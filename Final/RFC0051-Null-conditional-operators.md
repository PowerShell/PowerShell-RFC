---
RFC: 0051
Author: Aditya Patwardhan
Status: Final
SupercededBy:
Version: 0.1
Area: Language
Comments Due: 09/30/2019
Plan to implement: Yes
---

# Null conditional operators for coalescing, assignment and member access

The RFC proposes to introduce new operators for null coalescing, null conditional assignment.
Null conditional member access is being moved to additional consideration section.
The operators provide the script authors a short-hand for performing null checks.

## Motivation

    As a PowerShell script author,
    I can use null conditional operators,
    so that null condition checks are more succinct.

## User Experience

### Null assignment operator - `??=`

Assign the value to the variable if the variable value is null or the variable does not exist.

```powershell
$x ??= 'new value'
${x}??='new value'

$x? ??= 'new value'
${x?}??='new value'
```

### Null coalescing operator - `??`

Return the left hand side if it is not null, else return the right hand side.

```powershell
$x = $null
$x ?? 100
100
```

```powershell
$x = 'some value'
$x ?? 100
some value
```

```powershell
$x? = 'some value'
${x?}??100
some value
```

## Specification

The RFC proposes four new operators, the null assignment operator `??=`, the null coalescing operator `??`, and the two null conditional member access `?.` and `?[]`.

### Null assignment operator - `??=`

The `??=` operator check the left hand side to be null or undefined and assigns the value of right hand side if null, else the value of left hand side is not changed.

```powershell
$todaysDate ??= (Get-Date)
```

The value of `Get-Date` is assigned to `$todaysDate` if it is null or undefined, if not null, the value is not changed.

### Null coalescing operator - `??`

The `??` operator checks the left hand side to be null or undefined and returns the value of right hand side if null, else return the value of left hand side.

```powershell
(Get-Module Pester -ListAvailable) ?? (Install-Module Pester)
```

If the module `Pester` is not installed, then install the module.

### Null conditional member access operators - `?.` and `?[]`

Null member access operators can used on scalar types and array types.
Return the value of the accessed member if the variable is not null.
If the value of the variable is null, then return null.

```powershell
$x = $null
${x}?.property
${x?}?.property

${x}?[0]
${x?}?[0]

${x}?.MyMethod()
```

The `?.` and `?[]` operators access the specified member if the value of the variable is not null.
If it is null, then null is returned.

```powershell
${x}?.propname
${x}?[0]
${x}?.MyMethod()
```

The property `propname` is accessed and it's value is returned only if `$x` is not null.
Similarly, the indexer is used only if `$x` is not null.
If `$x` is null, then null is returned.

The `?.` and `?[]` operators are member access operators and do not allow a space in between the variable name and the operator.

PowerShell allows `?` as part of the variable name.
Hence disambiguation is required when the operators are used without a space between the variable name and the operator.
To disambiguate, the variables must use `{}` around the variable name like: `${x?}?.propertyName` or `${y}?[0]`.

### Updates to operator precedence

The operator precedence token flags will be updated to add gaps and place the null coalesce opeator at position 7, above comparison and below add/subtract.
Currently, the binary operator mask is set at `0x07`, which will be updated to `0x0f`.
This will ensure that `??` is evaluated before operators like `-eq`, `-and`, `-bor` etc, but after `+` and `-`.

This enables scenarios like:
```PowerShell
'x' -eq $null ?? 'x' evaluates to 'x' -eq ($null ?? 'x')

 2 + $null ?? 3 evaluates to (2 + $null) ?? 3
```

### Additional considerations

The `??=` and `??` operators are binary operators and check for the value of the left hand side to be null or an undefined variable.
They can be used with a space between the variable name and the to operator.
If the left hand side is null, then `??=` assigns the right hand side to the variable while, the `??` operator returns the right hand side.
When they are used without a space between variable name and the operator, a `{}` must be used around the variable name.

## Alternate Proposals and Considerations

### Disallow `?` in variable names

It was considered to disallow `?` in variable names, to avoid confusion whether the operator is a null conditional operator or the variable name has `?` in it.
This proposal was rejected to maintain backward compatibility.

### Prefer new operators when tokenizing variable names

It was considered to look ahead when tokenizing the variable names and check for `??=`, `??`, `?.` or `?[]`.
If the token is found the assume that it is a null conditional check and not a member access of a variable name ending in `?`.
To maintain backward compatibility and this proposal was rejected.

### Use a different sigil for null-conditional operators

It was determined that the familiarity of `?.` and `??` operators to `C#` programmers will make usage of the new operators easier to understand than introducing new sigils
