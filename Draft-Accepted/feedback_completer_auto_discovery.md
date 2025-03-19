# Auto Discovery for Feedback provider and Tab-Completer

## Motivation

Today, to enable a feedback provider or a tab-completer for a native command,
a user has to import a module or run a script manually, or do that in their profile.
There is no way to auto-discover a feedback provider or a tab-completer for a specific native command.

As a tool author, I want to provide a feedback provider or a tab-completer along with my tool installation,
but I don't want to mess with user's profile to make the feedback provider or tab-completer discoverable.

As a user, I want my feedback providers and tab-completers for the specific native commands to be loaded lazily,
instead of having to use my profile to load them at session startup.

Relevant GitHub issues:
- [Enable loading of Feedback providers via config](https://github.com/PowerShell/PowerShell/issues/25115)
- [Add a way to lazily autoload argument completers](https://github.com/PowerShell/PowerShell/issues/17283)

## Target Scenario

For PowerShell commands (Function or Cmdlet), I presume the completion or feedback support,
if there is any, will be from the same module.
So, when the command becomes available, the feedback provider and/or tab-completer will become available too.

Therefore, the auto-discovery for feedback provider and tab-completer targets native commands only.

## Goals

- A tool can deploy its feedback provider and/or tab-completer without needing to update a file at a central location, such as the user's profile.
- A tool can remove its deployment cleanly without needing to update a file at a central location.
- PowerShell can discover feedback providers and tab-completers automatically, and load one based on the right trigger.
- A user can enable or disable the auto-discovery for a feedback provider or tab-completer.

## Specification

The proposal is to adopt the existing mechnism used in Bash, Zsh and Fish's completion system --
have directories contain individual completion scripts for various commands and applications,
whose file names should match names of the commands.
Those completion scripts are loaded only when their corresponding commands' completion is triggered for the first time.

We will have separate directories for feedback providers and tab-completers, for 2 reasons:

- Trigger is different
  - feedback provider for a specific native command -- execute the command will trigger the discovery.
  - tab-completer for a specific native command -- tab complete on the command will trigger the discovery.
- Implementation is different
  - feedback provider -- a module or an assembly (binary implementation only)
  - tab-completer -- a module or a script (binary or script implementation)

The folders for feedback providers and tab-completers will be placed under the same path where modules folders are currently located:

- In-box path: `$PSHOME/Feedbacks` and `$PSHOME/Completions`
- Current user path: `<personal>/PowerShell/Feedbacks` and `<personal>/PowerShell/Completions`
- All user path: `<shared>/PowerShell/Feedbacks` and `<shared>/PowerShell/Completions`

### Feedback Provider

There are 2 kinds of feedback providers:

1. One that covers different commands and different scenarios. For example,
  the [WinGet CommandNotFound](https://www.powershellgallery.com/packages/Microsoft.WinGet.CommandNotFound/1.0.4.0) feedback provider
  acts on `"CommandNotFound"` error and suggests the `WinGet` installation command if the tool that user tried to execute can be installed with `WinGet`.

2. One that convers a specific command. For example, a feedback provider for `git`.

The 1st kind doesn't have a specific corresponding trigger, so the auto-discovery for it only happens at the startup of a session.</br>
The 2nd kind can use the native command name as the specific trigger, and the auto-discovery happens when the command gets executed.

The strucutre of the `Feedbacks` folder is as follows:
```none
Feedbacks
│
├───_startup_
│   ├───linux-command-not-found
│   │       linux-command-not-found.json
│   │
│   └───winget-command-not-found
│           winget-command-not-found.json
├───git
│       git.json
│
├───kubectl
│       kubectl.json
│
├───...
```

Each item within `Feedbacks` is a folder.
- `_startup_`: feedback providers declared in this folder will be auto-loaded at the startup of an interactive `ConsoleHost` session.
- `git` or `kubectl`: feedback provider for a specific native command should be declared in a folder named after the native command. It will be auto-loaded when the native command gets executed.

Within each sub-folder, a JSON file named after the folder name should be defined to configure the auto-discovery for the feedback provider.

```JSONC
{
    "module": "<module-name-or-path>",  // Module to load to register the feedback provider.
    "disable": false, // Control whether auto-discovery should find this feedback provider.
}
```

About the `module` key:
- Its value could be the path to an assembly DLL, in which case the DLL will be loaded as a binary module.
- The processing order is
  - First, expand the string. So, the value could contain PowerShell variables such as `$env:ProgramData`.
  - Second, try resolving the value as a path using PowerShell built-in API.
    - if it's successfully resolved as a path, use the path for module loading; Otherwise
    - if the value contains directory separator, then discovery fails; Otherwise
    - use the expanded value as the module name for module loading

#### Execution Flow

1. At startup, auto-load the enabled feedback providers from the `_startup_` folder of the `Feedbacks` paths. Do the auto-loading before processing user's profile.
2. Report what feedback providers were processed and the time taken, in the streaming way.
3. In `NativeCommandProcessor`, when a native command gets executed,
   - Check if a feedback provider for this native command is available in the `Feedbacks` paths;
   - If so, check if the specified module already loaded;
   - If not, load it at the `DoComplete` phase, so the feedback provider will be ready right after the native command finishes.

**[NOTE:]** It would be better if we can check if a feedback provider for the native command has already registered before checking the paths,
but unlike the completer registration for native commands, today there is no mapping between a feedback provider and a native command.
Also, it's allowed to register multiple feedback providers for a single native command.

#### Discussion Points

1. Should we expand the string value for `module` key, or always treat the value as literal?
   It feels like a useful feature, but could come with security implication, especially in WDAC/AppLocker enforced environment.
   - Note: PowerShell data file (`.psd1`) doesn't allow environment variables.

2. Should we add another key to indicate the target OS?
   - A feedback provider may only work on a specific OS, such as the `"WinGet CommandNotFound"` feedback provider only works on Windows.
   - Such a key could be handy if a user wants to share the feedback/tab-completer configurations among multiple machines via a cloud drive.

3. Do we really need a folder for each feedback provider?
   For example, can we simply have the files `git.json` and `kubectl.json` under the `Feedbacks` folder, and the files `linux-command-not-found.json` and `winget-command-not-found.json` under the `_startup_` folder?
   - Since it's possible to have non-module feedback provider that comes with a DLL only,
     then the DLL might need to be deployed along with the configuration.
     In that case, the tool that deploys the DLL will either copy the DLL directly to `Feedbacks`,
     or create a sub-folder named after the tool. Having a folder for each feedback provider makes sense in that case.
   - We will need a folder for each tab-completer because the completion implementation may just be a script file
     that needs to be deployed along with the completer's configuration.
     It's good to keep consistency in the folder structure for both feedback provider and tab-completer.

### Tab Completer

Similarly, there are 2 kinds of tab-completers for native commands:

1. One that covers a wide range of native commands. For example, the [Unix Completion](https://www.powershellgallery.com/packages/Microsoft.PowerShell.UnixTabCompletion/0.5.0) module dynamically detects all the native commands that Bash or Zsh has built-in completion for on a Linux or macOS machine, and support the same completion for them in PowerShell.

2. One that covers a specific native command. For example, the [AzCLI tab completion script](https://learn.microsoft.com/cli/azure/install-azure-cli-windows?tabs=azure-cli&pivots=winget#enable-tab-completion-in-powershell).


The 1st kind doesn't have a specific corresponding trigger, so the auto-discovery for it only happens at the startup of a session.</br>
The 2nd kind can use the native command name as the specific trigger, and the auto-discovery happens when user tab completes on the command.

The strucutre of the `Completions` folder is as follows:
```none
Completions
│
├───_startup_
│   └───unix-completer
│           unix-completer.json
├───git
│       git.json
│
├───az
│       az.json
│
├───...
```

Each item within `Completions` is a folder.
- `_startup_`: tab-completer declared in this folder will be auto-loaded at the startup of an interactive `ConsoleHost` session.
- `git` or `az`: tab-completer for a specific native command should be declared in a folder named after the native command. It will be auto-loaded when user tab completes on the command.

Within each sub-folder, a JSON file named after the folder name should be defined to configure the auto-discovery for the tab-completer.

```JSONC
{
    "module": "<module-name-or-path>",  // Module to load to register the completer.
    "script": "<script-path>",  // Script to run to register the completer.
    "disable": false,  // Control whether auto-discovery should find this completer.
}
```

About the `module` and `script` keys:
- `module` key take precedence. So, if both keys are present, `script` will be ignored.
- For `module`, use the same resolution as in the `Feedback Provider` section above.
- For `script`, its value must be a `.ps1` file
  - If it fails to resolve as a `.ps1` file, the auto-discovery fails;
  - Otherwise, run the script. (_the script may run in PSReadLine's module scope, will that be a problem? How to run it in the top-level session?_)

#### Execution Flow

1. At startup, auto-load the enabled tab-completers from the `_startup_` folder of the `Completions` paths. Do the auto-loading before processing user's profile.
2. Report what tab-completers were processed and the time taken, in the streaming way.
3. In the completion system, when tab completion is triggered on a native command,
   - Check if a tab-completer for this native command already exists;
   - If not, check if a tab-completer for this native command is available in the `Completions` paths;
   - If so, register it either by loading the module or running the script, and then call the tab-completer.

**[NOTE:]** For a specific native command, you can only have 1 active completer for it. So, we will check if a completer already exists for a native command before searching in the paths.

#### Discussion Points

Same discussions as in `Feedback Provider` section:
1. Should we expand the string value for `module` and `script` keys, or always treat the value as literal?
2. Should we add another key to indicate the target OS?

Different discussions:
1. Do we really need a folder for each feedback provider?
   - [dongbo] Yes, I think we need.
   Appx and MSIX packages on Windows have [many constraints](https://github.com/PowerShell/PowerShell/issues/17283#issuecomment-1522133126) that make it difficult to integrate with a broader plugin ecosystem. The way for such an Appx/MSIX tool to expose tab-completer could be just running the tool with a special flag, such as `<tool> --ps-completion`,
   to output some PowerShell script text for the user to run. In that case, the user will need to manually save the script text to a file and place the file next to the `tool.json` file.
   Having a folder is useful to group them together in that case.

2. When running a script, it will run in the PSReadLine's module scope when completion is triggered from within PSReadLine.
   Will that be a problem? How to run it in the top-level session instead?
   - [dongbo] This is more of a implementation-detail question. It's not a problem for module thanks to https://github.com/PowerShell/PowerShell/pull/24909.

3. Today, a user can unregister a tab-completer by running `Register-ArgumentCompleter -CommandName <name> -ScriptBlock $null`.
   But it doesn't remove the `<name>` from cache, but only sets its value to `null`. We probably should fix it to make our cache lookup simpler.

4. Today, a user cannot see what commands have tab-completer registered. This may be out-of-scope for this RFC.

### When to enable/disable the feature

This feature is only for interactive session, so we need to decide on when the feature is enabled and when is disabled.

1. The feature only works when the PowerShell host is `ConsoleHost` or the `Visual Studio Code Host` (which uses `ConsoleHost` internally IIRC).
   - Today, only `ConsoleHost` consumes feedback providers.
2. The feature should be disabled when PowerShell starts with `-Command` or `-File` to run a command or a file and then terminates.
   - So, when `-noexit` is specifeid, the feature should be enabled if other conditions are satisfied.
2. Shall we disable the feature with the `-noninteractive` flag?
   - `PSReadLine` is disabled when this flag is specified, so maybe this feature should be disabled too.
3. Shall we disable the feature with the `-noprofile` flag?
   - The "loading at startup" part of the feature is similar to how profiles are processed, but it's not part of the profile.
4. Shall we add a flag (or flags) to allow user to disable this feature (or disable `feedback` and `completer` separately)?
5. We report progress when loading `feedback` or `completer` at startup, so how to allow users to disable the progress report?
   - We have the `-NoProfileLoadTime` flag today to not show the time taken for running profile.
6. How about on a WDAC/AppLocker enforced environment?
