---
RFC: RFCxxxx
Author: Thomas Nieto
Status: Draft
SupercededBy: N/A
Version: 1.0
Area: Formatting and Type Extension
Comments Due: 2020-03-30
Plan to implement: Yes
---

# Formatting and Type Extension DSL

PowerShell has an extensive formatting and type extension system.
Module authors wanting to take advantage of these features have to get acquainted with XML.

This RFC proposes a domain-specific language (DSL) to easily create formatting and type extensions in PowerShell without needing to use XML.
The design is inspired by the examples from [RFC0017-Domain-Specific-Language-Specifications](https://github.com/PowerShell/PowerShell-RFC/blob/master/3-Experimental/RFC0017-Domain-Specific-Language-Specifications.md).

## Motivation

    As a module author,
    I can create formatting and type updates,
    so I can save time without learning or using XML.

## User Experience

```powershell
TypeExtension [System.Management.Automation.PSModuleInfo] {
    ViewDefinition Module {
        ListControl {
            ListItem Name
            ListItem Length
        }

        TableControl {
            TableHeader -Width 10 -Alignment Left
            TableHeader -Width 10 -Alignment Left
            TableHeader Prerelease 10 Left
            TableHeader -Width 35 -Alignment Left
            TableHeader ExportedCommands -Alignment Left

            TableColumn ModuleType
            TableColumn Version
            TableColumn {
                if ($_.PrivateData -and $_.PrivateData.PSData)
                {
                        $_.PrivateData.PSData.PreRelease
                }
            }
            TableColumn Name
            TableColumn { $_.ExportedCommands.Keys }
        }

        WideControl {
            WideItem Name
        }
    }
}

TypeExtension [MamlCommandHelpInfo] {
    ViewDefinition CmdletHelp {
        CustomControl {
            CustomItem -Text Name
            CustomItem -NewLine 1

            Frame -LeftIndent 4 {
                CustomItem Name
                CustomItem -NewLine 2
            }

            CustomItem -Text Syntax
            CustomItem -NewLine 1

            Frame -LeftIndent 4 {
                CustomItem Syntax
                CustomItem -NewLine 1
            }
        }
    }
}

TypeExtension [System.Diagnostics.Process] {
    # Add alias properties
    Property Name -Alias ProcessName
    Property SI -Alias SessionId
    Property Handles -Alias HandleCount

    # Add note property
    Property __NounName -Value 'Process'

    # Add code property
    Property Parent -CodeReference [Microsoft.PowerShell.ProcessCodeMethods]::GetParentProcess

    # Add script property
    Property Company { $this.MainModule.FileVersionInfo.CompanyName }
    Property CPU { $this.TotalProcessorTime.TotalSeconds }

    # Add property set
    PropertySet PSConfiguration @('Name', 'Id', 'PriorityClass', 'FileVersion')

    # Add default display properties
    DefaultDisplayPropertySet @('Id', 'Handles', 'CPU', 'SI', 'Name')
}

TypeExtension [System.Xml.XmlNode] {
    # Add code method
    Method ToString -CodeReference [Microsoft.PowerShell.ToStringCodeMethods]::XmlNode
}

TypeExtension [System.Security.Cryptography.X509Certificates.X509Certificate2] {
    # Add script property with getter and setter
    Property SendAsTrustedIssuer -Accessors {
        Get = {
            [Microsoft.Powershell.Commands.SendAsTrustedIssuerProperty]::ReadSendAsTrustedIssuerProperty($this)
        }

        Set = {
            $sendAsTrustedIssuer = $args[0]
            [Microsoft.Powershell.Commands.SendAsTrustedIssuerProperty]::WriteSendAsTrustedIssuerProperty($this,$this.PsPath,$sendAsTrustedIssuer)
        }
    }
}

TypeExtension [Microsoft.PowerShell.Commands.HistoryInfo] {
    # Add default key property
    DefaultKeyPropertySet Id
}

TypeExtension [System.Net.IPAddress] {
    # Add script property
    Property IPAddressToString { $this.ToString() }

    # Add display property
    DefaultDisplayProperty IPAddressToString

    # Set serialization depth
    SerializationMethod -Depth 1
}
```

## Specification

The formatting keywords use `PSControl` builders to create control definitions.

### TypeExtension

Creates a type extension for a type(s).

```powershell
TypeExtension [-TypeName] <string[]> [-Definition] <scriptblock> [<CommonParameters>]
```

* `-TypeName` specifies the type names this type extension applies to.
* `-Definition` specifies the formatting and type extension attributes.

### ViewDefinition

Creates view definitions.
Any number of `ViewDefinition` keywords can appear in `TypeExtension` keyword.
Any number of `Control` keywords can appear in `ViewDefinition`.

```powershell
ViewDefinition [-Name] <string> [-Definition] <scriptblock> [<CommonParameters>]
```

* `-Name` specifies the name of the control.
* `-Definition` specifies the `ListControl`, `TableControl`, `WideControl`, or `CustomControl`.

### GroupBy

The `GroupBy` keyword is used to define a condition to group objects. Can be used only once in each `Control` keyword.

```powershell
GroupBy [-ScriptBlock] <scriptblock> [-Label <string>] [-CustomControl <scriptblock>] [<CommonParameters>]

GroupBy [-PropertyName] <string> [-Label <string>] [-CustomControl <scriptblock>] [<CommonParameters>]
```

* `-ScriptBlock` groups by a script block condition.
* `-PropertyName` groups by a property name.
* `-Label` specifies a label for the group by item.
* `-CustomControl` defines a custom control for group by header.

### Entry

```powershell
TypeExtension @([Microsoft.PowerShell.Commands.X509StoreLocation],
                [System.Security.Cryptography.X509Certificates.X509Certificate2],
                [System.Security.Cryptography.X509Certificates.X509Store]) {
    ViewDefinition ThumbprintTable {
        GroupBy PSParentPath

        TableControl {
            TableHeader -Width 41
            TableHeader -Width 20
            TableHeader EnhancedKeyUsageList

            TableColumn Thumbprint
            TableColumn Subject
            TableColumn { $_.EnhancedKeyUsageList.FriendlyName }
        }
    }

    ViewDefinition ThumbprintList {
        ListControl {
            Entry {
                EntrySelectedBy -TypeName [Microsoft.PowerShell.Commands.X509StoreLocation]
                ListItem Location
                ListItem StoreName
            }

            Entry {
                EntrySelectedBy -TypeName [System.Security.Cryptography.X509Certificates.X509Store]
                ListItem Name
            }

            ListItem { $_.SubjectName.Name } -Label Subject
            ListItem { $_.IssuerName.Name } -Label Issuer
            ListItem Thumbprint
            ListItem FriendlyName
            ListItem NotBefore
            ListItem NotAfter
            ListItem Extensions
        }
    }
}
```

The `Entry` keyword is used in conjunction with `EntrySelectedBy` to create sections with different type name conditions.
`Entry` can appear any number of times in a `Control` keyword.
There must be at least one item in a `Control` outside of an `Entry`.

```powershell
Entry [-Definition] <scriptblock> [<CommonParameters>]
```

* `-Definition` create a section for a different condition for items in the entry using `EntrySelectedBy`.

### EntrySelectedBy

The `EntrySelectedBy` keyword is used create type name conditions for the control.
`EntrySelectedBy` can appear any number of times in an `Entry` keyword.

```powershell
EntrySelectedBy -TypeName <string[]> [<CommonParameters>]
```

* `-TypeName` specifies type name conditions for items in the `Entry`.

### ListControl

```powershell
ListControl [-Definition] <scriptblock> [-OutOfBand] [<CommonParameters>]
```

* `-Definition` specifies `ListItem` and any conditions based on type names using the `Entry` and `EntrySelectedBy` keywords.
* `-OutOfBand` force this view even if a previous object has selected a different view.

### ListItem

`ListItem` adds list items to a `ListControl`.
Can be used any number of times in a `ListControl` or `Entry` keywords.

```powershell
ListItem [-ScriptBlock] <scriptblock> [-Label <string>] [-FormatString <string>] [<CommonParameters>]

ListItem [-PropertyName] <string> [-Label <string>] [-FormatString <string>] [<CommonParameters>]
```

* `-ScriptBlock` specifies a script block item.
* `-ParameterName` specifies a property name item.
* `-Label` apply a label to the item.
* `-FormatString` defines a specific formatting string on an item.

### TableControl

```powershell
TableControl [-Definition] <scriptblock> [-AutoSize] [-HideTableHeaders] [-OutOfBand] [<CommonParameters>]
```

* `-Definition` specifies `TableHeader`, `TableColumn`, and any conditions based on type names using the `Entry` and `EntrySelectedBy` keywords.
* `-AutoSize` specifies if columns will be automatically sized based on contents.
* `-HideTableHeaders` specifies if table headers will be hidden.
* `-OutOfBand` forces this view even if a previous object has selected a different view.

### TableHeader

`TableHeader` adds headers to a `TableControl`.
Can be used any number of times in a `TableControl` keyword.

```powershell
TableHeader [[-Label] <string>] [[-Width] <int>] [[-Alignment] {Undefined | Left | Center | Right}] [<CommonParameters>]
```

* `-Label` specifies the text appearing on the table header.
* `-Width` specifies the column width.
* `-Alignment` specifies if the header will be undefined, left, right, or center aligned.

### TableColumn

`TableColumn` adds columns `TableControl`.
Can be used any number of times `TableControl` or `Entry` keywords.

```powershell
TableColumn [-ScriptBlock] <scriptblock> [[-FormatString] <string>] [[-Alignment] {Undefined | Left | Center | Right}] [<CommonParameters>]

TableColumn [-PropertyName] <string> [[-FormatString] <string>] [[-Alignment] {Undefined | Left | Center | Right}] [<CommonParameters>]
```

* `-ScriptBlock` specifies a script block item.
* `-ParameterName` specifies a property name item.
* `-FormatString` specifies formatting a string on an item.
* `-Width` specifies the column width.
* `-Alignment` specifies if the content will be undefined, left, right, or center aligned.

### WideControl

```powershell
WideControl [-Definition] <scriptblock> [-Columns <uint>] [-AutoSize] [-OutOfBand] [<CommonParameters>]
```

* `-Definition` specifies `WideItem` and any conditions based on type names using the `Entry` and `EntrySelectedBy` keywords.
* `-Columns` specifies number of columns to be displayed.
* `-AutoSize` specifies if columns will be automatically-sized based on contents.
* `-OutOfBand` forces this view even if a previous object has selected a different view.

### WideItem

`WideItem` adds items to a `WideControl`.
Can only be used once in each `WideControl` or `Entry` keywords.

```powershell
WideItem [-ScriptBlock] <scriptblock> [-FormatString <string>] [<CommonParameters>]

WideItem [-PropertyName] <string> [-FormatString <string>] [<CommonParameters>]
```

* `-ScriptBlock` specifies a script block item.
* `-ParameterName` specifies a property name item.
* `-FormatString` specifies a formatting string on an item.

### CustomControl

```powershell
CustomControl [-Definition] <scriptblock> [-OutOfBand] [<CommonParameters>]
```

* `-Definition` specifies `CustomItem`, `Frame` and any conditions based on type names using the `Entry` and `EntrySelectedBy` keywords.
* `-OutOfBand` forces this view even if a previous object has selected a different view.

### CustomItem

`CustomItem` adds items to a `CustomControl`.
Can be used any number of times in a `CustomControl`, `Entry`, or `Frame` keywords.

```powershell
CustomItem [-ScriptBlock] <scriptblock> [-SelectedByTypeName <string>] [-SelectedByScriptBlock <scriptblock>] [-EnumerateCollection] [<CommonParameters>]

CustomItem [-PropertyName] <string> [-SelectedByTypeName <string>] [-SelectedByScriptBlock <scriptblock>] [-EnumerateCollection] [<CommonParameters>]

CustomItem -NewLine <int> [<CommonParameters>]

CustomItem -Text <string> [<CommonParameters>]

CustomItem -CustomControl <scriptblock> [-SelectedByTypeName <string>] [-SelectedByScriptBlock <scriptblock>] [-EnumerateCollection] [<CommonParameters>]
```

* `-ScriptBlock` specifies a script block item.
* `-ParameterName` specifies a property name item.
* `-SelectedByTypeName` sets a type name item selection condition.
* `-SelectedByScriptBlock` sets a script block item selection condition.
* `-NewLine` specifies the number of new lines to add.
* `-Text` specifies the text to add.
* `-EnumerateCollection` indicates if the collection should be enumerated.

### Frame

`Frame` adds a frame to a `CustomControl`.
Can be used any number of times in a `CustomControl` or `Entry` keywords.

```powershell
Frame [-Definition] <scriptblock> [-LeftIndent <uint>] [-RightIndent <uint>] [-FirstLineHanging <uint>] [-FirstLineIndent <uint>] [<CommonParameters>]
```

* `-Definition` specifies `CustomItem` and any conditions based on type names using the `Entry` and `EntrySelectedBy` keywords.
* `-LeftIndent` the number of characters to shift to the right.
* `-RightIndent` the number of characters to shift to the left.
* `-FirstLineHanging` the number of characters to shift the first line to the left.
* `-FirstLineIndent` the number of characters to shift the first line to the right.

### Property

Adds a property to a type.
Can be used any number of times in `TypeExtension` keyword.

```powershell
Property [-Name] <string> [-ScriptBlock] <scriptblock> [-Accessor {Get | Set}] [-Hidden] [<CommonParameters>]

Property [-Name] <string> -CodeReference <PSMethod> [-Accessor {Get | Set}] [-Hidden] [<CommonParameters>]

Property [-Name] <string> -Value <Object> [-Hidden] [<CommonParameters>]

Property [-Name] <string> -Accessors <hashtable> [-Hidden] [<CommonParameters>]

Property [-Name] <string> -Alias <string> [-Hidden] [<CommonParameters>]
```

* `-Name` specifies the property name.
* `-ScriptBlock` defines a script property.
* `-CodeReference` defines a code property.
* `-Value` defines the value for a note property.
* `-Accessor` if the script block or code reference is a getter or setter.
* `-Accessors` defines a script or code property in a hashtable containing Get and Set keys.
* `-Alias` defines a reference to alias property.
* `-Hidden` specifies that the property is hidden.

### Method

Adds a method to a type.
Can be used any number of times in `TypeExtension` keyword.

```powershell
Method [-Name] <string> [-ScriptBlock] <scriptblock> [<CommonParameters>]

Method [-Name] <string> [-CodeReference] <PSMethod> [<CommonParameters>]
```

* `-Name` specifies the method name.
* `-ScriptBlock` defines a script method.
* `-CodeReference` defines a code method.

### Standard Members

Adds a property to a type.
Can be used once of each `DefaultDisplayPropertySet`, `DefaultKeyPropertySet`, and `DefaultDisplayProperty` in a `TypeExtension` keyword.

```powershell
DefaultDisplayPropertySet [[-Property] <string[]>] [<CommonParameters>]

DefaultKeyPropertySet [[-Property] <string[]>] [<CommonParameters>]

DefaultDisplayProperty [[-Property] <string>] [<CommonParameters>]
```

* `-Property` specifies the property name.

### Serialization

Defines serialization settings for a type.
Can be used once of each `SerializationMethod` and `TargetTypeForDeserialization` in a `TypeExtension` keyword.

```powershell
SerializationMethod [-Property] <string> [-Depth <uint>] [<CommonParameters>]

SerializationMethod [-Properties] <string[]> [-InheritPropertySerializationSet <bool>] [-Depth <uint>] [<CommonParameters>]

SerializationMethod [-AllPublicProperties] [-Depth <uint>] [<CommonParameters>]

TargetTypeForDeserialization [-TypeName] <string> [<CommonParameters>]
```

* `-TypeName` specifies the type name.
* `-Property` specifies the property to use for serialization.
* `-Properties` specifies properties to use for serialization.
* `-Depth` specifies how many levels of type objects are serialized as strings.
* `-AllPublicProperties` serialize all public properties of the type.

### TypeConverter/TypeAdapter

Defines type conversion and type adapter types.
Can be used once of each `TypeConverter` and `TypeAdapter` in a `TypeExtension` keyword.

```powershell
TypeConverter [-TypeName] <string> [<CommonParameters>]

TypeAdapter [-TypeName] <string> [<CommonParameters>]

```

* `-TypeName` specifies the type name.

## Alternate Proposals and Considerations
