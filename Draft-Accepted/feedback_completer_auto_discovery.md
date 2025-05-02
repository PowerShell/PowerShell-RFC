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

- In-box path: `$PSHOME/feedbacks` and `$PSHOME/completions`
- Current user path: `<personal>/PowerShell/feedbacks` and `<personal>/PowerShell/completions`
- All user path: `<shared>/PowerShell/feedbacks` and `<shared>/PowerShell/completions`

### Feedback Provider

There are 2 kinds of feedback providers:

1. One that covers different commands and different scenarios. For example,
  the [WinGet CommandNotFound](https://www.powershellgallery.com/packages/Microsoft.WinGet.CommandNotFound/1.0.4.0) feedback provider
  acts on `"CommandNotFound"` error and suggests the `WinGet` installation command if the tool that user tried to execute can be installed with `WinGet`.

2. One that convers a specific command. For example, a feedback provider for `git`.

The 1st kind doesn't have a specific corresponding trigger, so the auto-discovery for it only happens at the startup of a session.</br>
The 2nd kind can use the native command name as the specific trigger, and the auto-discovery happens when the command gets executed.

The structure of the `feedbacks` folder is as follows:
```none
feedbacks
│
├───_startup_
│   ├───linux-command-not-found
│   │       linux-command-not-found.psd1
│   │
│   └───winget-command-not-found
│           winget-command-not-found.psd1
├───git
│       git.psd1
│
├───kubectl
│       kubectl.psd1
│
├───...
```

Each item within `feedbacks` is a folder.
- `_startup_`: feedback providers declared in this folder will be auto-loaded at the startup of an interactive `ConsoleHost` session.
- `git` or `kubectl`: feedback provider for a specific native command should be declared in a folder named after the native command. It will be auto-loaded when the native command gets executed.

Within each sub-folder, a `.psd1` file named after the folder name should be defined to configure the auto-discovery for the feedback provider.

```powershell
@{
   module = '<module-name-or-path>[@<version>]',  # Module to load to register the feedback provider.
   arguments = @('<arg1>', '<arg2>'),  # Optional arguments for module loading.
   disable = $false,  # Control whether auto-discovery should find this feedback provider.
}
```

About the `module` key:
- The `@<version>` part is optional, which allows specifying the module version if needed.
- Its value could be the path to an assembly DLL, in which case the DLL will be loaded as a binary module.
- The processing order is
  - First, expand the string. So, the value could contain PowerShell variables such as `$env:ProgramData`.
  - Second, try resolving the value as a path using PowerShell built-in API.
    - if it's successfully resolved as a path, use the path for module loading; Otherwise
    - if the value contains directory separator, then discovery fails; Otherwise
    - use the expanded value as the module name for module loading

#### Execution Flow

1. At startup, auto-load the enabled feedback providers from the `_startup_` folder of the `feedbacks` paths. Do the auto-loading before processing user's profile.
2. Report what feedback providers were processed and the time taken, in the streaming way.
3. In `NativeCommandProcessor`, when a native command gets executed,
   - Check if a feedback provider for this native command is available in the `feedbacks` paths;
   - If so, check if the specified module already loaded;
   - If not, load it at the `DoComplete` phase, so the feedback provider will be ready right after the native command finishes.

**[NOTE:]** It would be better if we can check if a feedback provider for the native command has already registered before checking the paths,
but unlike the completer registration for native commands, today there is no mapping between a feedback provider and a native command.
Also, it's allowed to register multiple feedback providers for a single native command.

#### Discussion Points

1. Should we add another key to indicate the target OS?
   - A feedback provider may only work on a specific OS, such as the `"WinGet CommandNotFound"` feedback provider only works on Windows.
   - Such a key could be handy if a user wants to share the feedback/tab-completer configurations among multiple machines via a cloud drive.

2. Do we really need a folder for each feedback provider?
   For example, can we simply have the files `git.psd1` and `kubectl.psd1` under the `feedbacks` folder, and the files `linux-command-not-found.psd1` and `winget-command-not-found.psd1` under the `_startup_` folder?
   - Since it's possible to have non-module feedback provider that comes with a DLL only,
     then the DLL might need to be deployed along with the configuration.
     In that case, the tool that deploys the DLL will either copy the DLL directly to `feedbacks`,
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

The strucutre of the `completions` folder is as follows:
```none
completions
│
├───_startup_
│   └───unix-completer
│           unix-completer.psd1
├───git
│       git.psd1
│
├───az
│       az.psd1
│
├───...
```

Each item within `completions` is a folder.
- `_startup_`: tab-completer declared in this folder will be auto-loaded at the startup of an interactive `ConsoleHost` session.
- `git` or `az`: tab-completer for a specific native command should be declared in a folder named after the native command. It will be auto-loaded when user tab completes on the command.

Within each sub-folder, a `psd1` file named after the folder name should be defined to configure the auto-discovery for the tab-completer.

```powershell
@{
   module = '<module-name-or-path>[@<version>]',  # Module to load to register the completer.
   script = '<script-path>',  # Script to run to register the completer.
   arguments = @('<arg1>', '<arg2>'),  # Optional arguments for module loading or script invocation.
   disable" = $false,  # Control whether auto-discovery should find this completer.
}
```

About the `module` and `script` keys:
- `module` key take precedence. So, if both keys are present, `script` will be ignored.
- For `module`, use the same resolution as in the `Feedback Provider` section above.
- For `script`, its value must be a `.ps1` file
  - If it fails to resolve as a `.ps1` file, the auto-discovery fails;
  - Otherwise, run the script. (_the script may run in PSReadLine's module scope, will that be a problem? How to run it in the top-level session?_)

#### Execution Flow

1. At startup, auto-load the enabled tab-completers from the `_startup_` folder of the `completions` paths. Do the auto-loading before processing user's profile.
2. Report what tab-completers were processed and the time taken, in the streaming way.
3. In the completion system, when tab completion is triggered on a native command,
   - Check if a tab-completer for this native command already exists;
   - If not, check if a tab-completer for this native command is available in the `completions` paths;
   - If so, register it either by loading the module or running the script, and then call the tab-completer.

**[NOTE:]** For a specific native command, you can only have 1 active completer for it. So, we will check if a completer already exists for a native command before searching in the paths.

#### Discussion Points

Same discussions as in `Feedback Provider` section:
1. Should we add another key to indicate the target OS?

Different discussions:
1. Do we really need a folder for each argument completer?
   - [dongbo] Yes, I think we need.
   The implementation of a completer can just be a `.ps1` script that needs to be deployed along with the completer's configuration.
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
6. How about on a System Lockdown Mode (SLM) or Restricted remoting environments?
   - We use `.psd1` file for metadata, which can be signed if needed.

### Unified Location for load-at-startup Configurations

There could be a similar demand for a predictor module. There won't be a specific trigger for any predictors,
so for a predictor to be auto-discovered, it has to be loaded at the startup of an interactive session.

Given that, maybe it's better to have a unified location for all the load-at-startup configurations:

- Have a `startup` folder at the same level of `feedbacks` and `completions` folders;
- All modules or scripts that need to be processed at session startup should have configurations deployed in the `startup` folder.

Each item within `startup` is a folder, whose name should be the friendly name of the component, e.g. `"UnixTabCompletion"`.
Within each sub-folder, a `.psd1` file named after the folder name should be defined to configure the auto-discovery of the component.

```powershell
@{
   module = '<module-name-or-path>[@<version>]',  # Module to load.
   script = '<script-path>',  # Script to run.
   arguments = @('<arg1>', '<arg2>'),  # Optional arguments for module loading or script invocation.
   disable = $false,  # Control whether auto-discovery should find this completer.
}
```

The configuration processing will be the same as what is described in the [Tab Completer](#tab-completer) section above.
Again, the `module` key take precedence. So, if both `module` and `script` keys are present, `script` will be ignored.


## Appx and MSIX Apps

What is discussed above requires a tool to deploy configurations to a pre-defined location during installation.
However, an Appx/MSIX package on Windows may be isolated and thus cannot access the location. See [some constraints](https://github.com/PowerShell/PowerShell/issues/17283#issuecomment-1522133126).
Therefore, to support auto-discovery of PowerShell resources like argument completer that come along with an Appx/MSIX package,
PowerShell has to proactively inspect the package instead of depending on the app registering its resources with the mechanism described above.

### Expose resources with App Extension

MSIX' standard recommendation is for the packaged app to declare an `appExtension` in its manifest.
Here is a snippet of the app manifest for MSIX package:

```xml
<Applications>
   ...

   <Extensions>
      <uap3:Extension Category="windows.appExtension">
         <uap3:AppExtension Name="add-in-contract" Id="add-in" PublicFolder="Public" DisplayName="Sample Add-in" Description="This is a sample add-in">
            <uap3:Properties>
               <!--Free form space-->
            </uap3:Properties>
         </uap3:AppExtension>
      </uap3:Extension>
   </Extensions>
</Application>
```

In this example, you will notice a `Properties` element within the `AppExtension` element [[reference]](https://github.com/MicrosoftDocs/msix-docs/blob/main/msix-src/msix-sdk/sdk-guidance.md#how-to-effectively-use-the-same-package-on-all-platforms-windows-10-and-non-windows-10). 
There is no validation performed in this section of the manifest file,
and this allows tool developers to specify the resources exposed to PowerShell.

An example would look like this:

```xml
<uap3:Extension Category="windows.appExtension">
   <uap3:AppExtension
      Name="com.microsoft.powershell.resources"
      Id="winget.powershell"
      PublicFolder="public"
      DisplayName="WinGet PowerShell Resources">
         <uap3:Properties>
            <Completer Type="Script" Name="winget.completer" File="assets\winget.completer.s1" />
            <Feedback Type="DLL" Name="winget.feedback" File="assets\WinGet.Feedback.dll" Discovery="auto"/>
         </uap3:Properties>
</uap3:Extension>
```

This model in Appx/MSIX packages can securely provide explicit details and avoid ambiguitities.
And it ensures clean behavior on uninstall - packages registered for the user are discoverable and
cleanly become invisible when the package is unregistered for the user.
No concerns about incomplete uninstalls or manual hackery or anything that leaves the system in inconsistent states.

### Resource Discovery

PowerShell can use the `AppExtensionCatalog` APIs to enumerate those extensions with the name/namespace of the extension,
and get any needed info about them [[reference]](https://learn.microsoft.com/uwp/api/windows.applicationmodel.appextensions.appextensioncatalog?view=winrt-26100).

A sample enumeration:

```c#
var catalog = new AppExtensionCatalog.Open("com.microsoft.powershell.resources")
foreach (var extension in catalog)
{
    ...use it...
}
```

Note that `AppExtensionCatalog` is a WinRT type. It implies that changes will be needed to the project file to use the API,
e.g. the `TargetFramework` needs to be Windows specific such as `net9.0-windows10.0.19041.0`.
This may be a trouble to PowerShell as it's cross-platform.

### Resource Access

It's recommended to use the Dynamic Dependency API to access files in an Appx/MSIX package.
The details about how to use the API can be found in [documentation](https://learn.microsoft.com/windows/apps/desktop/modernize/framework-packages/use-the-dynamic-dependency-api).

Dynamic Dependencies enables you to use several tech to interact with a packaged Powershell extension,
e.g. WinRT inproc and out-of-proc (`ActivateInstance`), inproc DLLs (`LoadLibrary`), inproc .NET assemblies (`Assembly.Load*()`), read a text file (`CreateFile(GetPackagePath()+"\foo.ps1"`) and other like tech.

Note that, inproc is generally discouraged for extensibility models given the reliability and security implications.
But it's technically feasible with Dynamic Dependencies as there are times it's necessary.
In our case, PowerShell will need to load assemblies inproc as it's how it works.
With Dynamic Dependency API, we can tell Windows that we are using a package's content,
and it will ensure Windows doesn't try to service the package (e.g. updating or removing) while in use.
It also adds the package to the Powershell process' package graph, making its resources available for `LoadLibrary`, `ActivateInstance` and other APIs.

However, there are a few potential issues that will restrict where and how we can support it in PowerShell:

1. There are 2 implementation of the API -- the Windows App SDK's dynamic dependency API AND the Windows 11's dynamic dependency API.
   Only the Windows 11's implementation supports referencing and using a main package and
   The Windows App SDK's implementation can only target a framework package [[reference]](https://learn.microsoft.com/windows/apps/desktop/modernize/framework-packages/use-the-dynamic-dependency-api#reference-and-use-a-main-package-windows-app-sdk-limitation).
   In our scenario, the extension for PowerShell resource should be declared in the tool's package, namely the main package.
   - A main package is the primary package that contains the core application binaries, assets, and manifest that define the app’s identity, capabilities, and behavior.
   - A framework package is a special type of MSIX package that contains shared code, libraries, or resources used by one or more apps — but does not contain an app itself.

2. The Windows 11's implementation targets Windows 11, version 22H2 (10.0; Build 22621), and later.
   It only provides C and C++ functions, no WinRT types, unlike the Windows App SDK's implementation.

3. I believe the Windows 11's implementation will provide WinRT types eventually.
   But consuming WinRT types would requires changes to the project, e.g. the `TargetFramework` needs to be Windows specific such as `net9.0-windows10.0.19041.0`.
   This may cause problem to PowerShell as it's cross-platform.

It means we can only support the feature in Windows 11, and only by using C/C++ functions for now.
When WinRT support comes in future, we may have trouble switching to that.

### Overall thinking about Appx/MSIX

The above discussion shows Appx/MSIX describes the App Extension for an app to declare PowerShell resources,
as well as for PowerShell to discover and access the resources inside the app pacakge.

However, there are still missing parts to the feature we are looking at in this spec:

1. How to deal with the resources that are supposed to be loaded at startup of PowerShell session, such as the `WinGetNotFound` feedback provider?
   - Technically, we can defined a special extension namespace for apps to declare such resources with.
     Then at startup, we get all extensions with that namespace and load all of them.
   - But, would that be performant enough to do at the startup?

2. How to allow user to disable the auto-discovery/loading of PowerShell resources from a particular Appx/MSIX app?
   - Shall we have configurations for indivual Appx/MSIX apps only for the purpose of disabling?

3. How to cache the information that an Appx/MSIX package doesn't have any PS resources declared and avoid the auto-discovery for it when user is using the tool?

Given that supporting auto-discovery/loading for Appx/MSIX packages is completely different from other applications,
It warrants another RFC to design for it specifically.
