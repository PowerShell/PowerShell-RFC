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

One of the big factors preventing existing Windows PowerShell users from moving to PowerShell Core has been cmdlet coverage. Specifically being able to use existing Windows PowerShell modules in PowerShell Core. Ease of UX is important here - working with 'WindowsPS-only' modules from PS Core should not be any different than working with 'PS Core - compartible' modules. Also this should not require any setup from the user.

## Motivation

    As a PowerShell user,
    I can use modules written for Windows PowerShell in PowerShell Core,
    so that I could migrate from Windows PowerShell to PowerShell Core.

## User Experience

Proof-of-concept test was successfull.
Examples will be added shortly...

## Specification

Today a module manifest (since Windows PowerShell 5) may contain a `CompatiblePSEditions` property to indicate if it is compatible with PowerShell Core (value of `Core`).
Absense of `Core` value in `CompatiblePSEditions` during module import signals to PS Core that it should process the module as described in this RFC.

When Import-Module in PS Core detects that user is attempting to load 'WindowsPS-only' module:
  1. it creates a hidden `WindowsPS Compatibility` Windows PS process (unless one already exists)
  2. creates PS Remoting connection to its IPC pipe
  3. loads the module in remote Windows PS process
  4. generates local proxy module/commands using IPC remoting connection and Import-PSSession cmdlet
  5. when 'WindowsPS-only' module unload request is detected - unload module in remote process, close PS remoting channel, close remote process

### PS Remoting Transport

IPC / Named Pipe is to be used for connections to remote PowerShell process. Today a named pipe listener is created with each Windows PowerShell process. IPC transport has good performance and secured endpoints, however (unlike, for example, SSH transport) it is not currently supported by New-PSSession.
So a new IPC parameter set will be added to New-PSSession.

### Lifetime of remote Windows PowerShell process and module

Overall RFC goal is to have farmiliar 'local module' user experience even though actuall operations are working with a module in a remote process. This drives following:

1. One remote Windows PS process corresponds to one local PS Core process. E.g. if a same user creates 2 local PS Core processes and loads 'WindowsPS-only' module in each one, this will result in creating 2 Windows PS processes and loading the actuall module in each one of them.
2. Local Load-Module for 'WindowsPS-only' module will have same behaviour/effect in remote Windows PS process as it would have been done locally on an compartible module. In addition, this will create `WindowsPS Compatibility` Windows PS process if one does not already exist.
2. Local Remove-Module for 'WindowsPS-only' module will have same behaviour/effect in remote Windows PS process as it would have been done locally on an compartible module. In addition, this will remove `WindowsPS Compatibility` Windows PS process if it is no longer needed.

### Objects returned by remote operations

With PS Remoting objects returned from remote side are not actual `live` objects. This will be the same in case of this RFC. Documentation will make sure that users understand that even though the experience will look like they are working with 'WindowsPS-only' modules locally, objects returned by these modules are deserialized copies.

## Alternate Proposals and Considerations

### Windows PowerShell Compatibility module

[Windows PowerShell Compatibility module](https://github.com/PowerShell/WindowsCompatibility) was initial step toward solving the problem of using WindowsPS modules in PS Core. It has following drawbacks:
1. It uses WinRM-based PS remoting which:
  i. requires setup
  ii. not the best transport performance-wise
1. Introduces additional cmdlets to work with 'WindowsPS-only' modules, i.e. does not have transparent user experience.

