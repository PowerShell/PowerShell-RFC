---
RFC: 0005
Author: Aditya Patwardhan
Status: Draft
Version: 0.1
Area: Help System
---

# Help Output Usability Improvements

The help content for PowerShell has various sections like syntax, description, parameters, related links, examples etc. Following is an example of help output for Get-Item. 

```PowerShell
NAME
    Get-Item

SYNOPSIS
    Gets the item at the specified location.


SYNTAX
    Get-Item [-Path] <String[]> [-Credential <PSCredential>] [-Exclude <String[]>] [-Filter <String>] [-Force]
    [-Include <String[]>] [-Stream <System.String[]>] [-UseTransaction <SwitchParameter>] [<CommonParameters>]

    Get-Item [-Credential <PSCredential>] [-Exclude <String[]>] [-Filter <String>] [-Force] [-Include <String[]>]
    [-Stream <System.String[]>] -LiteralPath <String[]> [-UseTransaction <SwitchParameter>] [<CommonParameters>]


DESCRIPTION
    The Get-Item cmdlet gets the item at the specified location. It does not get the contents of the item at the
    location unless you use a wildcard character (*) to request all the contents of the item.

    This cmdlet is used by Windows PowerShell providers to navigate through different types of data stores.


RELATED LINKS
    Online Version: http://go.microsoft.com/fwlink/p/?linkid=290495
    Clear-Item
    Copy-Item
    Invoke-Item
    Move-Item
    New-Item
    Remove-Item
    Rename-Item
    Set-Item
    Get-ChildItem
    Get-ItemProperty
    Get-PSProvider

REMARKS
    To see the examples, type: "get-help Get-Item -examples".
    For more information, type: "get-help Get-Item -detailed".
    For technical information, type: "get-help Get-Item -full".
    For online help, type: "get-help Get-Item -online"

```

## Motivation

As a PowerShell help user, I can have a simplfied visual and navigation experience when exploring help topics. When requesting help for a certain cmdlet, the user is often interested in seeing related cmdlets and links. In the above example, the user might want to explore help for Clear-Item, Remove-Item etc, or would like to see examples or detailed help. The help topics tend to be very verbose and difficult to browse through. 

## Specification
 
 The RFC proposes a way to improve navigation by making 'Related Links' and 'Remarks' sections have clickable links. On clicking the links, the help for linked topic is shown. To improve the readability, this RFC proposes syntax highlighting the examples. Following are the sample with the proposed changes.

RELATED LINKS <br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Online Version: http://go.microsoft.com/fwlink/p/?linkid=290495 <br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a name="clearItem">[Clear-Item](#clearItem)<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a name="copyItem">[Copy-Item](#copyItem)<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a name="invokeItem">[Invoke-Item](#invokeItem)<br />

REMARKS <br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;To see the examples, type: "<a name="examples">[get-help Get-Item -examples](#examples)".<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;For more information, type: "<a name="detailed">[get-help Get-Item -detailed](#detailed)".<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;For technical information, type: "<a name="full">[get-help Get-Item -full](#full)".<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;For online help, type: "<a name="online">[get-help Get-Item -online](#online)" <br />

EXAMPLES <br />
Example 1: Get the current directory
```PowerShell
PS C:\>Get-Item .

Directory: C:\
Mode                LastWriteTime     Length Name
----                -------------     ------ ----
d----         7/26/2006  10:01 AM            ps-test
```
This command gets the current directory. The dot (.) represents the item at the current location (not its
contents).


This command gets a list of all active processes running on the local computer. For a definition of each column,
see the "Additional Notes" section of the Help topic for Get-Help.

## Alternate Proposals and Considerations

