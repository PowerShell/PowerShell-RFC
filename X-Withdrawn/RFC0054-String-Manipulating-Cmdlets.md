---
RFC: '0054'
Author: Friedrich Weinmann
Status: Withdrawn
SupercededBy: n/a
Version: 1.0
Area: cmdlets/data manipulation
Comments Due: 
Plan to implement: Yes
---

# String Manipulating Cmdlets

Add cmdlets that manipulate strings and are capable of doing so on the pipeline.

## Motivation

Existing tools to manipulate strings - the most common data type manipulated in PowerShell are lacking in usability.
The .NET tools available through string and regex are inconvenient to use.
String-manipulating operators (such as `-split`) are unintuitive to explore, as their options are not easily discoverable.
Neither option allows use on the pipeline, forcing interruption of the pipeline or use of inefficient `ForEach-Object` calls.

From this come two main pain points this RFC is aimed to resolve:

Current string data manipulation ...

 - is technically insufficient from a performance perspective.
 - is unintuitive and inconvenient for the end user.

Given that this is a universal problem affecting literally all users and most code, this should be implemented as part of the core application, not an external third party module.

## Specification

### New Cmdlets

The solution should provide cmdlets that allow string manipulation on the pipeline.

The following operations should be supported:

 - `Add-String`    | Implements partial `-f` functionality to add content to strings, as well as the `.PadLeft()` and `.PadRight()` methods.
 - `Format-String` | Implements full `-f` functionality or the `String.Format()` method. Supports gathering multiple items from the pipeline before formatting.
 - `Get-SubString` | Implements the `.SubString()` method, the `-Trim` operator as well as the `.TrimStart()` and `.TrimEnd()` methods.
 - `Join-String`   | Implements the `-join` operator, allowing specifying the number of items to join in each batch.
 - `Set-String`    | Implements the `-Replace` operator as well as the `.Replace()` method.
 - `Split-String`  | Implements the `-Split` operator as well as the `.Split()` method.

### New Aliases

Given that part of the goal is to improve the user experience, introducing aliases - especially for those mapping operators where users are used to it - will improve user experience and accelerate adoption.

Thus here a proposed list of aliases:

 - `Add-String`    | `wrap` or `add`
 - `Format-String` | `format`
 - `Get-SubString` | None needed - `substring` will automatically resolve to `Get-SubString`
 - `Join-String`   | `join`
 - `Set-String`    | `replace`
 - `Split-String`  | `split`

## Alternate Proposals and Considerations

### Distribution as part of a module

Theoretically, these cmdlets could also be released and distributed as part of an external module.

The same could really be said of every single cmdlet that is shipped as part of the core PowerShell.
What makes a cmdlet core then?
Not bound to a specific use-case? Check.
Relevant to the vast majority of users? Check. (In fact vastly more so than quite a few other core cmdlets)

### Status Quo

The status quo - having users keep using operators and .NET methods -would have several disadvantages:

 - More coding effort to use
 - Additional resource cost (memory when interrupting pipeline, CPU when using `ForEach-Object` instead)
 - Harder to maintain code
 - Less discoverability for users
 - Harder to find documentation for users

## Notes:

 - This topic has already been discussed in an issue](https://github.com/PowerShell/PowerShell/issues/6697)
 - For an experimental implementation, [see the following PR](https://github.com/PowerShell/PowerShell/pull/6753)
