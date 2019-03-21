---
RFC:  RFCnnnn
Author:  Ilya Sazonov
Status:  Draft
SupercededBy:  N/A
Version: 0.1
Area: Engine
Comments Due: 2/25/2018
---

# `PowerShell Core` configuration

## Motivation

Consumers, developers, and enterprise system administrators should be able to flexibly and conveniently configure PowerShell Core and applications based on it.

A public PowerShell Core configuration API would allow a unified way to manage PowerShell-based applications in a consistent manner.

## Specification

`PowerShell Core` is configured using the following schemes:

- On Windows - Group Policy Objects (GPO), Group Policy Preferences (GPP) and settings files.
- On Unix - settings files.

The settings files have `JSON` format.

**Warning** The settings files differ from `PowerShell Core` _profile_ files, which are PowerShell scripts run at startup.

Configuration schemes allow to customize `PowerShell Core` in the most flexible way:

- Enterprise system administrators can use GPO, GPP and computer-wide settings files to apply approved configuration settings and mandatory security settings in a centralized manner. The same settings can be applied at user, application or startup levels.
- Developers and consumers can use user, application and startup level settings files.

### Configuration defaults

PowerShell Core has hardcoded defaults for all configuration options.

The default values must be `secure-by-default`.

For release versions hardcoded defaults must be the same as ones in re-installed configuration files. For preview versions they may vary (ex., enable experimental features and so on).

If during startup PowerShell Core cannot read configuration files it uses _hardcoded_ defaults.

If during operation PowerShell Core cannot read configuration files it continue to use _current_ (runtime) configuration values.

### Settings locations

`PowerShell Core` settings are grouped into `Policy settings` and `Regular settings`. Regular settings are normal configuration settings. Regular settings can be treated as default values. Policy settings is high priority and overlap regular settings. Policy settings are used by administrators to centrally manage applications.

Location | Policy settings | Regular settings
-| - | -
File section | "PowerShell": { "PolicySettings": {...} } | "PowerShell": { "RegularSettings": {...} }
File section | "OtherPowerShellApplication": { "PolicySettings": {...} } | "OtherPowerShellApplication": { "RegularSettings": {...} }
Registry key | Software\Policies\PowerShellCore | Software\PowerShellCore
Registry key | Software\Policies\PowerShellCore\OtherPowerShellApplication | Software\PowerShellCore\OtherPowerShellApplication

### Priority of applying settings

Because a configuration setting can be in several schemes, the setting wins according to the priority of its scheme.

#### Priorities for Policy settings in descending order

Scheme | Windows | Unix
-| - | -
GPO -> Computer Policy | HKLM\Software\Policies\PowerShellCore | /etc/powershell.config.json
GPO -> User Policy | HKCU\Software\Policies\PowerShellCore | See `Comment A` below
File -> Computer-Wide | %ProgramFiles%/PowerShell/powershell.config.json | /opt/Microsoft/powershell/powershell.config.json
File -> Application-Startup | pwsh -settingsfile `somepath/powershell.config.json` | pwsh -settingsfile `somepath/powershell.config.json`
File -> User-Wide | %APPDATA%/powershell.config.json | %XDG_CONFIG_HOME%/powershell.config.json
File -> Application-Wide | $apphome/powershell.config.json | $apphome/powershell.config.json

Defaults:

`%APPDATA%` - `C:\Users\useraccount\AppData\Roaming`

`%XDG_CONFIG_HOME%` - `HOME/.config`

#### Parameter `-settingsfile`

With `-settingsfile` parameter users can assign custom settings from the config file and overwrite user-wide and application-wide settings. Only users with elevated rights can overwrite computer-wide and user policy settings.

#### Priorities for Regular settings in descending order

Scheme | Windows | Unix
-| - | -
File -> Application-Startup | pwsh -settingsfile `somepath/powershell.config.json` | pwsh -settingsfile `somepath/powershell.config.json`
File -> Application-Wide | $apphome\powershell.config.json | $apphome/powershell.config.json
File -> User-Wide | %APPDATA%\powershell.config.json | ~/powershell.config.json
File -> Computer-Wide | %ProgramFiles%\PowerShell\powershell.config.json | /opt/Microsoft/powershell/powershell.config.json
GPO -> User Config | HKCU\Software\PowerShellCore | See `Comment A` below
GPO -> Computer Config | HKLM\Software\PowerShellCore | /etc/powershell.config.json

### Configuration settings

A set of configuration settings in GPO scheme and file scheme for policy settings and regular settings is the same. This allows to discover and configure settings in the simplest and fastest way.

#### Registry keys and settings

| Key | SubKey | Option | Type
| -| - | - | -
Software\Policies\PowerShellCore | - | -
Software\PowerShellCore | - | -
| | | ExecutionPolicy | String
| | | PipelineMaxStackSizeMB | DWORD
| | ConsoleSessionConfiguration | EnableConsoleSessionConfiguration | DWORD
| | ConsoleSessionConfiguration | ConsoleSessionConfigurationName | String
| | ModuleLogging | EnableModuleLogging | DWORD
| | ModuleLogging | ModuleNames | String
| | ProtectedEventLogging | EncryptionCertificate | DWORD
| | ScriptBlockLogging | EnableScriptBlockInvocationLogging | DWORD
| | ScriptBlockLogging | EnableScriptBlockLogging | DWORD
| | Transcription | EnableTranscripting | DWORD
| | Transcription | EnableInvocationHeader | DWORD
| | Transcription | OutputDirectory | String
| | UpdatableHelp | DefaultSourcePath | String
|Software\Policies\Microsoft\Windows\EventLog | ProtectedEventLogging | EnableProtectedEventLogging | DWORD

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

We could redesign `-settingsfile` startup parameter. If we'd support configuration sections for applications we'd can introduce `-settings` parameter to specify a particular configuration as `-settings OtherPowerShellApplication`. Ex.: `pwsh -settings Debug`.

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
