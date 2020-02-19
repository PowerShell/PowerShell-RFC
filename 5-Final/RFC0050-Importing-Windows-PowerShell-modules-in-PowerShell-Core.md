---
RFC: 0050
Author: Andrew Menagarishvili
Status: Final
SupercededBy:
Version: 0.1
Area: Language
Comments Due: 10/10/2019
Plan to implement: Yes
---

# Importing Windows PowerShell modules in PowerShell Core

One of the big factors preventing existing Windows PowerShell users from moving to PowerShell Core has been cmdlet coverage. Specifically being able to use existing Windows PowerShell modules in PowerShell Core. Non-compatibility is due to the different .NET runtimes used between Windows PowerShell and PowerShell Core. Ease of UX is important when addressing this problem - working with 'WindowsPS-only' modules from PS Core should not be any different than working with 'PS Core - compatible' modules. Also this should not require any setup from the user.

## Motivation

    As a PowerShell user,
    I can use modules written for Windows PowerShell in PowerShell Core,
    so that I could migrate from Windows PowerShell to PowerShell Core.

## New User Experience

Example that shows using commands from 'WindowsPS-only' module located in `System32` module path in PS Core:
```PowerShell
PS C:\> $PSVersionTable.PSEdition
Core
PS C:\> Import-Module DesktopOnlyModuleOnSystem32ModulePath
WARNING: Module DesktopOnlyModuleOnSystem32ModulePath is loaded in Windows PowerShell using WinPSCompatSession remoting session; please note that all parameter values and results of commands from this module will be deserialized objects. If you want to load this module into PowerShell Core please use Import-Module -SkipEditionCheck syntax.
PS C:\> (Get-Module DesktopOnlyModuleOnSystem32ModulePath).CompatiblePSEditions
Desktop
PS C:\> DesktopOnlyModuleOnSystem32ModulePathFunction
Success
```
Proof-of-concept test was successfull.

## Current User Experience

For reference, here is current behaviour that this RFC is targeting to change:
```PowerShell
PS C:\> $PSVersionTable.PSEdition
Core
PS C:\> Import-Module DesktopOnlyModuleOnSystem32ModulePath
Import-Module : Module 'C:\windows\system32\WindowsPowerShell\v1.0\Modules\DesktopOnlyModuleOnSystem32ModulePath\DesktopOnlyModuleOnSystem32ModulePath.psd1' does not support current PowerShell edition 'Core'. Its supported editions are 'Desktop'. Use 'Import-Module -SkipEditionCheck' to ignore the compatibility of this module.
```

## Specification

Today a module manifest (since Windows PowerShell 5) may contain a `CompatiblePSEditions` property to indicate if it is compatible with PowerShell Core (value of `Core`) or Windows PowerShell (value of `Desktop`). If `CompatiblePSEditions` property is missing then the value of `Desktop` is assumed.<br />
Functionality of this RFC will replace showing of the `PSEditionNotSupported` error message (`Module '{0}' does not support current PowerShell edition '{1}'. Its supported editions are '{2}'. Use 'Import-Module -SkipEditionCheck' to ignore the compatibility of this module.`) in scenarios where it is currently displayed:<br />
During `Import-Module` a module will be loaded into a separate Windows PowerShell process instead of current PowerShell Core if:<br />
(A module is located in `System32` module path) and ((`CompatiblePSEditions` is `Desktop`) or (`CompatiblePSEditions` is missing))<br />
If this condition is detected PowerShell Core:
  1. creates PS Remoting session `WinPSCompatSession` (unless it already exists) using redirected process streams transport (same one used by PS jobs). Internally this creates hidden `WindowsPS Compatibility` Windows PS process.
  2. generates local proxy module/commands using `WinPSCompatSession` remoting session and code of `Import-Module -PSSession` cmdlet;
  3. when 'WindowsPS-only' module unload request is detected, if no other module is using `WinPSCompatSession` remoting session - it is closed (this also closes remote Windows PS process).

Loading a module into `WinPSCompatSession` can be forced regardless of module path or value of `CompatiblePSEditions` manifest property using `-UseWindowsPowerShell` parameter of `Import-Module`.<br />
New functionality should also be supported during module autoload / command discovery.<br />
Loading of modules into `WinPSCompat` should be tracked in telemetry separately from loading other modules so `TelemetryType.WinCompatModuleLoad` needs to be added.

### PS Remoting Transport

Redirected process streams transport (same one used by PS jobs) is to be used for connections to remote Windows PS process.
A new WindowsPS-specific parameter set will be added to New-PSSession:  `New-PSSession -UseWindowsPowerShell [-Name <string[]>]`
Process streams transport automatically manages underlying process, so the lifetime of the `WindowsPS Compatibility` Windows PS process is the same as remoting session that owns it.

### Module deny list

This feature has a significant performance cost - a Windows PS process needs to be started and PS Remoting channel established. This can be a problem in some cases considering this feature also loads modules automatically during command discovery.
Some modules don't work well with de/serialized objects (e.g. pipeline cmdlet combinations of 'Hyper-V' module);<br />
Also there are cases when user never intended to load a module using this feature: for example, using any '-Job' cmdlet tries to load (and currently quietly fails) PS-Core-incompatible 'PSScheduledJob' module. With this feature enabled, 'PSScheduledJob' module is getting successfully loaded even though user never wanted it.<br />
For such cases it is necessary to implement a module deny list.<br />
For modules in the deny list this feature will not engage (and current behavior of generating 'PSEditionNotSupported' error will be maintained).<br />
Module deny list defined using a 'WindowsPowerShellCompatibilityModuleDenyList' setting in 'powershell.config.json' so that user can change it when needed.

Going forward, we will continue to update this list if any modules are discovered to have problems operating in this compatibility layer.

### Lifetime of 'compatibility' Windows PowerShell process and module

Overall RFC goal is to have familiar 'local module' user experience even though actual operations are working with a module in a separate 'compatibility' process. This drives following:

1. One 'compatibility' Windows PowerShell process corresponds to one local PowerShell Core process. E.g. if a same user creates 2 local PowerShell Core processes and loads 'WindowsPS-only' module in each one, this will result in creating 2 Windows PS processes and loading the actuall module in each one of them.
2. Lifetime of the module on 'compatibility' side (Windows PowerShell) should be the same as on local side (PS Core). This is important because some modules save their global state in-between command calls.
   * Local Load-Module for 'WindowsPS-only' module will have same behaviour/effect in 'compatibility' Windows PS process as it would have been done locally on an compartible module. In addition, this will create `WindowsPS Compatibility` Windows PS process if one does not already exist.
   * Local Remove-Module for 'WindowsPS-only' module will have same behaviour/effect in 'compatibility' Windows PS process as it would have been done locally on an compartible module. In addition, this will remove `WindowsPS Compatibility` Windows PS process if it is no longer needed.
3. When removing a module, there is an `OnRemove` event on the module that will execute. This event allows Compat proxy module to react to being removed and perform cleanup of 'compatibility' process if necessary.
4. PS process exit does Not perform graceful cleanup of modules, so it needs to be handled separately. There is an event that allows to react to the closing of the PowerShell process (to do 'compatibility' process cleanup): `Register-EngineEvent -SourceIdentifier ([System.Management.Automation.PsEngineEvent]::Exiting) -Action $OnRemoveScript`

### Objects returned by remote operations

With PS Remoting objects returned from remote 'compatibility' side are not actual `live` objects. This will be the same in case of this RFC. A warning will be generated every time a module is imported using this new functionality; also documentation will make sure that users understand that even though the experience will look like they are working with 'WindowsPS-only' modules locally, objects returned by these modules are deserialized copies.

## Alternate Proposals and Considerations

### Scenarios that need improvement

1. Ideally there should be 1-to-1 runspace afinity between PSCore and WinPS processes; currently this is many(runspaces of a single PSCore process)-to-1(runspace of a single WinPS process), which in some cases may cause conflicts.
2. PS providers are not supported across remoting boundaries.

### Windows PowerShell Compatibility module

[Windows PowerShell Compatibility module](https://github.com/PowerShell/WindowsCompatibility) was initial step toward solving the problem of using WindowsPS modules in PS Core. It has following drawbacks:
- It uses WinRM-based PS remoting which:
    + requires setup
    + not the best transport performance-wise
- Introduces additional cmdlets to work with 'WindowsPS-only' modules, i.e. does not have transparent user experience.
