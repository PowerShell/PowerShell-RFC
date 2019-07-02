---
RFC: RFC0000
Author: Ilya Sazonov
Status: Draft
Area: Commands
Comments Due: 05/30/2019
---

# Introduction

Historically, PowerShell has only used linguistic (`CurrentCulture`) and case-insensitive in many places to make string comparisons.
Goal of the RFC is to add more flexibility and performance in cmdlets based on string comparison operations.

## Motivation

In some scenarios PowerShell users need to do string comparisons in different culture than current culture.

Also sometimes PowerShell users need to do non-linguistic (Invariant culture) and binary/ordinal comparisons.

Such scenarios would be communicating with localized applications and searching in log files.

Users want to get performance comparable to utilites like `grep`, `egrep` and `ripgrep`.

## Specification

### Affected cmdlets

- `Compare-Object`
- `Group-Object`
- `Sort-Object`
- `Select-Object`
- `Select-String`

### Proposal

#### `Culture` and `Comparison` parameters

Add `Culture` and `Comparison` parameters in different parameter sets to all affected cmdlets if absent.

`Culture` parameter accepts C# `CultureInfo` type values.
It is default parameter set for backward compatibility and default value is `CurrentCulture`.

`Comparison` parameter accepts `Ordinal`, `Invariant` and `SimpelCaseFolding` values.

Users continue to use `CaseSensitive` parameter with both `Culture` and `Comparison` parameters.
Default is case-insensetive for backward compatibility.

##### Samples

```powershell
Select-String -Culture ru-RU -CaseSensetive         # lingustic as expected for Culture term
Select-String -Culture ru-RU                        # lingustic

Select-String -Comparison Ordinal -CaseSensetive    # non-lingustic
Select-String -Comparison Ordinal                   # lingustic - OrdinalIgnoreCase
Select-String -Comparison Invariant -CaseSensetive  # lingustic
Select-String -Comparison Invariant                 # lingustic - InvariantCultureIgnoreCase
Select-String -Comparison SimpleCaseFolding         # non-lingustic
```

#### `FastMatch` parameter

Add `FastMatch` parameter to `Select-String`.

The new parameter will invoke a simple Regex engine.

The engine is non-linguistic and based on SimpleCaseFolding like `ripgrep` utility.

The engine implements limited subset of Regex standard - "^","$",".","*","+","\d","\r","\n","\t","\w" without backtracking.

It is expected that the engine will be 5-10 times faster than full-featured .Net Regex.

##### Samples

```powershell
Select-String -Pattern "^Error.+count" -SimpleMatch      # Old. Linguistic - current default is CurrentCulture
Select-String -FastMatch "^Error.+count"                 # New. Non-linguistic - based on SimpleCaseFolding. Very fast.

Select-String -Pattern "Error" -SimpleMatch | Select-String -Pattern "^Error.+count"   # Old.
Select-String -FastMatch "^Error (.+) count" -Pattern "<Full Regex>"   # New. First do fast search a candidate string in large file
                                                                       # then parse it with full-featured Regex
```

#### `AsByteStream` parameter

A separate problem is that the `grep` / `ripgrep` utilities have an option to interpret the input stream from the file as byte ("binary" in their terms) which can be useful. `FileSystemProvider` offers us the `AsByteStream` parameter, which can also be useful here.
This parameter has been separated from the `Encoding` parameter. This parameter is used in Select-String but not in other above mentioned cmdlets. Perhaps we should unify the cmdlets to use `Encoding` and `A`sByteStream` wherever `Path` / `LiteralPath` parameters are used.

```powershell
Select-String -Path source.txt -AsByteStream -SimpleMatch "qwerty"
Select-String -Path source.txt -Encoding Utf8 -SimpleMatch "qwerty"
```

# Alternate Proposal

Use C# `CultureInfo` type for `Culture` parameter and introduce `Comparison` parameter corresponding to C# `StringComparison` type
(with values - Ordinal, OrdinalIgnoreCase, CurrentCulture, CurrentCultureIgnoreCase, InvariantCulture, InvariantCultureIgnoreCase and perhaps SimpleCaseFolding) in the same parameter set.

That means deprecating `CaseSensitive` parameter. It is not removed for backward compatibility but hided.

With the proposal developers use the same API for both script and C# development without confusing.
