---
RFC: RFC00NN
Author: Joey Aiello
Status: Draft-Accepted
SupercededBy: N/A
Version: 0.1
Area: External Module
Comments Due: 7/15/2017
---

# PowerShellCore Installer/Utility Module

A module called `PowerShellCore` that ships on the Gallery and eases installation and configuration of PowerShell Core on Windows.

## Motivation

Installation and configuration of PowerShell Core on Windows is tricky due to the wide range of Windows versions that we support (7-10 and Server 2008R2-2016) and because of the usage of some "system" concepts shared by Windows PowerShell like the WinRM remoting plugin.

## Goals

* I can install PowerShell Core across all supported versions of Windows and Windows PowerShell *at scale*.
* I can install PowerShell Core on certain versions of Windows and Windows PowerShell without remembering highly arcane commands or manually downloading a package from a well-known location.
* I can uninstall PowerShell Core using Windows PowerShell
* I can register PowerShell Core as a WinRM remoting endpoint across all supported versions of Windows and Windows PowerShell *at scale*.

## Specification

Ship a module on Gallery called `PowerShellCore` which contains the following cmdlets:

* Install-PowerShellCore
  * Download and install the latest PowerShell Core MSI and install it using the MSI PackageManagement provider
* Update-PowerShellCore
  * If PowerShell Core is already installed, this will grab the latest PowerShell Core MSI and do an in-place upgrade of the MSI-installed instance on the machine.
* Uninstall-PowerShellCore
  * Uninstall the PowerShell Core MSI using the MSI PackageManagement provider.
* Register-PowerShellCoreEndpoint
  * Call the Install-PowerShellRemoting.ps1 script in PowerShell Core's `$PSHome`.

These cmdlets allow me to do things like:

```powershell
Invoke-Command -ComputerName $TargetNode -Credential DOMAIN\BestAdmin -ScriptBlock {Install-Module PowerShellCore -Force; Install-PowerShellCore; Register-PowerShellCoreEndpoint}
```

`PowerShellCore`should start as a 0.1 release so that we can make breaking changes as we gather feedback in PowerShell Core 6.0 beta.

Right now, I think it's perfectly okay to ship all of these with nothing but common parameters.
Based on feedback to this RFC, we should prioritize a set of parameters that might be useful.
(For example, we may eventually add a `-Source` to `Install/Upgrade-PowerShellCore` that points to `GitHub` and some other, more official, source.)

## Alternate Proposals and Considerations

### Nano Server

Given that Nano Server currently has Docker containers with the latest version of PowerShell Core, Nano Server is out of scope for this RFC.
However, it's perfectly reasonable that a future version of `PowerShellCore` could extend itself to support Nano Server via the same set of cmdlets.

### Servicing

MSIs have well-known patterns for updates and servicing (e.g. WSUS, SCCM, etc).
While it's curently unclear *where* we should publish these MSIs to best enable these servicing patterns,
delivering an out-of-band application via an MSI is better than other alternatives for notifying administrators and user of security fixes and critical updates.

### Tracking "Portable" Copies of PowerShell

PowerShell Core can be installed in a "portable" manner by simply extracting a ZIP running `powershell.exe`.
(Similarly, "portable" PowerShell Core has its own `Install-PowerShellRemoting.ps1` that can be run as Administrator.)
It is currently out-of-scope to "register" or install "portable" copies of PowerShell Core using the `PowerShellCore` module.
(If you do want to build some automation around non-MSI installs, you might want to look into the [PSReleaseTools](https://www.powershellgallery.com/packages/PSReleaseTools) by @jdhitsolutions).

### Appx Package

Appx is a package format introduced in Windows 8 to support installation of Universal Windows Platform (UWP) applications both inside and outside of the Windows Store.
While we may still build an Appx package to release on the Store in the future,
this particular package will not be able to perform administrative actions from within the Store sandbox
(and will therefore only be useful in limited scenarios, like as a remoting client to manage other machines.)

Furthermore, this will never be supported on Windows 7 or Server 2008R2, and so it's a lower priority.