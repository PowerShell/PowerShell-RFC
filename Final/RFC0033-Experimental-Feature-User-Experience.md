---
RFC: RFC0033
Author: Steve Lee
Status: Final
SupercededBy: N/A
Version: 1.0
Area: Microsoft.PowerShell.Core
Comments Due: 12/1/2018
Plan to implement: Yes
---

# Experimental Feature User Experience

[Experimental Feature Flags](https://github.com/PowerShell/PowerShell-RFC/blob/master/5-Final/RFC0029-Support-Experimental-Features.md)
enable developers to expose experimental features to users to gather feedback before finalizing design.
That feature was focused on the developer and did not make it easy for users to discover and enable experimental features.
This RFC addresses the user experience.

## Motivation

    As a PowerShell user,
    I can discover and enable experimental features,
    so that I can try new capabilities safely and provide feedback.

## Specification

### Get-ExperimentalFeature *Breaking Change*

Currently, `Get-ExperimentalFeature` requires the `-ListAvailable` switch to enumerate available experimental features.
This was modeled after `Get-Module -ListAvailable`.
Without the switch, `Get-ExperimentalFeature` only shows the enabled experimental features, which by default is none.
Most users will try this cmdlet without the switch and assume there are no experimental features to try.

Since there are likely not many experimental features available at any point in time and less enabled,
it would make sense to remove the `-ListAvailable` switch and simply display all experimental features.
The current output already has a column indicating if the experimental feature is enabled allowing for easy filtering.

### System and User scope powershell.config.json

Enabling experimental features automatically requires creating or updating a `powershell.config.json` file in `$PSHOME`
which is read at startup which affects all users or from `$HOME\Documents\PowerShell\powershell.config.json` on Windows
and from `$HOME/.config/powershell/powershell.config.json` on Linux and macOS per user.

Experimental features are currently not being read from the user config and this RFC will enable reading
of the user config in addition to the system config.
In the case where the user config exists, it will take precedence over the system config for experimental features
in that the system config is not read for experimental features.
System config is not policy so this should be acceptable and expected.

### Enable and Disable cmdlets

```none
Enable-ExperimentalFeature [[-Name] <string[]>] [-Scope { CurrentUser | AllUsers }] [<CommonParameters>]

Disable-ExperimentalFeature [[-Name] <string[]>] [-Scope { CurrentUser | AllUsers }] [<CommonParameters>]
```

These cmdlets allow users to selectively enable and disable experimental features.

The `-Name` parameter shall accept `ValueFromPipelineByPropertyName` and is the name of the experimental feature.

The `-Scope` parameter is optional and defaults to `CurrentUser` and will create or update the
`powershell.config.json` in `$HOME\Documents\PowerShell\powershell.config.json` on Windows
and from `$HOME/.config/powershell/powershell.config.json` on Linux and macOS.
If `-Scope` is `AllUsers`, it will create or update `$PSHOME\powershell.config.json`.

Experimental features are read and enabled at PowerShell startup, so a warning message will be provided informing the user:
> Experimental feature changes will only be applied after restarting PowerShell.

Upon success, there is no output other than the warning message.

## Alternate Proposals and Considerations

### Enabled property

Currently, an experimental feature has an `Enabled` property that is `true` or `false`.
In addition to the warning message that a restart is required after changing experimental feature status,
the `Enabled` property could be changed to an enum instead of a boolean: True, False, Pending.
However, since experimental features is expected to be used by more advanced users,
this seems unnecessary.

Alternatively, we can add a `RestartRequired` property to indicate an enabled feature is pending.

### Features that were previously Experimental

In the case where an Experimental Feature no longer experimental (whether accepted or rejected),
the config file may still have that feature listed.
We can add a `State` property that has enum values: Available, NotAvailable.
This `State` can also convey the `RestartRequired` state.
If this becomes a problem in the future, we can have cmdlets to help clean-up non-valid settings in the
configuration (which will be more than just experimental features).
Since this is additive, this is currently outside the scope of this RFC.

### PowerShell instance specific configuration

The system (AllUsers) configuration is in `$PSHOME` and specific to that instance of PowerShell.
The user config is global to all instances of PowerShell which can create problems.
For experimental features, this is not a big issue as experimental features listed to
be enabled in the config file that don't exist are silently ignored.
However, other configuration that one may want to apply to a preview release and not a
stable release cannot be applied except for AllUsers currently.
This should be addressed in a separate configuration RFC.
