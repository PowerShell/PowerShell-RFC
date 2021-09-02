---
RFC: RFC00XX
Author: Dongbo Wang
Status: Draft
Area: Microsoft.PowerShell.Core
Comments Due: 9/30/2021
Plan to implement: Yes
---

# Loading Modules into Isolated AssemblyLoadContext

The [`AssemblyLoadContext`][alc-doc] in .NET Core represents an isolated scope for loading,
resolving, and potentially unloading a set of assemblies.

Today, a PowerShell process loads all assemblies in the default assembly load context (ALC).
For a given ALC instance, when an assembly is already loaded,
it's not allowed to load the same assembly again with a different version.
This results in the PowerShell issue [#2083](https://github.com/PowerShell/PowerShell/issues/2083).
Basically, when two modules depend on different versions of the same assembly,
if the module that depends on the lower version gets loaded first,
the other module cannot be loaded in the same session afterwards.

The proposal is to allow loading a module into a custom ALC.
All the module required assemblies, all the assemblies loaded by `Add-Type` in the module script,
as well as assemblies produced by the PowerShell classes declared in the module,
will be loaded into the custom ALC.

[alc-doc]: https://docs.microsoft.com/dotnet/api/system.runtime.loader.assemblyloadcontext?view=net-5.0

## Motivation

    As a PowerShell user, I'm able to use modules that depend on different versions of the same assembly in the same session.

## Specification

1. Create the type `PSModuleLoadContext` that derives from `AssemblyLoadContext`.
   This type will be the custom ALC used for the isolated modules.
   For the `Load-by-name` operations targeting this custom ALC, such as `Assembly.Load(string)`,
   the following algorithm describes how an assembly is probed:
   - Checking the custom ALC's cache
   - Calling the custom ALC's `Load` method
   - Checking the default ALC's cache
   - Running the default ALC's probing logic -- search in `TRUSTED_PLATFORM_ASSEMBLIES`
   - Raising the default ALC's `Resolving` handler
   - Raising the custom ALC's `Resolving` handler
   - Raising the `AppDomain.AssemblyResolve` handler

1. Update `Import-Module` with two new parameters:

    ```powershell
    ## Import the module to an isolated ALC.
    Import-Module [-Isolated]

    ## Import the module to the specified ALC.
    Import-Module [[-ReflectionContext] <PSModuleLoadContext>]
    ```
    - When `-Isolated` is specified,
      PowerShell creates a custom `AssemblyLoadContext` instance of the type `PSModuleLoadContext`,
      which will be associated with the module.

    - When `-ReflectionContext` is specified,
      the given `PSModuleLoadContext` instance will be associated with the module.

    Unless otherwise instructed by the module author,
    all assembly loading triggered by loading the module or executing the module
    will have the assembly loaded into the associated ALC.

    > NOTE: It's still possbile for a particular assembly to be loaded into a different ALC during the module loading/execution,
      because a module author can always explicitly choose where to load an assembly, by:
    > - calling `LoadXXX` methods directly on an `AssemblyLoadContext` instance;
    > - changing the active ALC with `AssemblyLoadContext.EnterContextualReflection`;
    > - calling `Assembly.LoadFrom` to load an assembly to the default context;
    > - calling `Assembly.LoadFile` to load an assembly to a new anonymous ALC;

1. A nested module should use the same ALC as the parent module.
   Nested modules belong to the parent module and commands are also exposed by the parent modules,
   so they should share the same ALC as the parent module.

1. The `ModuleInfo` object will have a new public read-only property named `ReflectionContext`.
   It will hold a reference to the `PSModuleLoadContext` instance associated with the module.

    > NOTE: The `SessionState` or `SessionStateInternal` was considered when thinking about where to store the custom `PSModuleLoadContext` instance.
    > It was not chosen because a binary module does not have a `SessionStateInternal` created for it.

1. The `ExecutionContext` will have a new internal property `ActiveReflectionContext`,
   which points to the currently active ALC that is supposed to be visible and to be used in PowerShell.
   If its value is `null`, then the currently active ALC is the default ALC.
   For details about why this property is added, see the [Update Assembly Loading Logic](#update_assembly_loading_logic) section.

1. `Import-Module` without `-Isolated` or `-ReflectionContext` will use the currently active ALC for loading assemblies unless `-Global` is specified.
   When `-Global` is specified, the default ALC will be used for loading assemblies.
    1. invoking `Import-Module A` from the global scope will have the default ALC used for the module A;
    1. invoking `Import-Module A` from the module B will have the module B's ALC used for module A.

1. A required module is imported essentially using `Import-Module -Name` today,
   and it's loaded into the current active session state (`EngineSessionState`).

   The proposal is to load the required module with the current active ALC.</br>
   Assuming module A requires module C, it will behave as follows in different scenarios:

    1. invoking `Import-Module A` from the global scope will have the default ALC used for both the modules C and A.</br>
       The modules C and A will be imported into the top session state and be globally visible;
    1. invoking `Import-Module A -Isolated` from the global scope will have the default ALC used for the module C,
       and have a new custom ALC used for the module A.</br>
       The modules C and A will be imported into the top session state and be globally visible;
    1. invoking `Import-Module A -ReflectionContext $moduleInfo-B.ReflectionContext` from the global scope will have the default ALC used for the module C,
       and have the specified module B's ALC used for module A.</br>
       The modules C and A will be imported into the top session state and be globally visible;
    1. invoking `Import-Module A` from the module B will have the module B's ALC used for both the modules C and A.</br>
       The modules C and A will be imported into the module B's session state and be visible from B's module scope.
    1. using `Import-Module -Isolated` or `Import-Module -ReflectionContext` within a module scope is strongly discouraged (maybe forbidden?),
       because the isolated module feature is designed to be a workaround for end users when facing conflicting modules,
       but not a feature for a module author to use by default.

   For details about the design decision and trade-off regarding required modules,
   see the [Handle Required Modules](#handle_required_modules) section.

### Update Assembly Loading Logic

Today, PowerShell uses `Assembly.Load` and `Assembly.LoadFrom` for various assembly loading scenarios,
such as module loading and `Add-Type`.
- `Assembly.Load` infers the active ALC, which is the default ALC in our case
  because the `System.Management.Automation.dll` is always loaded in the default ALC.
- `Assembly.LoadFrom` is hardcoded to always load an assembly into the default ALC.
  It also sets up an event handler to `AppDomain.AssemblyResolve` to handle loading additional assemblies from the same folder path
  for future implicit assembly loading requests.

In addition, we have two more forms of assembly loading:
- For the dynamic assembly generated by `Add-Type` via `Compilation.Emit`
  we load it by `AssemblyLoadContext.Default.LoadFromStream`.
- For the dynamic assembly generated by PowerShell class using the `System.Reflection.Emit` APIs,
  it's loaded into the inferred active ALC, which again is the default ALC today.

The assembly loading logic in PowerShell needs to be updated to make it aware of the correct ALC to use for the loading.
Furthermore, we need to set the contextual ALC before running a module command and restore it afterwards,
so that the explicit or implicit assembly loading triggered by the module command execution can be handled by the correct ALC.

- `AssemblyLoadContext.CurrentContextualReflectionContext` should be set before invoking a module command,
  and be restored to the previous context after the invocation.
- `AssemblyLoadContext.CurrentContextualReflectionContext` should be set before an invocation in a module context (`& $module <cmd-or-script>`),
  and be restored to the previous context after the invocation.

There is a risk if we completely depend on the `AssemblyLoadContext.CurrentContextualReflectionContext` to get the active ALC to work with,
because that property can be changed by a user.
If `CurrentContextualReflectionContext` is changed to an arbitrary ALC, we may run into weird type/assembly resolution issues.

So, I propose to add the new internal property `ExecutionContext.ActiveReflectionContext`,
which points to the currently active ALC that is supposed to be visible and usable in PowerShell.
If its value is `null`, then the currently active ALC is the default ALC.

Similarly, the setup and restore of this property should be done before and after command invocation:

- `ExecutionContext.ActiveReflectionContext` is set before invoking a module command, and restored to the previous context afterwards.
- `ExecutionContext.ActiveReflectionContext` is set before running a command or script block in a module context (`& $module <cmd-or-script>`),
  and restored to the previous context after the invocation.

Note that `ExecutionContext.ActiveReflectionContext` should be setup in `SetCurrentScopeToExecutionScope`,
which is called before the parameter binding in `DoPrepare`,
so that the type resolution during parameter binding can happen in the correct context.

### Update Type Resolution Logic

Now that we have custom ALC's for modules, the type resolving logic needs to be changed:

1. We should first look for the type within the `ExecutionContext.ActiveReflectionContext`,
and then revert back to the default ALC.
1. When reverting back to the default ALC,
we should ignore the assemblies that are loaded (and thus already searched) from the active ALC.

Since different versions of the same assemblies can be loaded into a custom ALC in PowerShell,
we may have type identity issues within a session -- type `C` and `C'` are the same type in terms of the fully-qualified type name,
but they are from two different assembly instances that are loaded into two ALC's.

In order to avoid type identity issue in type resolution and have it work consistently,
we need to do the following extra updates:

1. Today, we have a `TypeCache` data structure that maps a `Tuple<ITypeName, TypeResolutionState>` to a `Type`.
We will need to have separate `TypeCache` defined in the `PSModuleLoadContext` class,
so every custom ALC instance has its own type cache.
The existing `TypeCache` will become the type cache for the default ALC.

1. Today, we have `ExecutionContext.AssemblyCache` that holds all assemblies that are explicitly loaded by PowerShell.
Simiarly, we will need to have separate `AssemblyCache` defined in the `PSModuleLoadContext` class,
so every custom ALC instance has its own `AssemblyCache`.
The existing `AssemblyCache` will be for the default ALC exclusively.

1. All the `ITypeName` implementation types supports type caching.
The cached type of a `ITypeName` instance should be able to invalid the caching when `ExecutionContext.ActiveReflectionContext` changes.

### Handle Required Modules

When would a required module be used?

- For a binary module, a required module is mostly used to bring in assembly dependencies.
- For a script module, a required module is mostly used to bring in command dependencies.

According to how assembly resolution works with ALC,
when module A depends on a required module C to bring in assembly dependencies,
an assembly loaded by C is visible to A only if

- The module C is loaded with the default ALC, OR
- The module C is loaded with module A's ALC, assuming the module A uses a custom ALC.

Now, we come to the most tricky part regarding the required module: **it could be shared by multiple dependent modules**.
Let's imagine the following scenarios:

1. Scenario #1:

    > The modules A and B both depend on the required module C to bring in **assembly dependencies**.</br>
    > The user runs `Import-Module A -Isolated` and `Import-Module B -Isolated` from the global scope,
    > so the modules A and B will be loaded with their separate custom ALC's respectively.

    What ALC should the required module C be loaded with?

    In order to make the assembly loaded by module C visible to both the dependent modules A and B in this scenario,
    the module C has to be loaded with the default ALC.

1. Scenario #2:

    > The modules A and B both depend on the required module C to bring in **assembly dependencies**.</br>
    > The user runs `Import-Module A -Isolated` and `Import-Module B` from the global scope,
    > so the module A will be loaded with a custom ALC and the module B will be loaded with the default ALC.

    What ALC should the required module C be loaded with?

    Again, in order to make the assembly loaded by module C visible to both the dependent modules A and B in this scenario,
    the module C has to be loaded with the default ALC.

1. Scenario #3:

    > The modules A and B both depend on the required module C to bring in **command dependencies**.</br>
    > The commands that A and B depend on have parameters using types from the assembly loaded by module C.</br>
    > The user runs `Import-Module A -Isolated` and `Import-Module B -Isolated` from the global scope,
    > so the modules A and B will be loaded with their separate custom ALC's respectively.

    What ALC should the required module C be loaded with?

    Since the commands from the module C have parameters depending on types loaded by C,
    those types have to be visible to both the dependent modules A and B in order for them to use those commands properly.
    So again, the module C has to be loaded with the default ALC.

1. Scenario #4:

    > The modules A and B both depend on the required module C to bring in **command dependencies**.</br>
    > The commands that A and B depend on have parameters using types from the assembly loaded by module C.</br>
    > The user runs `Import-Module A -Isolated` and `Import-Module B` from the global scope,
    > so the module A will be loaded with a custom ALC and the module B will be loaded with the default ALC.

    What ALC should the required module C be loaded with?

    Similarly to the scenario #3, those parameter types need to be visible to both the dependent modules A and B,
    and thus the module C has to be loaded with the default ALC.

Given the above scenario analysis, it seems **loading the required module with the current active ALC is the most sensible choice**.

But, any drawbacks then? Let's consider the following scenario:

- Scenario #5:

    > The module A has the required module C, and the module B has the required module D.</br>
    > The modules C and D depend on a conflicting assembly.</br>
    > The user runs `Import-Module A -Isolated` and `Import-Module B -Isolated` from the global scope,
    > hoping to have both modules A and B work properly.

    If we choose to load the required module with the current active ALC,
    then in this scenario, both the modules C and D will be loaded with the default ALC.</br>
    However, they depend on a conflicting assembly, so loading the second will fail.
    Therefore, `Import-Module B -Isolated` will fail because the required module D cannot be loaded.

To make this scenario work, the required modules C and D cannot be loaded with the default ALC,
but instead, the module C has to be loaded with A's ALC, and the module D has to be loaded with B's ALC.</br>
That means **the required module needs to be loaded with the dependent module's ALC**.</br>
However, this will obviously break the scenarios 1-4 given that the required module could be shared by multiple dependent modules.

So, it looks we are in a dilemma due to the existing behavior that a required module is shared by all its dependent modules.

Then, how about we make a breaking change to this behavior for the isolated module?</br>
That is, when `-Isolated` or `-ReflectionContext` is used for `Import-Module`,
**we load the required module into the dependent module's session state and load it with the dependent module's ALC**,
so that every dependent module would have an exclusive instance of the required module for itself and they will share the same ALC.

At the first glance, this way would work for all scenarios analyzed above.
But hold on, let's think about the following scenario:

- Scenario #6:

    > Assuming `Az.X` and `Az.Y` modules depend on a conflicting assembly.
    > At the meantime, both modules depend on the same required module `Az.Account`
    > (all Azure PowerShell modules depend on `Az.Account` for authentication realted operations).
    >
    > The user runs `Import-Module Az.X -Isolated` and `Import-Module Az.Y -Isolated` from global scope,
    > hoping to have both modules work properly

    If we load the required module `Az.Account` into each dependent module's session state with the same ALC as the dependent module,
    then `Az.X` and `Az.Y` will be using two different instances of the `Az.Account` module,
    and thus won't be able to share the authentication context.
    Therefore, the user will have to authenticate twice when using `Az.X` and `Az.Y`,
    and the user account context won't be shared between those two modules.

So, it looks this breaking change is too big to make.

With all considered, I think there is no perfect way dealing with the required module to cover all scenarios.
So, let's look at the pros and cons of each candidate solution mentioned above.

1. Load the required module into the current active session state (this is the existing behavior) with the current active ALC.
   - Pros: Work for the scenario 1-4 and 6, and allow the required module to continue to be shared among dependent modules.
   - Cons: Do not work for the scenario 5 by default.
     But there is a simple workaround by importing both the module D and B into the same custom ALC.
      ```
      $module_d = Import-Module D -Isolated -PassThru
      Import-Module B -ReflectionContext $module_d.ReflectionContext
      ```

1. Load the required module into the current active session state (this is the existing behavior) with the dependent module's ALC.
   - Pros: Work for the scenario 5
   - Cons: Do not work for scenario 1-4 and 6, and there is no workaround.

1. Load the required module into the dependent module's session state with the dependent module's ALC.
   - Pros: Fully isolated with all dependencies.
   - Cons: The breaking change is very likely unacceptable.

So in a nutshell, I believe the 1st candidate solution is better suited overall.

## Potential Problems

### Module Interoperability

Imagine the assemblies A and B are loaded into ALC-M (isolated module M).
Assembly A exposes a cmdlet `Demo-Cmdlet` that has a parameter accepting an object of type `B::C`.

Scenario 1:
Let's say the default ALC doesn't contains the assembly B.
Then PowerShell won't be able to resolve the type `[B::C]` from outside the module M (e.g. from global scope),
because `[B::C]` can only be resolved within the ALC-M.

Scenario 2:
Let's say a different version of B was loaded into the default ALC,
and the result assembly instance is `B'`.
So, from the global scope,
one can create an object of `B'::C'` and call `Demo-Cmdlet` with it.
That call will result in a type casting exception with the confusing error like "cannot cast B::C to B::C".

- The behavior in the 1st scenario would be by design due to the nature of the module being isolated.
  The workaround would be `& $moduleM { [B::C] }`,
  but we have to admit that it would be confusing to PowerShell users,
  especially in cases like this example,
  where the exposed command expects the user to pass in an object of the type loaded by the module itself.

- The behavior in the 2nd scenario is because `B'::C'` and `B::C` are two different types,
  even though they have the same fully-qualified names and maybe members.
  This is a typical problem that could be caused by the type identity issue.

   This problem could also happen when `Import-Module M -Isolated` involves dealing with required modules.
   Since required modules will be loaded into the default ALC,
   there could be similar interoperability issues when M calls commands from its required modules.

We can mitigate the interoperability issue by improving the type resolution to make it aware of the module context,
with a new syntax for type reference: `[ModuleName\TypeName]`.

When the `ModuleName` part is specified,
PowerShell can check if such a module exists and whether it has a custom ALC.
If it's a module with a custom ALC,
then the type resolution will search assemblies from the custom-ALC and then the default-ALC to find the `TypeName`.
Otherwise, the type resolution will search the default-ALC only.

So in the 1st scenario above,
one can use `[M\B::C]` instead of `& $moduleM { [B::C] }`;
and in the 2nd scenario above,
it's possible to make it work as expected if we construct the object using `[M\B::C]` from the default-ALC.

> NOTE: The type identity issue is inevitable as soon as different versions of the same assembly can be loaded in the same PowerShell session.
> Improving the type resolution would definitely be helpful,
> but it would require the existing scripts to be updated before they can work with the modules loaded in separate ALC's.

### Caching Problem

PowerShell has a lot caches related to `System.Type`.
Some have `Type` as the value, such as the `TypeCache` in `TypeResolver` and the cached type for `ITypeName`.
Some have `Type` as the key, such as the member tables in `DotNetAdapter` and the binder caches in DLR.

For the former category, as was already mentioned above,
the cache data structure needs to be updated so as to make sure the cache is associated with a ALC.

  - For example, when type resolution happens for `[C]` that is running from an isolated module M,
    then `TypeResolver` should use the cache associated with ALC-M.

  - Another example, if a `ITypeName` has already resolved the type via reflection in default-ALC,
    and the `ScriptBlock` containing that `ITypeName` is now being executed in the context of an isolated module M,
    then the cached type referred by `ITypeName` should be invalided and another resolution should be triggered.
    Similarly, if the `ScriptBlock` has already been compiled into LINQ expression tree and have the delegate generated,
    then the cached delegate needs to be voided and another compilation needs to be enforced to make sure type resolution is correct.

For the latter category, theoretically they will continue to work without change,
because types loaded into different ALC's are different.

### Reclaiming Module AssemblyLoadContext

Starting from 3.0, .NET Core supports unloading an `AssemblyLoadContext` and all assemblies in it.
Ideally, we would like to reclaim the ALC associated with an isolated module when the module is removed.

The cache updates called out in the [Update Type Resolution Logic](#update-type-resolution-logic) section
were designed with the purpose of allowing the custom ALC to be reclaimed.
By having each custom ALC to hold the cache data structure within itself,
we won't need to hold a custom ALC or any types loaded in a custom ALC as the key or value for any caches in our engine code,
and thus there is no need to invalidate any engine cache.

However, we won't be able to guarantee that a custom ALC could be unloaded at all.
This is because Dynamic Language Runtime (DLR) holds types in its cache, which could be indefinitely.

The DLR types live in the default-ALC,
and it will generate a `DynamicMetaObject` object for a late-binding operation,
which is something like a key-value pair `<restrictions, expression>` that will be held in the DLR caches.
This is a perf improvement feature for dynamic languages like PowerShell because it allows you to skip an expensive late-binding operation when the `restrictions` matches.
That key-value pair could be held indefinitely,
so once types from a custom ALC is involved in a late binding operation,
such as property access, method invocation and etc,
it could be rooted in the GC Heap by the DLR cache, and the custom ALC cannot be unloaded afterwards.

## Summary

For the initial implementation, here are the goals and non-goals:

- Goals: work items called out in the [Specification](#specification) section,
  including updates to the module cmdlets, assembly loading, and type resolution.
- Non-goals: the new type reference syntax.

To emphasize again:

The type identity issue is inevitable as soon as different versions of the same assembly can be loaded in the same PowerShell session.
This will mainly affect modules that expect users to directly create objects of the types loaded by the modules.

However, I think the type identity issue is manageable, especially with the improved type reference syntax.
