---
RFC: RFCxxxx
Author: Steve Lee
Status: Draft
SupercededBy: N/A
Version: 1.0
Area: Desired State Configuration
Comments Due: Nov 22, 2020
Plan to implement: Yes
---

# DSC for PowerShell 7

[Desired State Configuration](https://docs.microsoft.com/powershell/scripting/dsc/overview/overview)
(DSC) is a platform in PowerShell enabling configuration as code.

## Background

DSC contains many components including:

- The `configuration` keyword that is part of the PowerShell engine
- Compilation to a configuration MOF file
- DSC resources (those that ship in Windows and those on PowerShellGallery.com)
- Local Configuration Manager (LCM) in Windows and maintained as open-source for Linux,
  a service that maintains and/or reports on configuration drift
- the DSC Pull Server, which is used by LCM's configured in pull mode to acquire new
  resources and configurations
- the `PSDesiredStateConfiguration` module that includes cmdlets like `Get-DscResource`

DSC ships as a component of Windows 10 and as an open source project for Linux.
It is also integrated into cloud-based server management services in multiple
public clouds including Microsoft Azure and Amazon Web Services (AWS).

The versions of DSC that ship in Windows PowerShell are tightly coupled with
[Windows Management Instrumentation (WMI)](https://docs.microsoft.com/windows/win32/wmisdk/wmi-start-page)
on Windows and
[Open Management Interface](https://www.dmtf.org/content/open-management-interface-omi-%E2%80%93-open-source-implementation-dmtf-cimwbem-standards)
(OMI) on Linux.
The *MI libraries are used to host the Local Configuration Manager (LCM) and APIs
are used to parse information store in Managed Object Framework (MOF) file format,
which is used for both configuration files and schema files.

### Challenges with the current versions of DSC

Very few community authored DSC resources exist for Linux because C++ is the only
supported language. Other than using the Script/nxScript resources combined with
a composite resource, it is near-impossible today to create a DSC solution that is cross-platform.

Today, third party solutions leveraging DSC resources need to call the LCM
to run Get/Set/Test, making it complicated to integrate with partner ecosystems.

## Scope of DSC implementation for PowerShell 7

Unlike DSC for Windows PowerShell, DSC for PowerShell 7 is only intended to provide
a platform for building or integrating with configuration solutions. For PowerShell 7,
the scope of DSC support is limited to the `configuration` keyword, authoring,
compilation, and using DSC resources via the PSDesiredStateConfiguration module.

This means that a default agent (LCM) is not included and existing pull servers are not supported.

### Motivation

DSC in PowerShell 7 should enable the following user scenarios:

    As a developer of configuration management solutions,
    I can easily integrate DSC resources
    so that my solution can leverage the PowerShell ecosystem and its users and contributors.

    As an IT Pro,
    I can directly invoke DSC resources
    so that I can build ad hoc cross platform configurations.

### Cross platform first

DSC has supported both Windows and Linux, however, the
[Linux support](https://docs.microsoft.com/powershell/scripting/dsc/getting-started/lnxGettingStarted)
was dependent on a Microsoft-maintained implementation of
[OMI](https://github.com/Microsoft/omi)
and did not encourage authoring Linux DSC resources in PowerShell,
making development inconsistent and difficult for non-expert users.

DSC for PowerShell 7 will prioritize being cross platform and having a consistent
user experience and capabilities on all supported platforms.

### Invoke-DscResource

About a year ago, we worked with
[Gael Colas](https://twitter.com/gaelcolas)
to author an RFC describing fundamental changes to
[`Invoke-DscResource`](https://docs.microsoft.com/powershell/module/psdesiredstateconfiguration/invoke-dscresource)
in PowerShell 7.0.
This enabled executing the Get, Set, and Test methods on DSC resources bypassing
the LCM on Windows and removing the need for OMI on Linux. This was the start
to enable cross platform DSC resources to be authored in PowerShell script (with
some limitations).

The changes to DSC shipped as an
[experimental feature](https://docs.microsoft.com/powershell/scripting/learn/experimental-features)
in PowerShell 7.0.

### Bring your own agent (BYOA)

A significant change from existing DSC for Windows PowerShell is there will be no
default agent (LCM) shipped with PowerShell 7. Instead, we want to make it easier
to integrate with existing configuration agents like
[Ansible](https://docs.ansible.com/ansible/latest/user_guide/windows_dsc.html),
[Chef](https://docs.chef.io/resources/dsc_resource/),
[Puppet](https://forge.puppet.com/puppetlabs/dsc),
[Azure Policy Guest Configuration](https://docs.microsoft.com/azure/governance/policy/concepts/guest-configuration),
and other popular configuration solutions.

Custom agents can use the existing PowerShell API (or call out to `pwsh`) to
directly invoke DSC resources. For ad hoc configuration management, solutions can
even use a PowerShell script run by TaskScheduler or cron.

Along with the lack of a default agent, there is no pre-defined push or pull
model for deploying DSC.

### Move away from WMI/OMI/MOF

DSC on Windows was originally designed relying on
[WMI](https://docs.microsoft.com/windows/win32/wmisdk/wmi-start-page)
as a host on Windows,
[OMI](https://www.dmtf.org/content/open-management-interface-omi-%E2%80%93-open-source-implementation-dmtf-cimwbem-standards)
as a host on Linux, and
[MOF](https://docs.microsoft.com/windows/win32/wmisdk/managed-object-format--mof-)
for configuration and schema.

MOF as a format is not something most developers are familiar with and there is no
tooling support. However, developers of DSC resources written in PowerShell script
still needed to learn MOF and also keep their MOF schema in sync with their script
implementation.

Today, when you execute a
[DSC configuration script](https://docs.microsoft.com/powershell/scripting/dsc/configurations/configurations),
the output is a MOF configuration file. In general, users do not need to read this
MOF file as it's intended to be used by machines. However, MOF parsing creates a
dependency on WMI/OMI. Instead, we will move towards a JSON configuration file format.

Since there is no default agent to understand the JSON configuration file, it will
be up to users or 3rd party agents to understand the configuration file.

Moving from WMI/OMI also means no longer supporting C++ resources nor Python based resources.

Example configuration file:

```powershell
Configuration FileResourceDemo
{
    Node "localhost"
    {
        File DirectoryCopy
        {
            Ensure = "Present" # Ensure the directory is Present on the target node.
            Type = "Directory" # The default is File.
            Recurse = $true # Recursively copy all subdirectories.
            SourcePath = "\\PullServer\DemoSource"
            DestinationPath = "C:\Users\Public\Documents\DSCDemo\DemoDestination"
        }

        Log AfterDirectoryCopy
        {
            # The message below gets written to the Microsoft-Windows-Desired State Configuration/Analytic log
            Message = "Finished running the file resource with ID DirectoryCopy"
            DependsOn = "[File]DirectoryCopy" # Depends on successful execution of the File resource.
        }
    }
}
```

Generated MOF:

```mof
/*
@TargetNode='localhost'
@GeneratedBy=slee
@GenerationDate=11/05/2020 11:09:47
@GenerationHost=SLEE-DESKTOP
*/

instance of MSFT_FileDirectoryConfiguration as $MSFT_FileDirectoryConfiguration1ref
{
  SourceInfo = "C:\\Users\\slee\\test\\file.dsc.ps1::6::9::File";
  DestinationPath = "C:\\Users\\Public\\Documents\\DSCDemo\\DemoDestination";
  SourcePath = "\\\\PullServer\\DemoSource";
  Recurse = True;
  Ensure = "Present";
  Type = "Directory";
  ModuleName = "PSDesiredStateConfiguration";
  ResourceID = "[File]DirectoryCopy";
  ModuleVersion = "1.0";
  ConfigurationName = "FileResourceDemo";
};

instance of MSFT_LogResource as $MSFT_LogResource1ref
{
  SourceInfo = "C:\\Users\\slee\\test\\file.dsc.ps1::15::9::Log";
  ModuleName = "PsDesiredStateConfiguration";
  Message = "Finished running the file resource with ID DirectoryCopy";
  ResourceID = "[Log]AfterDirectoryCopy";
  ModuleVersion = "1.0";
  DependsOn = {
    "[File]DirectoryCopy"
  };
  ConfigurationName = "FileResourceDemo";
};

instance of OMI_ConfigurationDocument
{
  Version="2.0.0";
  MinimumCompatibleVersion = "1.0.0";
  CompatibleVersionAdditionalProperties= {"Omi_BaseResource:ConfigurationName"};
  Author="slee";
  GenerationDate="11/05/2020 11:09:47";
  GenerationHost="SLEE-DESKTOP";
  Name="FileResourceDemo";
};
```

Example JSON representation of generated configuration:

```json
{
  "ConfigurationDocument": {
    "GenerationDate": "11/05/2020 11:09:47",
    "MinimumCompatibleVersion": "1.0.0",
    "CompatibleVersionAdditionalProperties": [
      "BaseResource:ConfigurationName"
    ],
    "Name": "FileResourceDemo",
    "Version": "2.0.0",
    "Author": "slee",
    "GenerationHost": "SLEE-DESKTOP"
  },
  "LogResource": [
    {
      "SourceInfo": "C:\\\\Users\\\\slee\\\\test\\\\file.dsc.ps1::15::9::Log",
      "ModuleVersion": "1.0",
      "Message": "Finished running the file resource with ID DirectoryCopy",
      "ResourceID": "[Log]AfterDirectoryCopy",
      "ModuleName": "PsDesiredStateConfiguration",
      "ConfigurationName": "FileResourceDemo",
      "DependsOn": [
        "[File]DirectoryCopy"
      ]
    }
  ],
  "FileDirectoryConfiguration": [
    {
      "ResourceID": "[File]DirectoryCopy",
      "Ensure": "Present",
      "Recurse": true,
      "ModuleVersion": "1.0",
      "DestinationPath": "C:\\\\Users\\\\Public\\\\Documents\\\\DSCDemo\\\\DemoDestination",
      "ConfigurationName": "FileResourceDemo",
      "SourceInfo": "C:\\\\Users\\\\slee\\\\test\\\\file.dsc.ps1::6::9::File",
      "ModuleName": "PSDesiredStateConfiguration",
      "Type": "Directory",
      "SourcePath": "\\\\\\\\PullServer\\\\DemoSource"
    }
  ]
}
```

Important differences:

- CIM class name prefixes are removed as they are not needed
- Instances are represented as an array of the type name
- The comment at the start is dropped as that information is available within `ConfigurationDocument`

### No longer require MOF schema by switching to improved class-based resources

Although DSC resources can be written as WMI/OMI providers, this requires writing
C/C++ code and compiling for specific Linux distros and processor architectures.
Writing PowerShell script DSC resources currently still requires a matching schema
file making it more complex. Instead, the current plan is to
[only support PowerShell class-based DSC resources](https://github.com/PowerShell/PowerShell/issues/13731)
in DSC for PowerShell 7. Whereas previously embedded objects were CIMInstance types,
they will now be .NET types based on PowerShell classes. Additional reasoning
and discussion is in the issue linked above.

Making the change from script-based resources to class-based resources could make
learning DSC a frustrating experience for new users. To resolve this, tooling will
be needed that scaffolds as much of the class requirements as possible based
on a .psm1 module file containing traditionally authored Get/Set/Test-TargetResource
functions. This would also aid in migrating the more than 1000 existing resources
for Windows.

### Open Source

The current
[PSDesiredStateConfiguration](https://docs.microsoft.com/powershell/module/psdesiredstateconfiguration/)
module is not Open Source. The intent is to clean up the code and prepare the GitHub
repository to be made public as an Open Source project.
However, DSC for PowerShell 7 does not depend on PSDesiredStateConfiguration being Open Source.

### PowerShell engine changes

To enable DSC for PowerShell 7 to evolve independently to PowerShell 7 itself, the
code in the engine that handles the DSC configuration keyword will be separated
as a plugin and made to be part of the PSDesiredStateConfiguration module.

## Alternate Proposals and Considerations

### Supporting existing script based resources

Existing script-based resources (not class-based) require a MOF schema file for
PowerShell to understand the types and nested types. We had prototyped using a JSON
based schema instead of the existing MOF schema file but it still required the DSC
resource author to create a schema file and ensure it aligns with the script implementation.

It is simpler to create a PowerShell class based resource which itself is the schema
removing the need to understand and maintain a separate schema file. In addition,
having a single model (class based) for authoring DSC resources will help the community
in the long term by having a single consistent way to write DSC resources, making
it easier for new authors to find
[documentation and examples](https://dsccommunity.org/blog/class-based-dsc-resources/).

### Tooling for parsing JSON configuration

To help developers of agents, we can provide APIs to parse the configuration
JSON file into a dependency graph as this would be required by every agent to
properly apply a configuration.
