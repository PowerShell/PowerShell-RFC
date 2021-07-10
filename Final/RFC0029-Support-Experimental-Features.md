---
RFC: RFC0029
Author: Dongbo Wang, Dan Travison
Status: Final
SupercededBy: N/A
Version: 1.0
Area: Engine and Configuration
Comments Due: March 22nd, 2018
Plan to implement: Yes
---

# Support Experimental Features via Configuration

This RFC is about adding the Experimental Feature support in PowerShell through the PowerShell configuration.

The Experimental Feature support in PowerShell is to provide a mechanism for experimental features to coexist
with existing stable features in PowerShell or PowerShell modules.
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
the `FileSystemProviderV2` can coexist with the existing one
and be exposed to the user as an experimental feature,
so that PowerShell team can get early feedback and issue reports.
At the same time, users can continue to use the existing `FileSystemProvider`
without being affected by the known or unknown issues introduced by the `FileSystemProviderV2`.

## Specification

### Experimental Feature Names

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
ModuleName.ExperimentName
```

### Enable Experimental Features

An experimental feature is enabled by adding the feature name to the experimental feature list
in the `powershell.config.json` file under `$PSHOME`.
The `JSON` schema should look like this:

```json
{
  "ExperimentalFeatures": [
    "A-Experimental-Feature-name",
    "Another-Experimental-Feature-Name"
  ]
}
```

> NOTE: the current proposal is to only use the instance configuration file `$PSHOME\powershell.config.json`
(or the settings file passed in through `pwsh -settingsFile`, which overrides the instance configuration file)
to read the enabled experimental feature list.
The per-user configuration file will not be used in the initial implementation.

PowerShell reads the experimental feature list from the configuration file only once.
It's done in the static constructor of the type `ExperimentalFeature`,
so the initialization is guaranteed to take place before any experimental feature code path is hit.

Feature names from the list will be populated to a read-only immutable HashSet that is static to the whole process.
The HashSet will be exposed to users via a public method `ExperimentalFeature.IsEnabled(string featureName)`,
as well as an automatic variable `$EnabledExperimentalFeatures`.
The method makes it easy to query for enabled features in C# code,
while the automatic variable makes it easy to query from PowerShell scripts.

The read-only immutable HashSet is a static field in the type `ExperimentalFeature`,
named `EnabledExperimentalFeatureNames`.

### Experimental Feature Implementation Scenarios

#### Code Path Divided

The first implementation scenario of an experimental feature is to divide the code path based on whether a feature is enabled.
This probably would be the most common pattern for implementing any experimental features.
PowerShell exposes the enabled experimental feature names through both APIs and an automatic variable,
so it could be very easy for both C# code and PowerShell script to access this information.

#### Cmdlet and Parameter

The second implementation scenario is to manipulate cmdlets and parameters.
There are three categories that we will discuss as follows.

##### Add New Cmdlets

There are two cases in this scenario:

- A new cmdlet is added by the experimental feature.
  The new cmdlet should be exposed when the experimental feature is enabled.
- A new cmdlet is added by the experimental feature to override an existing cmdlet with the same name.
  When the experimental feature is enabled,
  the existing cmdlet should be hidden and the new cmdlet should be exposed.

An example for the first case:
> A new cmdlet `Enable-SSHRemoting` is added as part of an experimental feature about Remoting over SSH.
We want to expose this cmdlet only if this experimental feature is enabled.

An example for the second case:
> We want to rewrite the `Invoke-WebRequest` cmdlet completely from scratch.
We don't want to make changes directly to the existing code due to the risk of regression to the existing cmdlet.
Instead, we prefer to write new code from scratch in a new C# type
and replace the existing `Invoke-WebRequest` when this experimental feature is enabled.

The proposal to this scenario is to add a new attribute `ExperimentalAttribute` that can be applied to a cmdlet.
The attribute takes two arguments: `experimentName` and `experimentAction`.

- `experimentName` is a string, indicating the experimental feature the attribute is associated with.
- `experimentAction` is an enum with three members,
  and it indicates the action to take on the cmdlet that has the attribute declared.

```c#
public enum ExperimentAction
{
    // Represent an undefined action, used as the default value only.
    None,

    // Hide the cmdlet/parameter when the corresponding experimental feature is enabled.
    Hide,

    // Show the cmdlet/parameter when the corresponding experimental feature is enabled.
    Show
}
```

> Context: Cmdlet Registration -- The process that PowerShell reads assemblies or scripts
and create cmdlets for the cmdlet/function implementations defined in those sources.

During the Cmdlet Registration, PowerShell can decide whether to process or ignore a cmdlet with the following workflow:

- Check if `ExperimentalAttribute` is declared
  - if so, check if the associated experimental feature is enabled
    - if so, ignore the cmdlet if the action to take is `Hide`;
      process the cmdlet if the action to take is `Show`.
    - if not, ignore the cmdlet if the action to take is `Show`;
      process the cmdlet if the action to take is `Hide`
  - if not, process the cmdlet as if no `ExperimentalAttribute` is declared.

The following code snippets are examples of using this attribute for cmdlet/function:

```c#
[Experimental("PSWebCmdletV2", ExperimentAction.Show)]
[Cmdlet(Verbs.Invoke, "WebRequest")]
public class InvokeWebRequestCommandV2 : WebCmdletBaseV2 { ... }

[Experimental("PSWebCmdletV2", ExperimentAction.Hide)]
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

> Implementation Note: the attribute `ExperimentalAttribute` could be applied to a script block directly.
Execution of such a script block should error out when the associated experimental feature is not enabled.

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
In order to evaluate the impact of the breaking change,
we want to hide it only if the experimental feature is enabled.

The same `ExperimentalAttribute` can also be used in this scenario.
During the Cmdlet Registration, when processing parameters for a cmdlet/function,
PowerShell can decide whether to process or ignore a given parameter with the following workflow:

- Check if `ExperimentalAttribute` is declared
  - if so, check if the associated experimental feature is enabled
    - if so, ignore the parameter if the action to take is `Hide`;
      process the parameter if the action to take is `Show`.
    - if not, ignore the parameter if the action to take is `Show`;
      process the parameter if the action to take is `Hide`
  - if not, process the parameter as if no `ExperimentalAttribute` is declared.

The following code snippets are examples of using this attribute for cmdlet/function:

```c#
[Experimental("PSNewAddTypeCompilation", ExperimentAction.Show)]
[Parameter(ParameterSet = "NewCompilation")]
public CompilationParameters CompileParameters { ... }

[Experimental("PSNewAddTypeCompilation", ExperimentAction.Hide)]
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
Parameters `-ScriptBlock` and `-FilePath` need to be added to the new parameter set
when the experimental feature is enabled.

An example for the second case:
> We want to try out a breaking change that involves removing a parameter set.
For the parameters that are in the parameter set,
they should be removed from the parameter set when the experimental feature is enabled.

The proposal to this scenario is to add two properties `ExperimentName` and `ExperimentAction` to `ParameterAttribute`.
So a `ParameterAttribute` can be associated with an experimental feature.
When the experimental feature is enabled,
PowerShell can choose to process or ignore the `ParameterAttribute` depending on the `ExperimentAction`.

The following code snippets are examples of using the new properties in a `Parameter` attribute:

```c#
[Parameter("PSAddTheSet", ExperimentAction.Show, Position = 1, Mandatory = true, ParameterSetName = "SetAboutToAdd")]
[Parameter("PSRemoveTheSet", ExperimentAction.Hide, Position = 1, Mandatory = true, ParameterSetName = "SetAboutToRemove")]
public ScriptBlock ScriptBlock { ... }
```

```powershell
param(
    [Parameter("PSAddTheSet", "Show", Mandatory = $true, ParameterSetName = "SetAboutToAdd")]
    [Parameter("SetAboutToRemove", "Hide", Mandatory = $true, ParameterSetName = "SetAboutToRemove")]
    [string] $Name,
)
```

Note that, if a parameter is declared with multiple `ParameterAttribute` that are all associated with experimental features respectively
and all those experimental features are not enabled,
then the parameter is considered not enabled and thus will be ignored when processing parameters.

#### PowerShell Class and Members

> NOTE: Not implemented in the first iteration of the feature work.

PowerShell Class is generated during the script compilation,
so runtime checks have no control over it.
If an experimental feature of a module involves new declaration of PowerShell Class,
the PowerShell Class will be exposed through `using Module <module-name>`
even if the experimental feature is not enabled.

The proposal to this scenario is to also apply the `ExperimentalAttribute` to PowerShell Class and its members.
When processing `TypeDefinitionAst`, PowerShell can filter out a `TypeDefinitionAst` node
depending on whether the experimental feature is enabled and the `ExperimentAction` value.
The filtering will need to be done in the following places (could be more):

- [`SymbolTable.AddTypesInScope(Ast ast)`](https://github.com/PowerShell/PowerShell/blob/master/src/System.Management.Automation/engine/parser/SymbolResolver.cs#L168)
- [`Compiler.GenerateTypesAndUsings(ScriptBlockAst rootForDefiningTypesAndUsings, List<Expression> exprs)`](https://github.com/PowerShell/PowerShell/blob/master/src/System.Management.Automation/engine/parser/Compiler.cs#L2199)
- [`PSModuleInfo.CreateExportedTypeDefinitions(ScriptBlockAst moduleContentScriptBlockAsts)`](https://github.com/PowerShell/PowerShell/blob/master/src/System.Management.Automation/engine/Modules/PSModuleInfo.cs#L612)
- [`CompletionCompleters.CompleteType(CompletionContext context, string prefix = "", string suffix = "")`](https://github.com/PowerShell/PowerShell/blob/master/src/System.Management.Automation/engine/CommandCompletion/CompletionCompleters.cs#L5844)

The searching for `TypeDefinitionAst` nodes should be done in one method,
so that there is no duplication of the filtering logic for `ExperimentalAttribute`.

As for the type members with the `ExperimentalAttribute` attributes,
[`TypeDefiner`](https://github.com/PowerShell/PowerShell/blob/master/src/System.Management.Automation/engine/parser/PSType.cs#L17) needs to be updated to process or skip a type member when processing a `TypeDefinitionAst` node,
depending on whether the experimental feature is enabled and the `ExperimentAction` value.

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
an internal static read-only collection `EngineExperimentalFeatures` will be used to track all available engine experimental features.
The items in the collection are of the type `ExperimentalFeature`, whose definition looks as follows:

```c#
public class ExperimentalFeature
{
    // Feature name
    public string Name { get; }
    // Feature description
    public string Description { get; }
    // Feature source (e.g. PSEngine, or module name)
    public string Source { get; }
    // Feature is enabled or not
    public bool Enabled { get; }
}

internal static readonly ReadOnlyCollection<ExperimentalFeature> EngineExperimentalFeatures;
```

When adding a new experimental feature, an instance representing the feature needs to be added to the collection for it to be discoverable to users.
The `Source` for engine experimental features should be specified as `PSEngine`.

#### Module Experimental Feature

For experimental features in modules,
metadata about an experimental feature should be kept in the module manifest.
We use the `PrivateData.PSData` section of a module manifest to expose the experimental features from the module.
Here is an example:

```powershell
PrivateData = @{
    PSData = @{
        ExperimentalFeatures = @(
            @{Name = "PSWebCmdletV2", Description = "Rewrite the web cmdlets for better performance"}
            @{Name = "PSRestCmdletV2", Description = "Rewrite the REST API cmdlets for better performance"}
        )
    }
}
```

PowerShell module components, such as `Import-Module` and module analysis,
will be updated to incorporate this metadata to the resulted `PSModuleInfo` object.

The property `ExperimentalFeatures` of the type `ReadOnlyCollection<ExperimentalFeature>` is added to `PSModuleInfo`,
to hold the exposed features from the module.

When any experimental feature are enabled, a unique file name will be used for caching the module analysis results,
so that the experimental commands won't affect a PowerShell session that has no experimental feature enabled.

#### Get-ExperimentalFeature

A new cmdlet `Get-ExperimentalFeature` will be added to return all experimental features of a PowerShell session.
The returned experimental features are represented by the type [`ExperimentalFeature`](#powerShell-engine-experimental-feature).
The syntax signature of the cmdlet will look as follows:

```powershell
Get-ExperimentalFeature [[-Name] <string[]>] [-ListAvailable] [<CommonParameters>]
```

By default, it returns enabled experimental features only.
When `-ListAvailable` is specified, it returns all available experimental features.

For the experimental features that are enabled,
the property `Enabled` will be set to `true` accordingly.

### Experimental Feature Dependencies

> NOTE: Not implemented in the first iteration of the feature work.

An experimental feature in a module can depend on another experimental feature.
The latter could be a PowerShell engine feature,
or a feature from the same module,
or one from another module.
Given an experimental feature name,
it's easy to tell whether it's a PowerShell engine feature
or a feature from a module,
and in the latter case, the module name.

There are two kinds of dependencies on an experimental feature:

- No new experimental feature is declared,
  but new functionality is turned on when the dependent experimental feature is enabled.
  For example, the `ScriptAnalyzer` automatically turns on the `DomainSpecificLanguage` related rules
  when the experimental feature `PSDomainSpecificLanguage` is enabled.
- A new experimental feature is declared,
  and the new functionality is turned on only if the new experimental feature is enabled.
  For example, `Pester` has an experimental implementation using the `DomainSpecificLanguage`,
  and users can try it out only if having the new experimental feature enabled.

For the first scenario, the dependency is implicit and nothing needs to be done on the engine side.
The new functionality will be turned on when the dependent experimental feature is enabled.

For the second scenario, the dependency is explicit and should be declared in the module manifest.
When loading such a module, if an experimental feature of the module is enabled,
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
    PSData = @{
        ExperimentalFeature = @{
            Name = "PSWebCmdletV2"
            Description = "Rewrite the web cmdlets for better performance"
            Dependency = "<experimental-feature-name>", "..."
        }
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
    // Feature is enabled or not
    public bool Enabled { get; }
    // Features it depends on
    public ExperimentalFeature[] Dependency { get; }
}
```

### Test Experimental Features

The regular daily test pass should run without any experimental feature enabled.
However, our CI system should be able to run tests with a specific experimental feature turned on,
so that CI builds can verify PRs for an experimental feature work.

The proposal is as follows:

- Add the file `test/tools/TestMetadata.json` for declaring tests for experimental features.
  - Key is the experimental feature name;
  - Value is an array of paths for the folder or files that Pester should run when the feature is turned on.
    If the value is an empty array, then all tests will be executed for the feature.
- Add `-ExperimentalFeatureName` to `Start-PSPester` to support running tests with the specified feature turned on.
- Update `AppVeyor.psm1` and `travis.ps1` to read `TestMetadata.json` and run tests for each declared features.

The tests for an experimental feature should be written in one of the following ways:

```powershell
Describe "Tests for Experimental Feature X" {
  BeforeAll {
    $skip = -not $PSEnabledExperimentalFeatures.Contains("X")
    ...
  }

  It "..." -Skip:$skip {
    ...
  }
}

try {
  $skip = -not $PSEnabledExperimentalFeatures.Contains("X")
  $originalDefaultParameterValues = $PSDefaultParameterValues.Clone()
  $PSDefaultParameterValues["it:skip"] = $skip

  Describe "Tests for Experimental Feature X" {
    ...
  }
}
finally {
  $global:PSDefaultParameterValues = $originalDefaultParameterValues
}
```

### Experimental Feature Incompatibility Mitigation

> NOTE: Not implemented in the first iteration of the feature work.

There will be cases where experimental features are incompatible either intentionally or unintentionally.

- An example of an intentional incompatibility occurs when two alternative, but incompatible solutions, are being tested.
- An example of an unintended incompatibility can occur when two experimental features prove to be incompatible through usage.

A minimal implementation will support defining a list of experimental feature pairs in the `powershell.config.json` file.
At startup, attempts to use incompatible features would produce warnings and consider both as disabled.

An opportunistic implementation will support an exclusion value in the experimental feature declaration
for cases where an author is intentionally providing alternative/incompatible implementations.

### Static Code Analysis

> NOTE: Not implemented in the first iteration of the feature work.

Static code analysis support should be provided to address experimental feature authoring.
Two areas are particularly important:

1. Validating whether an experimental feature is marked correctly.
   A use case for this occurs when an experimental feature is referenced in a module but not, or incorrectly defined in the manifest.
   From a script perspective, a `PSScriptAnalyzer` rule would be a reasonable solution.
   From a compiled code perspective, a reflection-based tool may be needed.

2. Validating whether references to an non-existing experimental feature have been removed.
   At some point, the experiment is completed and either removed from or incorporated into the code base.
   At this point, a tool to scan script or compiled code would be needed to ensure any references to the experiment have been removed.

## Open Issues

### Whether experimental features can be included in PowerShell GA releases (Closed)

The conclusion is: it's OK to have experimental features included in PowerShell GA releases.

This allows the development of an experimental feature to span across GA releases,
and also allows an experimental feature to get more feedback.

### Whether experimental feature dependencies should be parsed when reading the configuration

When reading the experimental feature list from configuration file,
shall we parse the dependencies at the same time or defer the dependency resolution until module loading?

As described in the [`Experimental Feature Dependencies`](#experimental-feature-dependencies) section,
an experimental feature may have a chain dependency on other experimental features.
In that section, the proposal is to check for dependencies at module loading time.

Another proposal is to resolve the dependencies when loading the configuration file,
so that PowerShell can fail early when dependencies are not satisfied.
The fail-early idea is desired,
however, PowerShell loads the enabled experimental feature list at very early stage,
where the engine is not ready for the operations required for resolving dependencies,
such as module discovery.
Also, there will be an impact to the startup time if we do this resolution when loading the configuration file.
Lastly, the module that exposes an experimental feature may not be in the module path yet
when PowerShell starts,
in which case it's impossible to resolve the experimental feature name.

### How to override the instance configuration file -- rewrite command-line parsing vs. use environment variable

A new command-line flag `-settingsFile` was added to `pwsh` to allow
a user to override the instance configuration file `$PSHOME\powershell.config.json` using another configuration file.
This flag is very useful in testing the experimental features.

However, there are some amount of work that occurs before the command-line parameter parsing,
so if that amount of work has experimental features,
it would be hard to flag them in testing using `-settingsFile`.
This problem is seen on Linux with logging configuration due to
the logging happens at a very early stage when PowerShell starts.

There are two proposals here for solving this problem:

1. Move the command-line parameter parsing earlier in the startup call-path.
   The main parsing logic depends on other components like the `InitialSessionState`,
   so it cannot be moved to a very early stage.
   A possible solution is to parse the command-line parameters in two passes.
   The first pass is to convert the arguments into a more structured format,
   and process flags like `-settingsFile` at a very early stage.
   The second pass is the main parsing logic.
2. Use an environment variable `PSSettingsFile` to point to the configuration file to use.
   By using the environment variable,
   overriding the instance configuration file can happen as early as the singleton instance of `PowerShellConfig` gets created,
   which would be the first time PowerShell attempts to read the configuration file.

The first proposal will cost way more than the second proposal.
Since the configuration file overriding functionality is mainly for testing,
the second proposal may be more efficient cost-wise.

### How to run tests for each experimental feature regularly (Closed)

The proposed [CI updates](#test-experimental-features) will enable running tests for each experimental feature in daily builds.

### Whether to have telemetry for experimental feature usage (Closed)

The conclusion is: yes, we need telemetry for experimental feature usage.

However, more work (compliance and etc.) needs to be done to enable telemetry in PowerShell Core.
So for now, we don't have telemetry for experimental feature usage.

## Alternate Proposals and Considerations

No alternate proposals.

---------------
## PowerShell Committee Decision

### Voting Results

Joey Aiello: Accept

Bruce Payette: Accept

Steve Lee: Accept

Hemant Mahawar: Accept

Dongbo Wang: Accept

Kenneth Hansen: Accept

### Majority Decision

Commmittee agrees that this RFC satisfies the experimental feature flag feature and recognizes that not all aspects of this RFC are currently implemented.

### Minority Decision

N/A
