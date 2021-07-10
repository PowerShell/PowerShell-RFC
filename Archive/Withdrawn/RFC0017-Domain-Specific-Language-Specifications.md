---
RFC: RFC0017
Author: Robert Holt
Status: Withdrawn
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

#### Examples pre-existing in PowerShell

* [PowerShell `configuration` modules](https://blogs.msdn.microsoft.com/powershell/2013/11/05/understanding-configuration-keyword-in-desired-state-configuration/) &mdash; DSC configuration modules (Note: this already uses the `DynamicKeyword` mechanisms)
* [Pester](https://github.com/pester/pester) &mdash; testing framework
* [psake](https://github.com/JamesKovacs/psake) &mdash; build automation
* [simplex](https://github.com/beefarino/simplex) &mdash; PowerShell providers
* [ShowUI](https://showui.codeplex.com/) &mdash; WPF UI specification
* [PowerShell Data Files](http://ramblingcookiemonster.github.io/PowerShell-Configuration-Data/) &mdash; PowerShell-native data specification format

#### Motivating candidates for porting to a PowerShell DSL

* [Types.ps1xml](https://msdn.microsoft.com/en-us/library/dd878306(v=vs.85).aspx) &mdash; type addition to PowerShell
* [Format.ps1xml](https://msdn.microsoft.com/en-us/library/dd878339(v=vs.85).aspx) &mdash; format/presentation specification for PowerShell types and resources

The goal, then, would be to create a standard mechanism for PowerShell that all of these DSLs
could be written and maintained in.

## Specifications

This RFC proposes a C#-based mechanism for defining DSLs in PowerShell, similar
to current mechanisms for Cmdlet definition.

A user defines their DSL by instantiating a `Keyword` abstract base class and
marking their class with the `[Keyword()]` attribute. Within the class, functionality is
defined in normal C# code. Parameters are defined with properties decorated with
`[KeywordParameter()]` and inner-scoped keywords can be defined with nested classes.

A DSL keyword is parametrized in `Body` and `Use`:

| Parameter | Variants                              | Default   | Meaning                        |
| :-------: | :-----------------------------------: | :-------: | :----------------------------- |
| `Body`    | `Command`, `ScriptBlock`, `Hashtable` | `Command` | The syntax of the expression/block following the keyword
| `Use`     | `Required`, `Optional`, `RequiredMany`, `OptionalMany` | `OptionalMany` | How many times the keyword may occur |

Similarly, a parameter can be marked `Mandatory` to declare that it must be provided or given
a `Position` where it should be found if not passed by name.

The functionality of a keyword can then be provided by providing instantiations for
the following delegate properties:

| Property        | Type                                             | Function |
| :-------------: | :----------------------------------------------  | :------- |
| `PreParse`      | `Func<DynamicKeyword, ParseError[]`              | Provide an action to execute prior to parsing the keyword statement |
| `PostParse`     | `Func<DynamicKeywordStatementAst, ParseError[]>` | Provide an action to execute after parsing the keyword statement |
| `SemanticCheck` | `Func<DynamicKeywordStatementAst, ParseError[]>` | Check the validity of an otherwise valid keyword statement |

These properties all default to `null`, signifying no operation.

This file is then compiled and the assembly loaded into PowerShell with the
`using module <keyword-name>` statement.

The following is an example aiming to implement a subset of the `types.ps1xml`
functionality. Begin an example of the desired language:

```powershell
using module TypeExtension

# Extend the System.Array type
TypeExtension [System.Array] {
    # Add a new Sum method (from Bruce Payette's "Windows PowerShell in Action", p. 435)
    Method Sum {
        $acc = $null
        foreach ($e in $this)
        {
            $acc += $e
        }
        $acc
    }


    # Add an alias property
    Property Count -Alias Length
}

# Extend the System.IO.DirectoryInfo type
TypeExtension [System.IO.DirectoryInfo] {
    # Add a method reference to an existing static method
    Method Mode -CodeReference [Microsoft.PowerShell.Commands.FileSystemProvider]::Mode
}

# Add a DateTime property to the System.DateTime class
TypeExtension [System.DateTime] {
    Property DateTime {
        if ((& {Set-StrictMode -Version 1; $this.DisplayHint}) -ieq "Date")
        {
            "{0}" -f $this.ToLongDateString()
        }
        elseif ((& {Set-StrictMode -Version 1; $this.DisplayHint }) -ieq "Time")
        {
            "{0}" -f $this.ToLongTimeString()
        }
        else
        {
            "{0} {1}" -f $this.ToLongDateString(), $this.ToLongTimeString()
        }
    }
}
```

To define this language, a `Keyword` class is instantiated as follows:

```csharp
using System.Management.Automation.Language;

[Keyword(Body = DynamicKeywordBodyMode.ScriptBlock)]
class TypeExtension : Keyword
{
    public TypeExtension(Type extendedType)
    {
        ExtendedType = extendedType;
    }

    [Parameter(Position = 0, Mandatory = true)]
    public Type ExtendedType
    {
        get;
        set;
    }

    [Keyword()]
    class Method : Keyword
    {
        public Method(string name)
        {
            Name = name;
            PostParse = ExtendTypeWithMethod;
            SemanticCheck = CheckMethodExtension;
        }

        [Parameter(Position = 0, Manadatory = true)]
        public string Name
        {
            get;
            set;
        }

        [Parameter(Position = 1)]
        public ScriptBlock MethodAction
        {
            get;
            set;
        }

        [Parameter()]
        public PSMethod CodeReference
        {
            get;
            set;
        }

        ParseError[] ExtendTypeWithMethod(DynamicKeywordStatementAst kwAst)
        {
            // Implementation code here here
        }

        ParseError[] CheckMethodExtension(DynamicKeywordStatementAst kwAst)
        {
            // Semantic checking goes here
        }
    }

    [Keyword()]
    class Property : Keyword
    {
        public Property(string name)
        {
            Name = name;
            PostParse = ExtendTypeWithProperty;
            SemanticCheck = CheckPropertyExtension;
        }

        [Parameter(Position = 0, Mandatory = true)]
        public string Name
        {
            get;
            set;
        }

        [Parameter(Position = 1)]
        public ScriptBlock PropertyAction
        {
            get;
            set;
        }

        [Parameter()]
        public string Alias
        {
            get;
            set;
        }

        ParseError[] ExtendTypeWithProperty(DynamicKeywordStatementAst kwAst)
        {
            // Implementation code here
        }

        ParseError[] CheckPropertyExtension(DynamicKeywordStatementAst kwAst)
        {
            // Semantic checking goes here
        }
    }
}
```

## Alternate Proposals and Considerations

### Argument types for `Keyword` methods

Currently the `Keyword` base class specifies its arguments with types internal
to the PowerShell parser, but it may be preferable to not expose these internal types
as a user interface directly. However, since we wish to provide a customizable DSL,
most of the internals of a `DynamicKeyword` and its AST will need to be exposed in some
way, and `DynamicKeywordStatementAst` is exactly the type conceived to encapsulate
that information.

Conceivably an interface could be provided along the lines of:

```csharp
PostParse(object[] positionalParameters, Dictionary<string, object> namedParameters)
{
    ...
}
```

However, this would limit the extent to which a user could customize their keyword
instantiation.

### Provided implementations for Keyword functionality

Further to the above point, keyword instantiation could be made simpler by providing
default virtual methods that simplify the keyword declaration process (for example
by parsing parameters). Users with more sophisticated needs could then override
these methods to suit their needs. This would provide a useful way to empower most
users to make simple DSLs while not precluding more enthusiastic language customizations.
Such methods may be out of scope for the current RFC.

### Bootstrapping the C# specification language for a PowerShell version

To extend the suggestions for ease of use above, the C# keyword definition language
could, in principle, be used to define a keyword definition DSL within PowerShell itself.
Such an implementation is likely out of scope for this RFC however.

### DSC modules and CIM configurations

Small DSL specification language already exists in PowerShell for DSC module
files (`.schema.psm1`) and CIM configurations. These may be of consideration
both as a guide for a more general implementation, and in terms of conflicts
and code duplication &mdash; although there are certain specific functionalities
that are tied to these language features.

### Attributes vs interface implementation for C# Specifications

In the current draft above, specification of a DSL keyword involves the redundancy
of both declaring the `[Keyword]` attribute and extending the `Keyword`
class. This is also the scheme used by `Cmdlet`, however for this reduced case it
may not be necessary.

### Runtime Behavior Specification

Currently there is no suggestion as to the design or implementation of specifying
the runtime behavior of a keyword -- what the keyword actually does when executed.
Runtime behavior can presently be added using PowerShell itself, but specification
in C# may be a better subject of a different RFC.

### Existing Implementations for Dynamic Keywords in PowerShell

The PowerShell parser and AST already support the concept of a
`DynamicKeyword`, which this proposal is built around. The following table
attempts to draw an equivalence between desired DSL features, current dynamic
keyword support, and proposed C# syntax:

| DSL Function            | `DynamicKeyword` Property            | C# Syntax               |
| :---------------------  | :----------------------------------  | :---------------       |
| Keywords                | `Name`                               | `Name = ...` attr       |
| Nested keywords         | Keyword stack                        | Inner class             |
| Keyword use constraints | ? (Semantics check?)                 | `Use = ...` attr                       |
| Body syntax             | `BodyMode`                           | `Body = ...` attr       |
| Arguments               | `DynamicKeywordProperty`             | `[Property]`            |
| Parameters              | `DynamicKeywordParameter`            | `[Parameter]`           |
| Custom types (enums)    | `DynamicKeywordProperty.Values`      | `enum`                  |
| Parse time action       | `DynamicKeyword.(Pre|Post)Parse`     | C# code/interface       |
| Semantics check         | `DynamicKeyword.SemanticCheck`       | C# code/interface       |
| Runtime action          | None                                 | Out of scope            |

-----
PowerShell Committee Decision

Voting Results

Jason Shirk: Accept

Joey Aiello: Accept

Bruce Payette: Accept

Steve Lee: Accept

Hemant Mahawar: Accept

Majority Decision

Commmittee agrees that this RFC is sufficient to move to experimental stage and begin prototype code for further feedback.

Minority Decision

N/A
