---
RFC: RFC0000
Author: Ilya Sazonov
Status: Draft
Area: Commands
Comments Due: 05/30/2019
---

# `CurrenCulture` and comparisons

Historically, PowerShell has used `CurrenCulture` and case-insensetive in many places to make string comparisons.

## Motivation

In some scenarios PowerShell users need to do string comparisons in different culture than current culture.

Also sometimes PowerShell users need to do Invariant culture and binary/ordinal comparisons.

Such scenarios would be communicating with localized applications and searching in log files.

## Specification

### Affected cmdlets

- `Compare-Object`
- `Group-Object`
- `Sort-Object`
- `Select-Object`
- `Select-String`

### Proposal

There are two proposals - first is more simple for script writers and backward compatibility and the other for developers.

### Proposal

For `Culture` parameter add to C# `CultureInfo` type values `Invariant` and `Ordinal` values.

Users continue to use `CaseSensitive` parameter.

The proposal leads to the fact that developers are forced to use different API in scripts and C#.

## Alternate Proposal

Use C# `CultureInfo` type for `Culture` parameter and introduce `Comparison` parameter corresponding to C# `StringComparison` type
(with values - Ordinal, OrdinalIgnoreCase, CurrentCulture, CurrentCultureIgnoreCase, InvariantCulture, InvariantCultureIgnoreCase and perhaps SimpleCaseFolding).

That means deprecating `CaseSensitive` parameter. It is not removed for backward compatibility but hided.

With the proposal developers use the same API for both script and C# development without confusing.
