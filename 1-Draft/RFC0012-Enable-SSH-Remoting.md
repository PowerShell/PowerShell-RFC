---
RFC: RFC0012
Author: Paul Higinbotham
Status: Draft
Version: 1.0
Area: Remoting
---

# Enable SSH Remoting
Currently PowerShell SSH remoting is complicated by the need to have SSH installed and configured on all involved machines.  Most of this is just normal SSH package installation and configuration, but it can be made easier for PowerShell remoting by creating a new cmdlet to handle the details.  The goal is to enable at minimum SSH based remoting configuration with password prompt that allows interactive PowerShell remoting.  Alternate forms of authentication, such as key base authentication, will not be supported as it requires safely generating and distributing private/public keys.  Endpoint configuration is not yet supported in SSH based remoting but when it is this cmdlet should also ensure that a default endpoint is created when needed.

The cmdlet will perform the following functions:
  - Client scope
     + Check for SSH installation
     + Install SSH client package as needed
     + Use default SSH configuration

  - Server scope
     + Check for SSHD installation
     + Install SSHD server package as needed
     + Use default SSHD configuration (with password prompt allowed)
     + Add PowerShell subsystem configuration to allow hosting an SSH remote PowerShell session
     + Create default PowerShell endpoint as needed (and when supported)
     + Start service/daemon

  - All scope
     + Install and configure for client
     + Install and configure for server

## Motivation
As an IT administrator, I can easily set up machines for SSH based PowerShell remoting by running the Enable-SSHRemoting cmdlet.

## Specification
The proposal is to create a new 'Enable-SSHRemoting' cmdlet that configures basic SSH based PowerShell remoting on a machine.

```PowerShell
Enable-SSHRemoting -Scope <'Client','Server','All'> -Force
```

- Scope parameter
   + This parameter takes an enum type that specifies what is to be configured on the machine.  It can take the following arguments 'Client', 'Server', 'All'.
   + 'Client' provides a configuration where the machine can only initiate PowerShell remote connections but cannot accept connections or host remote sessions.
   + 'Server' provides a configuration where the machine can accept connections and host SSH remote sessions but cannot initiate remote connections.
   + 'All' provides both Client and Server remoting capabilities.
   + The default scope is 'All'

- Force parameter
   + This will overwrite any previous configuration settings (including SSH settings) and cause the SSHD service/daemon to restart as needed.

Examples:

Set up Client remoting
```PowerShell
PS> Enable-SSHRemoting -Scope Client -Force
Installing SSH package...

PS> Enter-PSSession -SSHTransport -HostName computerName -UserName userName
The authenticity of host 'computerName' can't be established.
ECDSA key fingerprint is SHA256:AeRUkQg4QJjZDNI07Do4BBpmutic4UR35bKw3FBV6a4.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'computerName' (ECDSA) to the list of known hosts.
[computerName]: PS> 
```

Set up Server remoting
```PowerShell
PS> Enable-SSHRemoting -Scope Server -Force
Installing SSH package...
Configuring SSHD service...
Starting SSHD service... 
```

Set up All remoting
```PowerShell
PS> Enable-SSHRemoting -Force
Installing SSH package...
Configuring SSHD service...
Starting SSHD service...

PS> Enter-PSSession -SSHTransport -HostName localhost -UserName userName
The authenticity of host 'localhost' can't be established.
ECDSA key fingerprint is SHA256:AeRUkQg4QJjZDNI07Do4BBpmutic4UR35bKw3FBV6a4.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'localhost' (ECDSA) to the list of known hosts.
[localhost]: PS> 
```

## Alternate Proposals and Considerations
One difficulty with this is installing SSH as needed.  Different platforms will have different installation packages.  This could be simplified by detecting if SSH is installed and if not then asking the user to first install it before continuing.  But I feel it is worthwhile to make installation automatic for supported platforms.  The cmdlet should be designed so that it is easy to add new supported platforms.  We should support the same platforms that PowerShell is supported on: Win32, Ubuntu 14/16, CentOS7, MacOS10.
