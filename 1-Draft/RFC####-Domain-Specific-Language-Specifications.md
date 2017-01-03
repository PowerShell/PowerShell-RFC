---
RFC:
Author: Robert Holt
Status: Draft
Area: Domain Specific Languages, Object Schemas
Comments Due:
---

# Domain-Specific Language Specifications

PowerShell provides rich metaprogramming features with a natural-language-oriented
interface, but currently lacks a true, simple mechanism for language extensions and
keyword addition.

Users have already created their own domain-specific languages (DSLs) in PowerShell
to suit their needs using a variety of language features, however a canonical DSL
definition mechanism could standardize this, in addition to providing more natural
operability within PowerShell, such as:

  * Syntax highlighting
  * Autocompletion
  * Semantic checking
  * Better readability
  * Reproducible, schema-based formats
  * Informative error messages
  * Less XML (in the case of `.ps1xml` files)
  * Specification of parse-time functionality

While not allowing arbitrary changes to the PowerShell language, a DSL mechanism
would seek to let users define their own language functionality, especially with
respect to types, with a well-defined and standardized syntax.

## Motivation

As a PowerShell user, I can create a domain-specific language
to specify a (print format | xml document | object-type
resource configuration | testing setup) schema
using a familiar PowerShell syntax, so that I can repeatably
instantiate that schema and leverage PowerShell's semantic
functionality for field checking and autocompletion.

### Examples of Motivating DSLs in PowerShell

* [Pester](https://github.com/pester/pester) &mdash; testing framework
* [psake](https://github.com/JamesKovacs/psake) &mdash; build automation
* [simplex](https://github.com/beefarino/simplex) &mdash; PowerShell providers
* [ShowUI](https://showui.codeplex.com/) &mdash; WPF UI specification
* [Types.ps1xml](https://msdn.microsoft.com/en-us/library/dd878306(v=vs.85).aspx) &mdash; type addition to PowerShell
* [Format.ps1xml](https://msdn.microsoft.com/en-us/library/dd878339(v=vs.85).aspx)
* [PowerShell `configuration` modules](https://blogs.msdn.microsoft.com/powershell/2013/11/05/understanding-configuration-keyword-in-desired-state-configuration/)
* [PowerShell Data Files](http://ramblingcookiemonster.github.io/PowerShell-Configuration-Data/)

The goal, then, would be to create a standard mechanism for PowerShell that all of these DSLs
could be written and maintained in.

## Specifications

This RFC proposes two mechanisms for defining a DSL for PowerShell, one PowerShell-native and
one in C#. In both cases, the DSL is defined in a file and loaded in with
the `using module` syntax.

Two possibilities currently exist: to define DSL specification language within the PowerShell
language itself, or leveraging existing C# schema parsing so that DSLs are specified in C#
and then loaded into PowerShell.

As an example for either proposal, take a short Pester script (from [here](https://github.com/pester/Pester/blob/master/Examples/Validator/Validator.Tests.ps1)):

```powershell
function MyValidator($thingToValidate)
{
    $thingToValidate.StartsWith("s")
}

Describe "MyValidator"
{
    It "passes things that start with the letter S"
    {
        $result = MyValidator "summer"
        $result | Should Be $true
    }

    It "does not pass a param that does not start with S"
    {
        $result = MyValidator "bummer"
        $result | Should Be $false
    }
}
```

A DSL keyword is parametrized in `Name`, `Body` and `Use`:

| Parameter | Variants                              | Default   | Meaning                        |
| :-------: | :-----------------------------------: | :-------: | :----------------------------- |
| `Name`    | `NoName`, `Required`, `Optional`      | `NoName`  | Whether the keyword has a name |
| `Body`    | `Command`, `ScriptBlock`, `Hashtable` | `Command` | The syntax of the expression/block following the keyword
| `Use`     | `Required`, `Optional`, `RequiredMany`, `OptionalMany` | `Required` | How many times the keyword may occur |

### C# DSL Specifications

Using C#'s existing attribute parsing capabilities, we can define a DSL using an annotated
C# `class` definition and then load it in to PowerShell as described above. This has the
advantages of being simple, well-documented and easy to maintain.

Since a PowerShell keyword must have a semantic action, defined keywords must
also implement the `IPSDslKeyword` interface.
Rather than being methods, these are specified as nullable delegates.
However, since the keyword must have some action, if both are `null` this should be an error.

Using the example above, we might define a schema in C# as follows:

```csharp
[PSDsl]
class Pester
{
    [PSKeyword(Name = NameMode.Required, Body = BodyMode.ScriptBlock)]
    class Describe : IPSDslKeyword
    {
        [PSKeyword(Name = NameMode.Required, Body = BodyMode.ScriptBlock)]
        class It
        {
            [PSKeywordParameterType]
            enum Assertion
            {
                Be,
                BeExactly
            }

            [PSKeyword]
            class Should
            {
                [PSKeywordArgument]
                Assertion assertion;

                [PSKeywordArgument]
                System.Management.Automation.PSObject expectedOutput;

                Func<DynamicKeyword, ParseError[]> PreParse
                {
                    get
                    {
                        return (dynamicKeyword) => {
                            // Pre-parse action code...
                        };
                    }
                }

                Func<DynamicKeywordStatementAst, ParseError[]> PostParse
                {
                    get
                    {   return (dynamicKeywordStatementAst) => {
                            // Post-parse action code...
                        };
                    }
                }

                Func<DynamicKeywordStatementAst, ParseError[]> SemanticCheck
                {
                    get
                    {
                        return (dynamicKeywordStatementAst) => {
                            // Semantic checking code...
                        }
                    }
            }
            }

            Func<DynamicKeyword, ParseError[]> PreParse
            {
                get
                {
                    return (dynamicKeyword) => {
                        // Pre-parse action code...
                    };
                }
            }

            Func<DynamicKeywordStatementAst, ParseError[]> PostParse
            {
                get
                {   return (dynamicKeywordStatementAst) => {
                        // Post-parse action code...
                    };
                }
            }

            Func<DynamicKeywordStatementAst, ParseError[]> SemanticCheck
            {
                get
                {
                    return (dynamicKeywordStatementAst) => {
                        // Semantic checking code...
                    }
                }
            }
        }
    }
}
```

### PowerShell-native DSL Syntax

The other proposed solution is to extend the PowerShell language itself to allow the definition
of a DSL in a PowerShell syntax, e.g. in a `.psm1` file.

In this scheme, a new DSL is defined with the `DSL` keyword, with keywords within that DSL
defined with the `Keyword` keyword. Terms not following the `Keyword` keyword are properties.
Every DSL keyword (including the top-level keyword following `DSL`) can have an arbitrary
number of keywords and properties, which are scope dependent.

Using a DSL defintion, we would define a schema for the example above as:

```powershell
DSL Pester
{
    Keyword Describe -Name Required -Body ScriptBlock
    {
        Keyword It -Name Required -Body ScriptBlock
        {
            Keyword Should
            {
                param($assertion, $expectedOutput)
            }

            PreParse
            {
                param([DynamicKeyword] $dynamicKeyword)

                # Pre-parse action code...
            }

            PostParse
            {
                param([DynamicKeywordStatementAst] $dynamicKeywordStatementAst)

                # Post-parse action code...
            }

            SemanticCheck
            {
                param([DynamicKeywordStatementAst] $dynamicKeywordStatementAst)

                # Semantic checking code...
            }
        }
    }
}
```

### Importing a `DynamicKeyword` definition module

```powershell
using module Pester

$tests = Pester
{
    Describe "MyValidator"
    {
        # ... as in the original example above
    }
}
```


## Alternate Proposals and Considerations

### DSC modules and CIM configurations
Small DSL specification language already exists in PowerShell for DSC module
files (`.schema.psm1`) and CIM configurations. These may be of consideration
both as a guide for a more general implementation, and in terms of conflicts
and code duplication &mdash; although there are certain specific functionalities
that are tied to these language features.

### XML interoperability
The current proposal makes some assumptions about common properties and how innate
they should be to a DSL specification, especially the `Name` parameter, which is
present in the XML an ordinary node, but is a standard parameter to DSL keywords.
The current proposal may need to be changed to support more flexible translation of
XML formats.

The other feature missing from the proposed DSL specification format is the ability
to translate XML attributes. These could either be specified as extra, ad-hoc keyword
parameters, or within the block of the keyword with another syntax.

Conversely, it may be preferable to discard certain common parts of existing
XML schemas, such as `<Members>`, `<MemberSet>`, `<GetScriptBlock>`, etc. and
make them implicit to prevent needless verbosity. However, this would limit
non-XML applications. A workaround might be to include either a flag or extra parameters
to configure whether a user wants this functionality.

### Syntax flexibility
Further to the above point, alternate syntaxes may be preferable for both DSL
definition and instantiation. A more hashtable-like syntax (employing the `=`
symbol) common to both may be simpler for the language:

```powershell
Keyword Bar =
{
    ...
}
```

Another area to change syntax is in attribute specification, where a
type constraint could be specified in a more parameter-like way:

```powershell
Keyword Foo
{
    [string] MyAttribute
}
```

### Attributes vs interface implementation for C# Specifications
In the current draft above, specification of a DSL keyword involves the redundancy
of both declaring the `[PSKeyword]` attribute and implementing the `IPSDslKeyword`
interface. Ideally, only one of these would be used, but in a way that most
helpfully specifies the properties required, while still being consistent with
the rest of the specification scheme.

### Properties vs methods on `IPSDslKeyword` interface
Currently the `IPSDslKeyword` interface specifies parse-time actions be
provided as nullable delegates, so users can communicate when no action exists.
The possibly simpler alternative is to implement a method that the parser simply calls,
and which the user can leave empty if they wish no action to take place.

### Existing Implementations for Dynamic Keywords in PowerShell
The PowerShell parser and AST already support the concept of a
`DynamicKeyword`, which this proposal is built around. The following table
attempts to draw an equivalence between desired DSL features, current dynamic
keyword support, and proposed C# syntax:

| DSL Function            | `DynamicKeyword` Property            | C# Syntax               |
| :---------------------  | :----------------------------------  | :---------------       |
| Keywords                | `Name`                               | `Name = ...` attr       |
| Nested keywords         | Keyword stack                        | Inner class             |
| Keyword use constraints | ? (Semantics check?)                 | ?                       |
| Body syntax             | `BodyMode`                           | `Body = ...` attr       |
| Arguments               | `DynamicKeywordProperty`             | `[PSKeywordArgument]`   |
| Parameters              | `DynamicKeywordParameter`            | `[PSKeywordParameter]`  |
| Custom types (enums)    | `DynamicKeywordProperty.Values`      | `enum`                  |
| Action                  | `DynamicKeyword.(Pre|Post)Parse`     | C# code/interface       |
| Semantics check         | `DynamicKeyword.SemanticCheck`       | C# code/interface       |