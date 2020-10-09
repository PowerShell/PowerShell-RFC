---
RFC: RFCXXXX
Author: Jason Helmick
Status: Draft
Area: Shell
Comments Due: 10/31/2020
---

# Predictive IntelliSense

Predictive IntelliSense is an addition to the interactive (shell) experience to assist in command
discovery and accelerate full command execution. The prediction suggestion appears as colored text
following the user’s cursor. This enables new and experienced users of PowerShell to discover, edit,
and execute full commands based on matching predictions from the user’s history or additional
providers.

Additional providers enhance historic predictions by providing domain specific commands and task
completions. Predictive IntelliSense includes an extension model to support the registration of
additional providers.

Predictions may be displayed in either **InlineView** or **ListView** depending on the user’s
preference. Below, predictions are displayed with InlineView.

![image](./media/pred-wlc.png)

Below, predictions are displayed with a dropdown in ListView.

![image](./media/pred-listview.png)

## Motivation

Tab completion has accelerated the success of both new and experienced PowerShell users. New users
get the benefit of discovery; seeing available cmdlets and parameters as options while interactively
typing. Experienced users receive the benefit of acceleration; typing less while using the **<tab>** key
to quickly complete a command.

The increasing amount of technology translates to an increase in cmdlets and full command
complexity. Predictive IntelliSense is an addition to the concept of Tab Completion; helping the
user discover, build and edit full commands based on the user’s history or additional plugins.

## Release plan

- Proposed 7.1 GA
  - Currently in PowerShell 7.1 PReview 7
  - Predictive IntelliSense Whole Line Completion
  - Historical Predictions
  - Extension Framework
- Future PSReadLine previews
  - Predictive IntelliSense Whole Line Completion + ListView
  - Support for additional providers using extension framework
  - Previews begin in October with PSReadLine 2.2.0-beta1

## Goals/Non-Goals

 Goals:

- Provide suggestions from PSReadLine history
- The user will be able to enable and disable Predictive IntelliSense
- The user will be able to change between InlineView and ListView
- The user will have keyboard shortcuts to navigate and edit a prediction
- The user will register additional providers when desired
- The user may customize the color of a prediction to support accessibility needs
- The user will be able to search history for similar predictions

Non-goals:

- Due to limit use, PowerShell Session history is not part of the historical suggestion
- VSCode implementation. We agree this is important for a consistent experience and are working to
  bring this feature in a future release

## Specification

The proposal is to add Predictive IntelliSense and Extension model to improve the user interactive experience.

### Key Design Considerations Predictive IntelliSense

Predictive IntelliSense displays a best match command completion from command history. The
prediction may change if a better match is found as the user types additional information. The
predicted command may be accepted, edited, or ignored.

![Prediction - Historical](./media/pre-wlc.png)

### Install and Remove Predictive IntelliSense

Predictive IntelliSense is a part of the PSReadLine module. Updating or installing the current beta
release from PSGallery will provide the Predictive IntelliSense feature.

The current release is PSReadLine 2.1.0 Beta 2:

```powershell
Install-Module PSReadLine -RequiredVersion 2.1.0-beta2 -AllowPrerelease
```

It is possible to remove, delete, the PSReadLine module to disable Predictive IntelliSense. This
creates a negative impact to customers as it would remove all features of PSReadLine. For this
reason, Predictive IntelliSense has its own PSReadLine configuration to enable and disable
predictions.

### Enabling and Disabling Predictions

By default, the feature Predictive IntelliSense is disabled. The impact of enabling this feature by
default may cause new and existing users to become confused at the shell prompt. The feature is
enabled and disabled with a PSReadLine command Set-PSReadLineOption that is executed at the shell
prompt or in the user’s Profile.

Predictive IntelliSense currently supports four arguments for a prediction source;

* None - This option disables Predictive IntelliSense
* History - This option uses only the PSReadLine history for predictions
* Plugin – This option uses only registered plugin modules for predictions
* HistoryAndPlugin – This option uses both for predictions

To enable Predictive IntelliSense, enter the following command in the shell or in the users Profile:

```powershell
Set-PSReadLineOption -PredictionSource History
```

To disable Predictive IntelliSense, enter the following command in the shell or in the users Profile:

```powershell
Set-PSReadLineOption -PredictionSource None
```

### Changing the Prediction View

Predictions are displayed in one of two view depending on the user preference. The default view is InlineView.

* InlineView – This is the default view and displays the prediction inline with the user’s typing.
  This view is similar to other shells Fish and ZSH.
* ListView – ListView provides a dropdown list of predictions below the line the user is typing.
  Users may quickly scan the list, highlight and select the desired prediction.

Users may change the view at the command line, or in the user’s PowerShell profile.

```powershell
Set-PSReadLineOption -PredictionViewStyle InlineView
```

### Change the Prediction Color for Accessibility

By default, Predictions appear on the same line the user is typing in a different customizable
color. To support Accessibility needs, the prediction color is settable in the shell or users
Profile.

![pi-color](./media/pi-color.png)
![pi-color](./media/pi-color2.png)

```powershell
Set-PSReadLineOption -Colors @{ Prediction = '#8A0303'}
Set-PSReadLineOption -Colors @{ Prediction = '#2F7004'}
```

Multiple types of color code values are supported in PSReadLine. For more information see
Set-PSReadLineOption in
[PSReadLine](https://docs.microsoft.com/en-us/powershell/module/PSReadline/Set-PSReadlineOption?view=powershell-7)

Examples of different color code values:

```powershell
 # Use a ConsoleColor enum
Set-PSReadLineOption -Colors @{ Prediction = 'DarkRed'}
Set-PSReadLineOption -Colors @{ Prediction = [ConsoleColor]::DarkRed}

 # 24 bit color escape sequence
Set-PSReadLineOption -Colors @{ Prediction = "$([char]0x1b)[38;5;100m"}

 # RGB value
Set-PSReadLineOption -Colors @{ Prediction = "#8181f7"}
```

### Key Bindings for Predictions

Key bindings control cursor movement and additional features within the prediction. To support users
running Predictive IntelliSense on multiple platforms, key bindings are user-settable in the Shell
and user’s PowerShell profile.

Example of setting a key binding in the shell or user’s profile:

```powershell
Set-PSReadLineOption -HistorySearchCursorMovesToEnd
Set-PSReadLineKeyHandler -Key UpArrow -Function HistorySearchBackward
Set-PSReadLineKeyHandler -Key DownArrow -Function HistorySearchForward
```

List of suggested key bindings defined in **SamplePSReadLineProfile.ps1**

Key Binding   | PSReadLine Function    | Description
------------- | -------------------    | -----------
Up Arrow	    | HistorySearchBackward  | Move backward through history
Down Arrow	  | HistorySearchForward	 | Move forward through history
F7	          | Profile Code	         | Opens Out-GridView with full history
F1	          | Profile Code	         | Opens the Help window for the current command or parameter
Ctrl-b	      | Profile Code	         | Add command to history
Ctrl-q	      | TabCompleteNext	       | Windows style Tab for Emacs mode
Ctrl-Q	      | TabCompletePrevious	   | Windows style Tab for Emacs mode
Ctrl-c	      | Copy	                 | Clipboard interaction for Emacs mode
Ctrl-v	      | Paste	                 | Clipboard interaction for Emacs mode
Ctrl-d,Ctrl-c | CaptureScreen	         | Captures screen for pasting to email or blog
Alt-d	        | ShellKillWord	         | Token based movement bound to Emacs word movement
Alt-Backspace | ShellBackwardKillWord  | “
Alt-b	        | ShellBackwardWord	     | “
Alt-f	        | ShellForwardWord	     | “
Alt-B	        | SelectShellBackwardWord| “
Alt-F	        | SelectShellForwardWord | “
“ or ‘	      | Profile Code	         | Insert Smart quotes
( or { or [	  | Profile Code	         | Insert matching braces
Backspace	    | Profile Code    	     | Delete previous character
Alt-w	        | Profile Code	         | Save current line in history – don’t execute
Ctrl-V	      | Profile Code    	     | Paste clipboard as a Here string
Alt-(	        | Profile Code	         | Insert parenthesis around selection or entire line
Alt-‘	        | Profile Code	         | Toggle quotes on argument under cursor
Alt-%	        | Profile Code	         | Replace aliases with resolved commands

### Search prediction history based on cursor location

The prediction history from PSReadLine is searchable when the user begins typing into the shell.
Users may search the entire prediction history for commands matching under the current cursor
position. Users may search prediction history based on 'Verb', 'Noun' and 'Parameter' of a cmdlet
using the up/down arrow keys.

* Searching history from cmdlet verb.
![Verb](./media/pi-arrow-verb.gif)

* Searching histroy from cmdlet noun.
![noun](./media/pi-arrow-noun.gif)

* Searching history from parameter.
![Param](./media/pi-arrow-param.gif)

### Edit predictions before accepting the prediction

Users may navigate prediction tokens to make changes to parameters, arguments, and additional
pipeline commands. The prediction model will update the displayed prediction from the closest
matches in PSReadLine history or added provider.

In the example below, a user navigates the prediction, pausing to change the -Property argument and
receiving an updated prediction.

![arg change](./media/pi-fb.gif)

## Prediction Extension Model

Additional providers enhance historic predictions by providing domain specific commands and task
completions. Predictive IntelliSense includes an extension model to support the registration of
additional providers.

## Goals/Non-Goals

 Goals:

* Provide extension model to support additional providers
* Support the discovery of providers using Get-Subsystem
* Render predictions from providers within performance guidelines 20ms
* Provide Module owners method of registration

Non-goals:

* Register/Unregister-Subsystem cmdlets are not planned because it's targeting binary subsystem
  implementations, which should deal with registration/unregistration via APIs.
* Server-side provider privacy is the responsibility of the provider module

## Extension Model Specification

### Predictive Suggestion Contract to Support Addition Providers

Below is an abbreviated code listing of the API contract to support suggestions from additional
providers.

```csharp
/// The class represents a predictive suggestion generated by a predictor.
public sealed class PredictiveSuggestion
{
/// Gets the suggestion.
public string SuggestionText { get; }

/// Gets the tooltip of the suggestion.
public string? ToolTip { get; }

/// Initializes a new instance of the <see cref="PredictiveSuggestion"/> class.
/// <param name="suggestion">The predictive suggestion text.</param>
public PredictiveSuggestion(string suggestion)
: this(suggestion, toolTip: null)
```

### Discovery of Provider Metadata and Registration

Providers are registered automatically when the user installs a provider module. Removal of
registration occurs when the module is removed. Users will need to discover installed providers
along with additional metadata, such as version information to assist users in upgrading and
diagnosing problems.

To list the currently installed providers and their respective metadata:

```powershell
Get-SubSystem
```

### Expected Provider Performance

Additional providers often will communicate with an internet connect service to provide suggestions.
To minimize the appearance of slow responses, we estimate that the maximum roundtrip time of 20ms is
permissible to not negatively impact users.

If the roundtrip time is exceeded, the predictor model will provide historical suggestions, or
suggestions from another provider that completed in the allowed time.

* Default required provider performance = 20ms

The current roundtrip time in PSReadLine using the Tab Completion feature is approximately **2-3ms**. We
believe that this current default roundtrip time for additional providers may still exceed user
expectations. With additional testing and feedback, we may need to reduce the roundtrip time to less
than **20ms**.

### Registration for Module Owners

Predictive IntelliSense is an extensible model offering module authors the opportunity to create their own predictor plugins. Customers using Predictive IntelliSense will benefit from increased coverage of domain specific technologies and other helpful predictors.

Module authors writing new predictors will need to register their predictor with the extension
model. Predictor registration is accomplished with the **OnImport** and **OnRemove** methods
executed on module startup and removal. Additional documentation is available for
[IModuleAssemblyInitializer
(OnImport)](https://docs.microsoft.com/en-us/dotnet/api/system.management.automation.imoduleassemblyinitializer?view=powershellsdk-7.0.0)
and
[IModuleAssemblyCleanup (OnRemove)](https://docs.microsoft.com/en-us/dotnet/api/system.management.automation.imoduleassemblycleanup?view=powershellsdk-7.0.0).

* Example of registering a predictor using **OnImport**

```csharp
// The initializer to register the predictor implementation at module loading time.

    public class PredictorInitializer : IModuleAssemblyInitializer
    {
        public void OnImport()
        {
            var predictor = new MyPredictor();
            SubsystemManager.RegisterSubsystem<IPredictor,MyPredictor>(predictor);
        }
    }
```

* Example of unregistering a predictor using **OnRemove**

```csharp
// Clean up the registeration when unloading the module.

    public class PredictorCleanup : IModuleAssemblyCleanup
    {
        public void OnRemove(PSModuleInfo psModuleInfo)
        {
            SubsystemManager.UnregisterSubsystem<IPredictor>(MyPredictor.Identifier);
        }
    }
```

In the above example, the predictor implementation is elsewhere in the module code with the type
name **MyPredictor**. An object of **MyPredictor** is created in **OnImport** and **OnRemove**. To
remove (unregister) your implementation instance, you need the **GUID** id of your implementation.