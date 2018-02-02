---
RFC: RFC
Author: Dongbo Wang, Dan Travison
Status: Draft
SupercededBy: N/A
Version: 1.0
Area: Engine and Configuration
Comments Due: March 2nd, 2018
Plan to implement: Yes
---

# Support Experimental Features via Configuration

This RFC is about adding the Experimental Feature support in PowerShell through the PowerShell configuration.

The Experimental Feature support in PowerShell is to provide a mechanism for experimental features to coexist
with exsiting stable features in PowerShell or PowerShell modules.
An experimental feature would not be enabled
unless a user opt in by properly configuring the `powershell.config.json` file.
Experimental features should not affect any stable features
and not present any side effects
when they are all disabled.

## Motivation

This is a highly demanded feature which we believe will vastly improve the productivity of
PowerShell developers on experimenting or iterating new features in PowerShell and PowerShell modules.

The Experimental Feature support would make it easy for users to try out features
that are in early or intermediate stages of development,
and thus, allow the developers to get early feedback
and make adjustments to the design or implementation based on the feedback.

For users that do not care about the new features,
they can continue to enjoy the bug fixes and other improvements from the new releases
without worrying about regressions, issues, or side effects caused by the new features.

For example, PowerShell team plans to rewrite the `FileSystemProvider` in future
to reduce the code complexity and improve the performance.
During the development,
the `FileSystemProvider vnext` can coexist with the existing one
and be exposed to the user as an experimental feature,
so that PowerShell team can get early feedback and issue reports.
At the same time, users can continue to use the existing `FileSystemProvider`
without being affected by the known or unknown issues introduced by the `FileSystemProvider vnext`.

## Specification

### Experimental Feateure Names

Experimental features for PowerShell components and built-in modules should be named as `PSXXX`,
where `XXX` is the real feature name, which is a single word in camel case.
For example:

```none
PSDomainSpecificLanguage
PSFileSystemProviderV2
```

Experimental features for a third-party PowerShell module should be named with
the module name as the fully qualified namespace to reduce the chance of collision.
The recommended naming conversion is to use the module name followed by a single-word feature name, all in camel case.
For example:

```none
ModuleName.FeatureName
```

### Enable Experimental Features

An experimental feature is enabled by adding the feature name to the experimental feature list
in the `powershell.config.json` file under `$PSHOME`.
The `JSON` schema should look like this:

```json
{
  "ExperimentalFeatures": [
    "A-Feature-name",
    "Another-Feature-Name"
  ]
}
```

PowerShell reads the experimental feature list from the configuration file only once when starting up.

- For `pwsh`, the list will be read in `ConsoleHost` as soon as possible,
  so that experimental features targeting the `ConsoleHost` can be turned on early enough.
- When creating a Runspace, PowerShell will check if the list has been read,
  and read the list if not yet.
  This is to make sure experimental features can be enabled when an application is hosting PowerShell.

Feature names from the list will be populated to a read-only immutable HashSet that is static to the whole process.
The HashSet will be exposed to users via an API as well as an automatic variable.
The API makes it easy to query for enabled features in C# code,
while the automatic variable makes it easy to query from PowerShell scripts.

### Experimental Feature Implementation Scenarios

#### Code Path Divided

The first implementation scenario of an experimental feature is to divide the code path based on whether a feature is enabled.
This probably would be the most common pattern for implementing any experimental fatures.
PowerShell exposes the enabled experimental feature names through both an API and an automatic variable,
so it could be very easy for both C# code and PowerShell script to access this information.

#### Cmdlet and Parameter

The second implementation scenario is manipulate cmdlets and parameters.
There are three categories that we will discuss as follows.

##### Add New Cmdlets

There are two cases in this scenario:

- A new cmdlet is added by the experimental feature.
  The new cmdlet should be exposed when the experimental feature is enabled.
- A new cmdlet is added by the experimental feature to overide an existing cmdlet with the same name.
  When the experimental feature is enabled,
  the existing cmdlet should be hidden and the new cmdlet should be exposed.

An example for the first case:
> A new cmdlet `Enable-SSHRemoting` is added as part of an experimental feature about Remoting over SSH.
We want to expose this cmdlet only if this experimental feature is enabled.

An example for the second case:
> We want to rewrite the `Invoke-WebRequest` cmdlet completely from scratch.
We don't want to make changes directly to the existing code due to the risk of regression to the existing cmdelt.
Instead, we prefer to write new code from scratch in a new C# type
and replace the existing `Invoke-WebRequest` when this experimental feature is enabled.

The proposal to this scenario is to add a new attribute `[Experimental()]` that can apply to a cmdlet.
The attribute takes two arguments: `FeatureName` and `FeatureAction`.

- `FeatureName` is a string, indicating the experimental feature the attribute is associated with.
- `FeatureAction` is an enum with two members (shown as follows),
  and it indicates the action to take on the cmdlet that has the attribute declared.

```c#
public enum FeatureAction
{
    Hide,
    Show
}
```

> Context: Cmdlet Registration -- The process that PowerShell reads assemblies or scripts
and create cmdlets for the cmdlet/function implementations defined in those sources.

During the Cmdlet Registration, PowerShell can decide whether to process or ignore a cmdlet with the following workflow:

- Check if the `[Experimental()]` attribute is declared
  - if so, check if the associated experimental feature is enabled
    - if so, ignore the cmdlet if the action to take is `Hide`;
      process the cmdlet if the action to take is `Show`.
    - if not, ignore the cmdlet if the action to take is `Show`;
      process the cmdlet if the action to take is `Hide`
  - if not, process the cmdlet as is today

The following code snippets are examples of using this attribute for cmdlet/function:

```c#
[Experimental("PSWebCmdletV2", FeatureAction.Show)]
[Cmdlet(Verbs.Invoke, "WebRequest")]
public class InvokeWebRequsetCommandV2 : WebCmdletBaseV2 { ... }

[Experimental("PSWebCmdletV2", FeatureAction.Hide)]
[Cmdlet(Verbs.Invoke, "WebRequest")]
public class InvokeWebRequestCommand : WebCmdletBase { ... }
```

```powershell
function Enable-SSHRemoting {
    [Experimental("PSSSHRemoting", "Show")]
    [CmdletBinding()]
    param()
    ...
}
```

##### Add New Parameters

There are two cases in this scenario:

- A new parameter is added to a cmdlet by the experimental feature.
  The new cmdlet parameter should be exposed when the experimental feature is enabled.
- An existing parameter is removed from a cmdlet by the experimental feature.
  The existing cmdlet parameter should be hidden when the experimental feature is enabled.

An example for the first case
> We want to add a new parameter to `Add-Type` for a new feature for compiling C# code.
We want to expose this parameter only if the experimental feature is enabled.

An example for the second case:
> We want to remove a problematic parameter from `Add-Type`.
In order to evaluate the impact of the breaking chagne,
we want to hide it only if the experimental feature is enabled.

The same `[Experimental()]` attribute can also be used in this scenario.
During the Cmdlet Registration, when processing parameters for a cmdlet/function,
PowerShell can decide whether to process or ignore a given parameter with the following workflow:

- Check if the `[Experimental()]` attribute is declared
  - if so, check if the associated experimental feature is enabled
    - if so, ignore the parameter if the action to take is `Hide`;
      process the parameter if the action to take is `Show`.
    - if not, ignore the parameter if the action to take is `Show`;
      process the parameter if the action to take is `Hide`
  - if not, process the parameter as is today

The following code snippets are examples of using this attribute for cmdlet/function:

```c#
[Experimental("PSNewAddTypeCompilation", FeatureAction.Show)]
[Parameter(ParameterSet = "NewCompilation")]
public CompilationParameters CompileParameters { ... }

[Experimental("PSNewAddTypeCompilation", FeatureAction.Hide)]
[Parameter()]
public CodeDom CodeDom { ... }
```

```powershell
param(
    [Experimental("PSNewFeature", "Show")]
    [string] $NewName,

    [Experimental("PSNewFeature", "Hide")]
    [string] $OldName
)
```

##### Control Parameter Sets

There are two cases in this scenario:

- Add an existing parameter to a parameter set.
  The parameter set could be an existing one, or a new one created for the experimental feature.
  The parameter should be in the parameter set if the experimental feature is enabled.
- Remove an existing parameter from a parameter set.
  The parameter should not be in the parameter set if the experimental feature is enabled.

An example for the first case:
> We want to support a new transport layer for `Invoke-Command`,
which will introduce a new parameter set.
Parameters `-ScirptBlock` and `-FilePath` need to be added to the new parameter set
when the experimental feature is enabled.

An example for the second case:
> We want to try out a breaking change that involves removing a parameter set.
For the parameters that are in the parameter set,
they should be removed from the parameter set when the experimental feature is enabled.

The proposal to this scenario is to add two properties `FeatureName` and `FeatureAction` to the `[Parameter()]` attribute.
So a `Parameter` attribute can be associated with an experimental feature.
When the experimental feature is enabled,
PowerShell can choose to process or ignore the `Parameter` attribute depending on the `FeatureAction`.

The following code snippets are examples of using the new properties in a `Parameter` attribute:

```c#
[Parameter(Position = 1, Mandatory = true,
           ParameterSetName = "SetAboutToAdd",
           FeatureName = "PSAddTheSet",
           FeatureAction = FeatureAction.Show)]
[Parameter(Position = 1, Mandatory = true,
           ParameterSetName = "SetAboutToRemove",
           FeatureName = "PSRemoveTheSet",
           FeatureAction = FeatureAction.Hide)]
public ScriptBlock ScriptBlock { ... }
```

```powershell
param(
    [Parameter(Mandatory = $true, ParameterSetName = "SetAboutToAdd",
               FeatureName = "PSAddTheSet", FeatureAction = "Show")]
    [Parameter(Mandatory = $true, ParameterSetName = "SetAboutToRemove",
               FeatureName = "SetAboutToRemove", FeatureAction = "Hide")]
    [string] $Name,
)
```

#### PowerShell Class and Members

PowerShell Class is generated during the script compilation,
so runtime checks have no control over it.
If an experimental feature of a module involves new declaration of PowerShell Class,
the PowerShell Class will be exposed through `using Module <module-name>`
even if the experimental feature is not enabled.

The proposal to this scenario is to also apply the `[Experimental()]` attribute
to PowerShell Class and its members.
When processing `TypeDefinitionAst`, PowerShell can filter out a `TypeDefinitionAst` node
depending on whether the experimental feature is enabled and the `FeatureAction` value.
The filtering will need to be done in the following places (could be more):

- [`SymbolTable.AddTypesInScope(Ast ast)`](https://github.com/PowerShell/PowerShell/blob/master/src/System.Management.Automation/engine/parser/SymbolResolver.cs#L168)
- [`Compiler.GenerateTypesAndUsings(ScriptBlockAst rootForDefiningTypesAndUsings, List<Expression> exprs)`](https://github.com/PowerShell/PowerShell/blob/master/src/System.Management.Automation/engine/parser/Compiler.cs#L2199)
- [`PSModuleInfo.CreateExportedTypeDefinitions(ScriptBlockAst moduleContentScriptBlockAsts)`](https://github.com/PowerShell/PowerShell/blob/master/src/System.Management.Automation/engine/Modules/PSModuleInfo.cs#L612)
- [`CompletionCompleters.CompleteType(CompletionContext context, string prefix = "", string suffix = "")`](https://github.com/PowerShell/PowerShell/blob/master/src/System.Management.Automation/engine/CommandCompletion/CompletionCompleters.cs#L5844)

The searching for `TypeDefinitionAst` nodes should be done in one method,
so that there is no duplication of the filtering logic for the `[Experimental()]` attribute.

As for the type members with the `[Experimental()]` attributes,
[`TypeDefiner`](https://github.com/PowerShell/PowerShell/blob/master/src/System.Management.Automation/engine/parser/PSType.cs#L17) needs to be updated to process or skip a type member when processing a `TypeDefinitionAst` node,
depending on whether the experimental feature is enabled and the `FeatureAction` value.

#### Exposed Public APIs

An experimental feature may expose public C# APIs.
Public C# APIs cannot be hidden.
Even if we alter the `DotNetAdapter` to filter out those public APIs from the experimental feature in PowerShell,
they will still be visible to applications that host PowerShell.
Therefore, we will not do anything about the public APIs exposed from experimental features.

### Experimental Feature Discovery

A user should be able to discover the available experimental features in PowerShell,
including the ones in the PowerShell engine as well as the ones from modules.

#### PowerShell Engine Experimental Feature

For experimental features in PowerShell engine,
an internal enum `EngineExperimentalFeatures` will be used to track the names of all available engine experimental features.
When adding a new experimental feature, its name needs to be added to the enum for the feature to be discoverable to users.
The feature description needs to be added as a resource string,
and the id of the resource string needs to be constructible from the feature name,
such as `XXX-Description` where `XXX` is the feature name.
Here is an example of the enum and a description resource string.

```c#
internal enum EngineExperimentalFeatures
{
    None,
    PSDomainSpecificLanguage,
    PSFileSystemProviderV2
}
```
```xml
// EngineExperimentalFeature.resx
<data name="PSFileSystemProviderV2-Description" xml:space="preserve">
  <value>Rewrite file system provider for better performance.</value>
</data>
```

#### Module Experimental Feature

For experimental features in modules,
metadata about an experimental feature should be kept in the module manifest.
We use the `PrivateData` section of a module manifest to expose the experimental features from the module.
Here is an example:

```powershell
PrivateData = @{
    ExperimentalFeature = @{
        Name = "PSWebCmdletV2"
        Description = "Rewrite the web cmdlets for better performance"
    }
}
```

PowerShell module components, such as `Import-Module` and module analysis,
will be updated to incorporate this metadata to the resulted `PSModuleInfo` object.

#### Get-ExperimentalFeature

A new cmdlet `Get-ExperimentalFeature` will be added to return all available experimental features of a PowerShell session.
The returned experimental features are represented by the type `ExperimentalFeature`,
whose definition looks as follows:

```c#
public class ExperimentalFeature
{
    // Feature name
    public string Name { get; }
    // Feature description
    public string Description { get; }
    // Feature source (e.g. module name)
    public string Source { get; }
}
```

The cmdlet first goes through the enum members of `EngineExperimentalFeatures` to find all engine experimental features.
For each of them, the description resource string will be retrieved and the `Source` will be set as `PSEngine`.

Then the cmdlet goes through all loaded modules and get the available experimental features from each module.
For each of them, the `Name` and `Description` are from the metadata. The `Source` will be the module name.

Lastly, the cmdlet can go through not-yet-loaded modules that are in the module path,
in a similar way as `Get-Command` does,
with the help of the module analysis code.

### Experimental Feature Dependencies

> NOTE: This section may not be in scope for the first iteration of our implementation.

An experimental feature in a module can depend on another experimental feature.
The latter could be a PowerShell engine feature,
or a feature from the same module,
or one from another module.
Given an experimental feature name,
it's easy to tell whether it's a PowerShell engine feature
or a feature from a module,
and in the latter case, the module name.

When loading a module, if an experimental feature of the module is enabled,
PowerShell needs to check whether the dependencies are satisfied,
in a way that's similar to how PowerShell processes the nested modules.
Here is the high-level description of the scenarios:

- If the enabled experimental feature depends on a PowerShell engine experimental feature
  that is not enabled, then fail the loading with the appropriate error message.
- If the enabled experimental feature depends on an experimental feature from the same module
  that is not enabled, then fail the loading with the appropriate error message.
- If the enabled experimental feature depends on an experimental feature from another module,
  then the loading would fail unless
  - the latter module is specified in the `NestedModules`
  - the latter experimental feature is enabled

In order to represent the dependencies,
the metadata of an experimental feature in `.psd1` needs to add a `Dependency` field as follows:

```powershell
PrivateData = @{
    ExperimentalFeature = @{
        Name = "PSWebCmdletV2"
        Description = "Rewrite the web cmdlets for better performance"
        Dependency = "<experimental-feature-name>", "..."
    }
}
```

correspondingly, a new property `Dependency` needs to be added to the type `ExperimentalFeature`:

```c#
public class ExperimentalFeature
{
    // Feature name
    public string Name { get; }
    // Feature description
    public string Description { get; }
    // Feature source (e.g. module name)
    public string Source { get; }
    // Features it depends on
    public ExperimentalFeature[] Dependency { get; }
}
```

## Alternate Proposals and Considerations

No alternate proposals.
