---
RFC: RFC00XX
Author: Paul Higinbotham
Status: Draft
Area: Microsoft.PowerShell.Core
Comments Due: 10/22/2021
Plan to implement: Yes
---

# Custom Remote Connections

PowerShell currently supports six types of remote connections.

- WinRM remote connections
- SSH remote connections
- Named pipe local connections
- Local process connections
- Hyper-V local VM connections (Windows only)
- Hyper-V local Container connections (Windows only)

But these connections are implemented internally in the PowerShell engine, and adding new connection types requires modifying the PowerShell engine.
This RFC is a proposal to add support for optional custom remote connections, so that new remoting connection types can be created and used without having to change PowerShell engine code.  

Custom endpoints and listening services are out of scope for this proposal.
PowerShell remoting does not provide endpoint hosting or connection listeners, and relies on external solutions such as what WinRM and SSH provide.

## Motivation

As a third party contributor or PowerShell team member, I can create a new remoting connection type as a PowerShell module.
Users can download the module and use it with PowerShell remoting cmdlets.

### Example 1

PowerShell core used to provide an experimental remoting connection from non-Windows machines to Windows machine WinRM endpoints, through an OMI bridge.
But it is no longer supported and OMI changes break this connection on many non-Windows platforms.  

As a user and community member, I want to create a module that will support and maintain this connection independent of the PowerShell team and without having to submit changes to PowerShell engine code.  

```powershell
PS /Users/Paul> Install-PSResource -Name PSWSManConnection -Repository PSGallery
PS /Users/Paul> $session = New-PSWSManPSSession -Computer WindowsMachine1 -Credential Domain\UserName

PowerShell credential request
Enter your credentials.
Password for user Domain\UserName: **************

PS /Users/Paul> Enter-PSSession $session
[WindowsMachine1]: PS C:\> Users\UserName> 
```

This example installs the `PSWSManConnection` module from the PowerShell Gallery onto a macOS machine.
Next the user runs the module provided `New-PSWSManPSSession` cmdlet to create a `PSSession` session instance using the new connection.
The `PSSession` object is then used with PowerShell's `Enter-PSSession` cmdlet to interact directly with the remote session from the command line.

### Example 2

I have created a remote shell solution that uses custom multi-factor authentication, and is installed on all of the machines I am responsible for managing.
I want to use PowerShell remoting over this custom shell connection.  

I can do this by creating a PowerShell module, `MyCustomConnection`, that uses my custom remote shell.  

```powershell
PS /Users/Paul> $sessions = New-MyCustomConnection -Computer $computerList -Credential $creds -UseMFA
PS /Users/Paul> $results = Invoke-Command -Session $sessions -File ./Scripts/WeeklyManagementTasks.ps1
PS /Users/Paul> Invoke-AnalyzeResults $results
PS /Users/Paul> $sessions | Remove-PSSession
```

This example creates a set of remote sessions from a list of computers to be managed, using a connection and authentication from a custom module.
Once the sessions are created, existing PowerShell core remoting cmdlets are used to invoke script on the sessions and manage them.
The PowerShell `Invoke-Command` is used to run a weekly management script on all sessions in parallel, and then analyze the returned results.
Lastly, the sessions to the remote machines are all closed using the PowerShell `Remove-PSSession` cmdlet.

## Design

The design proposal is to allow third parties to create their own custom remoting connections by deriving from and providing custom implementations of the `RunspaceConnectionInfo` and `ClientSessionTransportManagerBase` abstract classes.
These implementations would be part of a PowerShell binary module, which also includes a cmdlet that returns a live connection instance as a `PSSession` object.
The `PSSession` object can then be passed to existing PowerShell remoting cmdlets to interact with the session.
The implementation binary can also include other relevant cmdlets and expose public APIs for creating and managing the custom connection.

### RunspaceConnectionInfo

The `System.Management.Automation.Runspaces.RunspaceConnectionInfo` abstract class is already publicly accessible.
Derivations of this class are used to define specific connection types, including new properties needed for the connection or authentication.
A derivation must also provide an implementation for the `CreateClientSessionTransportManager` method, which returns an instance of the client session transport manager for the new connection.  

### ClientSessionTransport

The `ClientSessionTransportManagerBase` abstract class is not currently publicly accessible, and will need to be made public.
Derivations of this class are used to bind live connection data streams to the PowerShell remoting layer.  

Once a connection is established, the data streams are bound to internal PowerShell remoting components through a `ClientSessionTransportManager`, which is derived from the public `ClientSessionTransportManagerBase` abstract class.  

The derived class will override a few virtual functions, the most important of which is the `CreateAsync` method.
This method runs immediately before protocol messaging begins, and needs to perform two vital functions:

- Create a text writer wrapper object, that is used to write protocol messages to the target input data stream.

- Start a listening thread for protocol messages from the target output data stream.

### Data Streams

All connection data streams used in the transport must be derived from .NET TextReader or TextWriter classes.
Connections typically have two data streams, one for writing messages to the target, and one for reading messages from the target.
However, some connections have a third error stream that require a special reader thread.
An example would be a Process connection that uses stdOut for reading from the target, stdIn for writing to the client, and stdError for reading connection error messages.
The stdError messages are not part of the remoting protocol and need to be handled separately.

### Creating a connection

The PowerShell module implementing the new remote connection should supply all necessary cmdlets for creating and managing the connection.
In particular it should have a cmdlet that returns a `PSSession` object.

```powershell
PS C:\> Import-Module -Name MyCustomConnection
PS C:\> New-MyCustomSession -ComputerName remoteVM1 -Cred $myCredentials

Id Name            Transport ComputerName    ComputerType    State         ConfigurationName     Availability
 -- ----            --------- ------------    ------------    -----         -----------------     ------------
  1 Runspace1       MyCustom  remoteVM1       RemoteMachine   Opened        DefaultShell             Available
```

However, if the new connection info class is made public accessible, the PowerShell remoting API can also be used to create a connection instance.

```powershell
PS C:\> Import-Module -Name MyCustomConnection
PS C:\> $connectionInfo = [MyCustomConnection.CustomConnectionInfo]::new($computerName, $creds)
PS C:\> $runspace = [runspacefactory]::CreateRunspace($connectionInfo)
PS C:\> $runspace.Open()
PS C:\> $runspace

 Id Name            ComputerName    Type          State         Availability
 -- ----            ------------    ----          -----         ------------
  2 Runspace2       remote-1        Remote        Opened        Available
```

### Endpoints

A remote connection must include a PowerShell session on the target machine, that handles remoting protocol messages from the client.
This is called the remoting endpoint.
An endpoint typically involves a service that listens for client connection requests, and securely creates the PowerShell session under the correct account and with correct permissions.
For example, both WSMan and SSH have services that do this.
Endpoints such as these are outside the scope of this feature proposal, and it is assumed that any custom connection scheme will include some sort of endpoint support.  

PowerShell currently supports a 'server' running mode, where remoting protocol messages are channeled through the process stdIn/stdOut data streams.
Any custom endpoint solution will leverage this to establish a PowerShell session and handle remoting protocol messages.
Currently, this is the only way to run PowerShell as an endpoint server, and should be sufficient for any custom solution.
Also this server mode is available on all supported PowerShell versions (v5.1 - v7.1+).

#### Example - PowerShell in server mode

This example illustrates how PowerShell can be run in server mode.

```powershell
# Create PowerShell endpoint session (with default shell configuration)
# Protocol handles messages through the child process stdIn/stdOut process pipes
$proc = Start-Process -FilePath pwsh.exe -ArgumentList '-servermode -nologo -noprofile' -PassThru

# Pass child process object to custom connection endpoint handler that reads/writes connection streams from stdIn/stdOut pipes
[MyCustomConnection.CustomConnectionEndpoint]::SetEndPointProcess($proc)
```

### Implementation Example

This is a simple example of how to create a custom ConnectionInfo and Transport implementation

```csharp
namespace MyCustomConnection
{
    // Derived custom connection information class
    public sealed class CustomConnectionInfo : RunspaceConnectionInfo
    {
        // New multi-factor authentication switch for connection.
        // Otherwise use existing 'ComputerName' and 'Credential' properties from derived class.
        public SwitchParameter MFA { get; private set; }

        // Message reader and writer.
        internal StreamReader messageReader { get; private set; }
        internal StreamWriter messageWriter { get; private set; }

        // Override and implement this method that creates the custom client session transport with
        // bound live connection data streams.
        // This method is called from within the internal PowerShell remoting implementation.
        public override BaseClientSessionTransportManager CreateClientSessionTransportManager(
            Guid instanceId,
            string sessionName,
            PSRemotingCryptoHelper cryptoHelper)
        {
            // Establish connection with this connection information and create connection reader/writer
            // data streams.
            StartConnection();

            return new CustomConnectionClientSessionTransportManager(
                this,
                instanceId,
                cryptoHelper);
        }
    }

    // Derived custom client session transport manager class.
    public sealed class CustomConnectionClientSessionTransportManager : ClientSessionTransportManagerBase
    {
        CustomConnectionInfo _connectionInfo;

        private CustomConnectionClientSessionTransportManager()
        { }

        // Constructor.
        internal CustomConnectionClientSessionTransportManager(
            CustomConnectionInfo connectionInfo,
            Guid runspaceId,
            PSRemotingCryptoHelper cryptoHelper) : base (runspaceId, cryptoHelper)
        {
            if (connectionInfo == null) { throw new PSArgumentException("connectionInfo"); }
            _connectionInfo = connectionInfo;
        }

        // Start connection protocol.
        // This is called by internal PowerShell remoting implementation.
        public override void CreateAsync()
        {
            // Create writer for this connection.
            stdInWriter = new TransportTextWriter(_connectionInfo.messageWriter);

            // Start reader thread for this connection.
            StartReaderThread(_connectionInfo.messageReader);
        }
    }
}
```

## Alternate Design Options

### PowerShell Subsystem

Another way to implement a custom connection, other than as a PowerShell module, is as a PowerShell subsystem.
PowerShell subsystems are an experimental feature where external feature implementations can be registered with PowerShell.
A registered subsystem can integrate with PowerShell via a public interface, but also define cmdlets that become available in the session.  

This proposal does not use the engine subsystem infrastructure because it was deemed unnecessary.
Implementing custom connections as a PowerShell module is sufficient, and subsystems provide no additional value.

## Background Information

### PowerShell Remoting

PowerShell remoting consists of a number of internal engine components.
It provides a way to establish a PowerShell session on a remote target, which can be another physical machine, virtual machine, or local process.
Commands and scripts can then be run in the remote session, with data objects being returned to the client.
PowerShell manipulates object instances and so returned data are objects that are serialized when transported over the remoting connection, and then deserialized after arriving at the destination.
If the object type is known, PowerShell will deserialize it back into its original type.
Unknown types are deserialized into a general PowerShell `PSCustomObject` type, that is essentially a property bag.  

The core part of PowerShell remoting is the remoting protocol (PSRP) implementation, which handles all messages and data over the connection.
But there is also the need to create the connections, along with mechanisms to transport protocol messages/data over the connection.
These functions are considered to be outside PowerShell core remoting, and specific to each connection type.
PowerShell provides abstractions for these functions, but leaves the implementation to each connection type.  

#### Example - WinRM remoting

This is the first remoting connection supported in PowerShell 2.0 and it is based on Windows WinRM.
WinRM is Windows secure remote shell solution.
PowerShell remoting uses WinRM to establish a remote connection, and then create a PowerShell session on a remote target.
PowerShell uses the WinRM plugin architecture to create PowerShell target endpoint sessions as WinRM remote shell instances.
The PowerShell engine provides WinRM specific connection objects, `WSManConnectionInfo`, to create a connection through WinRM.
It also contains a transport implementation for transferring protocol messages and data through the WinRM connection.

#### Example - SSH remoting

SSH remoting was added to PowerShell v6.0 and is similar to WinRM remoting, in that SSH is also a secure remote shell solution.
SSH remoting works on all PowerShell platforms (Linux, macOS, Windows), that have SSH installed.
Similar to WinRM, SSH authenticates a user and establishes the PowerShell remoting connection to the target computer.
The connection is defined in an SSH specific `SSHConnectionInfo` object.
There is also an SSH specific transport implementation that transfer the protocol messages and data over the SSH connection.
The target endpoint PowerShell session is created by SSH as a configured subsystem, where PowerShell runs in a server mode that handles remoting protocol messages.

#### Example - Local process remoting

PowerShell jobs (`Start-Job`) use PowerShell remoting but the remote connection is to a local child process instead of a remote machine.
Per the common remoting pattern, there is a connection object, `NewProcessConnectionInfo`, that defines the connection, and a transport implementation used to transfer protocol message/data over the connection.
Establishing the connection means creating the child process where the job will run, and then cleaning up the child process after the connection/session is terminated.
