---
RFC:
Author: Robert Holt
Status: Draft
Area: Domain Specific Languages, Object Schemas
Comments Due:
---

# Domain Specific Language Specifications

Frequently in PowerShell contexts there is a need to define a common
layout or schema for PowerShell objects. Examples of such needs
include defining object types, `Format` specifications, creating
repeatable configurations, and specifying testing harnesses.

Such definitions may currently be accomplished with XML-defined
schemas, frequently in a `.ps1xml` file. Notably, there is native
support for DSC configuration modules (`.schema.ps1` files), but
this functionality is not yet generalized.

A PowerShell DSL specification would allow previously XML-bound
schemas to be written with a PowerShell syntax and to hook in to the
PowerShell engine to provide semantic checking, syntax highlighting
and autocompletion. This would allow the enforcement of a notion of
"typing" across objects, allowing the PowerShell engine to manage
the complexity of larger schemas and prevent misconfigurations.

## Motivation

### User Story
As an IT administrator, I can specify a (print format | xml document |
resource configuration | testing setup) schema
using a familiar PowerShell syntax, so that I can repeatably
instantiate that schema and leverage PowerShell's semantic
functionality for field checking and autocompletion.

## Specifications

This RFC proposes a domain-specific language definition syntax
for PowerShell, with a DSL defined in a file and loaded in with
the `using module` syntax.

Currently a schema is defined using XML, for example:
```xml
<Configuration>
  <Controls>
    <Name>FileSystemTypes-GroupingFormat</Name>
    <CustomControl>
      <CustomEntries>
        <CustomEntry>
          <CustomItem>
            <Frame>
              <LeftIndent>4</LeftIndent>
              <CustomItem>
                <ExpressionBinding>
                  <ScriptBlock>
                    "Directory: $($_.PSParentPath.Replace('Microsoft.PowerShell.Core\FileSystem::', ''))"
                  </ScriptBlock>
                </ExpressionBinding>
                <NewLine/>
              </CustomItem>
            </Frame>
          </CustomItem>
        </CustomEntry>
      </CustomEntries>
    </CustomControl>
  </Controls>

  <ViewDefinitions>
    <View>
      <Name>children</Name>
      <ViewSelectedBy>
        <SelectionSetName>FileSystemTypes</SelectionSetName>
      </ViewSelectedBy>
      <Groupby>
        <PropertyName>PSParentPath</PropertyName>
        <CustomControlName>FileSystemTypes-GroupingFormat</CustomControlName>
      </Groupby>
      <TableControl>
        <TableHeaders>
          <TableColumnHeader>
            <Label>Mode</Label>
            <Width>7</Width>
            <Alignment>left</Alignment>
          </TableColumnHeader>
          <TableColumnHeader>
            <Label>LastWriteTime</Label>
            <Width>25</Width>
            <Alignment>right</Alignment>
          </TableColumnHeader>
          <TableColumnHeader>
            <Label>Length</Label>
            <Width>14</Width>
            <Alignment>right</Alignment>
          </TableColumnHeader>
          <TableColumnHeader>
            <Label>Name</Label>
          </TableColumnHeader>
        </TableHeaders>
        <TableRowEntries>
          <TableRowEntry>
            <Wrap/>
            <TableColumnItems>
              <TableColumnItem>
                <PropertyName>Mode</PropertyName>
              </TableColumnItem>
              <TableColumnItem>
                <ScriptBlock>
                  [String]::Format("{0,10} {1,8}", $_.LastWriteTime.ToString("d"), $_.LastWriteTimeToString("t"))
                </ScriptBlock>
              </TableColumnItem>
              <TableColumnItem>
                <PropertyName>Length</PropertyName>
              </TableColumnItem>
              <TableColumnItem>
                <ScriptBlock>
                  if ($host.UI.SupportsVirtualTerminal)
                  {
                      $ext = [System.IO.Path]::GetExtension($_.Name)
                      if ($ext -eq '.exe')
                      {
                          $esc = [char]0x1B
                          "${esc}[93m$($_.Name)${esc}[0m"
                          return
                      }
                  }
                  $_.Name
                </ScriptBlock>
              </TableColumnItem>
            </TableColumnItems>
          </TableRowEntry>
        </TableRowEntries>
      </TableControl>
    </View>
  </ViewDefinitions>
</Configuration>
``` 

## Alternate Proposals and Considerations