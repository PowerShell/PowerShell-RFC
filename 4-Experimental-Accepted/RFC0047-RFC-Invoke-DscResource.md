---
RFC: RFC0047
Author: Gael Colas
Status: Experimental-Accepted
SupersededBy: N/A
Version: 0.9
Area: Microsoft.PowerShell.DesiredStateConfiguration
---

# Invoke-DscResource

Add cross-platform support for `Invoke-DscResource` in PowerShell 7+ without dependency
on the [Local Configuration Manager](https://docs.microsoft.com/en-us/powershell/dsc/managing-nodes/metaconfig)
(LCM), the
[Common Information Model (CIM) or Windows Management Instrumentation (WMI)](https://devblogs.microsoft.com/scripting/should-i-use-cim-or-wmi-with-windows-powershell/).

This RFC addresses the need to leverage the DSC ecosystem of resources from newer
versions of PowerShell, the way to decouple the execution of resources from the
LCM and CIM/WMI, and the user experience from a consumer and solution vendor/integrator
point of view.

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

As we don't plan on changing the usage much, plus the functions is currently not available outside of PowerShell, and there is currently no command for PowerShell 7+, so there is no need to change the Syntax found in Windows PowerShell 5.1.

The increment in PowerShell [MAJOR](https://semver.org/#spec-item-8)'s version field is enough to indicate the change of public API (as per [Semantic Versioning](https://semver.org/)).

```text
Invoke-DscResource [-Name] <string> [-Method] <string> -ModuleName <ModuleSpecification> -Property <hashtable> [<CommonParameters>]
```

#### Default Execution Scope: Current runspace

We aim at enabling existing scripts using `Invoke-DscResource`, written for Windows
PowerShell 5.1, to "just work" in PowerShell 7+, but they will run resources
**in the current user context**.

```PowerShell
Invoke-DscResource -Name Script -ModuleName @{ModuleName='PSDscResources';ModuleVersion='2.12.0.0'} -Method 'Set' -Property @{
    GetScript  = '<# My Get ScriptBlock #>'
    SetScript  = '<# My Set ScriptBlock #>'
    TestScript = '<# My Test ScriptBlock #>'
} -Verbose
```

This should run in the current session state where the command is invoked.

#### PsDscRunAsCredential: throw exception if supplied

After more discussions (with @Jaykul & @TravisEz13) and thinking this through,
it is **not** right to build the support for PsDscRunAsCredential in this command
(more on why later in this RFC).

It's the user's responsibility to wrap the call in the required user context
(such as using [Invoke-CommandAs](https://www.powershellgallery.com/packages/Invoke-CommandAs)
by Mark Kellerman).

Although `PsDscRunAsCredential` is a recommended practice for DSC in WMF 5.1 to
implement [least-privilege Administrative Models](https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/implementing-least-privilege-administrative-models),
it will **not** be supported as part of this implementation of `Invoke-DscResource`.

Instead, we'll aim at providing an example function in a script published
on the PowerShell Gallery to proxy this command, so that it handles `PsDscRunAsCredential`
to invoke the resource in a different user context, using PowerShell Jobs.

If the key `PsDscRunAsCredential` is to be found amongst the keys of the
`-Property` parameter, then `Invoke-DscResource` will throw an exception,
unless the `Invoke-DscResource` is invoked with the (new) switch parameter `-IgnorePsDscRunAsCredential`,
in which case it will invoke the DSC Resource method after stripping
the `PsDscRunAsCredential` key/value pair from the properties.

So calling:

```PowerShell
Invoke-DscResource -Name Script -ModuleName @{ModuleName='PSDscResources';ModuleVersion='2.12.0.0'} -Method 'Set' -Property @{
    GetScript  = '<# My Get ScriptBlock #>'
    SetScript  = '<# My Set ScriptBlock #>'
    TestScript = '<# My Test ScriptBlock #>'
    PsDscRunAsCredential = $Credential
} -Verbose

```

Will **throw an exception**, because of the `PsDscRunAsCredential` key.

> NOTES: We're avoiding to take dependencies on other technologies that would require extra configurations or permissions (such as remoting), or would not be available on other OSes.

#### Independent and isolated execution

`Invoke-DscResource` in PowerShell 7+ will not be aware of other instances being executed, and as such it will be possible to execute several instances in parallel when isolated in their own runspace, or run in parallel with the LCM.

This means that it enables concurrent execution, but also risks conflict if two conflicting resources are run simultaneously.

It is up to the user to sequence the execution safely, or to create appropriate resources.

#### Output & Types

##### Get

The `GET` function invoked via the LCM (WMF 5.1) returns a CIM representation of a hashtable,
and will be a `hashtable` for `Invoke-DscResource` in PS7+.

##### Test

The `TEST` function in WMF 5.1 returns a CIM object, with a `[bool]` NoteProperty
`InDesiredState` that has the boolean result of the test.

```PowerShell
InDesiredState
--------------
True
```

In PS7+, `Invoke-DscResource -Method Test ...` will return an object (not CIM)
that has the same `InDesiredState` Property.

##### Set

The `SET` function in WMF 5.1 returns a CIM object, with a `[bool]` NoteProperty
`RebootRequired`, corresponding whether the `$global:DSCMachineStatus` has been
set to 1 in the resource.

```PowerShell
RebootRequired
--------------
False
```

In PS7+, `Invoke-DscResource -Method Set ...` will return an object (not CIM) that
has the same `RebootRequired` Property, based on whether `$global:DSCMachineStatus`
has been set to 1. This property from the returned object is what the users should
use, and not the `$global:DSCMachineStatus` variable, subject to manipulation.

To avoid the manipulation of behavior by global variable,
the `$global:DSCMachineStatus` variable will be reset before invoking the resource.

For the same security concerns and to avoid misuse, the `Invoke-DscResource` cmdlet
will remove the variable `$global:DSCMachineStatus` from the session after invoking
the DSC Resource.

# Out of Scope for initial work & other notes

We're aware that some extra work or feature could be solved at the same time, but we're trying to have the MVP (minimum viable product) out as soon as possible, to help addressing the points raised in the [Motivation](#Motivation) section.

## DSC Resource Parameters & Supported Types

As the `Invoke-DscResource` does not need to serialize and deserialize parameters
using CIM (via MOF objects), it is not limiting the types it can accept for the
DSC Resource functions (Get, Set, Test).

In simple words, a resource could in theory accept an `[hashtable]` as a parameter
type, instead of a `[Microsoft.Management.Infrastructure.CimInstance[]]`.

The downside here is that it's not backward compatible, for this reason, we
can't recommend any other type to be used for now.

Also, not all types are easily serializable and deserializable, so be careful
when the Configuration Data has to be passed to the resource invocation over
the network.

## PsDscRunAsCredential Support not built-in

Here's why we've decided to not handle the `PsDscRunAsCredential` from the `Invoke-DscResource`
advanced function, and instead will try to publish a wrapper on the PS Gallery.

To be truly cross platform and support invoking a resource's method under another
credential, Linux/Unix OSes creates another process under that user. In Windows
it is also possible to Impersonate a user, using `Win32.AdvApi32` for instance.

In WMF 5.1, the LCM is in charge of this. But in PowerShell 7+, the only cross platform
way to do this is by using PowerShell Jobs, which are relatively slow, heavy, and
a bit more tricky to troubleshoot, as they need a way to serialize and deserialize
objects.

In PowerShell, executing commands as job returns what we informally call "dead objects":
the object after it was serialized and then deserialized, pertaining most of its
properties but not an "live" object that still has its methods available.

Here, we could have `Invoke-DscResource` to always use Jobs, but that would severely
impact performance, and would make troubleshooting difficult.

We could, as initially thought, handle both case: directly as the current user when
`PsDscRunAsCredential` is not specified, and as Job when specified, but it would
then have two different behavior, depending on the `-Property` value (and not
the command's actual signature), obfuscating the potential issue to the user.

For those reasons, and with the possibility that an elegant or standardized solution
emerges from the community, we believe it's best to leave it outside the scope of
the `Invoke-DscResource` MVP.

In order to enable existing users to support the same behavior than the LCM,
we will try to provide, separately, a function that wraps around this `Invoke-DscResource`
and creates a job when `PsRunAsCredential` is specified, but this won't be part
of the `PSDesiredStateConfiguration` module work.

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

# Alternate Proposals and Considerations

none
