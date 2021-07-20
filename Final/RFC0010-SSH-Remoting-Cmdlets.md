---
RFC: 10
Author: Paul Higinbotham
Status: Final
Version: 0.1
Area: Remoting
Comments Due: October 28, 2016
---

# SSH Remoting Cmdlets

PowerShell remoting over SSH (Secure Shell) was added to PowerShell core as an alternative to traditional WinRM based remoting.  SSH is popular and ubiquitous in the Linux world and is now supported on Windows through the [Win32-OpenSSH] effort.  This makes it an attractive candidate for PowerShell multiplatform remoting.  

The current implementation of SSH remoting is at an evaluation stage.  Existing remoting cmdlets were given new parameter sets for establishing a remote connection through SSH that reflects how SSH works.  So the parameters were HostName, UserName, and KeyFilePath to support SSH based interactive password prompt and key identity authentication.   

```PowerShell
ssh UserName@HostName -identity_file (optional)
```

This works fine for a proof of concept but PowerShell is all about automation and the remoting cmdlets are designed to support automation as well as one off interactive sessions.  Remoting automation is provided with cmdlet parameter sets that take lists of computer names for fan-out execution of commands or scripts.  In Windows, fan-out remoting is implemented with PSCredential objects and requires credential passing that WinRM supports.  SSH does not currently support credential passing but does support key based identity authentication that can be used for fan-out.  This would mean that private/public keys need be generated and distributed appropriately to all affected machines, but once this is done fan-out remoting automation can be easily achieved with some modifications to the remoting cmdlets.  

In addition SSH must be configured appropriately on all machines including what authentication modes are allowed and with PowerShell SSH remoting enabled.  It is desirable to have cmdlets that assist with this (similar to Enable-PSRemoting for WinRM remoting), but is outside the scope of this RFC and will be addressed in a separate one.  

So PowerShell SSH based remoting cmdlets need to support both authentication keys for automation/fan-out scenarios where no user interaction is desired, as well as user password prompt for single PowerShell remote sessions.

[Win32-OpenSSH]: https://github.com/PowerShell/Win32-OpenSSH/

## Motivation

As an IT administrator, I can use existing PowerShell remoting cmdlets to start interactive sessions or run fan-out automation across both Linux and Windows platforms using SSH remoting.

## Specification

The proposal is to add two new parameter sets to the three primary remoting cmdlets (New-PSSession, Enter-PSSession, Invoke-Command) that provide for SSH password prompt and key authentication connection semantics.  The first parameter set is intended more for interactive command line use.  The second parameter set is designed for automation fan-out where SSH connection information can be specified for each target computer.

### Specification Update
<blockquote>
<p>After discussing comments and experimenting with parameter set variations I have the final parameter set specification.  Ideally the SSH based remoting parameter sets would be a simple extension of the existing WinRM based remoting parameter sets.  However, SSH remoting is significantly different from WinRM remoting and trying to combine the two parameter sets results in a confusing mishmash. So we will keep the separate parameter sets.  Another issue I ran into was while trying to use "ComputerName" in all parameter sets.  Unfortunately this resulted in breaking positional based use of the existing parameter sets and I ultimately had to use "HostName" instead of "ComputerName" for SSH parameter sets. On reflection I feel this is actually a good thing.  SSH based remoting is different enough from WinRM remoting where I feel making the SSH remoting cmdlet parameter sets distinct is less confusing.  
</p>
</blockquote>

First Parameter Set  
- HostName parameter
   + Specifies one or more target computers.
   + This parameter is required.
- UserName parameter
   + Specifies an optional user name to be used for the connection. 
   + If not specified the current log on user name is used.
   + A password prompt will occur if SSHD service is configured for it, otherwise SSH key authentication must be configured.
- KeyFilePath parameter
   + Alias: IdentityFilePath.
   + Specifies an optional private key file path used to authenticate a user account on each target machine.
      + This is equivalent to the SSH program -identity_file parameter.
      + Password prompt fallback occurs on key authentication failure if SSHD service is configured for it.
   + If not specified then SSHD will use console prompt or authentication keys stored in known locations depending on SSHD configuration.
- SSHTransport parameter
   + This is a switch parameter that specifies using the SSH transport.
   + It is needed to distinguish the SSH parameter set from other WinRM parameter sets.

Second ParameterSet
- SSHConnection parameter
   + Takes a HashTable object that specifies connection information.
      - HostName (ComputerName)           (required).
      - UserName                          (optional).
      - KeyFilePath (IdentityFilePath)    (optional).

Examples:

Parameter set syntax
```powershell
New-PSSession [-HostName] <string[]> [-Name <string[]>] [-UserName <string>] [-KeyFilePath <string>] [-SSHTransport] [<CommonParameters>]

New-PSSession -SSHConnection <hashtable[]> [-Name <string[]>] [<CommonParameters>]


Invoke-Command -ScriptBlock <scriptblock> -HostName <string[]> [-AsJob] [-HideComputerName] [-UserName <string>] [-KeyFilePath <string>] [-SSHTransport] [-InputObject <psobject>] [-ArgumentList <Object[]>] [<CommonParameters>]

Invoke-Command -ScriptBlock <scriptblock> -SSHConnection <hashtable[]> [-AsJob] [-HideComputerName] [-InputObject <psobject>] [-ArgumentList <Object[]>] [<CommonParameters>]

Invoke-Command -FilePath <string> -HostName <string[]> [-AsJob] [-HideComputerName] [-UserName <string>] [-KeyFilePath <string>] [-SSHTransport] [-InputObject <psobject>] [-ArgumentList <Object[]>] [<CommonParameters>]

Invoke-Command -FilePath <string> -SSHConnection <hashtable[]> [-AsJob] [-HideComputerName] [-InputObject <psobject>] [-ArgumentList <Object[]>] [<CommonParameters>]


Enter-PSSession [-HostName] <string> [-UserName <string>] [-KeyFilePath <string>] [-SSHTransport] [<CommonParameters>]
```

Interactive session with implicit user
```PowerShell
PS C:\> Enter-PSSession -SSHTransport -HostName WindowsVM1
[WindowsVM1]: PS C:\> whoami
Domain\CurrentUser
[WindowsVM1]: PS C:\>
```

Interactive session with explicit user
```PowerShell
PS C:\> Enter-PSSession -SSHTransport -HostName LinuxVM1 -UserName LinuxUser
LinuxUser@LinuxVM1's password:
[LinuxVM1]: PS C:\> whoami
LinuxUser
[LinuxVM1]: PS C:\> 
```

Interactive session with explict key authentication
```PowerShell
PS C:\> Enter-PSSession -SSHTransport -HostName LinuxVM1 -UserName LinuxUser -IdentityFilePath c:\Keys\.ssh\LinuxUserKey_rsa
[LinuxVM1]: PS C:\> whoami
LinuxUser
[LinuxVM1]: PS C:\> 
```

Interactive session with persisted session
```PowerShell
$session = New-PSSession -SSHTransport -HostName WindowsVM1
Invoke-Command -Session $session -ScriptBlock { $MyVariable = "Hello!" }
Enter-PSSession -Session $session
[WindowsVM1]: PS C:\> $MyVariable
Hello!
[WindowsVM1]: PS C:\>
```

Fan-out remoting automation
```PowerShell
$connectionList = @(
    @{
        ComputerName = 'WindowsVM1'
        UserName = 'WindowsUser'
        KeyFilePath = 'c:\Keys\.ssh\WindowsUser_rsa'
    },
    @{
        HostName = 'LinuxVm1'
        UserName = 'LinuxUser'
        IdentityFilePath = 'c:\Keys\.ssh\LinuxUser_rsa'
    }
)
$job = Invoke-Command -SSHConnection $connectionList -ScriptBlock { Get-Process powershell } -AsJob
$job | Wait-Job | Receive-Job

Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName                    PSComputerName
-------  ------    -----      -----     ------     --  -- -----------                    --------------
      0       0        0         19       1.50  23781 781 powershell                     LinuxVM1
      0      36    79568      59896      20.00  10180   2 powershell                     WindowsVM1
      0      28    73328      52548       1.86  11652   2 powershell                     WindowsVM1
      0      63    39568      61760       3.16  17516   2 powershell                     WindowsVM1
      0      74    65012      73892       9.05  23596   2 powershell                     WindowsVM1
      0      62    36380      58408       3.17  24344   0 powershell                     WindowsVM1
```

Pros: 
 + Extends existing remoting cmdlets to use SSH as transport while keeping SSH connection semantics.  
 + Covers interactive and automation scenarios.
 + Supports persisted sessions.

Cons:
 + Does not support credential passing.
 + SSH and identity keys must be installed and configured correctly on all machines.

## Alternate Proposals and Considerations

Another approach is to think of SSH based remoting as completely separate from the existing WinRM based remoting.  PowerShell remoting was created on top of WinRM and so all cmdlets have WinRM specific features that may never apply to SSH remoting.  An alternative would be to have separate cmdlets that apply only to SSH remoting.

Pros:
 + New cmdlets would be optimized for SSH experience only.

Cons:
 + New cmdlets would be very similar to existing WinRM cmdlets and duplicate much of the functionality.
 + Increases the number of remoting cmdlets.

---------------
## PowerShell Committee Decision

### Voting Results

Jason Shirk: Accept 

Joey Aiello: Accept

Bruce Payette: Absent

Steve Lee: Accept

Hemant Mahawar: Accept

### Majority Decision

We covered alternatives in regards to an enum for -Transport and this brought up a need to revisit both authoring and displaying complex parameter sets.
Decided that the current RFC is sufficient for current implementation needs.
Should consider a wrapper cmdlet to simplify usage of SSH remoting with PowerShell.


### Minority Decision

N/A
