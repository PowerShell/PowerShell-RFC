---
RFC:
Author: Andrew Menagarishvili
Status: Draft
SupercededBy:
Version: 0.1
Area: Language
Comments Due: 10/10/2019
Plan to implement: Yes
---

# Windows PowerShell Compatibility Utilities

One of the big factors preventing existing Windows PowerShell users from moving to PowerShell Core has been cmdlet coverage. Specifically being able to use existing Windows PowerShell modules in PowerShell Core. Ease of UX is important here - working with 'WindowsPS-only' modules from PS Core should not be any different than working with 'PS Core - compatible' modules. Also this should not require any setup from the user.

## Motivation

    As a PowerShell user,
    I can use modules written for Windows PowerShell in PowerShell Core,
    so that I could migrate from Windows PowerShell to PowerShell Core.

## User Experience

Example that shows using commands from 'WindowsPS-only' module located in `System32` module path in PS Core:
```PowerShell
PS C:\> $PSVersionTable.PSEdition
Core
PS C:\> Import-Module DesktopOnlyModuleOnSystem32ModulePath
PS C:\> (Get-Module DesktopOnlyModuleOnSystem32ModulePath).CompatiblePSEditions
Desktop
PS C:\> DesktopOnlyModuleOnSystem32ModulePathFunction
Success
```
Proof-of-concept test was successfull.

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
During `Import-Module` a module wil be loaded into a separete Windows PowerShell process instead of current PowerShell Core if:<br />
(A module is located in `System32` module path) and ((`CompatiblePSEditions` is `Desktop`) or (`CompatiblePSEditions` is missing))<br />
If this condition is detected PowerShell Core:
  1. creates a hidden `WindowsPS Compatibility` Windows PS process (unless one already exists)
  2. creates PS Remoting connection to its IPC pipe
  3. loads the module in remote Windows PS process
  4. generates local proxy module/commands using IPC remoting connection and `Import-PSSession` cmdlet
  5. when 'WindowsPS-only' module unload request is detected - unload module in remote process, close PS remoting channel, close remote process

### PS Remoting Transport

IPC / Named Pipe is to be used for connections to remote PowerShell process. Today a named pipe listener is created with each Windows PowerShell process. IPC transport has good performance and secured endpoints, however (unlike, for example, SSH transport) it is not currently supported by New-PSSession.
So a new IPC-specific parameter set will be added to New-PSSession:  `New-PSSession [-Name <String>] -ProcessId <Int32>`
Here is the pseudo-code that shows the essence of proposed implementation of a new parameter set in New-PSSession:
```csharp
string WindowsPS_AppDomainName = GetPSHostProcessInfoCommand.GetAppDomainNamesFromProcessId(WindowsPS_ProcessId);
NamedPipeConnectionInfo connectionInfo = new NamedPipeConnectionInfo(WindowsPS_ProcessId, WindowsPS_AppDomainName);
TypeTable typeTable = TypeTable.LoadDefaultTypeFiles();
RemoteRunspace remoteRunspace = RunspaceFactory.CreateRunspace(connectionInfo, this.Host, typeTable) as RemoteRunspace;
remoteRunspace.Name = NamedPipeRunspaceName;
remoteRunspace.ShouldCloseOnPop = true
try
{
    remoteRunspace.Open();
}
catch (RuntimeException e)
{
    // handle connection errors
}
PSSession remotePSSession = new PSSession(remoteRunspace);
this.RunspaceRepository.Add(remotePSSession);
WriteObject(remotePSSession);
```

### Lifetime of remote Windows PowerShell process and module

Overall RFC goal is to have familiar 'local module' user experience even though actual operations are working with a module in a remote process. This drives following:

1. One remote Windows PS process corresponds to one local PS Core process. E.g. if a same user creates 2 local PS Core processes and loads 'WindowsPS-only' module in each one, this will result in creating 2 Windows PS processes and loading the actuall module in each one of them.
2. Lifetime of the module on remote side (Windows PS) should be the same as on local side (PS Core). This is important because some modules save their global state in-between command calls.
   * Local Load-Module for 'WindowsPS-only' module will have same behaviour/effect in remote Windows PS process as it would have been done locally on an compartible module. In addition, this will create `WindowsPS Compatibility` Windows PS process if one does not already exist.
   * Local Remove-Module for 'WindowsPS-only' module will have same behaviour/effect in remote Windows PS process as it would have been done locally on an compartible module. In addition, this will remove `WindowsPS Compatibility` Windows PS process if it is no longer needed.
3. When removing a module, there is an `OnRemove` event on the module that will execute. This event allows Compat proxy module to react to being removed and perform cleanup of remote process if necessary.
4. PS process exit does Not perform graceful cleanup of modules, so it needs to be handled separately. There is an event that allows to react to the closing of the PowerShell process (to do remote Compat process cleanup): `Register-EngineEvent -SourceIdentifier ([System.Management.Automation.PsEngineEvent]::Exiting) -Action $OnRemoveScript`

### Objects returned by remote operations

With PS Remoting objects returned from remote side are not actual `live` objects. This will be the same in case of this RFC. Documentation will make sure that users understand that even though the experience will look like they are working with 'WindowsPS-only' modules locally, objects returned by these modules are deserialized copies.

## Alternate Proposals and Considerations

### Windows PowerShell Compatibility module

[Windows PowerShell Compatibility module](https://github.com/PowerShell/WindowsCompatibility) was initial step toward solving the problem of using WindowsPS modules in PS Core. It has following drawbacks:
- It uses WinRM-based PS remoting which:
    + requires setup
    + not the best transport performance-wise
- Introduces additional cmdlets to work with 'WindowsPS-only' modules, i.e. does not have transparent user experience.

