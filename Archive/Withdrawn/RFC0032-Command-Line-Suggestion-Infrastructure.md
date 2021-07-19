---
RFC: RFC
Author: Dongbo Wang
Status: Withdrawn
SupercededBy: N/A
Version: 1.0
Area: Engine
Comments Due: April 30th, 2018
Plan to implement: No
---

# Command-line Suggestion Infrastructure

This RFC is to propose a design of command-line suggestion infrastructure in PowerShell engine.

## Motivation

[PowerShell in Azure Cloud Shell](https://docs.microsoft.com/en-us/azure/cloud-shell/overview) targets a very specific PowerShell use scenario -- Azure resource management.
It is an Azure service, which means it is possible to collect telemetry about how people use the shell. Given these two facts,
it is interesting to see whether ML can be leveraged to study the telemetry data within the targeted scenario,
and provide useful command-line suggestions in the cloud shell based on the user input.

It should be pointed out that, the command-line suggestion is different from tab completion in both purpose and its inner working.
From the perspective of purpose,
tab completion targets a specific syntactic part of the typed input, such as a parameter name or a member,
while command-line suggestion targets the whole input string and gives suggestions like how a search engine reacts to the user input.
From the perspective of inner working,
tab completion mainly depend on inference within the runtime,
while command-line suggestion depends on studying the pattern of the history inputs.

Of course, in order for PowerShell cloud shell to do this, basic infrastructure support is required in both the PowerShell engine and host.
PowerShell itself should make it easy to plug in a suggestion engine via a module, as well as present the suggestion results to the user.
With this support, the cloud shell can focus on the design of the suggestion engine,
and enable it in the could environment by simply loading the suggestion engine as a module.

## Specification

### Interface with Suggestion Engine

The proposal is to have a similar infrastructure as the tab completion today.
A cmdlet named `CommandLineSuggestion` will be used as the interface between PowerShell and a suggestion engine.
The suggestion engine could be completely PowerShell-agnostic.
To turn it into a module that works with PowerShell,
you just need to have the cmdlet `CommandLineSuggestion` as a thin wrapper over its suggestion APIs.

The syntax of `CommandLineSuggestion` looks as follows:

```powershell
CommandLineSuggestion [-InputScript] <string> [-HistoryInput] <string[]> [-Async] [<CommonParameters>]
```

- The parameter `-InputScript` is mandatory, which takes the user input for suggestions.
- The parameter `-HistoryInput` can be mandatory or optional, which takes the relevant history inputs as a context for the suggestion.
  Given the privacy concerns, only the history from the current local/remote session is passed to the cmdlet by PowerShell,
  which can be retrieved easily with `Get-History`.
  This may seem no much value due to the limitation of the data set,
  but it won't be a problem in cloud shell as the telemetry will provide much richer context for the suggestion engine to work.
  From that perspective, we can see the history context is not necessarily needed by all suggestion engines,
  and therefore, a suggestion engine module can indicate whether or not it needs this information by making `-HistoryInput` mandatory or optional.
  In case `-HistoryInput` is optional, PowerShell won't bother to collect the history inputs.
- The parameter `Async` is optional, which instructs the cmdlet to run synchronously or asynchronously.
  When running synchronously, the cmdlet returns a collection of strings as the suggestion results.
  When running asynchronously, the cmdlet returns a `Task<Collection<string>>` that represents a already started asynchronous operation.

When running in a remote session, the tab completion calls `TabExpansion2` in the remote Runspace to get the completion results.
This is necessary for tab completion because inference needs to happen in the right context.
However, this is not necessary for command-line suggestion.
`CommandLineSuggestion` will always be invoked from the default local Runspace (`[runspace]::DefaultRunspace`),
so the task-based asynchronous behavior of the cmdlet is guaranteed to work properly
(serialization and de-serialization would break the async behavior if invoking the cmdlet in a remote Runspace).
When `-HistoryInput` is mandatory, PowerShell will collect the history from the default local Runspace and the remote Runspace (if there is one),
and then pass the history context along when invoking `CommandLineSuggestion`.

### Interface with UI Component

Two static method will be exposed to interact with the UI component. They look as follows:

```c#
public static Collection<string> SuggestInput(string input, powershell powershell);
public static Task<Collection<string>> SuggestInput(string input, powershell powershell);
```

- The first method is synchronous, where `CommandLineSuggestion` is invoked without `-Async`.
- The second method is asynchronous, where `CommandLineSuggestion` is invoked with `-Async`.
  PowerShell will make sure the `Task<Collection<string>>` object is started before returning it from this method
  (`CommandLineSuggestion` should have started the task before writing it out).
- The `powershell` parameter indicates the current active Runspace of the host.
  This parameter is needed to get the history from the current session.
  When the host is in `Enter-PSSession`, `powershell` points to the remote runspace.
  So if it's needed, PowerShell can get the history from the remote Runspace as well as from the default local Runspace.

### Simple Local Experience

A simple local experience needs to be implemented to verify the design and show people what it looks like.

#### Suggestion Engine

To prove the concept, a simple local suggestion engine needs to be implemented.
It provides suggestions only based on the passed in history context with a string proximity algorithm, such as
[Levenshtein distance](https://en.wikibooks.org/wiki/Algorithm_Implementation/Strings/Levenshtein_distance) or
[Dice's coefficient](https://en.wikibooks.org/wiki/Algorithm_Implementation/Strings/Dice%27s_coefficient).
By using the string proximity algorithm, it can calculate the proximity rate between each history command-line and the current input,
and return suggestions based on the proximity rates.

#### Update PSReadline

To prove the concept, we need to update `PSReadline` to interact with the `SuggestInput` APIs and render suggestions on the host.
The rudimentary idea is to borrow and alter the existing UI experience of `Ctrl+Space` in `PSReadline` 2.0,
where completion options show up as a table and you can move cursor around to choose what you want.

- A custom key binding will be used to trigger the suggestion,
  and returned suggestions will be shown in a single-column table, ordered by the proximity rate of each suggestion.
  Similarly, a user can move the cursor with `Up` and `Down` arrow keys and confirm an option with the `Enter` key.
- For each returned suggestion string, the words that appear in the input will be rendered with green color,
  so that it's easy for user to see the what matches the input and what doesn't.
- Once the suggestions are shown up, you can continue typing and the suggestion list will be filtered based on the new input,
  which may result in the rendered suggestions to be reordered or shrunk.
  Note that the filtering doesn't call `SuggestInput` again.
  Instead, it applies a **filtering method** on the previously returned suggestions.
- After typing more on the host, if you think the filtered suggestions are not relevant anymore,
  you can press the key binding again to trigger another suggestion based on the current input.

The **filtering method** should ideally be returned by the suggestion engine as a delegate along with the suggestion results.
This is currently not enforced in the interface contacts we defined above.
However, we can change the interface contracts if the filtering method proves to be valuable.
For this simple local experience,
PowerShell will provide a filtering delegate that always depends on the string proximity algorithm to filter the given suggestions.

PowerShell could provide some utility methods to help the UI components,
such as a method to find all common words between two strings.

### Cloud Shell Experience

The goal of this infrastructure design is to allow a custom suggestion engine to be completely self-contained and PowerShell-agnostic.
The cloud shell team can design the suggestion engine without having PowerShell in mind.
All they need is to provide the synchronous and the task-based asynchronous APIs for getting suggestions.
Then, it can be easily plug in to PowerShell by wrapping their APIs in the cmdlet `CommandLineSuggestion`.

## Alternate Proposals and Considerations

A C# interface between PowerShell and suggestion engine was considered initially.
It's in the form of a C# interface or abstract class,
and a suggestion engine needs to implement them to integrate with PowerShell.

Then I found it's hard to register and un-register a suggestion engine when loading and unloading a module.
Using a cmdlet as the interface makes things easy because the registration and un-registration is already taken care of by PowerShell module.
