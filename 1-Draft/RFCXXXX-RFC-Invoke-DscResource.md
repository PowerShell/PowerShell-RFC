---
RFC: RFCXXXX
Author: Gael Colas
Status: Draft
SupersededBy: N/A
Version: 0.6
Area: Microsoft.PowerShell.DesiredStateConfiguration
---

# Invoke-DscResource

Add cross-platform support for `Invoke-DscResource` in PowerShell 7+ without dependency on LCM and WMI.

This RFC addresses the need to leverage the DSC ecosystem of resources from newer versions of PowerShell, the way to decouple the execution of resources from the LCM and CIM/WMI, and the user experience from a consumer and solution vendor/integrator point of view.

## Motivation

1. DSC has been cited by users and solution partners as a blocker for moving from Windows PowerShell to PowerShell 7+.

2. As part of the latest survey on DSC usage started in June 2019, the top requested feature was support for Invoke-DscResource in PSCore without WMI dependency.

3. The DSC Ecosystem and Community would greatly benefit from enabling DSC resources to be used in imperative scripts, in current user context.

## Specification

### Invoke-DscResource in a module, decoupled from PowerShell

`Invoke-DscResource` is directly related to `Get-DscResource` in terms of user experience: Get provides the discoverability before the invocation.

`Invoke-DscResource` does not need to be part of the PowerShell engine (same as `Get-DscResource`), and we should aim to decouple PowerShell's engine from the DSC Ecosystem. While it's convenient for the `Configuration` keyword to have some of its implementation done in PowerShell's engine for parsing, `Invoke-DscResource` has no such requirements and should live in an independent module evolving in its own timeline, that will be bundled and shipped with PowerShell releases.

Open-sourcing this module should be a priority, but is out of scope for this RFC.

### Backward compatibility with Invoke-DscResource from PS 5.1

It is **not the intention to have feature parity** between the version found with Windows PowerShell 5.1 and the one described in this RFC.

While we attempt to maintain the same command syntax, the behavior will not be on par with `Invoke-DscResource` as found in PowerShell 5.1.

Specifically, we already know some features that **will not be supported** in this initial scope of work:

- Support for non-PowerShell resources (i.e. the native/binary or Python resources won't be supported)
- Running as System by default ([Discussed later in this document](#Default-Execution-Scope:-Current-runspace))
- Schema validation of invocation/results (It will only validate against the Resource's functions' signatures)

#### Syntax

As we don't plan on changing the usage, plus the functions is currently not available outside of PowerShell, and there is currently no command for PowerShell 7+, there is no need to change the Syntax found in Windows PowerShell 5.1.

The increment in PowerShell [MAJOR](https://semver.org/#spec-item-8)'s version field is enough to indicate the change of public API (as per [Semantic Versioning](https://semver.org/)).

```text
Invoke-DscResource [-Name] <string> [-Method] <string> -ModuleName <ModuleSpecification> -Property <hashtable> [<CommonParameters>]
```

#### Default Execution Scope: Current runspace

We aim at enabling existing scripts using `Invoke-DscResource`, written for Windows PowerShell 5.1, to "just work" in PowerShell 7+, but **in the current user context** when the **PsDscRunAsCredential** DSC common property is **not** supplied.

```PowerShell
Invoke-DscResource -Name xFile -ModuleName @{ModuleName='PSDscResources';ModuleVersion='2.12.0.0'} -Method 'Set' -Properties @{
    GetScript  = '<# My Get ScriptBlock #>'
    SetScript  = '<# My Set ScriptBlock #>'
    TestScript = '<# My Test ScriptBlock #>'
} -Verbose
```

This should run in the current session state where the command is invoked.

#### PsDscRunAsCredential: New Process as different user

It's the user's responsibility to either leverage [PsDscRunAsCredential](https://docs.microsoft.com/en-us/powershell/dsc/configurations/runasuser) to execute the resource in a user context that has the required privilege, or wrapping the call in a user context that runs as system (such as using [Invoke-CommandAs](https://www.powershellgallery.com/packages/Invoke-CommandAs) by Mark Kellerman).

`PsDscRunAsCredential` is a recommended practice to implement [least-privilege Administrative Models](https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/implementing-least-privilege-administrative-models).

Since we're now bypassing the LCM, we need to provide the feature in the `Invoke-DscResource`.

The Parameter will be extracted from the `-Properties` argument, and used to invoke the Resource in a process running as that user.
It will be equivalent as (from within the `Invoke-DscResource` command point of view):

So calling:

```PowerShell
Invoke-DscResource -Name Script -ModuleName @{ModuleName='PSDscResources';ModuleVersion='2.12.0.0'} -Method 'Set' -Properties @{
    GetScript  = '<# My Get ScriptBlock #>'
    SetScript  = '<# My Set ScriptBlock #>'
    TestScript = '<# My Test ScriptBlock #>'
    PsDscRunAsCredential = $Credential
} -Verbose

```

What will be executed will be equivalent (in terms of scope) to:

```PowerShell
Start-Job -Credential $Credential -ScriptBlock {
    Invoke-DscResource -Name Script -ModuleName @{ModuleName='PSDscResources';ModuleVersion='2.12.0.0'} -Method 'Set' -Properties @{
        GetScript  = '<# My Get ScriptBlock #>'
        SetScript  = '<# My Set ScriptBlock #>'
        TestScript = '<# My Test ScriptBlock #>'
    } -Verbose
}
```

> NOTES: We're avoiding to take dependencies on other technologies that would require extra configurations or permissions (such as remoting), or would not be available on other OSes.

#### Independent and isolated execution

`Invoke-DscResource` in PowerShell 7+ will not be aware of other instances being executed, and as such it will be possible to execute several instances in parallel when isolated in their own runspaces, or run in parallel with the LCM.

This means that it enables concurrent execution, but also risks conflict if two conflicting resources are run simultaneously.

It is up to the user to sequence the execution safely, or to create appropriate resources.

# Out of Scope for initial work & other notes

We're aware that some extra work or feature could be solved at the same time, but we're trying to have the MVP (minimum viable product) out as soon as possible, to help addressing the points raised in the [Motivation](#Motivation) section.

## Invoke-DscResource will not clear the Builtin Provider Cache

The Built-in provider cache, located in `$env:ProgramData\Microsoft\Windows\PowerShell\Configuration\BuiltinProvCache`, is currently cleared by the LCM (in WMF 5.1).

With an Invoke-DscResource advanced function as described in this RFC, it is not guaranteed to have enough permissions to that path (it's running in the current user context unless using `PSDscRunAsCredential`), nor do we assume exclusivity (LCM might be in use and running).

For those reasons, it is not reasonable to expect `Invoke-DscResource` for PowerShell 7 to clear the cache, at least for the scope of the MVP.

DSC Resources that rely on this may experience unexpected behavior (compared to running `Invoke-DscResource` in WMF 5.1). It is up to the maintainers of those resource modules to handle (or not) this new possibility.

## File Resource not supported

Just for clarification, as the File resource is Native/Binary:

```text
PS C:\ > Get-DscResource File

ImplementedAs   Name    ModuleName  Version    Properties
-------------   ----    ----------  -------    ----------
Binary          File                           {DestinationPath, Attributes, Checksum, Content...
```

This resource won't be supported with this initial work. Only Resources `ImplementedAs PowerShell`, will be supported.

## Composite Resources not supported

The **Composite resources** won't be supported for this initial scope. The support for composite would require `Invoke-DscResource` to understand and extract individual Resources called within the composite which is currently handled either in the PowerShell engine or through the `Configuration` function in the PSDesiredStateConfiguration module.

Although it would be great to enable this scenario, decoupling the `Configuration` keyword from the MOF compilation is out of scope for this initial work.
