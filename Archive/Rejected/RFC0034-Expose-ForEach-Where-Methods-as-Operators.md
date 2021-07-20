---
RFC: RFC0034
Author: Michael Klement
Status: Rejected
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

* `.ForEach()` and `.Where()` will be exposed as _binary operators_ `-foreach` and `-where`, respectively.

* Unlike the methods (which return `[System.Collections.ObjectModel.Collection[psobject]]` instances), the operators will return `[object[]]` arrays for consistency with existing array-aware operators such as `-eq` and `-match`.

* The _optional_ arguments supported by the `.ForEach()` and `.Where()` methods will be surfaced as optional RHS array elements, analogous to the `-split` operator's optional arguments, for instance - see below.

* The new operators will have the same precedence as the group of equal-precedence operators that operator `-like` falls into - see [`Get-Help about_Operator_Precedence`](https://github.com/PowerShell/PowerShell-Docs/blob/staging/reference/6/Microsoft.PowerShell.Core/About/about_Operator_Precedence.md)

* In terms of handling errors occurring in the RHS script block as well as `return`, `continue` and `break` semantics there, the new operators will exhibit the same behavior as the `.ForEach()` and `.Where()` methods.

### Syntax forms (meta syntax borrowed from [`about_Split`](https://github.com/PowerShell/PowerShell-Docs/blob/staging/reference/6/Microsoft.PowerShell.Core/About/about_Split.md]))

#### `-foreach`

For script-block-based transformation, optionally with arguments passed to each script block invocation.

```none
<collection> -foreach <ScriptBlock>[, <Arguments[]>]
```

For type conversion.
Note: 

 * Arguably, this form is not needed, because a simple array-valued cast will do:  
`[string[]] (1, 2, 3)` rather than `1, 2, 3 -foreach [string]`

 * However, for symmetry with .ForEach() it should probably be implemented.

```none
<collection> -foreach <Type>
```

For property access / method calls:
Note:

* For mere property extraction, there's a simpler alternative: member enumeration:  
`((get-date), (get-date)).Ticks` vs.  `(get-date), (get-date) -foreach 'Ticks'`

* However, there's actually a distinct advantage to using `-foreach`: bypassing the _ambiguity_ of member enumeration:  
`('ab', 'cde') -foreach 'Length'` will unambigiously access the _elements'_ `.Length` property, 
whereas `('ab', 'cde').Length` returns the _array's_ length (element count).

```none
<collection> -foreach "propertyName"[, <value>]
<collection> -foreach "methodName"[, <Arguments[]>]
```

#### `-where`

Filtering a collection based on the Boolean outcome of a script block, optionally in one of serveral modes and with a limit on how many objects to return:

```none
<collection> -where <ScriptBlock>[, <mode>[, <numberToReturn>]]
```

* `<mode>` is technically a [`[System.Management.AutomationWhereOperatorSelectionMode]`](https://docs.microsoft.com/en-us/dotnet/api/system.management.automation.whereoperatorselectionmode?view=powershellsdk-1.1.0) enumeration value, but may be specified as a string (e.g., `'Split'`)
* The semantics of `[int]` `<numberToReturn>` depend on `<mode>`; `0`, the default, requests that _all_ matching elements be returned.
  * Given https://github.com/PowerShell/PowerShell/issues/4765, consider interpreting negative values - which currently generate an execption - as requests to return elements from the _end_ of the results collection - this should obviously apply both to the method and operator forms.
* As with `-split`, arguments are strictly positional; notably, specifying `<numberToReturn>` requires that `<mode>` value also be specified, even if as `'Default`.

[@KirkMunro's blog post](http://www.powershellmagazine.com/2014/10/22/foreach-and-where-magic-methods/) contains the details.

It's probably worth creating a new conceptual help topic titled `about_Collection_Operators` to describe these new operators, with links to existing help topics for those existing operators that _also_ support collection-valued LHS input, such as `-eq`, `-like`, and `-replace`, among others.

## Alternate Proposals and Considerations

The alternative is to make do with the existing _method_ implementation, restricting their use to a more developer-savvy crowd comfortable with method syntax.

Note that even the existing methods [lack proper documentation](https://github.com/PowerShell/PowerShell-Docs/issues/2307) as of this writing.
