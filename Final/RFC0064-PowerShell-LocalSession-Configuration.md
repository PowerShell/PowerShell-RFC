---
RFC: RFC0064
Author: Paul Higinbotham
Status: Draft
Area: Microsoft.PowerShell.Core
Comments Due: 5/20/2022
Plan to implement: Yes
---

# PowerShell Local Session Configuration

PowerShell currently supports extensive session configuration for WinRM remoting endpoints.
The configuration is defined in a .pssc file, which can be passed to the `Register-PSSessionConfiguration` cmdlet when creating a remoting endpoint.
This session configuration is limited to remoting endpoints based on WinRM (Windows Remote Management) connections only.
All other PowerShell remoting connections, such as SSH based remoting, cannot define endpoint configurations, and will connect to a PowerShell session having a default configuration.
A default session configuration is essentially a full PowerShell session without any restrictions.  

PowerShell session configuration is useful in remoting scenarios as it can restrict what is available to a user in that session, greatly improving the security stance of that session.
PowerShell uses remote session configuration to support JEA (Just Enough Administration) sessions, where a user role is limited to a small number of commands and actions that can be taken within the session.
Thus, the JEA session endpoint can be used to safely managed a high value machine, such as a domain controller, where the connected user can perform only a predetermined set of functions.  

But being able to configure a **local** PowerShell session is also useful.
For example, a machine policy, could enforce a PowerShell configuration that denies direct access to the file system, or the system registry, making it more difficult for malicious actors.  

Local sessions are also used to implement other PowerShell remoting connections, such as SSH, and being able to configure a local session would allow these non-WinRM remoting connections to have configurable endpoints.
This is important for high value machines where allowing a full PowerShell remote session is considered a security risk.  

## Motivation

As a system administrator, I want to create a PowerShell SSH remoting endpoint that does not allow full access to the target system but pre-defined allowed capabilities.

### Example 1

```powershell
PS C:\> Enter-PSSession -Host TargetComputer1 -User domain\user1 -ConfigurationName MySSHEndpoint
[domain\user1@TargetComputer1]: PS > dir c:\

Error: The term 'dir' is not recognized as a name of a cmdlet, function, script file, or executable program.
Check the spelling of the name, or if a path was included, verify that the path is correct and try again.

[domain\user1@TargetComputer1]: PS > Get-SystemState

com1           : @{data_len=1; inner=; io_port_base=1016}
com2           : @{data_len=1; inner=; io_port_base=760}
com3           : @{data_len=1; inner=; io_port_base=1000}
com4           : @{data_len=1; inner=; io_port_base=744}
...

[domain\user1@TargetComputer1]: PS > $NewSystemState | Set-SystemState
VERBOSE: System state successfully updated
[domain\user1@TargetComputer1]: PS > 
```

## Design

### PowerShell Session Configuration

PowerShell already supports session configuration through the `InitialSessionState` class.
It is used to initialize a PowerShell runspace, which essentially contains the runtime session configuration state.
The `InitialSessionState` class contains a number of static creation methods used to create predetermined configurations.
It also contains a number of properties that expose various session functions, such as LanguageMode, Providers, Variables, etc.
In addition, the `InitialSessionState` class has a static `CreateFromSessionConfigurationFile()` method that takes a path to a .pssc file, which contains session configuration options in PowerShell psd1 file format.  

```powershell
@'
@{
    SchemaVersion = '2.0.0.0'
    GUID = 'eaa72cae-fafe-4d3d-bf65-84ff02d58fd9'
    Author = 'contoso'
    Description = 'Test configuration'
    CompanyName = 'Unknown'
    Copyright = '(c) Contoso. All rights reserved.'
    SessionType = 'Default'
}
'@ | Out-File -FilePath c:\test.pssc

$isstate = [initialsessionstate]::CreateFromSessionConfigurationFile('C:\test.pssc')
[runspacefactory]::CreateRunspace($isstate)

Id Name            ComputerName    Type          State         Availability
 -- ----            ------------    ----          -----         ------------
  4 Runspace4       localhost       Local         BeforeOpen    None
```

So it is currently possible to create a configured PowerShell runspace through the PowerShell API.
But what most users want is a way to configure PowerShell from the command line, so that when running `pwsh.exe`, a PowerShell shell starts with the desired configuration.

### PowerShell Server Mode

The PowerShell executable, `pwsh.exe`, can be started in 'server mode' by including the non-public `-S` command line switch.
In this mode, PowerShell does not present a console window and prompt for command line input.
Instead, it runs a listener on the process standard input stream, listening for remoting protocol (PSRP) messages, and sending protocol message responses on the process standard output stream.
In this way PowerShell functions as a remoting endpoint for PowerShell remoting connections.
PowerShell's `job`, `named pipe`, and `SSH` remoting connections all use PowerShell in server mode as an endpoint to connect to.
In addition, the new [Custom Remote Connections feature](https://github.com/PowerShell/PowerShell-RFC/blob/master/Archive/Draft/RFC0063-Custom-Remote-Connections.md), will also use PowerShell in server mode for a remoting endpoint.  

However, PowerShell in server mode has no way to configure the session and currently runs in its normal 'default' configuration.  

### PowerShell `-ConfigurationFile` Command Line Switch

The main proposal of this RFC is to add a new command line switch to `pwsh.exe` that takes an optional file path string to a PowerShell configuration (.pssc) file.
This file would then be used to configure the default session runspace with the contained configuration information.
The new command line switch would make configuring PowerShell running in server mode very easy.  

For example, a PowerShell SSH connection endpoint is created by setting up a SSH 'subsystem' that runs `pwsh.exe`.
Multiple PowerShell remoting endpoints could be created by using the new `-ConfigurationFile` command line switch:

sshd_config file:

```powershell
...
# override default of no subsystems
#Subsystem  sftp        sftp-server
# PowerShell endpoint configurations
Subsystem   powershell1 /opt/microsoft/powershell/7/pwsh -S -NOLOGO -ConfigurationFile /etc/ssh/PSConfig1.pssc
Subsystem   powershell2 /opt/microsoft/powershell/7/pwsh -S -NOLOGO -ConfigurationFile /etc/ssh/PSConfig2.pssc
...
```

### Windows/WinRM Configuration Elements

Unfortunately, some of the configuration elements supported in a .pssc configuration file is for Windows/WinRM remote connections only.
For example, `virtual` and `group managed` accounts are Windows platform only.
And JEA role definitions are based on Windows platform accounts and groups.  

We want to separate Windows/WinRM specific configuration elements from what PowerShell supports implicitly.
The new `-ConfigurationFile` command line switch and policy setting will support PowerShell configurations only, and will throw a terminating error if unsupported WinRM/Windows configurations are specified.  

The following are Windows/WinRM configuration elements that will **not** be supported for local sessions:

```powershell
RunAsVirtualAccount
RunAsVirtualAccountGroups
GroupManagedServiceAccount
RoleDefinitions
RequiredGroups
PowerShellVersion
```

### PowerShell Configuration Elements

The following are configuration elements that **will** be supported:

```powershell
SessionType
TranscriptDirectory
MountUserDrive
UserDriveMaximumSize
ScriptsToProcess
LanguageMode
ExecutionPolicy
ModulesToImport
VisibleAliases
VisibleCmdlets
VisibleFunctions
VisibleExternalCommands
VisibleProviders
AliasDefinitions
FunctionDefinitions
VariableDefinitions
EnvironmentVariables
TypesToProcess
FormatsToProcess
AssembliesToLoad
```

## Alternate Design Options

### PowerShell `SessionConfiguration` Policy Setting

Additional to the `-ConfigurationFile` command line switch, session configuration could also be settable via Windows Group Policy or PowerShell configuration file.
The policy session configuration would be specified by either a configuration file path or a string defining the configuration.

```powershell
HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\PowerShellCore\SessionConfiguration
    ConfigurationFile   REG_SZ  'C:\Windows\System32\PowerShell\v7.2.3\SessionConfig.pssc'
    ConfigurationHash   REG_SZ  '@{ SchemaVersion = "2.0.0.0"; GUID = "eaa72cae-fafe-4d3d-bf65-84ff02d58fd9"; ...}'
```

### PowerShell `-Configuration` Command Line Switch

A possible variation is to also include a `-Configuration` command line that takes a text string containing session configuration information in PowerShell hash table format.
But this seems less useful, since configurations can be very complex.

```powershell
C:\> pwsh.exe -Configuration '@{SessionType = "Default"; ...}'
```

### Configuration Through Application Control Policies

Another idea is to extend this ability to specify a machine wide PowerShell session configuration to Application Control policies.
For example, the Windows WDAC (Windows Defender Application Control) policy could include the ability to specify a .pssc file used for PowerShell sessions when the system has WDAC polices enforced.
