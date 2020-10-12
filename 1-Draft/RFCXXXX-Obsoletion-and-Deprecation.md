---
RFC: RFCXXXX
Author: James Truher
Status: Draft
Area: Shell
Comments Due: 11/15/2020
---

# Obsolescence Behavior in PowerShell

PowerShell has a long history of ensuring that when behavior is available in an earlier release it will be available in future releases.
However, this may be difficult to do from a practical perspective;

- Producers of modules may not want to support historical behavior forever
- Sometimes, serious security issues are discovered in the functionality, and should no longer be used.
- .NET has the `Obsolete` attribute which indicates that a type or member should no longer be used, and may be removed in the future.

With regard to the C# attribute, there are two optional parameters; the first is the reason behind the obsolescence, the second to indicate that use is now an error.
Because c# is compiled, these warnings/errors are generated during the authoring/compilation process.
Since PowerShell provides a number of ways to "directly" use C# types, the user is not generally provided the opportunity to easily determine whether a type or member has the obsolete attribute.
While developers are sometimes painfully aware that types or members are obsolete, PowerShell users may be unaware regard to obsolescence.
It seems reasonable that script users would like to know when they are using an obsolete API or type.
With PowerShell there are two aspects of this problem:

- Cmdlets or Parameters may be designated obsolete
- Types and/or members of types may be designated as obsolete

It would be a better experience if the user notification was consistent between these two types of uses.

There are a number of behaviors which would be beneficial to the PowerShell users:

- Overriding the warning messages which occur when an obsolete type, member, cmdlet, or parameter is used.
- Providing warning messages when creating an instance of a type which has been marked obsolete.
- Providing warning messages when reference a member of a type which has been marked obsolete.
- An easy way to retrieve the cmdlets, parameters, types, and type members which have been marked obsolete.

Providing for these behaviors would improve the current experience when interacting with obsolete or deprecated elements.

## Current behaviors

If you're familiar with our current behavior regarding obsolescence, you can skip to [Desired Behavior](#desired-behavior) below.

The following shows the inconsistency between using obsolete cmdlets and types.

### Cmdlets/Functions

The following provides examples of the current behaviors for cmdlets.
It includes examples for both obsolete cmdlet and parameters.

Ex.1 Obsolete cmdlet

```powershell
PS> function Invoke-Obsolete1 {
>> [ObsoleteAttribute("Invoke-Obsolete1 is no longer supported")]
>> param ( )
>> }
PS> Invoke-Obsolete1
WARNING: The command 'Invoke-Obsolete1' is obsolete. Invoke-Obsolete1 is no longer supported
```

Ex.2 Obsolete parameter when the parameter is used

```powershell
PS> function Invoke-Obsolete2 {
>> param (
>> [ObsoleteAttribute("Use parameter2")]
>> [string]$parameter1,
>> [string]$parameter2
>> )
>> }
PS> Invoke-Obsolete2 -parameter1 value1
WARNING: Parameter 'parameter1' is obsolete. Use parameter2
```

Ex.3 Obsolete cmdlet and parameter are both used

```powershell
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

Ex.4 obsolete parameter is not used

```powershell

PS> function Invoke-Obsolete4 {
>> param (
>> [ObsoleteAttribute("Use parameter2")]
>> [string]$parameter1,
>> [string]$parameter2
>> )
>> }
PS> Invoke-Obsolete3 -parameter2 value2
```

Note that the warning is only emitted when the obsolete parameter _is used_.
If you use a parameter which is not marked obsolete, naturally no warning will be issued.

### Interacting with Types

The cmdlet behavior is not available for types, as the following example shows:

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

However, I can use this type from PowerShell without warning or error

```powershell
 PS> $a = [System.Management.Automation.PowerShellStreamType]::Information.Value__
7
```

Because PowerShell does not provide obsolete warnings for types it is possible to more easily perpetuate the use of obsolete types in script where this would not be possible if the code were compiled.

## Desired Behavior

### Cmdlets

The current behavior of cmdlets (script or compiled c#) is that when the cmdlet is _executed_ a warning will be emitted.
If the cmdlet uses a parameter which has been marked obsolete, another warning is emitted.
If the cmdlet is not marked obsolete, but an obsolete parameter is used, a warning is emitted only for the obsolete parameter.
This behavior has informed the behavior surrounding the use of obsolete types/members.
However, It is not easy to determine that a cmdlet has been marked obsolete via `Get-Command` or before use,
`Get-Command` will be extended to include obsolescence information.

### Proposed behavior for types

When an obsolete type or member is used, we will emit a warning:

- when a script is _parsed_
  - This means that when a script is executed or dot-sourced a warning may be emitted
- when a module is imported (a special case of parsing)
- when an obsolete type is _used_ in an interactive session

It will not warn on every reference at run-time as that might result in thousands of warnings if you're using an obsolete type/member in a loop.
This is to attempt to mimic the behavior of the compilation experience which is to warn at compilation, but not at run time.
Our analogous compilation is when the script is parsed, or when a module is loaded, or a function is created.

The following is an example for the suggested behaviors:

```powershell
PS> [obsoleteattribute("Use class2.")]
>> class class1 {
>> [obsoleteattribute("Use property2")]
>> [string]$property1
>> [string]$property2
>> }
PS> $i = [class1]::new()
WARNING: class1 is obsolete. Use class2.
PS> $i.property1 = "new value"
WARNING: property1 is obsolete. Use property2.

```

There is some asymmetry with behavior here, as Cmdlet obsolescence warnings occur when the cmdlet it _used_.
However, most PowerShell users do not notice the difference between our parsing phase and execution phase.

---
**NOTE:**
We should consider altering our current behavior for cmdlets to emit a warning only at parse time rather than run-time.

---

## Displaying and Overriding default behaviors

This section provides proposals on global behaviors as well as overriding these behaviors.

### Displaying obsolete behavior

Currently, all warnings are displayed based on the global preference variable `WarningPreference`.
Additional levels of granularity would be useful so different warnings may be controlled, much the same way that trap/catch works in PowerShell today.
While it is the case that you can use `-WarningAction SilentlyContinue` with cmdlets today to suppress obsolescence warnings, this will affect _all_ warnings.
In some cases, you may not want to suppress obsolete warnings while suppressing operational warnings (and vice-versa).

#### ObsoleteWarningPreference

This is a new ubiquitous variable will dictate the global behavior of PowerShell when an item has been made obsolete.
This has four possible values:

- SilentlyContinue
  - Warning messages will not be shown
- Continue
  - Warning messages will be shown
- NotAvailable
  - Any obsolete item will be not found.
   This means that cmdlets will produce a CommandNotFound error and types (or their obsolete members) will produce a runtime error.
   Additionally, the warning message will not be produced.
- Stop
  - Warning messages will be converted to terminating error messages

---
**NOTE**
A ubiquitous parameter could be added, but this seems less interesting as the user must have _a priori_ knowledge about the obsolete cmdlet, thus diluting its usefulness.
It may be possible to use a ubiquitous parameter via `PSDefaultParameterValues`, but this feels like _additional_ complexity rather than simplification.
Additionally, it would not be applicable to the use of obsolete _types_, further reducing its usefulness.

---

### Overriding Behaviors

It is not enough to provide default behaviors, as There are conditions where the default is not desired, so it must be possible to locally change PowerShell behaviors.
In order to support this scenario an additional cmdlet `Set-ObsoleteWarning` shall be provided.
This allows the user to specify for a specific warning what behavior is to be provided.

`Set-ObsoleteWarning -Id _WarningId_ -Action _PreferenceValue_`

The _WarningId_ may be retrieved with the `Get-ObsoleteItem` cmdlets.

`Get-ObsoleteWarning` will return the current overrides in place.

Enables individual tuning for a specific obsolete warning.

Detailed discussion on these tools is [below](#configuration-tools).

### Discovery

Being able to discover and explore the environment is one of PowerShells' great strengths.
We should provide tools which enable users to discover obsolete cmdlets and types in their environment.
PowerShell currently supports the attribute `ObsoleteAttribute` which should be sufficient.

## Additions to the module manifest

In order to quickly report on whether a module has some element which is obsolete, the module manifest will be extended to support providing a hint to obsolescence.
A new field in the `PrivateData` block shall be available.
The name `ObsoleteHints` is a hashtable of obsolete items in the module and is used when discovering obsolete items in unloaded modules.

```powershell
ObsoleteHints = @{ # Optional
    Commands = @( # Optional
        @{
          CommandName = _ObsoleteCommandName_ # Required - If omitted, an error shall be produced at module import time.
          Action = _SilentlyContinue_|_Continue_|_Stop_|_NotFound_ # Optional - This is the action to take. Default behavior is "continue" which will produce a warning.
          IsObsolete = _bool_ # Optional - default to $true, may be set to false if the command is not obsolete but has obsolete parameters.
          Message = _string_ # Optional - In the case of a command which only has obsolete parameters, this may be omitted. If omitted the value shall be ""
          Parameters = @(
            @{
              ParameterName= _ObsoleteParameterName_ # Required - The parameter which will be reported as obsolete. If Parameters is used, at least one parameter entry must be found. If omitted, it will produce an error.
              Message = _string_ # Optional - The message to provide when importing the module.
              Action = _SilentlyContinue_|_Continue_|_Stop_|_NotFound_ # Optional - This is the action to take. Default behavior is "continue" which will produce a warning.
            }
          )
        }
    )
    Types = @( # Optional
        @{
          TypeName = _ObsoleteTypeName_
          IsObsolete = _bool_ # Optional - default to $true, may be set to false if the type is not obsolete but has obsolete members.
          Message = _string_ # Optional - In the case of a command which only has obsolete members, this may be omitted. If omitted the value shall be ""
          Action = _SilentlyContinue_|_Continue_|_Stop_|_NotFound_ # Optional - This is the action to take. Default behavior is "continue" which will produce a warning.
          Members = @( # Optional - this is only allowed when there are obsolete members.
            @{
              MemberName = _ObsoleteMemberName_ # Optional - If omitted the MemberSignature will be used. If both MemberName and MemberSignature are omitted, this entry will produce an error.
              Message = _string_ # Optional - The message to omit when the module is loaded.
              MemberSignature = _string_ # Optional - used only to disambiguate multiple member names. If omitted, all members of this name will be assumed obsolete.
              MemberType = _string_ # Property|Field|Method|Constructor - Optional - used only disambiguate multiple members. If omitted, all members with the name will be assumed obsolete.
              Action = _SilentlyContinue_|_Continue_|_Stop_|_NotFound_ # Optional - This is the action to take. Default behavior is "continue" which will produce a warning.
            }
          )
        }
    )
}
```

The following is an example from the Microsoft.PowerShell.Utility module:

```powershell
...
PrivateData = @{
  PSData = @{
    ExperimentalFeatures = @(
      @{
        Name        = 'Microsoft.PowerShell.Utility.PSManageBreakpointsInRunspace'
        Description = 'Enables -BreakAll parameter on Debug-Runspace and Debug-Job cmdlets to allow users to decide if they want PowerShell to break immediately in the current location when they attach a debugger.'
      }
    )
    ObsoleteHints = @{
      Commands = @{
        CommandName = "Format-Hex"
        IsObsolete = $false
        Parameters = @{ ParameterName = "Raw"; Message = "Raw parameter is obsolete." }
      },
      @{
        CommandName = "Send-MailMessage"
        Message = "This cmdlet does not guarantee secure connections to SMTP servers. While there is no immediate replacement available in PowerShell, we recommend you do not use Send-MailMessage at this time. See https://aka.ms/SendMailMessage for more information."
        Action = "NotFound"
      }
    }
  }
}
...
```

- the MemberSignature is for disambiguation of obsolescence for the member of a type which may have overloaded signatures.
- If there are no obsolete types, then `Types` may be omitted.
- If there are no obsolete commands (cmdlets, functions, or scripts), then `Commands` may be omitted.
- If there are no obsolete parameters, then `Parameters` may be omitted.
- If there are no obsolete members, then `Members` may be omitted.

The actual `ObsoleteAttribute` must still be present in the command or type.
The source of truth once the module is imported is the presence of the `ObsoleteAttribute` in the source of the cmdlet, parameter, type, or member.
The elements are present in the manifest to not require that the module is loaded before being able to report on obsolescence.

## Configuration Tools

There are a number of elements that require configuration for managing the behavior when an element is marked for obsolescence.
Because of this, we shall provide cmdlets to aid with the creation of the configuration.
Three cmdlets, `Get-ObsoleteItem`, `Enable-ObsoleteItem`, and `Disable-ObsoleteItem` shall be provided.

### `Get-ObsoleteItem`

  This cmdlet retrieves those items which have been attributed as obsolete and their current configuration.
  By default this will return only cmdlets whose items which are currently in loaded modules.
  `-All` shall check all available modules and inspect the PrivateData element for obsolete items and report them, if found.
  `-Force` shall inspect all loaded types and members in PowerShell assemblies for the obsolete attribute and report them, if found. Note that this may take some time and should be used rarely.
  `-ItemType` allows you to retrieve a partial list based on the ItemType.
  Allowed values are `Type`, `Command`, `ExternalScript`

  The output of the `Get-ObsoleteItem` has at least the following properties:

```cs
enum ItemType {
    Type
    Command
    ExternalScript
}

enum MemberType {
    Parameter
    Property
    Field
    Method
    Type
    Event
    Constructor
}

class ObsoleteMember {
    [string]$Name
    [string]$Signature
    [string]$Message
    [bool]$IsError
    [string]$WarningId
    [MemberType]$MemberType
}

class ObsoleteItem {
    [string]$FullyQualifiedWarningId
    [string]$Message
    [bool]$IsError
    [string]$Module
    [ItemType]$ItemType
    [bool]$IsObsolete
    [string]$WarningId
    [bool]$HasObsoleteMembers
    [List[ObsoleteMember]]$ObsoleteMembers
}
```

The FullyQualifiedWarningId is a calculated property made up of the Name of the type or the cmdlet
the following is an example of the output from the `Get-ObsoleteItem` cmdlet:

```powershell
PS> Get-ObsoleteItem

   ItemType: Type

FullyQualifiedWarningId                  ObsoleteMembers           Message
------------------                       ---------------           -------
System.Management.Automation.PowerShell… none                      This enum type was used only in PowerShell Workflow and is now obsolete.
Microsoft.PowerShell.Commands.TextEncod… none                      This class is included in this SDK for completeness only. The members of this cla…
Microsoft.PowerShell.Commands.UtilityRe… none                      This class is obsolete
System.Management.Automation.Runspaces.… ImportPSSnapIn            Custom PSSnapIn is obsolete. Please use a module instead.
Microsoft.PowerShell.Commands.ByteColle… .ctor, .ctor, Offset      The constructor is obsolete. The property is depre...

   ItemType: Command

FullyQualifiedWarningId                       ObsoleteMembers           Message
-----------------------                       ---------------           -------
Microsoft.PowerShell.Utility\Format-Hex:Raw   Raw                       Raw parameter is obsolete.
Microsoft.PowerShell.Utility\Send-MailM…      none                      This cmdlet does not guarantee secure connections to SMTP servers. While there is…

   ItemType: ExternalScript

FullyQualifiedWarningId                  ObsoleteMembers           Message
-----------------------                  ---------------           -------
/Users/james/bin/invoke-obsolete.ps1:foo foo                       This is an obsolete script. foo is an obsolete attribute, use foo2 instead

PS> Get-ObsoleteItem -ItemType Type

   ItemType: Type

FullyQualifiedWarningId                  ObsoleteMembers           Message
-----------------------                  ---------------           -------
System.Management.Automation.PowerShell… none                      This enum type was used only in PowerShell Workflow and is now obsolete.
Microsoft.PowerShell.Commands.TextEncod… none                      This class is included in this SDK for completeness only. The members of this clas…
Microsoft.PowerShell.Commands.UtilityRe… none                      This class is obsolete
System.Management.Automation.Runspaces.… ImportPSSnapIn            Custom PSSnapIn is obsolete. Please use a module instead.
Microsoft.PowerShell.Commands.ByteColle… .ctor, .ctor, Offset      The constructor is obsolete. The constructor is obsolete. The property is obso…

PS> Get-ObsoleteItem -ItemType Command

   ItemType: Command

FullyQualifiedWarningId                  ObsoleteMembers           Message
------------------                       ---------------           -------
Microsoft.PowerShell.Utility\Format-Hex… Raw                       Raw parameter is obsolete.
Microsoft.PowerShell.Utility\Send-MailM… none                      This cmdlet does not guarantee secure connections to SMTP servers. While there is …

```

### `Set-ObsoleteBehavior`

  This cmdlet enables an item which specific override behavior for a specific WarningId.
  If a WarningId setting has already been set, it will be overridden with the new action value.

  ```powershell
  PS> Set-ObsoleteBehavior -WarningId Microsoft.PowerShell.Utility\7.0.0.0\Format-Hex:Raw -Action SilentlyContinue
  PS> Set-ObsoleteBehavior -WarningId /Users/james/bin/invoke-obsolete.ps1:foo -Action Continue
  ```

### `Get-ObsoleteBehavior`  

This command returns all the currently configured warning behavior.
By default it will return configured settings for those items which are currently available in the session.
The `-All` parameter will search through all modules (loaded and unloaded) and script returning the current behavior for all discovered obsolete items.

```powershell
PS> Get-ObsoleteBehavior

FullyQualifiedWarningId                                                     Action
-----------------------                                                     ------
Microsoft.PowerShell.Utility\Format-Hex:Raw                                 SilentlyContinue
Microsoft.PowerShell.Utility\Send-MailMessage                               Error
System.Management.Automation.Runspaces.InitialSessionState:ImportPSSnapIn   NotFound

PS> Get-ObsoleteBehavior -All

FullyQualifiedWarningId                                                     Action
-----------------------                                                     ------
Microsoft.PowerShell.Utility\Format-Hex:Raw                                 SilentlyContinue
/Users/james/bin/invoke-obsolete.ps1:foo                                    Continue
Microsoft.PowerShell.Utility\Send-MailMessage                               Error
System.Management.Automation.Runspaces.InitialSessionState:ImportPSSnapIn   NotFound
```

## Implementation Considerations

- WarningId does not generally ever have a value in the current WarningRecord.
This would need to be used consistently in order for suppression as defined above to work.

- With regard to interactive sessions, PSReadLine does many pre-fetches when the user types the `[...` while referencing types.
Currently, PSReadLine behavior puts types into the type cache even if they are not actually used.
This will likely have implications for reporting obsolete types and members in an interactive session.

## Follow on work

### Extendability

It should be possible to extend what produces the obsolescence warning.
Currently, we recognize only `ObsoleteAttribute`, but it seems reasonable that developers/authors will want to create their own attributes for deprecation and obsolescence.
It is not possible to extend `ObsoleteAttribute` as it is a sealed class.
We should consider a registration mechanism for obsolescence which allows expansion to other attributes.
