---
RFC: RFCNNNN
Author: Mathias R. Jessen
Status: Draft
SupercededBy: N/A
Version: 0.1
Area: Language
Comments Due: 2019-01-04
Plan to implement: Yes
---

# Native Support for .NET Interface Declarations

The PowerShell Classes feature introduced in PowerShell 5.0 allows for type definitions at runtime. Since open sourcing PowerShell Core, one of the most oft-repeated feature request I've heard from community members is the ability to also declare and define interfaces at runtime, usually for ease of implementing common software design pattern such as the Builder pattern.

## Motivation

    As a module author,
    I can leverage common OO design patterns that require abstract type definitions,
    so that my implementations are easily communicated and explained to other developers and module authors.

## Specification

Implementing native support for interface declarations require both syntactical and semantic changes to the language engine.

### Syntax

Based on the [original issue](https://github.com/PowerShell/PowerShell/issues/2223) raised by @vors, I imagine a simplified subset of the class syntax, as follows:

```powershell
# interface declaration with no members
interface IMyInterface { }

# interface declaration chaining declared members from another interface
interface IMyDisposable : IDisposable { }

# interface declaration with a method
interface IMyInterfaceWithMethod {
    [void] DoSomething([int]$IntParam)
}

# interface declaration with a property
interface IMyInterfaceWithProperty {
    [int]$MyProperty
}

# interface declaration extending an existing interface with properties and methods
interface IMyExtendedInterfaceWithMembers {
    [int]$MyProperty

    [void] DoSomething([int]$IntParam)
}
```

### Semantics

Currently, when the parser recognizes a `class` or `enum` declaration, it produces a `TypeDefinitionAst` made up of `PropertyMemberAst` and `FunctionMemberAst` children. The type definition AST is then dispatched to either TypeDefiner.DefineTypeHelper or TypeDefiner.DefineEnumHelper for creating and emitting the corresponding runtime type.

We can either add new `AbstractMemberAst` types specifically for the purpose of abstract type definitions, or modify the existing `MemberAst` types to account for abstract member constraints.

Going with separate AST types for abstract type members would reduce the number of branches and special cases required in the rest of the code base, but the obvious benefit of reusing existing AST types for member declarations is that we can do not have to extend the existing AST visitor surface (ie. no `IAstVisitor3 : IAstVisitor2`, `AstVisitor3 : AstVisitor2, IAstVisitor3` etc.). This may present hidden parsing issues for tooling developers though.

Regardless of design changes to the AST surface, we can either define a new `TypeDefiner.DefineInterfaceHelper` for emitting interface types, or we can modify the existing `DefineTypeHelper` definition to take the `Interface` attribute into account and modify all the other emitting functions being called by it.

In my [prototype implementation](https://github.com/IISResetMe/PowerShell/tree/feature/interfaces), I've opted to: 

 1. extend the AST visitor surface with a set of `AbstractMemberAst` types,
 2. update the relevant AstVisitor interfaces and implementations, and
 3. dispatch type definition to a new `DefineInterfaceHelper` class

## Alternate Proposals and Considerations

There are a couple of open questions around the design and implementation of interface declaration support:

 - Should we allow control over getter/setter declarations?
   - If so, should we replicate the syntax from C#?
 - Existing class definitions need to [mark interface member implementations virtual](https://github.com/PowerShell/PowerShell/issues/8302)
