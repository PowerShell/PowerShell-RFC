---
RFC: RFC0015
Author: James Truher
Status: Draft
Version: 0.1
Area: Startup, Initial Configuration
Comments Due: 1/1/2017
---

# Startup Configuration Settings for PowerShell Core

Full PowerShell uses the registry to store configuration data which controls many different behaviors when PowerShell starts.
A number of different locations in the registry are used to store config which makes it difficult to manage and retrieve configuration.
This is obviously a problem for non-Windows systems which require system wide configuration, but the registry is not present.

## Motivation

* As a user, I would like to be sure that I do not have to set additional configuration (such as execution policy) when my shell starts.
* As an admin, I would like to be able to control the execution policy and other settings of the shell as it starts.
* As a tester, I often need to have configuration set _before_ the PowerShell engine is running to enable certain test behaviors.

The registry exists only on Windows, non-Windows platforms still need a mechanism for applying system configuration.
Additionally, since PowerShell Core may have multiple installations on a single system, configuration must be specific to the version installed.

## Specification

The file $PSHOME/PowerShell.Config.psd1 shall contain the configuration to be used at startup, the format of which is a PowerShell data file.
If the file $PSHOME/PowerShell.Config.psd1 is found, the configuration therein will be read and applied to configurable elements within the PowerShell engine.
This file shall be read-only, e.g., current tools which currently write to these locations in the registry will return an error if used.
Also, changes shall be read only at startup time, changes made to the file will not affect any running sessions.

Additionally, a PowerShell._version_.Config.psd1 file may be placed in the users $HOME directory.
If a PowerShell._version_.Config.psd1 file is found in the users $HOME directory corresponding to the version of the PowerShell executable, all configuration will be applied from that file in addition to that found in the file in $PSHOME.
The precedence of the config files if found in both $HOME and $PSHOME shall be that the file $HOME/PowerShell._version_.Config.psd1 will override any settings in $PSHOME/PowerShell.Config.psd1.

Initially, the file shall allow for the following first level keys:

* InternalTestHooks
    * This allows test configuration to be set before the PowerShell engine has started
* PowerShell
    * This allows for configuration of ExecutionPolicy, ConsolePrompting, DisablePromptForUpdateHelp, PSModulePath

Additional keys and subkeys may be added as needed.
Keys which are not supported shall generate a _warning_ and be ignored.

The configuration file in $PSHOME shall be applicable only to the PowerShell executable found therein.
The configuration file in $HOME (PowerShell._version_.Config.psd) shall be applicable to only to the matching version of PowerShell Core.
Additionally, the parameter `-ConfigurationFile` shall be present in the PowerShell executable, which takes an array of strings which represent paths to config files.
When the `-ConfigurationFile` parameter is present, those files only will be read for configuration data.

The config file should have a published schema to aid in file creation.

### Additional Possibilities
* A global PowerShell.Config.psd1 file could be placed in a common location (such as /etc or %ALLUSERSPROFILE%) which could be applicable to all PowerShell sessions, but that is not in scope for this proposal.
* Tools to ease the creation/alteration/removal of settings should be created at a future date.

### Additional Optional Work
We should have a mechanism for creating default config files, we could easily modify existing code for New-PSSessionConfiguraitonFile to do this, however this is not a requirement.

### Sample configuration file
```powershell
@{
    InternalTestHooks = @{
        LogPipelineCommandToFile = $true
        PipelineLogFile = "C:/tmp/command.log"
        }
    PowerShell = @{
        ShellId = @{ 
            "Microsoft.PowerShell" = @{
                ExecutionPolicy = "Unrestricted"
                PSModulePath = @{
                    Windows = "C:/PowerShellInstalls/6.0.0/Modules"
                    Ubuntu  = "/PowerShellInstalls/6.0.0/Modules"
                    MacOS   = "/PowerShellInstalls/6.0.0/Modules"
                    }
                }
            "ScriptedDiagnostics"  = @{
                ExecutionPolicy = "AllSigned"
            }
        }
    }
}
```

### Precedence

If both $PSHOME configuration file defines PowerShell as follows:
```powershell
    PowerShell = @{
        ShellId = @{ 
            "Microsoft.PowerShell" = @{
                ExecutionPolicy = "Restricted"
                }
            "ScriptedDiagnostics"  = @{
                ExecutionPolicy = "AllSigned"
            }
        }
```
And $HOME configuration file defines it as:
```powershell
    PowerShell = @{
        ShellId = @{ 
            "Microsoft.PowerShell" = @{
                ExecutionPolicy = "Unrestricted"
                }
            "ScriptedDiagnostics"  = @{
                ExecutionPolicy = "Unrestricted"
            }
        }
```
The the resultant configuration will be as defined by the file in $HOME.

However, if the configuration does not overlap then the settings will not be overridden:

```powershell
$PSHOME
    PowerShell = @{
        ShellId = @{ 
            "Microsoft.PowerShell" = @{
                ExecutionPolicy = "Restricted"
                }
        }

$HOME
    PowerShell = @{
            "ScriptedDiagnostics"  = @{
                ExecutionPolicy = "Unrestricted"
            }
        }
```
The resultant configuration would be:
```powershell
    PowerShell = @{
        ShellId = @{ 
            "Microsoft.PowerShell" = @{
                ExecutionPolicy = "Restricted"
            "ScriptedDiagnostics"  = @{
                ExecutionPolicy = "Unrestricted"
            }
        }
```


## Out of Scope
This specification is applicable only to startup preferences and does not apply to setting Policy.
Setting Policy behavior, such as found via GroupPolicy, may be addressed in a later specification, if desired.

## Alternate Proposals and Considerations

### EnvironmentalVariables used to set configuration
This has a number of problems, the foremost is that environment variables may be changed or even deleted by the user.
The current proposal provides for those settings to be unalterable by a user which does not have permission to alter the file contents.

### Datafile Formats
The format for the datafile could take many forms; XML, json, Linux Config, .yml, or .ini.
I chose PowerShell Data format assuming that anyone that is configuring PowerShell is familiar with PowerShell constructs, and code is available to convert a PSD1 file to an object before the engine is started.
Also, PSD1 files may conin comments, which may be very helpful in describing the settings.
The config file location is somewhat forced due to PowerShell's side-by-side requirements, so a global location (.e.g., `/etc/`) would greatly complicate the configuration file to cover multiple versions.

## PowerShell Committee Decision

### Voting Results

Jason Shirk: Absent 

Joey Aiello: Reject

Bruce Payette: Reject

Steve Lee: Reject

Hemant Mahawar: Reject

### Majority Decision

Committee decided to reuse an existing json file that is already parsed at startup and to move the telemetry configuration into that file.
Although the current implementation does pull in the json parser, it doesn't appear to have a significant impact on startup as it's been in code since PSCore6 was public.

### Minority Decision

N/A

