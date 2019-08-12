---
RFC: RFC0043
Author: Dongbo Wang
Status: Withdrawn
SupercededBy: N/A
Version: 1.0
Area: Microsoft.PowerShell.Core
Comments Due: 4/10/2019
Plan to implement: No
---

# Loading Modules into Isolated AssemblyLoadContext

Assembly isolation is a missing feature in PowerShell.
Today, a PowerShell process loads all assemblies in the default load context.
When an assembly is already loaded, it's not allowed to load the same assembly again with a different version.
This results in the PowerShell issue [#2083](https://github.com/PowerShell/PowerShell/issues/2083).
Basically, when two modules depend on different versions of the same assembly,
if the module that depends on the lower version is loaded first,
the other module cannot be loaded in the same session anymore.

There are two layers of assembly isolation that we can consider to support

- Runspace level: Load PowerShell assemblies into a custom `AssemblyLoadContext`,
  then create a Runspace via Reflection within that `AssemblyLoadContext`.
  So that Runspace works with its own set of PowerShell assemblies in that load context,
  and all assemblies loaded in that Runspace also get loaded into that load context
  (Except for `Assembly.LoadFrom` and `Assembly.LoadFile`.
   `LoadFrom` always loads an assembly to the default load context;
   `LoadFile` always loads an assembly into a new load context).

- Module level: Load a module into a custom `AssemblyLoadContext`,
  the module required assemblies as well as all the assemblies loaded by `Add-Type` in the module script will be loaded into that load context.

Note: _This RFC focuses on the module-level isolation, **but it's written with the purpose to be withdrawn**._
      _While researching the feasibility of this idea, I realized that it's impossible to have it well supported due to the challenges discussed below._
      _This RFC serves as a summary of the research to explain the reasons why we will not proceed with this idea._

## Motivation

    As a PowerShell user, I'm able to use modules that depend on different versions of the same assembly in the same session.

## Specification

### Import-Module -Isolated

The idea is simple: Add the switch parameter `-Isolated` to `Import-Module`.
When specified, PowerShell creates a custom `AssemblyLoadContext` (ALC) instance and makes sure all assembly loadings for and within that module,
except for explicit user calls to `Assembly.LoadFrom` and `Assembly.LoadFile`,
load the assemblies into that custom load context.
The `ModuleInfo` object will hold a reference to the load context used for this module.

PowerShell assembly loading code and `Add-Type` needs to be changed to not call `Assembly.LoadFrom`.
Instead, PowerShell needs to call `LoadFromAssemblyPath` on the corresponding load context instance:

- It queries for the current `EngineSessionState` to see if it's from an isolated module;
- If yes, then the load context of that module is used, otherwise, default load context `AssemblyLoadContext.Default` is used.

_This means we will have type identity issues within a session_ -- type C and C' are the same type in terms of the fully-qualified type name,
but they are from two different assembly instances that are loaded into two load contexts.
This would raise multiple folds of challenges that will be discussed below

### Interoperability

Imagine the assemblies A and B are loaded into ALC-M (isolated module M).
Assembly A exposes a cmdlet `Demo-Cmdlet` that has a parameter accepting an object of type B::C.
Let's say a different version of B is loaded into the default-ACL, and the result assembly instance is B'.
So, from the default-ACL (default Runspace), one can create an object of B'::C' and call `Demo-Cmdlet` with it.
That call will result in a type casting exception with the confusing error like "cannot cast B::C to B::C".

This is because `B'::C'` and `B::C` are two different types,
even though they have the same fully-qualified names and maybe members.
This is the interoperability issue that we will face once PowerShell allow modules to be loaded into different load contexts.

This problem can be even worse when it comes to how `Import-Module M -Isolated` deals with M's nested modules and required modules.
Unless having all nested and required modules loaded into the same custom load context,
there could be interoperability issues when M calls cmdlets from its nested modules or required modules.

#### Mitigation by updating Type Resolution

We can mitigate the interoperability issue by enhance the `TypeResolver` to make it aware of the current `EngineSessionState` for a type resolution operation.
If it happens in the scope of a isolated module,
then `TypeResolver` first searches assemblies in the associated custom-ALC,
followed by searching in the default-ALC.

We also need to support a new syntax for the type reference in PowerShell:

```powershell
[ModuleName\TypeName]
```

When the `ModuleName` part is specified, PowerShell can check if such a module exists and whether it has a custom load context.
If it's a module with a custom load context, then the type resolution will search assemblies from the custom-ALC and then the default-ALC to find the `TypeName`.
Otherwise, the type resolution will search the default-ALC only.

So in the above imagined scenario, it's possible to make it work as expected if you construct the object using `[M\B::C]` from the default-ALC.
But it's non-intuitive and I think many existing scripts may fail when modules started to be loaded in separate load contexts.

### Caching Problem

PowerShell has a lot caches related to `System.Type`.
Some have `Type` as the value, such as the cache in `TypeResolver` and the type cache for `TypeNameAst`.
Some have `Type` as the key, such as the member tables in `DotNetAdapter` and the binder caches in DLR.

For the former category, the cache data structure needs to be updated so as to make sure the cache is associated with a load context.
So for example, when type resolution happens for `[C]` that is running from an isolated module M,
then `TypeResolver` should use the cache associated with ALC-M.

Another example, if a `TypeNameAst` has resolve the type via reflection in default-ALC,
and the `ScriptBlock` containing that `TypeNameAst` is being executed in the context of an isolated module M,
then the cached type referred by `TypeNameAst` should be voided and another resolution should be triggered.
Similarly, if the `ScriptBlock` has already been compiled into LINQ expression tree and have the delegate generated,
then the cached delegate needs to be voided and another compilation needs to be enforced to make sure type resolution is correct.

For the latter category, theoretically they will continue to work without change,
because types loaded into different load contexts are different.
Initially, I think we still need to replace the cache data structure with something like `ConditionalWeakTable<AssemblyLoadContext, ...>`,
so that a load context can be GC collected after the corresponding module is removed.
However, it turns out it's unlikely that the module load contexts can be reclaimed at all in PowerShell.
We will visit that topic in the next section.

As you can imagine, it will be a huge work item to just getting the current caching correct.
And besides, the bookkeeping to make sure caches from the first category are properly voided and rebuilt will be expensive and hurt performance.

### Cannot GC Module's AssemblyLoadContext

.NET Core 3.0 supports unloading an `AssemblyLoadContext` and all assemblies in it.
Ideally, we would like to reclaim the load context associated with an isolated module when the module is removed.
However, it's very likely not possible no matter how we update our caches to avoid holding an `AssemblyLoadContext` instance indefinitely.
This is because Dynamic Language Runtime (DLR) also holds types in its cache, indefinitely.

The DLR types live in the default-ALC, and it will generate a `DynamicMetaObject` object for a late-binding operation,
which is something like a key-value pair `<restrictions, expression>` that will be held in the DLR caches.
This is a perf improvement feature for dynamic languages like PowerShell because it allows you to skip an expensive late-binding operation when the `restrictions` matches.
That key-value pair will be held indefinitely, so once types from a custom load context is involved in a late binding operation, such as property access, method invocation and etc,
it will be rooted in the GC Heap by the DLR cache, and the custom load context cannot be unloaded.

## Summary

To summarize, supporting the module-level isolation doesn't seem to be the right way to go.
The interoperability issues caused by type identity would result in confusing errors that are not intuitive to resolve.
The caching issues caused by the necessary changes to `TypeResolver` would make the code hard to maintain and slower to run.

## Alternate Proposals and Considerations

### Runspace-level Isolation

The Runspace-level isolation seems to be more promising.

- The whole Runspace and all assemblies get loaded in the Runspace are isolated in a custom load context.
  So interoperability will not be a problem unless you receive an object by invoking something on another isolated Runspace.
- The existing caching needs no change.
  The `TypeResolver` implementation needs to be updated to first search assemblies in the Runspace load context,
  then search in the default load context.
- Runspace load context can be reclaimed after the Runspace is closed,
  because the Runspace is not held on by anything other than the user code.
- It fits in the server-application model better.
  Imagine we want to make a server process for PowerShell,
  and any hosts can connect to the server and request for a new session.
  It would be a requirement that an isolated session is created for each host so they are not interfered with each other.
  The Runspace-level isolation would be perfect in this scenario,
  since one server process would be able to satisfy multiple hosts by creating isolated sessions for them.

### About the scenario called out in Motivation

However, the Runspace-level isolation doesn't help the motivation called out above,
because it's still not possible to load same assemblies with different versions in the same Runspace load context.

There is probably an easier way to address that scenario -- register a handler to `AssemblyLoadContext.AssemblyResolve` event.
The resolving process is as follows:

- When `Assembly.Load` fails, the `AssemblyResolve` event will be triggered.
  - The event arg passed in contains the requesting assembly,
    so we can get the directory where the requesting assembly locates.
    - Search in that particular directory to find the exact match of the requested assembly.
      - if found, load it into the load context of the requesting assembly.

Yes, this can also cause interoperability issues,
but the chance that a user is affected by it should be low compared to supporting `Import-Module -Isolated`,
because only the conflicting assembly is loaded into a separate load context.

By far, only the assembly `Newtonsoft.Json` is reported as the conflicting assembly among different modules,
so chances are only this assembly will be loaded into a separate load context for most cases.
