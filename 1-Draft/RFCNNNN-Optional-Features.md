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

There are several important issues in the PowerShell language that cannot be corrected without introducing breaking changes. At the same time, the number of breaking changes introduced in a new version of PowerShell needs to be as minimal as possible, so that there is a low barrier to adoption of new versions, allowing community members to transition scripts and modules across versions more easily. Given that those two statements are in conflict with one another, we need to consider how we can optionally introduce breaking changes into PowerShell over time.

PowerShell has support for experimental features, which some may think covers this need; however, the intent of experimental features is to allow the community to try pre-release versions of PowerShell with breaking changes that are deemed necessary or new features that are not necessarily fully tested/polished so that they can more accurately assess the impact of those features. For release versions of PowerShell, an experimental feature has one of three possible outcomes:

1. The the experimental feature is deemed necessary, adequately tested/polished and accepted by the community as not harmful to adoption of new versions, in which case the experimental feature is no longer marked as experimental.
1. The experimental feature is deemed necessary, and adequately tested/polished, but considered harmful to adoption of new versions, in which case the experimental feature is changed to an optional feature.
1. The experimental feature is deemed not useful enough, in which case the experimental feature is deprecated.

In some cases a breaking change may be implemented immediately as an optional feature, when it is known up front that such a breaking change would be considered harmful to adoption of new versions of PowerShell if it was in place by default yet is still found important enough to implement as an optional feature.

Given all of that, we need to add support for optional features in PowerShell so that what is described above becomes a reality.

As an example of a feature that will be optional if implemented, see RFCNNNN-Propagate-Execution-Preferences-Beyond-Module-Scope or RFCNNNN-Make-Terminating-Errors-Terminate.

## Motivation

As a script, function, or module author,
I can enable optional features for specific users or in specific scopes,
so that I can leverage new functionality that may break existing scripts without risk.

## User experience

```powershell
# Create a module manifest, specifically enabling or disabling one or more optional features in the manifest
New-ModuleManifest -Path ./test.psd1 -OptionalFeatures @('OptionalFeature1',@{Name='OptionalFeature2';Enabled=$false}) -PassThru | Get-Content

# Output:
#
# @{
#
# <snip>
#
# # Optional features enabled or disabled in this module.
# OptionalFeatures = @(
#     'OptionalFeature1'
#     @{Name='OptionalFeature2';Enabled=$false}
# )
#
# }

# Create a script file, enabling or disabling one or more optional features in the file
@'
#requires -OptionalFeature OptionalFeature1
#requires -OptionalFeature OptionalFeature2 -Disabled

<snip>
'@ | Out-File -FilePath ./test.ps1

# Get the current optional feature configuration for the current user and all users
Get-OptionalFeatureConfiguration

# Output:
#
#     For: AllUsers
#
# Name                              Session                  NewManifest
# ----                              ------                   -----------
# OptionalFeature1                  False                    True
# OptionalFeature2                  True                     False
# OptionalFeature4                  True                     True
# OptionalFeature5                  False                    False
#
#
#     For: CurrentUser
#
# Name                              Session                  NewManifest
# ----                              ------                   -----------
# OptionalFeature2                  False                    True
# OptionalFeature3                  False                    True
#

# Get a list of optional features, their source, and their descriptions
Get-OptionalFeature

# Output:
#
# Name                              Source                              Description
# ----                              ------                              -----------
# OptionalFeature1                  PSEngine                            Description of optional feature 1
# OptionalFeature2                  PSEngine                            Description of optional feature 2
# OptionalFeature3                  PSEngine                            Description of optional feature 3
# OptionalFeature4                  PSEngine                            Description of optional feature 4

# Enable an optional feature in current and future PowerShell sessions for all
# users in PowerShell.
Enable-OptionalFeature -Name OptionalFeature1 -For AllUsers

# Output:
# None

# Disable an optional feature in current and future PowerShell sessions for the
# current user in PowerShell.
Disable-OptionalFeature -Name OptionalFeature1 -For CurrentUser

# Output:
# None, unless the feature was explicitly enabled for all users and is being
# disabled only for the current user as an override, as is the case here,
# in which case they get prompted to confirm. This is described below.

# Enable an optional feature in all new module manifests created with
# New-ModuleManifest in the current and future PowerShell sessions for any user
# in PowerShell.
Enable-OptionalFeature -Name OptionalFeature2 -For AllUsers -InNewModuleManifests

# Output:
# None

# Enable an optional feature in all new module manifests created with
# New-ModuleManifest in the current and future PowerShell sessions for the
# current user in PowerShell.
Disable-OptionalFeature -Name OptionalFeature3 -InNewModuleManifests

# Output:
# None

# Enable and disable an optional feature the duration of the script block being
# invoked.
Use-OptionalFeature -Enable OptionalFeature1 -Disable OptionalFeature2 -ScriptBlock {
    # Do things using OptionalFeature1 here
    # OptionalFeature2 cannot be used here
}
# If OptionalFeature1 was not enabled before this invocation, it is still no
# longerenabled here. If OptionalFeature2 was enabled before this invocation,
# it is still enabled here. All to say, their state before the call is
# preserved.
```

## Specification

Unlike experimental features, which can only be enabled or disabled in PowerShell sessions created after enabling or disabling them, optional features can be enabled or disabled in the current PowerShell session as well as in future PowerShell sessions. This is necessary to allow certain functionality to be "lit up" in packaged modules or scripts.

Below you will find details describing how this functionality will be implemented.

### System and User powershell.config.json configuration files

Enabling optional features automatically in future PowerShell sessions requires creating or updating one of two `powershell.config.json` configuration files that are read on startup of a new PowerShell session:

* one in `$PSHOME`, which applies to all user sessions
* one in `$HOME\Documents\PowerShell\powershell.config.json` on Windows or `$HOME/.config/powershell/powershell.config.json` on Linux and macOS, which applies only to current user sessions.

This RFC will enable optional feature defaults to be read from these configuration files, with current user configuration taking precedence over system (all users) configuration. System config is not policy so this should be acceptable and expected.

### Add parameter to New-ModuleManifest

`[-OptionalFeatures <object[]>]`

This parameter would configure specific optional features in the new module manifest that is generated.

The values provided to this parameter would be combined with optional features that are enabled or disabled by default according to the session configuration files, with the values specified in the `New-ModuleManifest` command overriding the settings for optional features with the same name that are configured in the configuration files. Entries in this collection would either be string (the name of the optional feature to enable) or a hashtable with two keys: `name` (a string) and `enabled` (a boolean value). The hashtable allows an optional feature to be specifically disabled instead of enabled within a module, which is necessary if an older module does not support a newer optional feature yet.

A terminating error is generated if the same optional feature name is used twice in the collection passed into the `-OptionalFeatures` parameter.

### Add parameter set to #requires statement

`#requires -OptionalFeatures <string[]> [-Disabled]`

This parameter set would enable, or disable if `-Disabled` is used, optional features identified by `-Name` in the current script file.

### New command: Get-OptionalFeatureConfiguration

```none
Get-OptionalFeatureConfiguration [[-Name] <string[]>] [-For { CurrentUser | AllUsers }] [<CommonParameters>]
```

This command would return the current configuration of optional features that are available in PowerShell, read from the configuration files.

The properties on the `S.M.A.OptionalFeatureConfiguration` object would be `Name`, `Session`, `NewManifest`, and `For`, defined as follows:

|Property Name|Description|
|--|--|
|`Name`|A string value that identifies the optional feature name|
|`Session`|A boolean value that identifies whether the optional feature is enabled or disabled in the current and new PowerShell sessions|
|`NewManifest`|A boolean value that identifies whether the optional feature is enabled or disabled in manifests created by new module manifests in the current and new PowerShell sessions|
|`For`|An enumeration flag identifying whether the optional feature configuration was set up for the `CurrentUser` or `AllUsers`|

The default output format is of type table with the properties `Name`, `Session`, and `NewManifest` with the results grouped by `For`.

When this command is invoked with the `-For` parameter, the results are automatically filtered for that configuration file. The default value for `-For` is both flags, showing configuration values from both configuration files.

### New command: Get-OptionalFeature

```none
Get-OptionalFeature [[-Name] <string[]>] [<CommonParameters>]
```

This command will return a list of the optional features that are available in PowerShell, along with their source and description.

The properties on the `S.M.A.OptionalFeature` object would be `Name`, `Source`, `Description`, defined as follows:

|Property Name|Description|
|--|--|
|`Name`|A string value that identifies the optional feature name|
|`Source`|A string value that identifies the area of PowerShell that is affected by this optional feature|
|`Description`|A string value that describes the optional feature|

The default output format would be of type table with the properties `Name`, `Source`, and `Description`.

### Enabling and disabling optional features in current and future PowerShell sessions

```none
Enable-OptionalFeature [-Name] <string[]> [-For { CurrentUser | AllUsers }] [-InNewModuleManifests] [-WhatIf] [-Confirm] [<CommonParameters>]

Disable-OptionalFeature [-Name] <string[]> [-For { CurrentUser | AllUsers }] [-InNewModuleManifests] [-WhatIf] [-Confirm] [<CommonParameters>]
```

It is important to note up front that there are three default states for an optional feature: enabled by default, implicitly disabled by default, and explicitly disabled by default. The only time an optional feature needs to be explicitly disabled by default is if it is enabled by default in the AllUsers configuration file and a specific user wants to disable it for their sessions. This impacts how `Disable-OptionalFeature` works.

The `Enable-OptionalFeature` command will enable an optional feature in current and future PowerShell sessions either globally (if the `-InNewModuleManifests` switch is not used) or only in manifests created by `New-ModuleManifest`.

The `Disable-OptionalFeature` command will disable an optional feature in current and future PowerShell sessions either globally (if the `-InNewModuleManifests` switch is not used) or only in manifests created by `New-ModuleManifest`. If the feature is being disabled for `AllUsers` and the optional feature is completely disabled in that configuration file as a result of this command, the entry is removed from the configuration file. If the feature is being disabled for `AllUsers` and there is no entry in the system (all users) configuration file, nothing happens. If the feature is being disabled for the `CurrentUser` there is no entry in the system (all users) or current user configuration files, nothing happens. If the feature is being disabled for the `CurrentUser` and the optional feature is enabled in the `AllUsers` configuration file, users will be informed that this feature is enabled for all users and asked to confirm that they want to explicitly disable this feature for the current user in the current and future PowerShell sessions. They can always re-enable it later.

### New command: Use-OptionalFeature

```none
Use-OptionalFeature [[-Enable] <string[]>] [[-Disable] <string[]>] [-ScriptBlock] <ScriptBlock> [-Confirm] [<CommonParameters>]
```

This command would enable or disable the optional features whose names are identified in the `-Enable` and `-Disable` parameters for the duration of the `ScriptBlock` identified in the `-ScriptBlock` parameter, and return the features to their previous state afterwards. This allows for easy use of optional features over a small section of code. If neither `-Enable` or `-Disable` are used, a terminating error is thrown.

### Checking optional feature states within the PowerShell runtime

Optional features can be enabled or disabled in a session, module, script, or script block. Since enabling or disabling an optional feature can happen at different levels, the current state of an optional feature should be maintained in a stack, where the validation logic simply peeks at the top of the stack to see if the feature is enabled or not, popping the top of the stack off when appropriate (when leaving the scope of the module, script, or script block where the feature is enabled).

## Alternate proposals and considerations

### Extend experimental features to support the enhancements defined in this RFC

At a glance, experimental features and optional features are very similar to one another, so it was proposed that it may make sense to have them both use the same functionality when it comes to enabling/disabling them in scripts and modules; however, experimental features have a completely different intent (to try out new functionality in a PowerShell session), are only for future PowerShell sessions, and they only have a single on/off state. On the other hand, optional features are for the current and future PowerShell sessions, for the current user or all users, and may be enabled or disabled in various scopes within those sessions. For that reason, this approach doesn't seem like a viable solution. If we want to change that, perhaps someone should file an RFC against experimental features to make that change.

### Enable/disable optional features in jobs according to their current state when the job is launched

Jobs run in the background have their own session, and therefore will not automatically have optional features enabled or disabled according to the current state when the job is launched. We should consider updating how jobs are launched in the engine such that they do "inherit" optional feature configuration to allow them to use optional features without additional code.

You might think that you can accomplish this using `#requires` in a script block launched as a job, but that doesn't work -- the `#requires` statement is ignored when you do this at the moment. For example, the see the results of the following script when launched from PowerShell Core:

```PowerShell
Start-Job {
    #requires -PSEdition Desktop
    $PSVersionTable
} | Receive-Job -Wait
```

The result of that command shows that the version of PowerShell where the job was run was Core, not Desktop, yet the job ran anyway despite the `#requires` statement. This may be a bug. If it is, and if that bug is corrected, then you could use #requires to enable/disable features, but regardless it would still be preferable (and more intuitive) for jobs to "inherit" the current optional feature configuration when they are invoked. This includes jobs launched with `Start-Job`, `Start-ThreadJob`, the `&` background operator, parallelized `foreach` statements or `ForEach-Object` commands, or the generic `-AsJob` parameter.
