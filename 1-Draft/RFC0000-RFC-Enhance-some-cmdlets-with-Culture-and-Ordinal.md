---
RFC: RFC0000
Author: Ilya Sazonov
Status: Draft
Area: Commands
Comments Due: 11/10/2019
---

# Introduction

Historically, PowerShell has only used linguistic (`CurrentCulture`) and case-insensitive in many places to make string comparisons.
Goal of the RFC is to add more flexibility and performance in cmdlets based on string comparison operations.

## Motivation

In some scenarios PowerShell users need to do string comparisons in different culture than current culture.

Also sometimes PowerShell users need to do non-linguistic, binary/ordinal, comparisons.

Such scenarios would be communicating with localized applications and searching in log files.

Users want to get performance comparable to utilites like `grep`, `egrep` and `ripgrep`.
In the RFC context it is achieved by `Ordinal` and `OrdinalIgnoreCase` C# string comparison.

## Specification

### Affected cmdlets

- `Compare-Object`
- `Group-Object`
- `Sort-Object`
- `Select-Object`
- `Select-String`

### Proposal

#### `Culture` parameter

Add `Culture` parameter to all affected cmdlets if absent.

`Culture` parameter accepts C# `CultureInfo` type, `Ordinal`, `Invariant` and `Current` values.
It is default parameter set for backward compatibility and default value is `Current` (C# `CurrentCulture`).

#### Additional considerations

C# Regex implementation works only with current culture so new `Culture` parameter can only work with `SimpleMatch` parameter.

To address the Regex limitation we could consider a custom __simple__ Regex implementation in another RFC.

#### Examples

```powershell
Select-String -SimpleMatch -Culture ru-RU -CaseSensitive      # lingustic as expected for Culture term
Select-String -SimpleMatch -Culture ru-RU                     # lingustic

Select-String -SimpleMatch -Culture Ordinal -CaseSensitive    # non-lingustic - corresponds to C# `Ordinal`
Select-String -SimpleMatch -Culture Ordinal                   # lingustic in .Net Core 3.* - corresponds to C# `OrdinalIgnoreCase`
                                                 # It will be non-lingustic in .Net Core 5.* (Simple Case Folding). See https://github.com/dotnet/corefx/issues/41333 .
Select-String -SimpleMatch -Culture Invariant -CaseSensitive  # lingustic
Select-String -SimpleMatch -Culture Invariant                 # lingustic - corresponds to `InvariantCultureIgnoreCase`
```

##### Pester examples

```powershell
"file" | Select-String -Pattern "file" -Culture "tr-TR" -SimpleMatch | Should -BeExactly "file"
"file" | Select-String -Pattern "fIle" -Culture "tr-TR" -SimpleMatch | Should -BeNullOrEmpty
"file" | Select-String -Pattern "fIle" -Culture "tr-TR" -SimpleMatch -CaseSensitive | Should -BeNullOrEmpty
```
