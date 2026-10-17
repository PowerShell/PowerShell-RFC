---
RFC: RFCnnnn
Author: Kirk Munro
Status: Draft
SupercededBy: 
Version: 0.1
Area: Engine
Comments Due: September 15, 2019
Plan to implement: Yes
---

# Execution preferences must persist beyond module or script scope

PowerShell has a long-standing issue where execution preferences such as those
defined by the `-ErrorAction`, `-WarningAction`, `-InformationAction`,
`-Debug`, `-Verbose`, `-WhatIf` and `-Confirm` common parameters, or those
defined by any of the `$*Preference` variables, do not persist from one module
to another module or script, nor do they persist from a script to another
module or script. This is a result of how modules and scripts are scoped within
PowerShell. It impacts modules written by Microsoft as well as scripts and
modules written by the community, and it is often identified as a bug when it
shows up in various places. You can see some of the discussion around this in
[PowerShell/PowerShell#4568](https://github.com/PowerShell/PowerShell/issues/4568).

Regardless of whether you are authoring a script, an advanced function, or a
cmdlet, you should be able to do so knowing that all execution preferences will
be carried through the entire invocation of a command, end to end, regardless
of the implementation details of that command.

Cmdlets already work this way today.

You can invoke cmdlets within a script, or within an advanced function defined
in a module, and the execution preferences used during the invocation of that
script or advanced function will be respected by the cmdlets that are invoked.
This RFC is about making it possible for scripts or advanced functions to work
that way as well.

It is important to note that the only way to implement this feature such that
it is not a breaking change is by making it
[optional](https://github.com/PowerShell/PowerShell-RFC/pull/220); however,
even with it optional, it can be enabled by default for new modules to correct
this problem going forward (using `$PSDefaultParameterValues`), and since it is
optional, existing scripts and modules could be updated to support it as well,
when they are ready to take on the responsibility of testing that out. That
would allow this feature to be adopted more easily for new modules, while
existing modules could be updated over time.

## Motivation

As a scripter or a command author,<br/>
I should never have to care what type of command (cmdlet vs advanced function)
I am invoking,<br/>
so that I can can focus on my script without having to worry about the
implementation details of commands I use.

As a command author,<br/>
I should be able to change a published command from an advanced function to a
cmdlet or vice versa,<br/>
so that as long as I keep the same functionality in the command, scripts and
modules where that command is in use should behave the same way.

## User experience

```powershell
# First, create a folder for a new module
$moduleName = 'RFCPropagateExecPrefDemo'
$modulePath = Join-Path `
    -Path $([Environment]::GetFolderPath('MyDocuments')) `
    -ChildPath PowerShell/Modules/${moduleName}
New-Item -Path $modulePath -ItemType Directory -Force > $null

# Then, create the manifest, with the PersistCommandExecutionPreferences optional
# enabled by default, to correct the behaviour in a non-breaking way; downlevel
# versions of PowerShell would ignore the optional feature flags)
$nmmParameters = @{
    Path = "${modulePath}/${moduleName}.psd1"
    RootModule = "./${moduleName}.psm1"
    FunctionsToExport = @('Test-1')
    OptionalFeatures = @('PersistCommandExecutionPreferences')
    PassThru = $true
}
New-ModuleManifest @nmmParameters | Get-Content

# Output:
#
# @{
#
# RootModule = './RFCPropagateExecPrefDemo.psm1'
#
# # Private data to pass to the module specified in RootModule/ModuleToProcess.
# # This may also contain a PSData hashtable with additional module metadata
# # used by PowerShell.
# PrivateData = @{
#
#     <snip>
#
#     PSData = @{
#
#         # Optional features enabled in this module.
#         OptionalFeatures = @(
#             'PersistCommandExecutionPreferences'
#         )
#
#         <snip>
#
#     } # End of PSData hashtable
#
#     <snip>
#
# } # End of PrivateData hashtable
#
# }

# Then, create the script module file, along with a second module in memory
# that it invokes, and import both modules
$scriptModulePath = Join-Path -Path $modulePath -ChildPath ${moduleName}.psm1
New-Item -Path $scriptModulePath -ItemType File | Set-Content -Encoding UTF8 -Value @'
    function Test-1 {
        [cmdletBinding()]
        param()
        Test-2
    }
'@
Import-Module $moduleName
New-Module Name test2 {
    function Test-2 {
        [cmdletBinding()]
        param()
        Write-Verbose 'Verbose output'
    }
} | Import-Module

# When invoking the Test-2 command with -Verbose, it shows verbose output,
# as expected.
Test-2 -Verbose

# Output:
#
# VERBOSE: Verbose output

# Thanks to this feature, when invoking the Test-1 command with -Verbose, it
# also shows verbose output. In PowerShell 6.2 and earlier, no verbose output
# would appear as a result of this command due to the issue preventing
# execution preferences from propagating beyond module/script scope.
Test-1 -Verbose

# Output:
#
# VERBOSE: Verbose output
```

## Specification

To resolve this problem, a new optional feature called
`PersistCommandExecutionPreferences` would be defined in PowerShell. When this
optional feature is enabled, it would change how common parameters are passed
from one advanced function to another advanced function or a script that uses
`CmdletBinding`.

Today if you invoke a script or advanced function with `-ErrorAction`,
`-WarningAction`, `-InformationAction`, `-Debug`, `-Verbose`, `-WhatIf`, or
`-Confirm`, the corresponding `$*Preference` variable will be set within the
scope of that command. That behaviour will remain the same; however when the
optional feature is enabled, in addition to that behaviour, the names and
values you supplied to those common parameters stored in a new
`ExecutionPreferences` dictionary property of the `$PSCmdlet` instance. Once
`$PSCmdlet.ExecutionPreferences` is set, any common parameters that are stored
in `$PSCmdlet.ExecutionPreferences` that are not explicitly used in the
invocation of another command within that command scope will be automatically
passed through if the command being invoked supports common parameters.

It is important to note that parameter/value pairs in
`$PSCmdlet.ExecutionPreferences`, which represent command execution
preferences, would take priority and be applied to a command invocation before
values in `$PSDefaultParameterValues`, which represents user/module author
parameter preferences (i.e. if both dictionaries have a value to be applied to
a common parameter, only the value in `$PSCmdlet.ExecutionPreferences` would be
applied).

As per the optional feature specification, the optional feature can be enabled
in a module manifest (see example above), or a script file via `#requires`. For
more details on how that works, see #220.

## Alternate proposals and considerations

### Rip off the bandaid

[Some members of the community](https://github.com/PowerShell/PowerShell/issues/6745#issuecomment-499740080)
feel it would better to break compatibility here. On the plus side, not having
to deal with this an an optional parameter would be ideal; however, to increase
adoption of PowerShell 7, it would be better to make the transition from
PowerShell 5.1 into 7 easier by having as few breaking changes as possible.

One way to achieve this while supporting users who don't want the breaking
change would be to inverse the optional feature, where the breaking change is
in place and users opt out of the breaking change instead of opting into it.
Another way would be to change the optional feature design such that users can
turn them off in scripts/modules if those scripts/modules are not ready to use
the breaking change. See the alternate proposals and considerations section of
#220 for more information.

### Support `-DebugAction`, `-VerboseAction`, and `-ProgressAction` if those common parameters are added

[PowerShell/PowerShell#10238](https://github.com/PowerShell/PowerShell/pull/10238)
will add `-DebugAction`, `-VerboseAction`, and `-ProgressAction` common
parameters to PowerShell if it is merged. These common parameters would also
need to propagate properly with `-ErrorAction`, `-WarningAction`, and
`-InformationAction`.
