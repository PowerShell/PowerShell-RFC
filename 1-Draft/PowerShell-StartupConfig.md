---
RFC: 
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

The file PowerShell.Config.psd1 shall contain the configuration to be used at startup, the format of which is a PowerShell data file.
This file would be read-only, e.g., current tools which currently write to these locations in the registry would return an error if used.
Also, changes would be read only at startup time, so changes made to the file would not affect any running sessions.
If the file $PSHOME/PowerShell.Config.psd1 is found, the configuration therein will be read and applied to configurable elements within the PowerShell engine.

Additionally, a PowerShell.Config.psd1 file may be placed in the users $HOME directory.
If a PowerShell.Config.psd1 file is found in the users $HOME directory, all configuration will be applied from that file in addition to that found in the file in $PSHOME.
The precedence of the config files if found in both $HOME and $PSHOME shall be that the file $PSHOME/PowerShell.Config.psd1 will override any settings in $HOME/PowerShell.Config.psd1.

Initially, the file would allow for only specific first level keys:

* Version
    * The minimum version to which the configuration applies, if the version is less than specified, the configuration is ignored
* InternalTestHooks
    * This allows test configuration to be set before the PowerShell engine has started
* WSMAN
    * This allows for configuration of WSMAN remoting
* PowerShell
    * This allows for configuration of ExecutionPolicy, ConsolePrompting, DisablePromptForUpdateHelp

The configuration file can be provided based on the platform.

Keys which are not supported would generate a _warning_ and be ignored. 

Support for additional keys can be added when needed.

The configuration file in $PSHOME would be applicable only to the PowerShell executable found there.   
The configuration file in $HOME would be applicable to _all_ versions of PowerShell Core.

### Additional Possibilities
* A global PowerShell.Config.psd1 file could be placed in a common location (such as /etc or %ALLUSERSPROFILE%) which could be applicable to all PowerShell sessions, but that is not in scope for this proposal.
* Tools to ease the creation/alteration/removal of settings could be created at a future date.

### Sample configuration file
```powershell
@{
    Version = "6.0.0-alpha"
    InternalTestHooks = @{
        LogPipelineCommandToFile = $true
        PipelineLogFile = "C:/tmp/command.log"
        }
    PowerShell = @{
        ShellId = @{ 
            "Microsoft.PowerShell" = @{
                ExecutionPolicy = "Unrestricted"
                }
            "ScriptedDiagnostics"  = @{
                ExecutionPolicy = "AllSigned"
            }
        }
    WSMANConfig = @{
        PlugIn = @{ 
            "Microsoft.PowerShell.Workflow" = @"
            <PlugInConfiguration xmlns="http://schemas.microsoft.com/wbem/wsman/1/config/PluginConfiguration" Name="microsoft.powershell.workflow" Filename="%windir%\system32\pwrshplugin.dll" SDKVersion="2" XmlRenderingType="text" UseSharedProcess="true" ProcessIdleTimeoutSec="1209600" RunAsUser="" RunAsPassword="" AutoRestart="false" Enabled="true" >
            <InitializationParameters>
            <Param Name="PSVersion" Value="5.1"/>Param Name="AssemblyName" Value="Microsoft.PowerShell.Workflow.ServiceCore, Version=3.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35, processorArchitecture=MSIL"/>
            <Param Name="PSSessionConfigurationTypeName" Value="Microsoft.PowerShell.Workflow.PSWorkflowSessionConfiguration"/>
            <Param Name="SessionConfigurationData" Value="&lt;SessionConfigurationData&gt;
                &lt;Param Name=&quot;ModulesToImport&quot; Value=&quot;%windir%\system32\windowspowershell\v1.0\Modules\PSWorkflow&quot;/&gt;
                &lt;Param Name=&quot;PrivateData&quot;&gt;
                &lt;PrivateData&gt;
                &lt;Param Name=&quot;enablevalidation&quot; Value=&quot;true&quot; /&gt;
                &lt;/PrivateData&gt;
                &lt;/Param&gt;
                &lt;/SessionConfigurationData&gt;" />
            </InitializationParameters>
            <Resources>
            <Resource ResourceUri="http://schemas.microsoft.com/powershell/microsoft.powershell.workflow" SupportsOptions="true" ExactMatch="true">
            <Security xmlns="http://schemas.microsoft.com/wbem/wsman/1/config/PluginConfiguration" Uri="http://schemas.microsoft.com/powershell/microsoft.powershell.workflow" ExactMatch="true" Sddl="O:NSG:BAD:P(A;;GA;;;BA)(A;;GA;;;RM)S:P(AU;FA;GA;;;WD)(AU;SA;GXGW;;;WD)"/>
            <Capability Type="Shell"/>
            </Resource>
            </Resources>
            <Quotas MaxMemoryPerShellMB="2147483647" MaxIdleTimeoutms="2147483647" MaxConcurrentUsers="2147483647" IdleTimeoutms="7200000" MaxProcessesPerShell="2147483647" MaxConcurrentCommandsPerShell="2147483647" MaxShells="2147483647" MaxShellsPerUser="2147483647"/>
            </PlugInConfiguration>
"@
        "Microsoft.PowerShell" = @"
             <PlugInConfiguration xmlns="http://schemas.microsoft.com/wbem/wsman/1/config/PluginConfiguration" Name="microsoft.powershell" Filename="%windir%\system32\pwrshplugin.dll" SDKVersion="2" XmlRenderingType="text" Enabled="true" >
             <InitializationParameters>
             <Param Name="PSVersion" Value="5.1"/>
             </InitializationParameters>
             <Resources>
             <Resource ResourceUri="http://schemas.microsoft.com/powershell/microsoft.powershell" SupportsOptions="true" ExactMatch="true">
             <Security xmlns="http://schemas.microsoft.com/wbem/wsman/1/config/PluginConfiguration" Uri="http://schemas.microsoft.com/powershell/microsoft.powershell" ExactMatch="true" Sddl="O:NSG:BAD:P(A;;GA;;;BA)(A;;GA;;;IU)(A;;GA;;;RM)S:P(AU;FA;GA;;;WD)(AU;SA;GXGW;;;WD)"/>
             <Capability Type="Shell"/>
             </Resource>
             </Resources>
             <Quotas MaxMemoryPerShellMB="2147483647" MaxIdleTimeoutms="2147483647" MaxConcurrentUsers="2147483647" IdleTimeoutms="7200000" MaxProcessesPerShell="2147483647" MaxConcurrentCommandsPerShell="2147483647" MaxShells="2147483647" MaxShellsPerUser="2147483647"/>
             </PlugInConfiguration>
"@
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
But $HOME configuration file defines it as:
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
The the resultant configuration will be as defined by the file in $PSHOME.

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


## Alternate Proposals and Considerations

### EnvironmentalVariables used to set configuration
This has a number of problems, the foremost is that environment variables may be changed or even deleted by the user.
The current proposal provides for those settings to be unalterable by a user which does not have permission to alter the file contents.

### Datafile Formats
The format for the datafile could take many forms; XML, json, Linux Config, or .ini.
I chose PowerShell Data format assuming that anyone that is configuring PowerShell is familiar with PowerShell constructs, and code is available to convert a PSD1 file to an object before the engine is started.
The config file location is somewhat forced due to PowerShell's side-by-side requirements, so a global location (.e.g. `/etc/`) would greatly complicate the configuration file to cover multiple versions.

