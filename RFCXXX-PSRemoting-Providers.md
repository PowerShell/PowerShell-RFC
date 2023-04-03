---
RFC: RFCXXX
Author: Jordan Borean
Status: Draft
SupercededBy:
Version: 0.1
Area: remoting
Comments Due: May 3, 2023
Plan to implement: No
---

# Improving PSRemoting Providers with an official provider registration

## Motivation

There are 2 problems this RFC is aiming to help with:

+ As a user I want to have a unified way of using PSRemoting providers in my `Invoke-Command` calls so that I can simplify my code.

+ As a module author I want to be simplify the work needed to implement and use my custom remoting providers

### Current Landscape

PowerShell Remoting has a few builtin mechanisms used by PowerShell to run commands on a remote instance of PowerShell and with the introduction of custom remoting plugins in PowerShell 7.3 this list is potentially limitless.
The UX of how a user specifies what transport to use for a PSRemoting plugin is currently hardcoded to specific parameters or custom cmdlets a module might implement.
This makes it difficult for users new to PSRemoting, let alone PowerShell, to understand how remoting is done in scripts.
For example the main cmdlet used is `Invoke-Command` and it currently has 17 parameter sets used to select the PSRemoting transport.
Some of the provider specific parameters in this cmdlet are:

|Parameter|Transport|
|-|-|
|`-HostName`/`-SSHConnection`|SSH|
|`-ComputerName`/`-ConnectionUri`|WinRM|
|`-VMId`/`-VMName`|Hyper-V|
|`-ContainerId`|Containers|
|`-Session`|Custom|

Here is it in action:

```powershell
# Connects over WinRM
Invoke-Command -ComputerName WinRMHost -ScriptBlock { 'test' }

# Connects over SSH
Invoke-Command -HostName SSHHost -ScriptBlock { 'test' }
```

The only way for the user to know this is to read through the docs or look at existing examples.
This is no simple task considering the official docs for [Invoke-Command](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/invoke-command?view=powershell-7.3) first displays the 17 parameter set syntaxes which are most likely unrelated to the specific scenario the user has.

Custom provider are even more complicated where the user must manage the session lifetime themselves by using a cmdlet provided by the module that implemented the provider.

```powershell
$session = New-CustomTransportSession -Target ...
try {
    Invoke-Command -Session $session { 'test' }
}
finally {
   $session | Remove-PSSession
}
```

This has now turned a single cmdlet call into something more complex and error prone with the introduction of the `try/finally` that users could omit from their code.
As well as this, each provider needs to implement their own `New-CustomTransportSession` cmdlet rather than utilitise the hard work already done in `Invoke-Command`, `New-PSSession`, `Enter-PSSession`, etc.
This can lead to substandard error messages, cancellation problems, as the current interface for doing so has a few complexities around it.
It's also complicated further as now the creation of the session must be done by the user and the fan-out feature of `Invoke-Command` is no longer available.
For example the following can be used to connect to 2 hosts in parallel using the builtin SSH provider:

```powershell
Invoke-Command -HostName Host1, Host2 -ScriptBlock { 'test' }
```

Trying to implement this for custom providers would require some complicated solution like the following:

```powershell
$sessions = 'Host1', 'Host2' | ForEach-Object -Parallel {
    New-CustomTransportSession -Target $_
}

try {
    Invoke-Command -Session $session { 'test' }
}
finally {
    $sessions | Remove-PSSession
}
```

Introduction the `-Parallel` block itself even brings in more complications, for example `$using:var` is required when referring to vars outside of the scriptblock.
What was a single cmdlet has now turned into multiple lines without any real error handling.
This can be a fairly large obstacle for custom transport providers to overcome when it comes to getting people willing to try out their code vs what's built into PowerShell itself.

## Proposed Solution

The proposed solution would be to implement a registry of remoting providers that modules can register to.
This registration would include a unique prefix allowing users to simply specify the transport type as part of the host connection string.
More examples can be seen in the `User Experience` header but in short it would turn this:

```powershell
$sessions = 'Host1', 'Host2' | ForEach-Object -Parallel {
    New-CustomTransportSession -Target $_
}

try {
    Invoke-Command -Session $session { 'test' }
}
finally {
    $sessions | Remove-PSSession
}
```

Into

```powershell
Invoke-Command -ComputerName custom:Host1, custom:Host2
```

Existing providers can also have their own host prefix allowing things like `-ComputerName ssh:Host1` to connect over SSH to `Host1`.
While the other parameters can't be removed straight away, it now opens the possibility to simplify the existing parameter set in the future to just a single `-ComputerName` parameter

## User Experience

These examples use the [PWSExecConn POC](https://github.com/jborean93/PWSExecConn) as the example custom remoting provider to see what this could potentially look like.
To use this provider with `Invoke-Command` it would look like:

```powershell
Import-Module PWSExecConn

Invoke-Command -ComputerName psexec:Hostname -ScriptBlock { 'test' }
```

Specifying connection specific options can still be done using a custom cmdlet that outputs connection specific options that inherit from `PSSessionOption` like:

```powershell
Import-Module PWSExecConn

$pso = New-PWSExecSessionOption -LogonType Batch
Invoke-Command -ComputerName psexec:Hostname -ScriptBlock { 'test' } -SessionOption $pso
```

It is also theoretically possible to embed connection specific details in the target string, for example using a username and port with SSH.

```powershell
Invoke-Command -ComputerName 'ssh:username@target:2222' -ScriptBlock { 'test' }
```

The interpretation of the value beyond the provider identifier is entirely up to the provider itself.

This can also be used by a provider to provide multiple ways to specify a connection to the same provider type.
For example the [NamedPipeConnectionInfo](https://learn.microsoft.com/en-us/dotnet/api/system.management.automation.runspaces.namedpipeconnectioninfo?view=powershellsdk-7.3.0) connection type supports both a process ID or pipe name target.
This provider could register 2 provider names `pid` and `pipe` allowing the following to be possible:

```powershell
Invoke-Command -ComputerName 'pid:1234', 'pipe:PSHostPipe' -ScriptBlock { 'test' }
```

It should also be possible for users to see currently registered providers with a cmdlet like `Get-PSRemotingProvider`

```powershell
PS C:\> Get-PSRemotingProvider

Name     Provider
----     --------
pid      NamedPipeConnectionInfo
pipe     NamedPipeConnectionInfo
psexec   PSExecConnectionInfo
ssh      SSHConnectionInfo
winrm    WSManConnectionInfo
```

From an engine perspective this would require a new property on the [SessionState](https://learn.microsoft.com/en-us/dotnet/api/system.management.automation.sessionstate?view=powershellsdk-7.2.0) and [SessionStateProxy](https://learn.microsoft.com/en-us/dotnet/api/system.management.automation.runspaces.sessionstateproxy?view=powershellsdk-7.0.0) class to store the registered providers.
This property could be called `RemotingProvider` which would use something like this class definition:

```csharp
using System;
using System.Management.Automation;
using System.Management.Automation.Remoting;
using System.Management.Automation.Runspaces;

namespace System.Management.Automation;

// Parameters from Invoke-Command/etc
public sealed class PSConnectionOption
{
    public int Port { get; }
    public PSCredential Credential { get; }
    public PSSessionOption Options { get; }
    ...
}

// Called by pwsh internally when creating a new connection
public delegate RunspaceConnectionInfo BuildConnectionInfo(string target, PSConnectionOption options);

public sealed class PSRemotingProvider
{
    public string Name { get; set; }

    public string Provider { get; set; }

    public BuildConnectionInfo ConnectionBuilder { get; set; }
}

public sealed class RemotingProviderManagementIntrinsics
{
    public BuildConnectionInfo Get(string identifier)
    {}

    public PSRemotingProvider New(PSRemotingProvider provider)
    {}

    public void Remove(PSRemotingProvider provider)
    {}
}
```

When a module is imported it can register it's own custom provider identifier through the standard mechanisms:

```csharp
using System;
using System.Management.Automation;

namespace PWSExecConn;

public class OnModuleImportAndRemove : IModuleAssemblyInitializer, IModuleAssemblyCleanup
{
    private PSRemotingProvider? _provider = null;

    public void OnImport()
    {
        _provider = new("psexec", typeof(PSExecConnectionInfo).Name, CreateConnection);
        Runspace.DefaultRunspace.SessionStateProxy.RemotingProvider.New(_provider);
    }

    public void OnRemove()
    {
        if (_provider != null)
        {
            Runspace.DefaultRunspace.SessionStateProxy.RemotingProvider.Remove(_provider);
            _provider = null;
        }
    }

    public static PSExecConnectionInfo CreateConnection(string target, PSConnectionOption options)
    {
        return new PSExecConnectionInfo(target);
    }
}
```

Custom providers are also free to ignore all this and continue to use the existing behaviour exposed in PowerShell 7.3.

## Open Questions

+ Should the `-ComputerName` parameter be changed in favour of another parameter

The `-ComputerName` parameter has been around since the v2.0 days so it would make sense to try and reuse this.
Non provider prefixed connection strings can still continue to use WinRM by default for backwards compatibility.
Using a new parameter does make it easier to remove the WinRM cruft that is currently present in the existing `-ComputerName` parameter set but at the expense of adding yet another parameter set to the cmdlets.

+ Should users be able to define their own provider ids

It could be possible to expose a `New-PSRemotingProvider` cmdlet that allows users to define their own connection prefixes.
These prefixes could have a "default" connection option set associated with it that makes it easier to change connection defaults if desired.
For example this would register `psexec_system:` that runs each connection as the `SYSTEM` user (a PWSExec specific option) if that prefix was used.

```powershell
New-PSRemotingProvider -Id psexec_system -ConnectionInfo (New-PSWExecConnectionOption -RunAs System)

Invoke-Command -ComputerName psexec_system:Host1 -ScriptBlock { whoami }
```

Unfortunately the UX around this would need to be tidied up to allow a user to easy designate the connection info builder needed by PowerShell in this proposed scenario.

## Considerations

It should also be mentioned that [RunspaceConnectionInfo](https://learn.microsoft.com/en-us/dotnet/api/system.management.automation.runspaces.runspaceconnectioninfo?view=powershellsdk-7.0.0) and [PSSessionOption](https://learn.microsoft.com/en-us/dotnet/api/system.management.automation.remoting.pssessionoption?view=powershellsdk-7.2.0) currently contains a few WinRM specific properties.
These should be cleaned up to provide a simpler implementation interface for custom providers before this is implemented.

There are a few other rough edges in the custom remoting plugin interface that would be investigated and cleaned up to make it easier for people to implement their own providers.

## Alternate Proposals

An alternate proposal would be to have `-ComputerName` (or another parameter) accept an object or hashtable that specifies the transport provider and connection options.
For example this could be possible:

```powershell
Invoke-Command -HostInfo @(
    New-SSHConnectionInfo -ComputerName Host1 -UserName username
    New-WinRMConnectionInfo -ComputerName Host2 -Authentication Kerberos
    New-PWSExecConnectionInfo -ComputerName Host3 -RunAs System
) -ScriptBlock { 'test' }
```

Builtin providers would need to expose this for each of the providers it provides and custom providers would need to do the same as their own.
While simpler from an implementation perspective and it can include provider specific options in the connection, it does have the following downsides:

+ Not as easily discoverable for users, they would need to know what's available beforehand
+ It's a lot more verbose, not as simple as just `provider:HostName`
+ It's another cmdlet a custom module needs to implement and document
