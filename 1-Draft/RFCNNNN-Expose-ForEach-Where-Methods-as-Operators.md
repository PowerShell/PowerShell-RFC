---
RFC: RFC
Author: Michael Klement
Status: Draft
Version: 0.1
Area: Operators
Comments Due: 7/8/2018
Plan to implement: No
---

# Expose the .ForEach() and .Where() collection-transforming/filtering methods as regular PowerShell operators

[The `.ForEach()` and `.Where()` methods](http://www.powershellmagazine.com/2014/10/22/foreach-and-where-magic-methods/) introduced in PSv4 are better-performing and feature-richer expression-mode alternatives to the `ForEach-Object` and `Where-Object` cmdlets; they complement the memory-throttling but slow pipeline processing by the cmdlets with fast, all-in-memory, collected-up-front collection processing.

Semantically, these methods act _like_ operators (and are even referred to a such in source code and documentation fragment), but are syntactically implemented as _methods_.

Since PowerShell's _own_ functionality (as opposed to .NET functionality made _accessible_ by it) is surfaced as commands (cmdlets/function/scripts) and _bona fide operators_ such as `-match`, `.ForEach()` and `.Where()` should (also) be surfaced as array-valued binary operators `-foreach` and `-where`.

## Motivation

As a regular PowerShell user  
I can use operators `-foreach` and `-where`  
in order to transform/filter in-memory collections efficiently, using familiar operator syntax and `$_`-based script blocks, as conceptually clean complements to the `ForEach-Object` and `Where-Object` cmdlets.

Examples:

```powershell
$var = 1, 2, 3 -foreach { $_ + 1 }  # wishful thinking

# vs.
$var = foreach ($i in 1, 2, 3) { $i + 1 }
# or (slower)
$var = 1, 2, 3 | ForEach-Object { $_ + 1 }
#  ---

$var = 1, 2, 3 -where { $_ -gt 1 } # wishful thinking

# vs.
# (slower)
$var = 1, 2, 3 | Where-Object { $_ -gt 1 }
# or (conceptually *indirect*, given that there's no filtering loop):
$var = foreach ($i in 1, 2, 3)  { if ($i -gt 1) { $i } }
```

Advanced example (no `Where-Object` counterpart): split a collection in two:

```powershell
$odds, $evens = 1, 2, 3, 4 -where { $_ % 2 }, 'split'

# equivalent to:
$odds, $evens = (1, 2, 3, 4).Where({ $_ % 2 }, 'split')
```


## Specification

* `.ForEach()` and `.Where()` will be exposed as * binary operators `-foreach` and `-where`, respectively.

* Unlike the methods (which return `[System.Collections.ObjectModel.Collection[psobject]]` instances), the operators will return `[object[]]` arrays for consistency with existing array-aware operators such as `-eq` and `-match`.

* The _optional_ arguments supported by the `.ForEach()` and `.Where()` methods will be surfaced as optional RHS array elements, analogous to the `-split` operator's optional arguments, for instance.



It's probably worth creating a new conceptual help topic titled `about_Collection_Operators` to describe these new operators.


## Alternate Proposals and Considerations

The alternative is to make do with the existing _method_ implementation, restricting their use to a more developer-savvy crowd comfortable with method syntax.

Note that even the existing methods [lack proper documentation](https://github.com/PowerShell/PowerShell-Docs/issues/2307) as of this writing.

