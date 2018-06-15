---
RFC: RFCNNNN
Author: Steve Lee
Status: Draft
SupercededBy: n/a
Version: 0.1
Area: Engine
Comments Due: 6/30
Plan to implement: Yes
---

# Enable Discovery and Importing of Compatible in-box Windows PowerShell modules

The biggest factor preventing existing Windows PowerShell users from moving to PowerShell Core 6 has been cmdlet coverage.
Specifically being able to use existing cmdlets that ship with Windows.
The PowerShell team has been working with Windows teams to port their modules so that they work in PowerShell Core 6.
This RFC addresses the experience for the user when using PowerShell Core 6.1 with next version of Windows 10.

## Motivation

As an ITPro,
I can use existing Windows PowerShell modules in PowerShell Core 6,
so that I can deploy PowerShell Core 6 in my enterprise.

## Specification

The module manifest (since Windows PowerShell 5) may contain a `CompatiblePSEditions` property to indicate if it is compatible with
Windows PowerShell (value of `Desktop` even on Windows Server) or PowerShell Core (value of `Core`).
Currently, this property is only documentation and not observed by PowerShell.
The proposal is to modify PowerShell Core 6.1 to respect this property for in-box Windows PowerShell modules.

### Windows PSModulePath

Windows PowerShell, by default, has module paths in:

  1. User profile: $env:UserProfile\Documents\WindowsPowerShell\Modules
  2. Program files: $env:ProgramFiles\WindowsPowerShell\Modules
  3. System32: $env:Windir\System32\WindowsPowerShell\v1.0\Modules

The proposed change is to add *only* the `System32` path to `PSModulePath` on PowerShell Core 6.

Modules in the user profile should be re-installed under PowerShell Core 6.
Duplicate modules are something that can be addressed in PowerShellGet and outside the scope of this RFC.
Modules in `Program Files` that may be compatible with PowerShell Core 6 is addressed below.

Users who want to search the entirety of the Windows PowerShell `PSModulePath` should use `Add-WindowsPSModulePath` cmdlet
from the `WindowsCompatibility` module.

### PowerShell Core PSModulePath

Modules in the PowerShell Core `PSModulePath` are implicitly deemed compatible even if the manifest does not
declare compatibility.
A user who explicitly uses `Install-Module` within PowerShell Core 6 expect those modules to be available to PowerShell Core 6.

### CompatiblePSEditions for Windows System32

When we add the `System32` path to PowerShell Core's `PSModulePath`,
we will use additional logic to validate that the module is compatible with PowerShell Core.

If the module manifest contains `CompatiblePSEditions` with value `Core`, then that module is treated as compatible.
If the module manifest contains `CompatiblePSEditions` with only the value of `Desktop`, then that module is treated as incompatible.
If the module manifest does not contain `CompatiblePSEditions`, then that module is treated the same as `Desktop` which is incompatible.

By default, `Get-Module -ListAvailable` will only show modules found in `PSModulePath` that are declared as compatible.
Similarly, module auto-discovery and auto-loading will only find modules that are declared as compatible.

The `ModuleInfo` table format will add a column for `CompatiblePSEditions` (between `Name` and `ExportedCommands`).

### Using Modules not declared as Compatible

`Get-Module -ListAvailable` will support `-SkipCompatibilityCheck` switch to list all modules in `PSModulePath`.

If `Import-Module` is used with a module that is not declared in the manifest as compatible, it will throw a terminating error:

> The module '{0}' could not be imported as it is not declared as compatible.  Use `-SkipCompatibilityCheck` switch to override this check.

`Import-Module` will support `-SkipCompatibilityCheck` switch to try and import the module regardless of `CompatiblePSEditions`.
Since module auto-discovery does not expose cmdlets from modules not declared as compatible, explicit `Import-Module` with
`-SkipCompatibilityCheck` is required to use potentially incompatible modules.

## Alternate Proposals and Considerations

Open issue: Should `CompatiblePSEditions` column only be added in `ModuleInfo` table format if used with `-SkipCompatibilityCheck`?
