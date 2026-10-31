---
RFC: 
Author: soroshsabz
Status: Draft
SupercededBy: 
Version: 1.0
Area: Remoting
Comments Due: 2026-12-01
Plan to implement: With Help
---

# Just Enough Administration (JEA) on Linux

This RFC proposes a cross-platform evolution of Just Enough Administration (JEA) that makes role-based, constrained PowerShell endpoints possible on Linux (and, through the same mechanisms, over SSH on all supported platforms).
It is motivated by the discussion in [PowerShell/PowerShell#2198](https://github.com/PowerShell/PowerShell/issues/2198) and builds on the session configuration foundation delivered by [RFC0064](https://github.com/PowerShell/PowerShell-RFC/blob/master/Final/RFC0064-PowerShell-LocalSession-Configuration.md), which deliberately left role-based JEA semantics out of scope.

The requirement is not to make Linux behave like Windows, nor to replace SSH, Bash, or `sudo`.
The requirement is to provide a supported, PowerShell-native authorization boundary for systems, in particular appliances, that expose PowerShell as their management CLI.

## Motivation

```
As the administrator of a Linux appliance whose management CLI is PowerShell,
I can expose a constrained, role-based PowerShell endpoint over SSH,
so that operators can perform only the management operations they are authorized for,
without being granted root, an unrestricted shell, or access to the appliance implementation.
```

### Why SSH and sudo are not equivalent

A Linux appliance may use SSH as its transport, but SSH by itself does not provide the administrative model an appliance CLI requires:

- SSH authenticates the user, but does not define which PowerShell commands that user may execute.
- A normal SSH session can provide Bash, arbitrary native commands, file access, tunneling, port forwarding, and access to other management interfaces.
- `sudo` delegates individual Linux executables or command lines, but it does not naturally express PowerShell command visibility, parameter constraints, validation, or PowerShell-specific roles.
- A PowerShell script that internally invokes `sudo` is not, by itself, a security boundary. The effective restriction must be enforced by the host and authorization layer, not by the script author alone.
- If a user can obtain an unrestricted Bash or PowerShell session, restrictions imposed only inside a script can generally be bypassed.

Linux has excellent building blocks for delegation, including `sudo`, `doas`, polkit, file permissions, systemd services, SELinux, AppArmor, containers, and SSH forced commands.
However, none of these provide a consistent, PowerShell-aware endpoint model.

This is the same core problem JEA addresses on Windows: reduce the exposed administrative surface, delegate only the required operations, provide role-based access, and preserve auditing, this time expressed in the same language as the management interface.

### Appliance scenario

Consider an appliance with the following operational roles:

| Role | Authorized operations |
| --- | --- |
| NOC operator | Restart a service, collect diagnostics, rotate a certificate, retrieve a support bundle |
| Field engineer | Modify a limited set of network settings |
| Security administrator | Manage users and audit configuration |
| Platform administrator | Full access |

The appliance should not need to expose a root shell, an unrestricted Bash session, a full PowerShell session, its internal implementation, internal files, credentials, or unrelated administrative functions.
It should expose a narrowly defined management surface such as:

```powershell
Get-ApplianceHealth
Restart-ApplianceService -Name "collector"
New-ApplianceSupportBundle
Set-ApplianceNetworkAddress -Address ...
```

The authorization boundary must be enforced by the endpoint, not by asking every client application or operator to invoke the correct script manually.

### The three problems JEA must solve

JEA addresses three distinct problems:

1. **Connection authorization**, managing who is allowed to connect to a given endpoint.
2. **Command surface restriction**, managing what cmdlets and native commands a connected user may run, with which parameters and arguments.
3. **Privilege delegation**, managing what privileges those commands need in order to function.

On Windows today, all three are answered by WinRM session configurations together with Windows identity and account machinery (security descriptors, virtual accounts, group managed service accounts).
On Linux, problem 2 is largely answerable today with a session configuration (`.pssc`) file and `pwsh -ConfigurationFile`; problems 1 and 3 have no supported answer.
This RFC is primarily about answering problems 1 and 3 cross-platform, while keeping problem 2 consistent across platforms.

### What exists today, and the gap

- `pwsh -ConfigurationFile` (RFC0064) applies a session configuration file to any PowerShell session, including SSH server mode. This enables constrained endpoints on any platform.
- However, per RFC0064, the Windows/WinRM-only elements, `RoleDefinitions`, `RequiredGroups`, `RunAsVirtualAccount`, `RunAsVirtualAccountGroups`, and `GroupManagedServiceAccount`, are not supported and produce an error.
  Without `RoleDefinitions` there is no way to map *who connects* to *what they may do*: a `.pssc` endpoint grants the same capability set to every user who can reach it.
- The SSH remoting cmdlets accept a `-Subsystem` parameter that selects a named endpoint defined in `sshd_config`, and RFC0064 shows how to wire such endpoints by hand.
  There is no cmdlet support on Linux for registering, enumerating, validating, or removing endpoints; administrators must edit `sshd_config` manually, which is error-prone and hard to audit.
- Windows JEA remains tied to WinRM session configuration registration (`Register-PSSessionConfiguration`), which does not exist on Linux.
- There is no documented, supported model for elevating individual operations on Linux (the equivalent of virtual accounts or group managed service accounts), nor a unified audit story: transcripts exist, structured syslog/journald/auditd integration does not.

## User Experience

The following is the target experience on a Linux appliance.
It is illustrative; the exact cmdlet surface is finalized during review of this RFC.

### 1. Author a role capability

`/etc/powershell/RoleCapabilities/Maintenance.psrc`:

```powershell
@{
    GUID        = '6b2d42a4-19c4-4f0e-9a2b-2e30f3f7cf01'
    Author      = 'contoso-appliance'
    Description = 'NOC maintenance role'

    ModulesToImport          = 'Appliance'
    VisibleCmdlets           = @(
        'Get-ApplianceHealth'
        'Restart-ApplianceService'
        'New-ApplianceSupportBundle'
        @{
            Name       = 'Set-ApplianceNetworkAddress'
            Parameters = @{ Name = 'Address'; ValidatePattern = '^10\.0\.\d{1,3}\.\d{1,3}$' }
        }
    )
    VisibleExternalCommands  = @()   # native commands are denied by default
}
```

### 2. Author a session configuration that maps Linux groups to roles

`/etc/powershell/Endpoints/Maintenance.pssc`:

```powershell
@{
    SchemaVersion       = '2.0.0.0'
    GUID                = '6b2d42a4-19c4-4f0e-9a2b-2e30f3f7cf02'
    Author              = 'contoso-appliance'
    Description         = 'NOC maintenance endpoint'

    SessionType         = 'RestrictedRemoteServer'
    LanguageMode        = 'NoLanguage'
    TranscriptDirectory = '/var/log/powershell/transcripts'
    RoleDefinitions     = @{
        'noc-operators' = @{ RoleCapabilityFiles = '/etc/powershell/RoleCapabilities/Maintenance.psrc' }
        'field-eng'     = @{ RoleCapabilityFiles = '/etc/powershell/RoleCapabilities/Network.psrc' }
    }
}
```

Principals are Linux users and groups resolved through NSS (local accounts, or domain accounts via SSSD/LDAP/Active Directory), the same resolution `sudo` and `sshd` use today.
A user who is not a member of a mapped group is denied the endpoint.

### 3. Register the endpoint

```powershell
# On the appliance, as an administrator
Register-PSSessionConfiguration -Name Maintenance -Path /etc/powershell/Endpoints/Maintenance.pssc -Transport SSH
```

```output
Name          : Maintenance
Transport     : SSH (subsystem: Maintenance)
Configuration : /etc/powershell/Endpoints/Maintenance.pssc
RunAs         : psrv-maintenance (unprivileged service account)
sshd_config   : Subsystem Maintenance /usr/bin/pwsh -SSHServerMode -ConfigurationFile /etc/powershell/Endpoints/Maintenance.pssc
```

Registration validates the configuration file, verifies that endpoint resources are owned by root and are not writable by session users, and updates `sshd_config`.

### 4. Connect and operate

```powershell
Enter-PSSession -HostName appliance01.contoso.com -UserName alice -Subsystem Maintenance
```

```output
[Maintenance]: PS> Get-ApplianceHealth

Service    Status   Uptime
-------    ------   ------
collector  Running  3.2d
ingest     Running  3.2d

[Maintenance]: PS> Restart-ApplianceService -Name collector
VERBOSE: Restarting collector as psrv-maintenance (sudo rule: appliance-restart-collector)
[Maintenance]: PS> bash
bash: The term 'bash' is not recognized as an allowed command in this endpoint.
[Maintenance]: PS> Remove-Item /etc/shadow
Remove-Item: The term 'Remove-Item' is not recognized as a name of a cmdlet, function, script file, or executable program.
```

Every command, allowed or denied, is recorded in the endpoint transcript and emitted as a structured event to syslog/journald containing the endpoint name, principal, resolved role, and command.

## Specification

### Goals

- Role-based constrained endpoints (`RoleDefinitions` plus role capabilities) usable with `pwsh -ConfigurationFile`, on Linux and over SSH on all supported platforms.
- Platform-neutral principal resolution for role mapping: Windows security identifiers; Linux and macOS users and groups via NSS.
- Endpoint lifecycle cmdlets (`Register-`, `Get-`, `Set-`, `Unregister-PSSessionConfiguration` or equivalent) that manage SSH subsystem endpoints safely.
- An explicit, auditable privilege delegation model for constrained endpoints on Linux that integrates with `sudo`, systemd, and polkit rather than replacing them.
- A default-deny native command policy with argument validation.
- Session and command auditing, transcripts plus structured syslog/journald events, at parity with Windows.
- A design Windows JEA can adopt over time ("JEA v2"): one format and one mental model, with per-operating-system enforcement details.

### Non-goals

- Porting WinRM to Linux.
- Replacing `sudo`, polkit, systemd, SELinux, AppArmor, or SSH. JEA layers on top of these primitives.
- Making a constrained endpoint a substitute for operating-system hardening. The endpoint boundary reduces and constrains the exposed surface; defense in depth is still required.
- Full parity with Windows virtual accounts or group managed service accounts, which rely on Windows-specific identity machinery. Linux equivalents are proposed instead.

### Design

#### 1. Role-based session configuration files (cross-platform)

- `RoleDefinitions` and `RequiredGroups` in `.pssc` files become supported; today they error when used with `-ConfigurationFile`.
- Principal names resolve at connection time through a platform abstraction:
  - Windows: account and group SIDs, as today.
  - Linux and macOS: user and group identities resolved through NSS (`getpwnam`/`getgrnam`), which uniformly covers local files and domain sources (SSSD, LDAP, Active Directory).
- A connecting principal maps to the union of all roles its user and group memberships select, exactly as Windows JEA merges role capabilities today.
- Principals that map to no role are denied the endpoint. This provides endpoint-level connection authorization on top of SSH authentication.
- `.psrc` files are already platform-neutral; `New-PSRoleCapabilityFile` and `Test-PSSessionConfigurationFile` become available on all platforms.

#### 2. Endpoint registration and management

- The session configuration cmdlets (`Register-`, `Get-`, `Set-`, `Unregister-PSSessionConfiguration`) are extended with an SSH transport so that on Linux (and Windows OpenSSH) they can:
  - create and update the corresponding `sshd_config` `Subsystem` entry, as shown in RFC0064;
  - validate the `.pssc` and `.psrc` files;
  - enforce ownership and permissions on endpoint resources (owned by root or an administrator, not writable by session users);
  - signal `sshd` to reload its configuration.
- The default unrestricted `powershell` subsystem is left untouched for general remoting; an appliance may remove it.
  The tooling warns when a constrained endpoint coexists with unrestricted shell access for the same users.

#### 3. Hardening the SSH boundary

The endpoint is only a boundary if users cannot simply route around it:

- Tooling checks and documented guidance for restricting SSH for endpoint users, for example `Match User` blocks with forced commands, `X11Forwarding no`, `AllowTcpForwarding no`, and appropriate `PermitTTY` settings, or per-key `command=` forced-command entries in `authorized_keys`.
- The endpoint session runs as a dedicated, unprivileged service account declared by the endpoint, never as the connecting user and never as root, unless an explicit elevation policy (next section) says otherwise.

#### 4. Privilege delegation model

Elevation must be explicit, declared, and auditable. For each privileged operation, the endpoint policy selects one of the following mechanisms:

| Mechanism | Example use |
| --- | --- |
| Dedicated unprivileged service account with narrowly scoped rights | Most read and diagnostic operations |
| `sudo` command allowlist (`NOPASSWD`, specific binaries with fixed arguments) for the service account | `appliance-restart-collector` |
| systemd unit or `systemctl` delegation via a polkit action | Service lifecycle |
| File capabilities (`setcap`) on a hardened helper binary | Binding privileged network ports |
| Privileged helper daemon reached over a root-owned Unix domain socket with per-operation ACLs | Complex appliances |

- The chosen mechanism is part of the endpoint definition and is shown by `Get-PSSessionConfiguration`, so customers can audit what is elevated and how.
- Role capabilities may declare the operations they require; the elevation mapping remains at the endpoint or appliance layer so vendors keep control of platform integration.
- PowerShell does not invent a new privilege mechanism; it integrates with the operating system's mechanisms. This mirrors what virtual accounts and group managed service accounts provide on Windows, and gives Windows JEA a path to the same explicitness in a future version.

#### 5. Command surface restriction and native command policy

- Everything not explicitly visible is denied. `SessionType RestrictedRemoteServer` and `LanguageMode NoLanguage` are the defaults for role-based endpoints.
- `VisibleExternalCommands` is the only way to run native commands, and entries may carry argument validation patterns.
- Deny-by-default applies to all shells and interpreters (`bash`, `sh`, `pwsh`, `python`, and similar). Allowing them is possible but produces a prominent warning, because it effectively dissolves the boundary.
- Native commands are executed directly, without an intermediate shell, so argument validation remains meaningful. Paths that reintroduce shell string interpolation must remain forbidden.

#### 6. Escape prevention (threat model)

Assets: the appliance operating system, its privileged operations, its implementation, and its secrets.
Attacker: an authenticated endpoint user, or anyone holding their credentials or keys, attempting to exceed delegated authority.

The constrained session must prevent the following escape vectors:

| Escape vector | Mitigation |
| --- | --- |
| Running disallowed cmdlets, functions, or providers | Session type and visible-command filtering at runspace construction |
| Language-based reflection or direct .NET access | `NoLanguage`/`ConstrainedLanguage` by default |
| Native command execution, including shells | Default-deny `VisibleExternalCommands` |
| Tampering with endpoint scripts, modules, or configuration | Root-owned, non-writable resources; restricted and validated module/script paths |
| Environment or profile injection | No profiles in server mode; scrubbed environment |
| Launching a nested unrestricted `pwsh` | `pwsh` treated as a native command and denied by default |
| Routing around the endpoint via a plain SSH shell | SSH hardening (section 3) and tooling warnings |
| Out-of-band access to appliance interfaces | Documented operating-system and network hardening checklist |

#### 7. Auditing

- Transcripts written to `TranscriptDirectory`, which is already cross-platform.
- Structured session and command lifecycle events emitted to syslog/journald on Linux and the Event Log on Windows, including endpoint name, principal, resolved roles, command, and outcome, so appliances can forward them to SIEM systems.
- Documented hooks for auditd correlation where customers require kernel-level auditing.

#### 8. Transports

- SSH subsystem endpoints: primary for Linux and macOS, available on Windows through Win32-OpenSSH.
- Local constrained execution through `pwsh -ConfigurationFile` (RFC0064): the same `.pssc` semantics, useful for appliance local consoles and scheduled tasks.
- Named pipe and custom transports may adopt the same model later.

### Open questions

1. Which Windows JEA scenarios are already adequately served by existing Linux tools, and should therefore be documentation rather than product?
2. What are the exact principal resolution semantics on Linux for nested or indirect group membership from NSS or SSSD sources?
3. Should `RequiredGroups` keep its Windows JEA v1 semantics or be simplified for JEA v2?
4. Should role-based `.pssc` endpoints over SSH also be honored on Windows, to reduce JEA's dependency on WinRM over time?
5. How restrictive can the native command argument validation grammar be while remaining usable?
6. Which elevation mechanisms must the first release document and validate (sudo, polkit, systemd, helper daemon) for the appliance scenario?
7. What guarantees can be provided that a user restricted to endpoints cannot reach the default `powershell` subsystem: tooling checks, `Match` guidance, or enforcement?
8. What should the auditing event schema look like, and should it align with an existing standard?
9. How should appliance vendors package and support endpoints and role capabilities without exposing implementation details (module layout, signing)?

### Implementation outline (design first)

Per the #2198 discussion, design precedes code:

1. **Investigation and proof of concept.** Validate NSS resolution at connection time, `sshd_config` automation, one `sudo` and one polkit elevation pattern, and audit event emission. Publish findings against this RFC.
2. **Engine.** Support `RoleDefinitions` and `RequiredGroups` with `-ConfigurationFile` in server mode, behind an experimental feature.
3. **Tooling.** Cross-platform session configuration cmdlets and hardening checks.
4. **Elevation and audit guidance.** Documented patterns and an appliance hardening checklist.

## Alternate Proposals and Considerations

### Status quo: SSH, sudo, and wrapper scripts

Vendors can combine SSH, `sudo`, wrapper scripts, and restricted shells today. Problems with this approach:

1. Each vendor creates a different authorization model.
2. PowerShell commands and Linux commands are governed by separate policy systems.
3. Command and parameter validation is duplicated in wrapper programs.
4. The supported management experience differs between Windows and Linux appliances.
5. Vendors may accidentally create escape paths through argument handling, environment variables, writable scripts, command substitution, or unrestricted native process execution.
6. The result is difficult to reason about and difficult for customers to audit.

### SSH forced commands (`authorized_keys command=`, `ForceCommand`)

Sufficient for single-purpose delegation (one key, one command) and useful as a complementary SSH-layer control. Does not provide multiple roles per endpoint, PowerShell-aware parameter validation, transcripts, or a manageable endpoint model. Investigated in the #2198 discussion.

### polkit, systemd, or doas delegation

Fine-grained and auditable, but not PowerShell-aware: cannot express cmdlet visibility or parameter constraints, and does not integrate with PowerShell remoting sessions.

### Containers as the isolation boundary

Strong isolation, but a different operational model (image lifecycle, no shared host state). Unsuitable as the primary mechanism for an appliance management CLI, though vendors may combine both.

### Documentation-only outcome

For basic delegation needs, existing Linux tooling plus documentation may suffice, and where that is true this RFC should say so explicitly. It does not suffice for appliances whose primary CLI is PowerShell: for them, the endpoint is part of the product's security boundary.

### Implement as an out-of-tree module

Not feasible. Role mapping at runspace construction, server-mode configuration, and endpoint lifecycle management touch engine internals (`InitialSessionState`, the remoting host). This was also concluded in the #2198 discussion.

### A separate specification repository

A JEA v2 definition could be developed in its own repository, as suggested in the #2198 discussion. This RFC instead proposes the PowerShell-RFC process as the vehicle, so the design stays tied to the engine working groups and their acceptance process. The Committee may of course redirect.

## Required Help

- **Working Group / PowerShell Committee**: review of the threat model and of the enforcement points, especially whether role mapping belongs in `InitialSessionState` at runspace construction or in the remoting host.
- **Linux security expertise**: NSS/SSSD group resolution semantics, systemd and polkit integration, SELinux/AppArmor interaction, auditd correlation.
- **PowerShell team and engine contributors**: server mode (`-SSHServerMode`) hardening review, making `RoleDefinitions` work with `-ConfigurationFile`, and `sshd_config` management ergonomics.
- **Appliance vendors and administrators**: validation of the scenario, roles, and packaging questions above, and feedback on the elevation patterns.

## References

- [PowerShell/PowerShell#2198, JEA on Linux](https://github.com/PowerShell/PowerShell/issues/2198)
- [RFC0064, PowerShell Local Session Configuration](https://github.com/PowerShell/PowerShell-RFC/blob/master/Final/RFC0064-PowerShell-LocalSession-Configuration.md)
- [RFC0010, SSH Remoting Cmdlets](https://github.com/PowerShell/PowerShell-RFC/blob/master/Final/RFC0010-SSH-Remoting-Cmdlets.md)
- [PowerShell/PowerShell#17367, JEA session configuration breaks after upgrading past 7.2.1](https://github.com/PowerShell/PowerShell/issues/17367)
- [JEA documentation](https://learn.microsoft.com/powershell/scripting/security/remoting/jea/overview)
