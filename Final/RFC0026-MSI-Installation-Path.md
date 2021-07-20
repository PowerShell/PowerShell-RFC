---
RFC: RFC
Author: Joey Aiello
Status: Final
Version: 0.2
Area: Installation
Comments Due: March 27, 2018
Plan to implement: Yes
---

# MSI Installation Path Location

Today, when using the MSI package on Windows,
PowerShell Core is installed to `$env:ProgramFiles\PowerShell\<version>\`.
This path was chosen early in the development of PowerShell Core in order to provide a deterministic,
file-based way to query the installed version of PowerShell.
In this RFC, we will explore the possibility of installing to different paths in order to support certain side-by-side scenarios.

## Motivation

Now that PowerShell Core 6.0 has GA'd, some users would like to run a preview version of PowerShell Core 6.1 side-by-side with PowerShell Core.
Today, that's only possible if one version is installed via the MSI and another is installed via the "portable" ZIP package.
Unfortunately, this makes the ZIP-installed copy of PowerShell difficult to install.

Also, in working to support Microsoft Update (MU) for servicing of PowerShell Core,
we require that the path of all PowerShell Core binaries does not change between servicing.
For example, if needed to patch a hypothetical version 6.0.3 to 6.0.4, the location of `pwsh.exe` and all associated binaries cannot change.

Lastly, we now have multiple mechanisms for querying the version directly from the PowerShell binary itself.
In order to support *nix-y semantics, we've added a `--version` parameter to `pwsh` that returns the version.
We also provide the version of PowerShell in the FileVersion object associated with `pwsh`.

## Specification

* There shall be two "flavors" of MSIs available:
    * Stable: a production-ready, supported build.
    * Preview: an unsupported build that may contain incomplete new features and high-impact bugs,
      and should therefore not be used in production.
* All stable MSIs of PowerShell Core 6.x shall be installed to `$env:ProgramFiles\PowerShell\6\`.
* All preview MSIs of PowerShell Core 6.x shall be installed to `$env:ProgramFiles\PowerShell\6-Preview`.
* When a stable MSI is run on a machine that already has a stable PowerShell Core 6.x installed,
  an in-place update of that stable build shall be performed.
* When a preview MSI is run on a machine that already has a preview PowerShell Core 6.x installed,
  an in-place update of that preview build shall be performed.
* No installation of a preview build shall interfere with a stable build, and vice versa
    * This includes an implicit requirement that `Install-PowerShellRemoting.ps1`
    and `Enable-PSRemoting` create a different endpoint name for the preview build.
    * The preview of PowerShell Core 6.x shall not be placed on the `PATH`.
      Instead, we should create a subdirectory in `$PSHOME` called `bin`,
      create a `pwsh-preview.cmd` file that launches the preview `pwsh.exe`,
      and add `$PSHOME\bin` to the `PATH`.

## Alternate Considerations and Proposals

### Include the minor version number in the folder name

This proposal would install/update all stable PowerShell Core 6.x builds to `$env:ProgramFiles\PowerShell\6.0`,
and all stable PowerShell Core 6.1 builds to `$env:ProgramFiles\PowerShell\6.1`.
Similarly, previews of PowerShell Core 6.1 would be installed to `$env:ProgramFiles\PowerShell\6.1-preview`,
and previews of PowerShell Core 6.2 would be installed to `$env:ProgramFiles\PowerShell\6.2-preview`.

This is undesirable given the Modern Lifecycle Policy that PowerShell Core has adopted.
Essentially, we've given users 6 months to migrate off of an older minor version of PowerShell Core once a new one is released.
Therefore, it makes more sense to *update* users from stable 6.0 to 6.1 than leave a version lying around that will soon become unsupported.
If users want to maintain unsupported versions of PowerShell Core side-by-side,
they should do a portable ZIP install.

## Open Questions

* Should an update install of stable PowerShell Core 6.0 to 6.1 also do a cleanup of any remaining preview 6.1 installs on the box?
    * I'd posit "no" because the stable vs. preview branches should be independent installations.
      E.g. I may want to immediately do an update of my preview 6.1 to 6.2,
      and maybe I have profiles or modules in that preview `$PSHOME` that I want specific to my preview install.
