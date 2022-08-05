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

# Install-PowerShell7 Cmdlet for Windows

This RFC proposes the addition of the `Install-PowerShell7` cmdlet and new `PowerShell7` module for
Windows to perform a simplified installation of the latest stable release of PowerShell 7. Options
through parameters are provided for advanced installation scenarios.

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
PowerShell on Windows and to easily install PowerShell from **Windows PowerShell 5.1** with a
cmdlet. While this may not help customers that have post-deployment policy restrictions, it will
help the majority of customers discover and update to the latest version of PowerShell for Windows.

```
As an admin,
I can use Windows PowerShell 5.1 to provision multiple hosts with PowerShell 7 over PSRemoting,
so that I can leverage new capabilities in PS7.
```

```
As a user,
I can discover and install PowerShell 7 from Windows PowerShell 5.1 on my Windows machine,
so that I can leverage new capabilities in PS7.
```

## Shipping in Windows

There are several issues that prevent shipping PowerShell 7 in Windows;

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

To solve these issues, the proposal is to ship a new cmdlet in Windows and the PowerShell Gallery
that would;

- Be discoverable using normal PowerShell conventions
- Perform a simple download and install of the latest stable release of PowerShell
- Provide advanced options with parameters;
  - Specific version
  - Stable, Preview or LTS
  - Scope - location of installation based on permissions

## Goals/Non-Goals

Goals:

- Be discoverable as other built-in cmdlets
- By default, download and install the latest stable release to the users CurrentUser Scope
  - CurrentUser - default installation scope
  - AllUsers - Administrative users may explicit select this scope
- Allow users to use remoting for install at scale with Invoke-Command
- Allow advanced users to specify the version of PowerShell
- Allow advanced users to specify scope (location of installation)
- Allow advanced users to specify channel Stable/Preview/LTS
- Users should receive a warning to continue the installation (ConfirmImpact=High)

Non-goals:

- Removal of PowerShell (Remove-PowerShell7)
- Updating of PowerShell (Update-PowerShell7)
- Disconnect scenarios - Users that are air-gapped may download the appropriate release from GitHub.
- Handling of additional MSI switch options - i.e. Enable PSRemoting

## Specification

Cmdlet name: Install-PowerShell7

Alias: None

The default behavior is to download and extract the .ZIP package, then install the latest stable
release of PowerShell into the user scope allowed by permissions. If with administrative privilege,
install to the scope of AllUsers. If user does not have administrative privilege, install to the
scope CurrentUser.

- The package install location - $env:LOCALAPPDATA\Microsoft\PowerShell
- The installer should update the Users PATH for pwsh
- The installer should enable Microsoft Updates for scope AllUsers. Due to permission requirements,
  the scope CurrentUser will require manual updating.
- Confirm impact should be set to **High** to force a warning and the ability to abort

The following parameters can be added by experienced PowerShell users to customize the installation.

- Parameter: Version - overrides the default and allows the user to specify the version of
  PowerShell that will be installed. PowerShell version 7.0 and 7.1 are out of support, so version
  numbers should begin with 7.2. This would install the latest 7.2.x.
- Parameter: Scope - overrides the default and allows the user to specify the installation location
  (scope) AllUsers or CurrentUser.
- Parameter: Channel - options Stable/Preview/LTS - allows the user to select if they want the
  current preview or Long Term Service (LTS) release. Preview should not be allowed if value of
  parameter **-Version** is Generally Available (GA).
- Parameter: Passthru - to provide informative output post installation

## Syntax and Parameter Sets

### ParameterSet Version (Default)

```
Install-PowerShell7 [[-Version] <string>] [-Scope {AllUsers | CurrentUsers}] [-Passthru] [-WhatIf] [-Confirm]  [<CommonParameters>]
```

### ParameterSet Channel

```
Install-PowerShell7 -Channel {Preview | LTS | Stable} [-Scope {AllUsers | CurrentUser}] [-Passthru] [-WhatIf] [-Confirm]
[<CommonParameters>]
```

The **Version** parameter has the following validation attribute: [ValidatePattern("^7\.\d+")]

The **Channel** parameter has the following validation attribute: [ValidateSet("Preview", "LTS", "Stable")]

The **Scope** parameter has the following validation attribute: [ValidateSet("AllUsers", "CurrentUser")]

### Data model

The `Install-PowerShell7` cmdlet should support the **Passthru** parameter to provide useful
information post installation. The data model for the output is below;

```output
PSTypename   = "PSInstallationInfo"
[Version]Version      = $version
[string]Path         = (Get-Command pwsh).source
[string]Channel      = if ($Channel) { $channel} else {"Stable" }
[string]Computername = $env:COMPUTERNAME
```

## Demo.txt

- Install the latest stable version by default.

  ```powershell
  install-powershell7 -Passthru
  ```

  ```
  Version Path                                   Channel Computername
  ------- ----                                   ------- ------------
  7.2.3   C:\Program Files\PowerShell\7\pwsh.exe Stable
  ```

- To install the latest stable release of PowerShell over PowerShell Remoting:

  ```powershell
  Invoke-Command -Computername RemoteComputer1 -ScriptBlock {Install-PowerShell7}
  ```

- Install the LTS release using the current user scope.

  ```powershell
  Install-PowerShell7 -Passthru -Channel LTS -Scope CurrentUser
  ```

  ```
  Version Path                                   Channel Computername
  ------- ----                                   ------- ------------
  7.2.0   C:\Users\jeff\PowerShell\7\pwsh.exe    LTS     RemoteComp
  ```

- To install the latest stable release of PowerShell:

  ```powershell
  Install-PowerShell7
  ```

- To install a specific version of a PowerShell release:

  ```powershell
  Install-PowerShell7 -Version 7.2
  ```

- To install a specific version of a PowerShell release to a specific scope:

  ```powershell
  Install-PowerShell7 -Version 7.2 -Scope Allusers
  ```

- To install the latest preview from a specific version:

  ```powershell
  Install-PowerShell7 -Version 7.3 -Channel 'Preview'
  ```
