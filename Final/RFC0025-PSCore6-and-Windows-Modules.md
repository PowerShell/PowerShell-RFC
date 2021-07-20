---
RFC: RFC0025
Author: Steve Lee
Status: Final
SupercededBy: n/a
Version: 1.0
Area: Engine
Comments Due: 6/30
Plan to implement: Yes
---

# Enable Discovery and Importing of Compatible in-box Windows PowerShell modules

The biggest factor preventing existing Windows PowerShell users from moving to PowerShell Core 6 has been cmdlet coverage.
Specifically being able to use existing cmdlets that ship with Windows.
The PowerShell team has been working with Windows teams to port their modules so that they work in PowerShell Core 6.
This RFC addresses the experience for the user when using PowerShell Core 6.1 with next version of Windows 10 specifically
focused on the Windows PowerShell modules that are shipped within Windows.

## Motivation

As an ITPro,
I can use existing Windows PowerShell modules in PowerShell Core 6,
so that I can deploy PowerShell Core 6 in my enterprise.

## Specification

The module manifest (since Windows PowerShell 5) may contain a `CompatiblePSEditions` property to indicate if it is compatible with
Windows PowerShell (value of `Desktop` even on Windows Server) or PowerShell Core (value of `Core`).
Currently, this property is only documentation and not observed by PowerShell.
The proposal is to modify PowerShell Core 6.1 to respect this property for in-box Windows PowerShell modules.

### Windows PowerShell 5.1 PSModulePath

Windows PowerShell, by default, searches for modules from these paths in this order:

  1. User profile: $env:UserProfile\Documents\WindowsPowerShell\Modules
  2. Program files: $env:ProgramFiles\WindowsPowerShell\Modules
  3. System32: $env:Windir\System32\WindowsPowerShell\v1.0\Modules

### Proposed PowerShell Core PSModulePath Change

PowerShell Core, by default, searches for modules form these paths in this order:

  1. User profile: $env:UserProfile\Documents\PowerShell\Modules
  2. Program files: $env:ProgramFiles\PowerShell\Modules
  3. PSHOME: value of $PSHOME

The proposed change is to add *only* the `System32` path from Windows PowerShell 5.1 to `PSModulePath` on PowerShell Core 6
as the last path in `PSModulePath`.

This means that modules installed using Windows PowerShell with `-Scope CurrentUser` will not be available from PowerShell
Core 6.
To use those modules, the user needs to use `Install-Module -Scope CurrentUser` from within PowerShell Core 6.
Duplicate modules are something that can be addressed in PowerShellGet and outside the scope of this RFC.

Modules in `Program Files` that may be compatible with PowerShell Core 6 is addressed below.

Modules _not under_ Windows System32 path, for example in the PowerShell Core `PSModulePath`, are implicitly deemed compatible even
if the manifest does not declare compatibility.
A user who explicitly uses `Install-Module` within PowerShell Core 6 expect those modules to be available to PowerShell Core 6.

### CompatiblePSEditions for Windows System32

When we add the `System32` path to PowerShell Core's `PSModulePath`,
we will use additional logic to validate that the module is compatible with PowerShell Core.

If the module manifest contains `CompatiblePSEditions` with value `Core`, then that module is treated as compatible.
If the module manifest contains `CompatiblePSEditions` with only the value of `Desktop`, then that module is treated as incompatible.
If the module manifest does not contain `CompatiblePSEditions`, then that module is treated the same as `Desktop` which is incompatible.

By default, `Get-Module -ListAvailable` will only show modules found in `PSModulePath` that are declared as compatible.
Similarly, module auto-discovery and auto-loading will only find modules that are declared as compatible.

The `ModuleInfo` table format will add a column for `PSEdition` (shortened from `CompatiblePSEditions` between `Name` and `ExportedCommands`).

### Using Modules from System32 not declared as Compatible

`Get-Module -ListAvailable` will support `-SkipEditionCheck` switch to list all modules in `PSModulePath`.

If `Import-Module` is used with a module that is not declared in the manifest as compatible, it will throw a terminating error:

> The module '{0}' could not be imported as it is not declared as compatible.  Use `-SkipEditionCheck` switch to override this check.

`Import-Module` will support `-SkipEditionCheck` switch to try and import the module regardless of `CompatiblePSEditions`.
Since module auto-discovery does not expose cmdlets from modules not declared as compatible, explicit `Import-Module` with
`-SkipEditionCheck` is required to use potentially incompatible modules.

## Alternate Proposals and Considerations

Open issue: Should `CompatiblePSEditions` column only be added in `ModuleInfo` table format if used with `-SkipEditionCheck`?
Users who want to search the entirety of the Windows PowerShell `PSModulePath` should use `Add-WindowsPSModulePath` cmdlet
from the `WindowsCompatibility` module.

---------------
## PowerShell Committee Decision

### Voting Results

Joey Aiello: Accept

Bruce Payette: Accept

Steve Lee: Accept

Hemant Mahawar: Accept

Dongbo Wang: Accept

Kenneth Hansen: Accept

Jim Truher: Absent

### Majority Decision

Commmittee agrees to accept this RFC as written.

### Minority Decision

N/A
