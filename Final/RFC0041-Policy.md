---
RFC: RFC0041
Author: travisez13
Status: Final
SupercededBy:  N/A
Version: 0.1
Area: Engine
Comments Due: 6/30/2019
---

# `PowerShell Core` Policy

## Motivation

Consumers, developers, and enterprise system administrators should be able to flexibly and reliably configure PowerShell 7.

## Acknowledgement

I based this off of @iSazonov 's RFC, for just a slightly different purpose.
[PR #111](https://github.com/PowerShell/PowerShell-RFC/pull/111)

## Goals

1. Specify how PowerShell 7 will deal with having both Windows PowerShell and PowerShell Core Group Policy.
    - This is covered in [Policy settings Setting Fall-Back](#policy-settings-setting-fall-back).
1. Define how the `pwsh -settingsfile` switch should behave.
    - This is covered in  [Parameter `-settingsfile`](#Parameter--settingsfile)
1. Specify where setting will be read from on various supported platforms.

## Definitions

- **Computer-Wide settings/policy**  - setting or policy applied to an operating system environment (OSE), affecting all users of the OSE.
- **Per-User settings/policy** - setting or policy applied to a specific user of an OSE, and not applied to the OSE as a whole.

## Specification

`PowerShell 7` should be configured using the following schemes:

- On Windows - Group Policy Objects (GPO), and settings files.
- On Unix - settings files.

The settings files have `JSON` format.

**Warning** The settings files differ from `PowerShell 7` _profile_ files, which are PowerShell scripts run at startup.

Configuration schemes allow to customize `PowerShell 7` in the most flexible way:

- Enterprise system administrators can use GPO,
    Computer-wide settings files to apply approved configuration settings and mandatory security settings in a centralized manner.
    Most settings can be applied either to the user or computer-wide.
    We will cover the precedence of these in [Registry keys and settings](#registry-keys-and-settings).
- Developers and consumers can use user, or computer-wide level setting files.

### Configuration defaults

PowerShell 7 has hard-coded defaults for all configuration policies and settings.

The default values must be `secure-by-default`.

For release versions hard-coded defaults must be the same as ones in pre-installed configuration files. For preview versions they may vary (ex., enable experimental features and so on).

Computer-wide configuration includes security sensitive setting,
and failing to read those setting, for example if the file is locked or corrupted, could result in an insecure system.
So, if during startup, PowerShell 7 cannot read settings files from the Computer-Wide scope,
it fails to startup.
Because registry reads are more atomic,
this is not an issue for group policy settings,
but if we faced the same issues for these settings the solution would be the similar.

If during startup PowerShell 7 cannot read user configuration files it uses _hardcoded_ defaults and emits a warning.

PowerShell 7 does not update configuration values from modified configuration files during a given session after the configuration has been loaded.

### Settings locations

`PowerShell 7` settings are grouped into `Policy settings` and `Non-policy settings`.
Regular settings are normal configuration settings.
Regular settings can be treated as default and recommended values.
Policy settings have a higher precedence than regular settings.
See [Precedence for Policy settings in descending order](#precedence-for-policy-settings-in-descending-order).
Policy settings are used by administrators to centrally manage PowerShell.

| Location     | Policy settings                                           | Regular settings                                           |
|--------------|-----------------------------------------------------------|------------------------------------------------------------|
| File section | "PowerShell": { "PolicySettings": {...} }                 | "PowerShell": { "RegularSettings": {...} }                 |
| Registry key | Software\Policies\Microsoft\PowerShellCore                | Not Applicable                                             |

### Policy settings Setting Fall-Back

#### Motivation - Policy Setting Fall-Back

Help to transition from Windows PowerShell to PowerShell 7.

#### Implementation

For Policy Settings,
each policy should have a `Use Windows PowerShell Policy` which will indicate that the policy should the read from
`SOFTWARE\Policies\Microsoft\Windows\PowerShell` instead of `Software\Policies\Microsoft\PowerShellCore`.
The default in Group Policy is to have no policy, so it would not fall back to Windows PowerShell Policy, or apply PowerShell 7 policy.

### Precedence of applying settings

Because a configuration setting can be in several schemes, the setting wins according to the priority of its scheme.

#### Precedence for Computer-Wide settings in descending order

Note, this is listed as `Computer, Then User` in [Registry keys and settings](#registry-keys-and-settings).

| Scheme                      | Windows                                              | Unix                                                 |
|-----------------------------|------------------------------------------------------|------------------------------------------------------|
| GPO -> Computer Policy      | HKLM\Software\Policies\Microsoft\PowerShellCore      | See [Moving configuration out of PSHome][moving]     |
| GPO -> User Policy          | HKCU\Software\Policies\Microsoft\PowerShellCore      | %XDG_CONFIG_HOME%/powershell.config.json             |
| File -> Application-Startup | pwsh -settingsfile `somepath/powershell.config.json` | pwsh -settingsfile `somepath/powershell.config.json` |
| File -> Computer-Wide       | See [Moving configuration out of PSHome][moving]     | [Moving configuration out of PSHome][moving]         |
| File -> User-Wide           | %APPDATA%/powershell.config.json                     | %XDG_CONFIG_HOME%/powershell.config.json             |

Defaults:

`%APPDATA%` - `C:\Users\useraccount\AppData\Roaming`

`%XDG_CONFIG_HOME%` - `HOME/.config`

#### Parameter `-settingsfile`

With `-settingsfile` parameter users can assign custom settings from the config file and overwrite user-wide settings.

##### More definitions

- System Lock-down mode:
  When Windows Defender Application Control or AppLocker force PowerShell into Constrained Language mode and
  only trusted code runs in Full Language mode.
  See [PowerShell Constrained Language Mode](https://devblogs.microsoft.com/powershell/powershell-constrained-language-mode/)

##### Computer-wide and per-user policy settings

Admin/root users can overwrite computer-wide and per-user policy settings using `-settingsfile`,
only when not in System Lock-down mode.

This will have performance impact on startup, but only when `-settingsfile` is specified.

#### Precedence for Per-user settings in descending order

Note, this is listed as `User, then Computer` in [Registry keys and settings](#registry-keys-and-settings).

| Scheme                      | Windows                                              | Unix                                                 |
|-----------------------------|------------------------------------------------------|------------------------------------------------------|
| GPO -> User Config          | HKCU\Software\Policies\Microsoft\PowerShellCore      | %XDG_CONFIG_HOME%/powershell.config.json             |
| GPO -> Computer Config      | HKLM\Software\Policies\Microsoft\PowerShellCore      |  See [Moving configuration out of PSHome][moving]    |
| File -> Application-Startup | pwsh -settingsfile `somepath/powershell.config.json` | pwsh -settingsfile `somepath/powershell.config.json` |
| File -> User-Wide           | %APPDATA%\powershell.config.json                     | %XDG_CONFIG_HOME%/powershell.config.json             |
| File -> Computer-Wide       | See [Moving configuration out of PSHome][moving]     | /opt/Microsoft/powershell/powershell.config.json     |

#### Precedence for UpdatableHelp in descending order

Note, this is listed as `Computer` in [Registry keys and settings](#registry-keys-and-settings).

| Scheme                      | Windows                                              | Unix                                                 |
|-----------------------------|------------------------------------------------------|------------------------------------------------------|
| GPO -> Computer Config      | HKLM\Software\Policies\Microsoft\PowerShellCore      |  See [Moving configuration out of PSHome][moving]    |
| File -> Application-Startup | pwsh -settingsfile `somepath/powershell.config.json` | pwsh -settingsfile `somepath/powershell.config.json` |
| File -> Computer-Wide       | See [Moving configuration out of PSHome][moving]     | /opt/Microsoft/powershell/powershell.config.json     |

### Configuration settings

A set of configuration settings in GPO scheme and file scheme for policy settings and regular settings is the same. This allows to discover and configure settings in the simplest and fastest way.

#### Registry keys and settings

Notes:

- All policies are in `Software\Policies\Microsoft\PowerShellCore`.
- `ExecutionPolicy` is not in any SubKey.

| SubKey                      | Option                             | Type   | Precedence          |
|-----------------------------|------------------------------------|--------|---------------------|
| -                           | -                                  |        |                     |
| -                           | -                                  |        |                     |
|                             | ExecutionPolicy                    | String | Computer, Then User |
| ConsoleSessionConfiguration | EnableConsoleSessionConfiguration  | DWORD  | User, then Computer |
| ConsoleSessionConfiguration | ConsoleSessionConfigurationName    | String | User, then Computer |
| ModuleLogging               | EnableModuleLogging                | DWORD  | Computer, Then User |
| ModuleLogging               | ModuleNames                        | String | Computer, Then User |
| ProtectedEventLogging       | EncryptionCertificate              | DWORD  | Computer Wide       |
| ScriptBlockLogging          | EnableScriptBlockInvocationLogging | DWORD  | Computer, Then User |
| ScriptBlockLogging          | EnableScriptBlockLogging           | DWORD  | Computer, Then User |
| Transcription               | EnableTranscripting                | DWORD  | Computer, Then User |
| Transcription               | EnableInvocationHeader             | DWORD  | Computer, Then User |
| Transcription               | OutputDirectory                    | String | Computer, Then User |
| UpdatableHelp               | DefaultSourcePath                  | String | Computer Wide       |

I filed [#9632](https://github.com/PowerShell/PowerShell/issues/9632) on UpdatableHelp-DefaultSourcePath to make it allow Per-user settings.

#### JSON file settings format

```json
{
  "PowerShell": {
    "RegularSettings": {
      "ConsoleSessionConfiguration": {
        "EnableConsoleSessionConfiguration": true,
        "ConsoleSessionConfigurationName": "name"
      },
      "ProtectedEventLogging": {
        "EnableProtectedEventLogging": false,
        "EncryptionCertificate": [
          "Joe"
        ]
      },
      "ScriptBlockLogging": {
        "EnableScriptBlockInvocationLogging": true,
        "EnableScriptBlockLogging": false
      },
      "ScriptExecution": {
        "ExecutionPolicy": "RemoteSigned",
        "PipelineMaxStackSizeMB": 10
      },
      "Transcription": {
        "EnableTranscripting": true,
        "EnableInvocationHeader": true,
        "OutputDirectory": "c:\\tmp"
      },
      "UpdatableHelp": {
        "DefaultSourcePath": "f:\\temp"
      }
    },

    "PoliciesSettings": {
      ...
    }
  }
}
```

## Alternate Proposals and Considerations

### Automatically resolve Windows PowerShell policy conflicts

#### Motivation -  Automatically policy

This is a description of the alternative to [Policy settings Setting Fall-Back](#policy-settings-setting-fall-back).
The main purpose of describing the alternative is to describe why it should not be pursued.

#### Description

PowerShell could attempt to resolve policy conflicts between PowerShell 7 policy and Windows PowerShell policy.
This would make the `Precedence for Policy settings` not just a simple list but a complex set of rules that would not be easily understood.  See [this conversation](https://github.com/PowerShell/PowerShell/issues/9309?#issuecomment-480643922).

### Allowing environment variable in the JSON

A new RFC should be drafted about how to allow environment variables in the values in the JSON.
This would allow consistent files across platforms.

### Moving configuration out of PSHome

Per issues [9278](https://github.com/PowerShell/PowerShell/issues/9278) we need to move configuration out of PSHome,
follow that issue for issues related to new locations of files.

[moving]:#moving-configuration-out-of-pshome

### Allowing Regular settings from the registry in Windows

This is out of scope of this RFCs
