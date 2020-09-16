# Deprecation Behavior in PowerShell

PowerShell has a long history of ensuring that when some behavior is available it will be available in future releases.
However, this may be difficult to do from a practical perspective;

- .NET has an attribute `[Obsolete]` which indicates that a type or member should no longer be used, and may be removed in the future.
- Producers of modules may not want to support historical behavior forever
- Sometimes, serious security issues are discovered in the functionality, and should no longer be used.

With regard to the C# attribute, there are two optional parameters; the first is the reason behind the obsoletion, the second to indicate that use is now an error.
Because c# is compiled, these warnings/errors are generated during the authoring/compilation process.
Since PowerShell provides a number of ways to "directly" use C# types, the user is not provided the opportunity to determine whether a type has the obsolete attribue.
It seems reasonable that the users would like to know when they are using a deprecated API or type.

Current behavior for Cmdlets, Functions, and Parameters is as follows:

If the cmdlet has the obsolete attribute applied, the behavior when run is:

```powershell
PS> function Invoke-Obsolete1 {
>> [ObsoleteAttribute("Invoke-Obsolete1 is no longer supported")]
>> param ( )
>> }
PS> Invoke-Obsolete1
WARNING: The command 'Invoke-Obsolete1' is obsolete. Invoke-Obsolete1 is no longer supported

PS> function Invoke-Obsolete2 {
>> param (
>> [ObsoleteAttribute("Use parameter2")]
>> [string]$parameter1,
>> [string]$parameter2
>> )
>> }
PS> Invoke-Obsolete2 -parameter1 value1
WARNING: Parameter 'parameter1' is obsolete. Use parameter2

PS> function Invoke-Obsolete3 {
>> [ObsoleteAttribute("Use a different cmdlet")]
>> param (
>> [ObsoleteAttribute("Use parameter2")]
>> [string]$parameter1,
>> [string]$parameter2
>> )
>> }
PS> Invoke-Obsolete3 -parameter1 value1
WARNING: The command 'Invoke-Obsolete3' is obsolete. Use a different cmdlet
WARNING: Parameter 'parameter1' is obsolete. Use parameter2
```

This behavior is not available for types

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>netcoreapp3.1</TargetFramework>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="System.Management.Automation" Version="7.0.3" />
  </ItemGroup>
</Project>
```

```cs
using System;
using System.Management.Automation;

namespace p1
{
    public class Class1
    {
        public PowerShellStreamType streamtype;
    }
}
```

```powershell
PS> dotnet build
Microsoft (R) Build Engine version 16.8.0-preview-20414-02+a55ce4fbb for .NET
Copyright (C) Microsoft Corporation. All rights reserved.

  Determining projects to restore...
  All projects are up-to-date for restore.
  You are using a preview version of .NET. See: https://aka.ms/dotnet-core-preview
/Users/james/src/projects/obsolete/p1/Class1.cs(8,16): error CS0619: 'PowerShellStreamType' is obsolete: 'This enum type was used only in PowerShell Workflow and is now obsolete.' [/Users/james/src/projects/obsolete/p1/p1.csproj]

Build FAILED.

/Users/james/src/projects/obsolete/p1/Class1.cs(8,16): error CS0619: 'PowerShellStreamType' is obsolete: 'This enum type was used only in PowerShell Workflow and is now obsolete.' [/Users/james/src/projects/obsolete/p1/p1.csproj]
    0 Warning(s)
    1 Error(s)

Time Elapsed 00:00:01.07
```

However, I can use this type without any errors from PowerShell without warning or error

```powershell
 PS> [System.Management.Automation.PowerShellStreamType]::Information.Value__
7
```

Because PowerShell does not provide the obsolete information about the type it is possible to perpetuate obsolete types in script where this would not be possible if the code were compiled.

## Commands

There is currently no way to override the warning messages which occur when an obsolete cmdlet or parameter is used.
There is currently no way to provide warning messages when creating an instance of a type which has been marked obsolete.
There is currently no way to provide warning messages when reference a member of a type which has been marked obsolete.
There is currently no easy way to retrieve the cmdlets, parameters, types, and type members which have been marked obsolete.

### Discovering deprecated or obsolete cmdlets, functions, and types

It is possible to discover whether a cmdlet or function has been marked obsolete

### Displaying deprecation behavior

### Setting deprecation behavior

## Multiple areas for marking behavior deprecated

Beause deprecated code may be desired in both .NET and scripting, the attribute should have consistent behavior across all of the potential uses

Three types of behavior shall be supported.

- None
  - When this value is used, no action shall be taking essentially nullifying any output

- Warning
  - When this value is used, a warning shall be emitted each time the designated behavior occurs

- Error
  - When this value is used, a runtime error shall be generated and the currently executing pipeline shall stop

## Deprecation behavior for cmdlets and functions

## Deprecation for .NET types

## Additions to the module manifest

A new field in the `PrivateData` block shall be available.
The field name shall be "DeprecatedElements" is a hashtable of
@{
    Commands = @()
    Types = @()
}

If there are no deprecated types, then `Types` may be emitted
If there are no deprecated commands (cmdlets, functions, or scripts), then `Commands` may be omitted
The actual `obsoleteattribute` must still be present in the command or type.

## Configuration Tools

There are a number of elements that require configuration for managing the behavior when an element is marked for deprecation.
Because of this, we shall provide cmdlets to aid with the creation of the configuration.
Three cmdlets, `Get-DeprecatedItem`, `Enable-DeprecatedItem`, and `Disable-DeprecatedItem` shall be provided.

- `Get-DeprecatedItem`
  This cmdlet retrieves those items which have been attributed as deprecated and their current configuration.
  By default this will return only cmdlets whose items which are currently in loaded modules
  `-All` shall check all available modules and inspect the PrivateData element for obsolete items and report them, if found.
  `-Force` shall inspect all loaded types and members in PowerShell assemblies for the obsolete attribute and report them, if found. Note that this may take some time and should be used rarely.

- `Set-DeprecationBehavior`
  This cmdlet enables an item which

## DeprecationActionPreference

This is a new ubiquitous variable which dictates the behavior PowerShell shall present when an item has been deprecated.
This has four possible values:

- SilentlyContinue
  - Warning messages will not be shown
- Continue
  - Warning messages will be shown
- Stop
  - Warning messages will be converted to Error messages
- NotAvailable
  - Any deprecated item will be not found.
   This means that cmdlets will produce a CommandNotFound error and types (or their deprecated members) will produce a runtime error.

## Configuration for obsoletion

when specifying a command which will be shown as obsolete, the following information may be provided:

- The command name
- The module where the command resides
- The version of module which contains the obsolete command

The module and version are optional. If the module information is not provided, then the command _whereever it may be found_ will be marked as obsolete.

The follow json schema represents the configuration for configuring deprecations.
The file is read at startup time and not reread at anytime during the session

## Additional Considerations

A rule for PowerShell script analyzer should be created to inspect for obsolete cmdlets and types.

from an implementation, different mechanisms must be used to determine whether

## Open Questions

deprecation vs obsoletion (I've sort of settled on deprecated/deprecation)

- cmdlets
  - get-deprecateditem returns
    - types marked with obsolete
    - types which have members which are marked obsolete
    - cmdlet/function marked as obsolete
    - cmdlet/function which have parameters marked as obsolete

output:

```powershell
PS> Get-DeprecatedItem

```
