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

## Specifications

This RFC proposes a domain-specific language definition syntax
for PowerShell, with a DSL defined in a file and loaded in with
the `using module` syntax.

A new DSL is defined with the `DSL` keyword, with keywords within that DSL defined with
the `Keyword` keyword. Terms not following the `Keyword` keyword are properties. Every DSL
keyword (including the top-level keyword following `DSL`) can have an arbitrary number of
keywords and properties, which are scope dependent.

A DSL keyword is parametrized in `Name` and `Use`:

| Parameter | Variants                              | Default   | Meaning                        |
| :-------: | :-----------------------------------: | :-------: | :----------------------------: |
| `Name`    | `NoName`, `Required`, `Optional`      | `NoName`  | Whether the keyword has a name |
| `Use`     | `Required`, `Optional`, `RequiredMany`, `OptionalMany` | `Required` | How many times the keyword may occur |

As an example, a part of a typical `types.ps1xml` might look as follows:

```xml
<Types>
  <Type>
    <Name>System.Array</Name>
    <Members>
      <AliasProperty>
        <Name>Count</Name>
        <ReferencedMemberName>Length</ReferencedMemberName>
      </AliasProperty>
    </Members>
  </Type>
  <Type>
    <Name>System.Xml.XmlNode</Name>
    <Members>
      <CodeMethod>
        <Name>ToString</Name>
        <CodeReference>
          <TypeName>Microsoft.PowerShell.ToStringCodeMethods</TypeName>
          <MethodName>XmlNodeList</MethodName>
        </CodeReference>
      </CodeMethod>
    </Members>
  </Type>
  <Type>
    <Name>System.Management.Automation.PSDriveInfo</Name>
    <Members>
      <ScriptProperty>
        <Name>Used</Name>
        <GetScriptBlock>
          ## Ensure that this is a FileSystem drive
          if($this.Provider.ImplementingType -eq
          [Microsoft.PowerShell.Commands.FileSystemProvider])
          {
          $driveRoot = ([System.IO.DirectoryInfo] $this.Root).Name.Replace('\','')
          $drive = Get-WmiObject Win32_LogicalDisk -Filter "DeviceId='$driveRoot'"
          $drive.Size - $drive.FreeSpace
          }
        </GetScriptBlock>
      </ScriptProperty>
    </Members>
  </Type>
</Types>
```

Using a DSL defintion, we would define a schema for this:

```powershell
DSL Types
{
    Keyword Type -Name Required -Use RequiredMany
    {
        Keyword Members
        {
            Keyword AliasProperty -Name Required -Use OptionalMany
            {
                ReferencedMemberName -Type [string]
            }

            Keyword CodeMethod -Name Required -Use Optional
            {
                Keyword CodeReference
                {
                    TypeName -Type [string]
                    MethodName -Type [string]
                }
            }

            Keyword ScriptProperty -Name Required -Use OptionalMany -Number
            {
                GetScriptBlock -Type [scriptblock]
            }
        }        
    }
}
```

Then, to recreate the specific `types` XML file from earlier, we could instance it as so:

```powershell
using module Types

$typeSpec = Types
{
    Type -Name System.Array
    {
        Members
        {
            AliasProperty -Name Count
            {
                ReferencedMemberName = Length
            }
        }
    }

    Type -Name System.Xml.Node
    {
        Members
        {
            CodeMethod -Name ToString
            {
                CodeReference
                {
                    TypeName = Microsoft.PowerShell.ToStringCodeMethods
                    MethodName = XmlNodeList
                }
            }
        }
    }

    Type -Name System.Management.Automation.PSDriveInfo
    {
        Members
        {
            ScriptProperty -Name Used
            {
                GetScriptBlock =
                {
                    ## Ensure that this is a FileSystem drive
                    if ($this.Provider.ImplementingType -eq
                        [Microsoft.PowerShell.Commands.FileSystemProvider])
                    {
                        $driveRoot = ([System.IO.DirectoryInfo] $this.Root).Name.Replace('\','')
                        $drive = Get-WmiObject Win32_LogicalDisk -Filter "DeviceId='$driveRoot'"
                        $drive.Size - $drive.FreeSpace
                    }
                }
            }
        }
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