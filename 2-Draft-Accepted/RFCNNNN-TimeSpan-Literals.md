---
RFC: RFC<four digit unique incrementing number assigned by Committee, this shall be left blank by the author>
Author: Anselm Schüler
Status: Draft
Area: Syntax and Cmdlet implementation
Comments Due: <?>
Plan to implement: <Would like to, but I'm not sure if I'm able to>
---

# `TimeSpan` Literals and support for `TimeSpan` objects in Cmdlets such as Start-Sleep

A new syntax expansion that adds support for `TimeSpan` literals, and cmdlets such as `Start-Sleep` accepting `TimeSpan` objects by default - this entails adding new suffixes to numeric literals as [described here](https://docs.microsoft.com/en-us/powershell/module/Microsoft.PowerShell.Core/About/about_numeric_literals?view=powershell-7).  
This is helful as current methods for time specification have significant drawbacks (see Motivation).  
This solution is significantly more concise than using `New-TimeSpan` (see User Experience).  
The semantics of this method are more clear, as cmdlets with the `New` verb are usually used in variable assignments, i.e. for persistent objects.

## Motivation

Cmdlets such as `Start-Sleep` offer both `-Seconds` and `-Milliseconds` as parameters.
This style of time input is functional, but has a few issues:
- Expanding the number of units requires updating the argument handling and the function body itself
- Implementation in custom scripts or cmdlets requires boilerplate
- Available options may not be consistent across cmdlets

## User Experience

Example of expressions and their equivalents in current PowerShell:

| New syntax | With `New-TimeSpan` | With type conversion |
| --- | --- | --- |
| `10sec` | `New-TimeSpan -Seconds 10` | `[TimeSpan]100000000` |
| `15min` | `New-TimeSpan -Minutes 15` | `[TimeSpan]9000000000` |
| `0xFFyear` | `New-TimeSpan -Days (0xFF * 365)` | `[TimeSpan]80416800000000000` |

Example of user interaction:

```
Start-Sleep 20sec
```
Result: Sleep for 20 seconds

---

```
Start-Sleep 5min
```
Result: Sleep for 300 seconds

---

```
Start-Sleep 100ms
```
Result: Sleep for 0.1 seconds

---

```
$TimeSpan = 15sec
$TimeSpan
```
```

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

```
20dy
```
```

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

The following suffixes would now be recognized as `TimeSpan` objects:
- `min` minutes
- `sec` seconds
- `ms` milliseconds
- `h` hours
- `dy` days

These suffixes are not combinable with multiplier suffixes or type suffixes.

The `Start-Sleep` cmdlet is expanded to allow a `-TimeSpan` parameter, where a `TimeSpan` object can be passed.

The behaviour of `Start-Sleep` when only one anonymous parameter is given is changed such that a non-`TimeSpan` object is interpreted as seconds and a `TimeSpan` object is interpreted as argument to parameter `-TimeSpan`.

## Alternate Proposals and Considerations

These suffixes ideally do not interfere with the existing suffixes, as the types of the numbers would already be determined by the time unit. For example, `tick` will always be `Int64` (or in case of future changes the type used for `TimeSpan`'s `Ticks` property)

A year unit could be added, however, this requires deciding on the length of a year, which may be contentious.

## Related

Related proposal on the PowerShell issues page:  
https://github.com/PowerShell/PowerShell/issues/10712

Original proposal thread on the PowerShell issues page:  
https://github.com/PowerShell/PowerShell/issues/12305
