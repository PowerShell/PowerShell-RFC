---
RFC: RFCnnnn
Author: Kirk Munro
Status: Draft
SupercededBy: 
Version: 1.0
Area: Engine
Comments Due: July 15, 2019
Plan to implement: Yes
---

# Optional features in PowerShell

There are several important issues in the PowerShell language that cannot be corrected without introducing breaking changes. At the same time, the number of breaking changes introduced in a new version of PowerShell needs to be as minimal as possible, so that there is a low barrier to adoption of new versions, allowing community members can transition scripts and modules across versions more easily. Given that those two statements are in conflict with one another, we need to consider how we can optionally introduce breaking changes into PowerShell over time.

PowerShell has support for experimental features, which some may think covers this need; however, the intent of experimental features is to allow the community to try pre-release versions of PowerShell with breaking changes that are deemed necessary so that they can more accurately assess the impact of those breaking changes. For release versions of PowerShell, an experimental feature has one of three possible outcomes:

1. The breaking change in the experimental feature is deemed necessary and accepted by the community as not harmful to adoption of new versions, in which case the experimental feature is no longer marked as experimental.
1. The breaking change in the experimental feature is deemed necessary, but considered harmful to adoption of new versions, in which case the experimental feature is changed to an optional feature.
1. The breaking change in the experimental feature is deemed not useful enough, in which case the experimental feature is deprecated.

In some cases a breaking change may be implemented immediately as an optional feature, when it is known up front that such a breaking change would be considered harmful to adoption of new versions of PowerShell.

Given all of that, we need to add support for optional features in PowerShell so that what is described above becomes a reality.

As an example of a feature that will be optional if implemented, see RFCNNNN-Propagate-Execution-Preferences-Beyond-Module-Scope or RFCNNNN-Make-Terminating-Errors-Terminate.

## Motivation

As a script, function, or module author,
I can enable optional features in my scripts or modules,
so that I can leverage new functionality that could break existing scripts.

## User experience

```powershell
# Create a module manifest, specifically enabling one or more optional features in the manifest
New-ModuleManifest -Path ./test.psd1 -OptionalFeatures @('OptionalFeature1','OptionalFeature2') -PassThru | Get-Content

# Output:
#
# @{
#
# <snip>
#
# # Optional features enabled in this module.
# OptionalFeatures = @(
#     'OptionalFeature1'
#     'OptionalFeature2'
# )
#
# }

# Create a script file, enabling one or more optional features in the file
@'
#requires -OptionalFeature OptionalFeature1,OptionalFeature2

<snip>
'@ | Out-File -FilePath ./test.ps1

# Get a list of optional features that are available
Get-OptionalFeature

# Output:
#
#     EnabledIn: Script
#
# Name                              Source                              Description
# ----                              ------                              -----------
# OptionalFeature1                  PSEngine                            Description of optional feature 1
# OptionalFeature2                  PSEngine                            Description of optional feature 2
#
#
#     EnabledIn: NotEnabled
#
# Name                              Source                              Description
# ----                              ------                              -----------
# OptionalFeature3                  PSEngine                            Description of optional feature 3

# Enable an optional feature by default in PowerShell
Enable-OptionalFeature -Name OptionalFeature1

# Output:
# This works just like Enable-ExperimentalFeature, turning the optional
# feature on by default for all future sessions in PowerShell.

# Disable an optional feature by default in PowerShell
Disable-OptionalFeature -Name OptionalFeature1

# Output:
# This works ust like Disable-ExperimentalFeature, turning the optional
# feature off by default for all future sessions in PowerShell.

# Enable an optional feature by default in all new module manifests
# created with New-ModuleManifest in all future sessions in PowerShell.
Enable-OptionalFeature -Name OptionalFeature1 -NewModuleManifests

# Disable an optional feature by default in all new module manifests
# created with New-ModuleManifest in all future sessions in PowerShell.
Disable-OptionalFeature -Name OptionalFeature1 -NewModuleManifests
```

## Specification

Aside from closely (but not exactly, see below) mirroring what is already in place internally for experimental features in PowerShell, this RFC includes a few additional enhancements that will be useful for optional features, as follows:

### Add parameter to New-ModuleManifest

`[-OptionalFeatures <string[]>]`

This new parameter would assign specific optional features to new modules. Note that these would be in addition to optional features that are enabled by default in manifests created with `New-ModuleManifest`.

### Add parameter to #requires statement

`#requires -OptionalFeatures <string[]>`

This new parameter would enable optional features in the current script file.

### New command: Get-OptionalFeature

```none
Get-OptionalFeature [[-Name] <string[]>] [<CommonParameters>]
```

This command would return the optional features that are available in PowerShell. The default output format would be of type table with the properties `Name`, `Source`, and `Description`, and with the results grouped by the value of the `EnableIn` property. All of those properties would be of type string except for `EnabledIn`, which would be an enumeration with the possible values of `NotEnabled`, `Session`, `Manifest`, `Script`, and `Scope`. This differs from experimental features where `Enabled` is a boolean value. Given the locations in which an optional feature can be enabled, it would be more informative to identify where it is enabled than simply showing `$true` or `$false`. The enumeration values have the following meaning:

|Value|Description|How to set the feature up this way|
|--|--|--|
|NotEnabled|The optional feature is not enabled at all|Disable-OptionalFeature command|
|Session|The optional feature is enabled by default in all PowerShell sessions|Enable-OptionalFeature command|
|Manifest|The optional feature is enabled in the manifest for the current module|OptionalFeatures entry in module manifest|
|Script|The optional feature is enabled in the current script|#requires entry in script file|
|Scope|The optional feature is enabled the current scope|Use-OptionalFeature command|

### New command: Enable-OptionalFeature

```none
Enable-OptionalFeature [-Name] <string[]> [-NewModuleManifests] [-WhatIf] [-Confirm] [<CommonParameters>]
```

This command would enable an optional feature either globally (if the `-NewModuleManifests` switch is not used) or only in new module manifests created by `New-ModuleManifest`.

### New command: Disable-OptionalFeature

```none
Disable-OptionalFeature [-Name] <string[]> [-NewModuleManifests] [-WhatIf] [-Confirm] [<CommonParameters>]
```

This command would disable an optional feature either globally (if the `-NewModuleManifests` switch is not used) or only in new module manifests created by `New-ModuleManifest`. If the optional feature is not enabled that way in the first place, nothing would happen.

### New command: Use-OptionalFeature

```none
Use-OptionalFeature [-Name] <string[]> [-ScriptBlock] <ScriptBlock> [-Confirm] [<CommonParameters>]
```

This command would enable an optional feature for the duration of the `ScriptBlock` identified in the `-ScriptBlock` parameter, and return the feature to its previous state afterwards. This allows for easy use of an optional feature over a small section of code.

## Alternate proposals and considerations

### Extend experimental features to support the enhancements defined in this RFC

Experimental features and optional features are very similar to one another, so much so that they really only differ in name. Given the model for how both of these types of features are used, it may make sense to have them both use the same functionality when it comes to enabling/disabling them in scripts and modules. The downside I see to this approach is that optional features are permanent features in PowerShell while experimental features are not, so it may not be a good idea to support more permanent ways to enable experimental features such as `#requires` or enabling an experimental feature in a new module manifest.

### Supporting a `-Scope` parameter like the experimental feature cmdlets do

The `Enable-OptionalFeature` and `Disable-OptionalFeature` cmdlets could support a `-Scope` parameter like their experimental feature cmdlet counterparts do. I felt it was better to remove this for optional features, because it may be risky to allow a command to enable an optional feature in a scope above the one in which it is invoked, influencing behaviour elsewhere.

### Allow optional features to be disabled at a certain level

Certain optional features may be so important that PowerShell should be installed with them to be on by default. In cases where this happens, scripters should be able to indicate that they want the opposite behaviour in a script file or module, so that they can ensure any compatibility issues are addressed before the feature is enabled in that module/script. With this in mind, we could either allow optional features to be disabled at a certain level, or stick with enable only, inversing how the optional feature is designed such that turning it on effectively disables a breaking fix that was deemed important enough to have fixed by default.
