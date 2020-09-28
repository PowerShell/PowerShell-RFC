---
RFC: '0058'
Author: Kirk Munro
Status: Rejected
SupercededBy: 
Version: 0.1
Area: Engine
Comments Due: September 15, 2019
Plan to implement: Yes
---

# Optional features in PowerShell

How do you resolve longstanding issues in PowerShell that either cause bugs to
show up in unexpected places or that make PowerShell much more difficult to
program with than it should be, when resolving those issues would introduce
breaking changes that would impact existing automation solutions?

You could use Semantic Versioning, where breaking changes can be added as
long as the major version is incremented; however, in an interpreted language
with a community of users that expects backwards compatibility, those types of
changes are often unappreciated.

Another approach is to take an opt-in approach to breaking changes, where users
can optionally accept breaking changes for their automated solutions.

This RFC is about enabling those scenarios where breaking changes are deemed
valuable enough to offer as an option in PowerShell.

NOTE: It may appear up front that the Experimental Features functionality in
PowerShell solves this need, but that is not the case. Experimental Features
are features under development while feedback on the design and usage data is
gathered. While they may include breaking changes, they are not intended as a
way to enable breaking changes.

As an example of a features that would most likely be optional if implemented,
consider how command execution preferences could be propagated beyond script or
script module scope, or how to make terminating errors properly terminate
without requiring try/catch scaffolding to get the correct behavior. Both of
those issues have separate RFCs that include implementation as an optional
feature.

## Motivation

As a script, function, or module author,<br/>
I can enable optional features for specific users or in specific scopes,<br/>
so that I can leverage new functionality that may break existing scripts without risk.

## User experience

```powershell
# Create a module manifest, specifically enabling or disabling one or more optional
# features in the manifest. An optional feature cannot be in both -OptionalFeatures
# and -DisabledOptionalFeatures.
$manifest = New-ModuleManifest `
    -Path ./test.psd1 `
    -OptionalFeatures OptionalFeature1 `
    -DisabledOptionalFeatures OptionalFeature2 `
    -PassThru
$manifest | Get-Content

# Output:
#
# @{
#
# <snip>
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
#             'OptionalFeature1'
#         )
#
#         # Optional features disabled in this module.
#         DisabledOptionalFeatures = @(
#             'OptionalFeature2'
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

# Create a script file, enabling or disabling one or more optional features in the file
@'
#requires -OptionalFeature OptionalFeature1
#requires -OptionalFeature OptionalFeature2 -Disabled

<snip>
'@ | Out-File -FilePath ./test.ps1

# Get a list of optional features, whether or not they are automatically enabled,
# their source, and their descriptions
Get-OptionalFeature

# Output:
#
# Name                 AutoEnable Source              Description
# ----                 ---------- ------              -----------
# OptionalFeature1    CurrentUser PSEngine            Description of optional feature 1
# OptionalFeature2       AllUsers PSEngine            Description of optional feature 2
# OptionalFeature3             No PSEngine            Description of optional feature 3
# OptionalFeature4             No PSEngine            Description of optional feature 4

# Enable an optional feature in current PowerShell session.
Enable-OptionalFeature -Name OptionalFeature1

# Output:
# None

# Enable an optional feature in the current PowerShell session
# and future PowerShell sessions for all users.
Enable-OptionalFeature -Name OptionalFeature1 -AutoEnable AllUsers

# Output:
# None

# Enable an optional feature in the current PowerShell session
# and future PowerShell sessions for the current user.
Enable-OptionalFeature -Name OptionalFeature1 -AutoEnable CurrentUser

# Output:
# None

# Disable an optional feature in the current PowerShell session.
Disable-OptionalFeature -Name OptionalFeature2

# Output:
# None

# Enable and/or disable an optional feature the duration of the
# script block being invoked.
Use-OptionalFeature -Enable OptionalFeature1 -Disable OptionalFeature2 -ScriptBlock {
    # Do things using OptionalFeature1 here
    # OptionalFeature2 cannot be used here
}
# If OptionalFeature1 was not enabled before this invocation, it
# is still no longer enabled here. If OptionalFeature2 was enabled
# before this invocation, it is still enabled here. All to say,
# their state before the call is preserved.

# Returns true if the optional feature is enabled in the scope in
# which the command was invoked; false otherwise.
Test-OptionalFeature -Name OptionalFeature1
```

## Specification

Unlike experimental features, which can only be enabled or disabled in
PowerShell sessions created after enabling or disabling them, optional features
can be:

- enabled or disabled in the current PowerShell session;
- enabled in future PowerShell sessions;
- enabled or disabled in a specific module or script scope;

This allows certain functionality to be "lit up" in packaged modules or scripts.

Below you will find details describing how this functionality will be implemented.

### System and User powershell.config.json configuration files

Enabling optional features automatically in future PowerShell sessions requires
creating or updating one of two `powershell.config.json` configuration files
that are read on startup of a new PowerShell session:

* one in `$PSHOME`, which applies to all user sessions
* one in `$HOME\Documents\PowerShell\powershell.config.json` on Windows or
`$HOME/.config/powershell/powershell.config.json` on Linux and macOS, which
applies only to current user sessions.

This RFC will enable optional feature defaults to be read from these
configuration files, with current user configuration taking precedence over
system (all users) configuration. System config is not policy so this should be
acceptable and expected.

### Add parameters in New-ModuleManifest

`[-OptionalFeatures <string[]>]`

This parameter would enable specific optional features in the new module
manifest that is generated.

The values provided to this parameter would be added to a module manifest under
a new `OptionalFeatures` key. This key will be part of `PSData` to maintain
backwards compatibility. When the module is loaded in a version of PowerShell
that supports optional features, any optional features named in this key will
be enabled in the module scope.

A terminating error is generated if the same optional feature name is used
twice in the collection passed into the `-OptionalFeatures` parameter.

`[-DisabledOptionalFeatures <string[]>]`

This parameter would disable specific optional features in the new module
manifest that is generated.

The values provided to this parameter would be added to a module manifest under
a new `DisabledOptionalFeatures` key. This key will be part of `PSData` to
maintain backwards compatibility. When the module is loaded in a version of
PowerShell that supports optional features, any optional features named in this
key will be disabled in the module scope.

Allowing a feature to be disabled within a module is necessary if an older
module does not support a newer optional feature yet, and the module author
wants to ensure that module can be used even when an incompatible optional
feature is enabled in the session.

A terminating error is generated if the same optional feature name is used
twice across the `-OptionalFeatures` and `-DisabledOptionalFeatures`
parameters.

### Add parameter set to #requires statement

`#requires -OptionalFeatures <string[]> [-Disabled]`

This parameter set would enable, or disable if `-Disabled` is used, optional
features identified by `-Name` in the current script file.

### New command: Get-OptionalFeature

```none
Get-OptionalFeature [[-Name] <string[]>] [<CommonParameters>]
```

This command will return a list of the optional features that are available in
PowerShell, along with their auto-enable configuration, source and description.

The properties on the `S.M.A.OptionalFeature` object would be `Name`,
`AutoEnable`, `Source`, `Description`, defined as follows:

|Property Name|Description|
|--|--|
|`Name`|A string value that identifies the optional feature name|
|`AutoEnable`|An enumeration that identifies if the optional feature is auto-enabled. Values include `No`, `CurrentUser`, and `AllUsers`|
|`Source`|A string value that identifies the area of PowerShell that is affected by this optional feature|
|`Description`|A string value that describes the optional feature|

The default output format would be of type table with the properties `Name`,
`AutoEnable`, `Source`, and `Description`.

### Enabling and disabling optional features in current and future PowerShell sessions

```none
Enable-OptionalFeature [-Name] <string[]> [-AutoEnable { No | CurrentUser | AllUsers }]
[-WhatIf] [-Confirm] [<CommonParameters>]

Disable-OptionalFeature [-Name] <string[]> [-WhatIf] [-Confirm] [<CommonParameters>]
```

The `Enable-OptionalFeature` command will enable an optional feature in current
and future PowerShell sessions. To stop enabling an optional feature by default
in future PowerShell sessions, use `Enable-OptionalFeature -AutoEnable No`.

The `Disable-OptionalFeature` command will disable an optional feature in
the current PowerShell session.

### New command: Use-OptionalFeature

```none
Use-OptionalFeature [-Enable] <string[]> [[-Disable] <string[]>] [-ScriptBlock]
<ScriptBlock> [-Confirm] [<CommonParameters>]

Use-OptionalFeature -Disable <string[]> [-ScriptBlock] <ScriptBlock> [-Confirm]
[<CommonParameters>]
```

This command would enable or disable the optional features whose names are
identified in the `-Enable` and `-Disable` parameters for the duration of the
`ScriptBlock` identified in the `-ScriptBlock` parameter, and return the
features to their previous state afterwards. This allows for easy control of
optional features over a small section of code.

### New command: Test-OptionalFeature

```none
Test-OptionalFeature [-Name] <string> [<CommonParameters>]
```

This command would return true if the optional feature is enabled in the
current scope in the current session; false otherwise.

### Checking optional feature states within the PowerShell runtime

Much like Experimental Features, optional feature states will be managed using
a variable. A Read-Only `$EnabledOptionalFeatures` automatic variable will
contain the optional features that are currently enabled in the current scope.

## Alternate proposals and considerations

### Extend experimental features to support the enhancements defined in this RFC

At a glance, experimental features and optional features are very similar to
one another, so it was proposed that it may make sense to have them both use
the same functionality when it comes to enabling/disabling them in scripts and
modules; however, experimental features have a completely different intent (to
try out new functionality in a PowerShell session), are only for future
PowerShell sessions, and they only have a single on/off state. On the other
hand, optional features are for the current and future PowerShell sessions, for
the current user or all users, and may be enabled or disabled in various scopes
within those sessions. For that reason, this approach doesn't seem like a
viable solution. If we want to change that, perhaps someone should file an RFC
against experimental features to make that change.

### Enable/disable optional features in jobs according to their current state when the job is launched

Jobs run in the background have their own session, and therefore will not
automatically have optional features enabled or disabled according to the
current state when the job is launched. We should consider updating how jobs
are launched in the engine such that they do "inherit" optional feature
configuration to allow them to use optional features without additional code.

You might think that you can accomplish this using `#requires` in a script
block launched as a job, but that doesn't work -- the `#requires` statement is
ignored when you do this at the moment. For example, the see the results of the
following script when launched from PowerShell Core:

```PowerShell
Start-Job {
    #requires -PSEdition Desktop
    $PSVersionTable
} | Receive-Job -Wait
```

The result of that command shows that the version of PowerShell where the job
was run was Core, not Desktop, yet the job ran anyway despite the `#requires`
statement. This may be a bug. If it is, and if that bug is corrected, then you
could use #requires to enable/disable features, but regardless it would still
be preferable (and more intuitive) for jobs to "inherit" the current optional
feature configuration when they are invoked. This includes jobs launched with
`Start-Job`, `Start-ThreadJob`, the `&` background operator, parallelized
`ForEach-Object` commands, or the generic `-AsJob` parameter.
