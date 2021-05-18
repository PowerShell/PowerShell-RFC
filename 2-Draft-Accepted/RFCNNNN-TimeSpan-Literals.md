---
RFC: RFCNNNN
Author: Anselm Schüler
Status: Draft
Area: Syntax and Cmdlet implementation
Plan to implement: Yes
Breaking change: Yes
---

# `TimeSpan` Literals and support for `TimeSpan` objects in Cmdlets such as Start-Sleep

A new syntax expansion that adds support for `TimeSpan` literals, and cmdlets such as `Start-Sleep` accepting `TimeSpan` objects by default - this entails adding new suffixes to numeric literals as [described here](https://docs.microsoft.com/en-us/powershell/module/Microsoft.PowerShell.Core/About/about_numeric_literals).  
This is helpful as current methods for time specification have significant drawbacks (see Motivation).  
This solution is significantly more concise than using `New-TimeSpan` (see User Experience).

Initially, this will be implemented in an experimental feature.

## Motivation

### Practicality

Cmdlets such as `Start-Sleep` offer both `-Seconds` and `-Milliseconds` as parameters.
This style of time input is functional, but has a few issues:

- Expanding the number of units requires updating the argument handling and the function body itself
- Implementation in custom scripts or cmdlets may require boilerplate
- Available options may not be consistent across cmdlets

### Semantics

Commands such as `New-TimeSpan`, which is the premier way to create timespan objects, are usually used in a manner where the returned object is a persistent object that later gets mutated or is itself effectful.  
By using a suffix, the intent is made clearer by the more functional style.

## User Experience

Example of expressions and their equivalents in current PowerShell:

| New syntax | With `New-TimeSpan` | With type conversion |
| --- | --- | --- |
| `10sec` | `New-TimeSpan -Seconds 10` | `[TimeSpan]100000000` |
| `15min` | `New-TimeSpan -Minutes 15` | `[TimeSpan]9000000000` |
| `0xFFyear` | `New-TimeSpan -Days (0xFF * 365)` | `[TimeSpan]80416800000000000` |

### Examples of user interaction

```pwsh
Start-Sleep 20sec
```

Result: Sleep for 20 seconds

---

```pwsh
Start-Sleep 5min
```

Result: Sleep for 300 seconds

---

```pwsh
Start-Sleep 100ms
```

Result: Sleep for 0.1 seconds

---

```pwsh
Start-Sleep (1min + 30sec)
```

Result: Sleep for 90 seconds

---

```pwsh
Start-Sleep (2min + 35sec)
```

Result: Sleep for 155 seconds

---

```pwsh
$TimeSpan = 15sec
$TimeSpan
```

```none

Days              : 0
Hours             : 0
Minutes           : 0
Seconds           : 15
Milliseconds      : 0
Ticks             : 150000000
TotalDays         : 0.000173611111111111
TotalHours        : 0.00416666666666667
TotalMinutes      : 0.25
TotalSeconds      : 15
TotalMilliseconds : 15000


```

---

```pwsh
20day
```

```none

Days              : 20
Hours             : 0
Minutes           : 0
Seconds           : 0
Milliseconds      : 0
Ticks             : 17280000000000
TotalDays         : 20
TotalHours        : 480
TotalMinutes      : 28800
TotalSeconds      : 1728000
TotalMilliseconds : 1728000000


```

## Specification

The following suffixes would now be recognized and be turned into `TimeSpan` objects:

| Suffix    | Name          | Method Name           | Seconds   |
|-----------|---------------|-----------------------|-----------|
| `day`     | Days          | `FromDays`            | 86400     |
| `h`       | Hours         | `FromHours`           |  3600     |
| `min`     | Minutes       | `FromMinutes`         |    60     |
| `sec`     | Seconds       | `FromSeconds`         |     1     |
| `ms`      | Milliseconds  | `FromMilliseconds`    |     0.001 |
| `tick`    | Ticks         | `FromTicks`           |           |

These suffixes are not combinable with multiplier suffixes or type suffixes.

The `Start-Sleep` cmdlet is expanded to allow a `-TimeSpan` parameter, where a `TimeSpan` object can be passed.

The behaviour of `Start-Sleep` when only one anonymous parameter is given is that a non-`TimeSpan` object is interpreted as an argument to parameter `-Seconds` and a `TimeSpan` object is interpreted as argument to parameter `-TimeSpan`.

All of this is only enabled when an experimental feature called `TimeSpanLiterals` is enabled.

## Related

[Related proposal on the PowerShell issues page](https://github.com/PowerShell/PowerShell/issues/10712)

[Original proposal thread on the PowerShell issues page](https://github.com/PowerShell/PowerShell/issues/12305)
