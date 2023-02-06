---
RFC: RFC00XX
Author: Jason Helmick
Status: Draft
SupercededBy: N/A
Version: 1.0
Area: Core
Comments Due: 6/30/2022
Plan to implement: Yes
---

# Install-PowerShell7 Native Command

This RFC proposes adding a `install-powershell7` native command to Windows to perform a simplified
installation of the latest Long Term Service (LTS) release of PowerShell 7.

## Motivation

Many IT customers of PowerShell face a problem in which they don't control the landscape of software
pre-installed on the Windows Servers they manage. Explicitly installing PowerShell is a blocker for
some customers due to discoverability and some that have policies that don't allow for installing
software onto Windows after initial provisioning.

To compensate for these situations, customers will implement and use the "tools at hand" to perform
management tasks. This leaves **Windows PowerShell 5.1**, not **PowerShell 7**, as the only
automation solution available to many of them. While customers are missing out on useful new
features that they may wish to have, they are certainly not receiving the performance and security
improvements that come with each new release.

This specification is an effort to help IT admins that wish to use new and updated releases of
PowerShell 7. While this may not help customers that have post-deployment policy restrictions, it
will help the majority of customers discover and update to the latest version of PowerShell for
Windows.

```
As a user,
I can discover and install PowerShell 7 from Windows PowerShell 5.1 or `cmd.exe` on my Windows machine,
so that I can leverage new capabilities in PS7.
```

Additional scenarios are enabled when considering a native command versus a cmdlet. While Windows
users would find benefit and familiarity to a cmdlet, a native command opens up scenarios where
PowerShell is not pre-installed. Additionally, several installation issues due to open files are
avoided by using a native command and closing any open sessions of PowerShell.

## Shipping in Windows

There are several issues that prevent shipping PowerShell 7 in Windows:

- Size
- Support lifecycle differences

### Size constraints

 A complete installation of PowerShell 7 currently is around 200 MB (unpacked). This is due to
 including everything (modules and .NET assemblies) to provide parity with what Windows PowerShell
 5.1 with .NET Framework provides. There is currently no plan by Windows or the .NET team to ship
 newer .NET in Windows, so PS7 will need to continue to ship .NET with PS7.

### Support lifecycle

 Windows has a 5+5 year support lifecycle. .NET supports up to 3 years for Long Term Support (LTS)
 versions. So even if size was not an issue, we cannot check in a version of PS7 with a .NET that is
 not supported while Windows itself is supported.

### Solution

To solve these issues, the proposal is to ship a new command in Windows that would:

- Be locally discoverable
- Perform a simple download and install of the latest LTS release of PowerShell
- Include parameter options to select Stable, Preview, or LTS releases

## Goals/Non-Goals

Goals:

- By default, download and install the latest LTS release to the users CurrentUser Scope
  - CurrentUser - default installation scope location $env:LOCALAPPDATA\Microsoft\pwsh. If the path
    exists, remove and replace.
  - AllUsers - Administrative users may explicit select this scope. AllUsers requires administrative
    elevation and performs a silent install using the MSI defaults.
- Allow advanced users to specify channel Stable/Preview/LTS

Non-goals:

- Removal of PowerShell (Remove-PowerShell7)
- Updating of PowerShell (Update-PowerShell7)
- User selected versions prior to current (No version parameter)
- Disconnected scenarios - Users that are air-gapped may download the appropriate release from GitHub.
- Handling of additional MSI switch options - i.e. Enable PSRemoting
- Cross platform support
- Deployments at scale

## Specification

Command name: install-powershell7

The default behavior is to install the zip package silently to CurrentUser scope, with the latest LTS
release of PowerShell. If the user has administrative
privilege, they can choose to install the msi package silently to the scope of AllUsers.

- The stable package install location - $env:LOCALAPPDATA\Microsoft\PowerShell-Stable
  - If path exists, detect installed version and compare:
    - If versions equal: Provide an error "PowerShell 7 is already installed" and exit.
    - If versions not equal: Overwrite original installation.
- The preview package install location - $env:LOCALAPPDATA\Microsoft\PowerShell-Preview
  - If path exists, detect installed version and compare:
    - If versions equal: Provide an error "PowerShell 7 is already installed" and exit.
    - If versions not equal: Overwrite original installation.
- The installer should update the Users PATH for pwsh
- The installer should enable Microsoft Updates for scope AllUsers.
- The command should show a progress bar during the download and installation of PowerShell 7.
- The command should produce an error if the download location (US based url) fails.

The following parameters can be added by experienced PowerShell users to customize the installation.

- Parameter: Scope - overrides the default of CurrentUser and allows the user to specify the
  installation location AllUsers.
- Parameter: Channel - options Stable/Preview/LTS - allows the user to select if they want the
  current preview or LTS release.

## Syntax

The command `install-powershell7` is lower case and supports `-` or `--` parameters to be consistent
with other command-line tools.

```syntax
install-powershell7  [--channel <option>] [--scope <option>] [--force]
install-powershell7  [-c <option>] [-s <option>] [-f]
```

The `--channel <option>` must be one of the following:

- `--channel preview`
- `--channel stable`
- `--channel lts`

If you don't provide a channel option, the command defaults to `-channel lts`.

The `--scope <option>` must be one of the following:

- `--scope allusers`
- `--scope currentuser`

If you don't provide a scope option, the command defaults to `--scope currentuser`.

Scope selection has an impact on receiving auto-updates from Microsoft Update:

- `--scope allusers` is an MSI-based installation with Microsoft Update enabled.
- `--scope currentuser` is a ZIP-based installation. Users will need to manually
  update when a new version is released.

The `--force` switch clobbers a previous installation.

## Demo.txt

- Install the latest LTS version by default.

  ```powershell
  install-powershell7 
  ```

- Install the Stable release using the current user scope.

  ```powershell
  install-powershell7 --channel stable --scope currentuser
  ```

- Alternative (short) to install the stable release using the current user scope.

  ```powershell
  install-powershell7 -c stable -s currentuser
  ```
