---
RFC: 
Author: Peppe Kerstens
Status: Draft
Version: 1.0
Area: Microsoft.PowerShell.Commands.Management
Comments Due: 2026-06-18
Plan to implement: Yes
---

# Linux *-Service Cmdlets via D-Bus

This RFC proposes implementing the `*-Service` cmdlets for Linux (`Get-Service`, `Start-Service`, `Stop-Service`, `Restart-Service`, `Set-Service`, `New-Service`, `Remove-Service`) using the systemd D-Bus API (`org.freedesktop.systemd1`) instead of shelling out to `systemctl`.

## Motivation

    As a Linux PowerShell user,
    I can manage systemd services using the same cmdlets as on Windows,
    so that I can write cross-platform scripts without conditional logic or subprocess calls.

Currently, `*-Service` cmdlets are Windows-only. Linux users must use `systemctl` via `Invoke-Expression` or `Start-Process`, which breaks pipeline input, object output, and `ShouldProcess` safety.

## User Experience

The cmdlets behave identically to their Windows counterparts, returning `LinuxServiceController` objects that mirror `System.ServiceProcess.ServiceController` properties.

```powershell
# List all services
Get-Service

# Filter by name (wildcards supported)
Get-Service -Name 'ssh*'

# Start a service
Start-Service -Name 'sshd' -PassThru

# Change startup type
Set-Service -Name 'sshd' -StartupType Disabled

# Create a new service
New-Service -Name 'myapp' -BinaryPathName '/usr/local/bin/myapp' -Description 'My Application'

# Remove a service
Remove-Service -Name 'myapp'
```

```output
Status   Name               DisplayName
------   ----               -----------
Running  sshd               OpenSSH Daemon
Stopped  myapp              My Application
```

## Specification

### Implementation Details

1.  **D-Bus Integration**: All interactions use `Tmds.DBus.Protocol` to communicate with `org.freedesktop.systemd1`. No `systemctl` subprocess is spawned.
2.  **Output Type**: `LinuxServiceController` (inherits `System.ComponentModel.Component`), matching `ServiceController` on Windows.
3.  **Properties**: `Name`, `DisplayName`, `Status`, `StartType`, `ActiveState`, `SubState`, `ServiceName`.
4.  **Elevation**: Write cmdlets (`Start`, `Stop`, `Set`, `New`, `Remove`) rely on reactive error translation. D-Bus polkit errors (`InteractiveAuthorizationRequired`) are caught and translated to `"CmdletName requires root privileges. Use 'sudo pwsh'."`
5.  **ShouldProcess**: All write cmdlets support `-WhatIf` and `-Confirm`.
6.  **Pipeline Input**: All cmdlets accept pipeline input by value or property name.
7.  **Wildcard Support**: `Get-Service -Name` supports wildcards (`*`, `?`).
8.  **Template Units**: `Get-Service` filters out template units (e.g., `systemd-journald@.service`) that cannot be started directly.

### Cmdlet Mapping

| Cmdlet | D-Bus Method | Notes |
|---|---|---|
| `Get-Service` | `ListUnits`, `ListUnitFiles` | Merges active units with file states |
| `Start-Service` | `StartUnit(name, "replace")` | |
| `Stop-Service` | `StopUnit(name, "replace")` | |
| `Restart-Service` | `RestartUnit(name, "replace")` | |
| `Set-Service` | `EnableUnitFiles` / `DisableUnitFiles` | For `-StartupType` |
| `New-Service` | File write + `Reload` + `EnableUnitFiles` | Creates `.service` file in `/etc/systemd/system/` |
| `Remove-Service` | `StopUnit` + `DisableUnitFiles` + File delete + `Reload` | |

### Platform Branching

The implementation is guarded by `#if UNIX`. On Windows, the existing `ServiceController` implementation is used. On Linux, the D-Bus implementation is active.

### Error Handling

-   **DBusExceptionBase**: Translated to `ErrorRecord` with category `ResourceUnavailable`.
-   **Polkit Errors**: Translated to `InvalidOperationException` with message `"CmdletName requires root privileges. Use 'sudo pwsh'."`
-   **Unit Not Found**: `Get-Service` returns no output (matches Windows behavior).

## Alternate Proposals and Considerations

### Subprocess (`systemctl`)
Using `systemctl` via `ProcessStartInfo` is simpler but introduces:
-   Parsing overhead (text output must be parsed).
-   Localization issues (output varies by locale).
-   Performance penalty (process spawn per call).
-   Deadlock risk (stdout/stderr buffers).

D-Bus is the native API for systemd. `systemctl` is a CLI wrapper around the same D-Bus calls. Using D-Bus directly is the Linux equivalent of using Win32 APIs on Windows.

### Third-Party Modules
Modules like `PSSystemd` exist but are not cross-platform and do not match the Windows cmdlet surface. This proposal aligns with the goal of Windows-compatible cmdlets on Linux.

## References

-   [systemd D-Bus API](https://www.freedesktop.org/wiki/Software/systemd/dbus/)
-   [Tmds.DBus.Protocol](https://github.com/tmds/Tmds.DBus)
-   [PowerShell/PowerShell Fork](https://github.com/peppekerstens/PowerShell/tree/feature/service-unix-systemctl)
-   [Services.Linux.Native](https://github.com/peppekerstens/Services.Linux.Native)
