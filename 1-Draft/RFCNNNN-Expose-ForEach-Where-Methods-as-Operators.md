---
RFC: RFC
Author: Michael Klement
Status: Draft
Version: 0.1
Area: Operators
Comments Due: 7/8/2018
Plan to implement: No
---

# Expose the .ForEach() and .Where() "collection-operator" methods as regular PowerShell operators

[The `.ForEach()` and `.Where()` methods](http://www.powershellmagazine.com/2014/10/22/foreach-and-where-magic-methods/) introduced in PSv4 are better-performing and feature-richer expression-mode alternatives to the `ForEach-Object` and `Where-Object` cmdlets; they complement the memory-throttling but slow pipeline processing by the cmdlets with fast, all-in-memory, collected-up-front collection processing.

Syntactically, these methods act _like_ operators (and are even referred to a such in source code and documentation fragment), but are implemented as _methods_.

Since PowerShell's _own_ functionality (as opposed to .NET functionality made _accessible_ by it) is surfaced as commands (cmdlets/function/scripts) and _bona fide operators_ such as `-match`, `.ForEach()` and `.Where()` should (also) be surfaced as array-valued binary operators `-foreach` and `-where`.


## Motivation

As a regular PowerShell user  
I can use operators `-foreach` and `-where`  
in order to transform/filter in-memory collections efficiently, using familiar operator syntax and `$_`-based script blocks, as conceptually clean complements to the `ForEach-Object` and `Where-Object` cmdlets.

## Specification

## Alternate Proposals and Considerations
