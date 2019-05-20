---
RFC:  RFCnnnn
Author:  travisez13
Status:  Draft
SupercededBy:  N/A
Version: 0.1
Area: Engine
Comments Due: 6/30/2019
---

# `PowerShell Core` Policy

## Motivation

Consumers, developers, and enterprise system administrators should be able to flexibly and reliable way to configure PowerShell 7.

## Acknowledgement

I based this off of @iSazonov 's RFC, for just a slightly different purpose.
[PR #111](https://github.com/PowerShell/PowerShell-RFC/pull/111)

## Specification

`PowerShell 7` should be configured using the following schemes:

- On Windows - Group Policy Objects (GPO), Group Policy Preferences (GPP) and settings files.
- On Unix - settings files.

The settings files have `JSON` format.

**Warning** The settings files differ from `PowerShell 7` _profile_ files, which are PowerShell scripts run at startup.

Configuration schemes allow to customize `PowerShell 7` in the most flexible way:

- Enterprise system administrators can use GPO,
    GPP and computer-wide settings files to apply approved configuration settings and mandatory security settings in a centralized manner.
    The same settings can be applied at user, application or startup levels.
- Developers and consumers can use user, application and startup level settings files.

### Configuration defaults

PowerShell 7 has hard-coded defaults for all configuration options.

The default values must be `secure-by-default`.

For release versions hard-coded defaults must be the same as ones in re-installed configuration files. For preview versions they may vary (ex., enable experimental features and so on).

If during startup PowerShell 7 cannot read system configuration files it fails to startup.

If during startup PowerShell 7 cannot read user configuration files it uses _hardcoded_ defaults.

If during operation PowerShell 7 cannot read configuration files it continue to use _current_ (runtime) configuration values.

### Settings locations

`PowerShell 7` settings are grouped into `Policy settings` and `Regular settings`.
Regular settings are normal configuration settings.
Regular settings can be treated as default values.
Policy settings is high priority and overlap regular settings.
Policy settings are used by administrators to centrally manage applications.

| Location     | Policy settings                                           | Regular settings                                           |
|--------------|-----------------------------------------------------------|------------------------------------------------------------|
| File section | "PowerShell": { "PolicySettings": {...} }                 | "PowerShell": { "RegularSettings": {...} }                 |
| File section | "OtherPowerShellApplication": { "PolicySettings": {...} } | "OtherPowerShellApplication": { "RegularSettings": {...} } |
| Registry key | Software\Policies\PowerShellCore                          | Software\PowerShellCore                                    |

### Policy settings Setting Fall-Back

#### Motivation - Policy Setting Fall-Back

This is to allow Fall-back to Windows PowerShell policies.

#### Implementation

For Policy Settings,
each policy should have a `Use Windows PowerShell Policy` which will indicate that the policy should the read from
`SOFTWARE\Policies\Microsoft\Windows\PowerShell` instead of `Software\Policies\PowerShellCore`.

### Precedence of applying settings

Because a configuration setting can be in several schemes, the setting wins according to the priority of its scheme.

#### Precedence for Policy settings in descending order

| Scheme                      | Windows                                              | Unix                                                 |
|-----------------------------|------------------------------------------------------|------------------------------------------------------|
| GPO -> Computer Policy      | HKLM\Software\Policies\PowerShellCore                | /etc/powershell.config.json                          |
| GPO -> User Policy          | HKCU\Software\Policies\PowerShellCore                | See `Comment A` below                                |
| File -> Computer-Wide       | %ProgramFiles%/PowerShell/powershell.config.json     | /opt/Microsoft/powershell/powershell.config.json     |
| File -> Application-Startup | pwsh -settingsfile `somepath/powershell.config.json` | pwsh -settingsfile `somepath/powershell.config.json` |
| File -> User-Wide           | %APPDATA%/powershell.config.json                     | %XDG_CONFIG_HOME%/powershell.config.json             |
| File -> Application-Wide    | $apphome/powershell.config.json                      | $apphome/powershell.config.json                      |

Defaults:

`%APPDATA%` - `C:\Users\useraccount\AppData\Roaming`

`%XDG_CONFIG_HOME%` - `HOME/.config`

#### Parameter `-settingsfile`

With `-settingsfile` parameter users can assign custom settings from the config file and overwrite user-wide and application-wide settings.

##### Computer-wide and user policy settings

Admin/root users can overwrite computer-wide and user policy settings using `-settingsfile`,
only when not in System Lock-down mode.

This will have performance impact on startup, but only when `-settingsfile` is specified.

#### Priorities for Regular settings in descending order

| Scheme                      | Windows                                              | Unix                                                 |
|-----------------------------|------------------------------------------------------|------------------------------------------------------|
| File -> Application-Startup | pwsh -settingsfile `somepath/powershell.config.json` | pwsh -settingsfile `somepath/powershell.config.json` |
| File -> Application-Wide    | $apphome\powershell.config.json                      | $apphome/powershell.config.json                      |
| File -> User-Wide           | %APPDATA%\powershell.config.json                     | ~/powershell.config.json                             |
| File -> Computer-Wide       | %ProgramFiles%\PowerShell\powershell.config.json     | /opt/Microsoft/powershell/powershell.config.json     |
| GPO -> User Config          | HKCU\Software\PowerShellCore                         | See `Comment A` below                                |
| GPO -> Computer Config      | HKLM\Software\PowerShellCore                         | /etc/powershell.config.json                          |

### Configuration settings

A set of configuration settings in GPO scheme and file scheme for policy settings and regular settings is the same. This allows to discover and configure settings in the simplest and fastest way.

#### Registry keys and settings

| Key                              | SubKey                      | Option                             | Type   | Precedence          |
|----------------------------------|-----------------------------|------------------------------------|--------|---------------------|
| Software\Policies\PowerShellCore | -                           | -                                  |        |                     |
| Software\PowerShellCore          | -                           | -                                  |        |                     |
|                                  |                             | ExecutionPolicy                    | String | Computer, Then User |
|                                  | ConsoleSessionConfiguration | EnableConsoleSessionConfiguration  | DWORD  | User, then Computer |
|                                  | ConsoleSessionConfiguration | ConsoleSessionConfigurationName    | String | User, then Computer |
|                                  | ModuleLogging               | EnableModuleLogging                | DWORD  | Computer, Then User |
|                                  | ModuleLogging               | ModuleNames                        | String | Computer, Then User |
|                                  | ProtectedEventLogging       | EncryptionCertificate              | DWORD  | Computer Wide       |
|                                  | ScriptBlockLogging          | EnableScriptBlockInvocationLogging | DWORD  | Computer, Then User |
|                                  | ScriptBlockLogging          | EnableScriptBlockLogging           | DWORD  | Computer, Then User |
|                                  | Transcription               | EnableTranscripting                | DWORD  | Computer, Then User |
|                                  | Transcription               | EnableInvocationHeader             | DWORD  | Computer, Then User |
|                                  | Transcription               | OutputDirectory                    | String | Computer, Then User |
|                                  | UpdatableHelp               | DefaultSourcePath                  | String | Computer Wide       |

I filed [#9632](https://github.com/PowerShell/PowerShell/issues/9632) on UpdatableHelp-DefaultSourcePath to make it allow User settings.

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
  },

  "OtherPowerShellApplication": {
    "RegularSettings": {
      ...
    },
    "PolicySettings": {
      ...
    }
}
```

## Alternate Proposals and Considerations

### Automatically resolve Windows PowerShell policy conflicts

We could attempt to resolve policy conflicts between PowerShell 7 policy and Windows PowerShell policy.
This would make the `Precedence for Policy settings` not just a simple list but a complex set of rules that would not be easily understood.  See [this conversation](https://github.com/PowerShell/PowerShell/issues/9309?#issuecomment-480643922).

### Allowing environment variable in the JSON

A new RFC should be drafted about how to allow environment variables in the JSON.
This would allow consistent files across platforms.

### Comment A

Mainly for Unix we'd add `Users` section to computer wide JSON file (`/etc/powershell.config.json`) to allow administrators set policies and regular settings on user level base

```json
{
  "PowerShell": {
    "RegularSettings": {
      ...
    },
    "PolicySettings": {
      ...
    },
  "Users": {
    "Smith": {
      "PowerShell": {
        "RegularSettings": {
          ...
        },
        "PolicySettings": {
          ...
        }
      }
  }
}
```