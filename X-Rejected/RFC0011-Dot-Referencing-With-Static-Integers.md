---
RFC: 0011
Author: Kirk Munro
Status: Rejected
SupercededBy: N/A
Version: 0.1
Area: Parsing Literal Numbers
Comments Due: November 4, 2016
Feedback: https://github.com/PowerShell/PowerShell-Language-RFC/issues/38
---


# Parsing literal Numbers

PowerShell has multiple inconsistencies in the parser when parsing literal
numbers. Literal numbers include any of the following:

```PowerShell
3
3.
3.14
1e10
1e+10
1e-10
3l
3.l
3.14l
1e10l
1e+10l
1e-10l
3d
3.d
3.14d
1e10d
1e-10d
1e+10d
0xabcd
0xabcdl
```

In addition, any of these literal numbers may be preceded by a sign (+/-), and
any of these literal numbers may have a byte-size unit (KB/MB/GB/TB/PB)
appended to the end. With all supported pieces in place, you can end up with a
literal number that looks like one of these:

```PowerShell
-1e+10lmb # Equal to -10485760000000000
0xabcdefgb # Equal to 12089661849600000
```

Since literal numbers (including their sign, type suffix, and byte-size unit)
evaluate to objects like any other, be they int32, int64, decimal, or double,
and since objects have property and method members, you can invoke a member by
dot-referencing the member name immediately at the end of the literal number,
except for int32 or int64 literal numbers that don't have a type suffix nor a
byte-size unit. That is to say, you can invoke members directly at the end of
_most_ literal number formats, but you can't invoke members directly at the end
of int32 or int64 literal numbers that don't have a suffix. These are parsed
and evaluated just fine today:

```PowerShell
-1e+10lmb.GetType()
0xabcdefgb.GetType()
2.0.Equals(2.0)
3l.CompareTo(1l)
Update-TypeData -TypeName int64 -MemberName days -MemberType ScriptProperty `
    -Value {New-TimeSpan -Days $this}
3l.days
```

These, however, are not:

```PowerShell
Update-TypeData -TypeName int -MemberName days -MemberType ScriptProperty `
    -Value {New-TimeSpan -Days $this}
2.days
2.GetType()
2.Equals(3)
```

This demonstrates an inconsistency in how PowerShell handles literal numbers.
The workaround is to wrap the literal numbers in brackets, in which case you
can invoke members just fine. For example this works:

```
(2).GetType()
```

That extra set of brackets shouldn't be necessary though, because it isn't
necessary for all of the other literal number formats. This is simply a design
bug in the PowerShell parser. In vanilla PowerShell this isn't a huge issue,
because int32 and int64 objects don't have many properties or methods on them
that you would want to use with literal values; however, since PowerShell has
a rich extensibility model, there are many opportunities to add properties and
methods to the extended type system that make great sense to use with literal
values, and it is with all of these that having to use round brackets gets in
the way (example: ```Get-EventLog -LogName System -After 2.weeks.ago```).

The inconsistency continues beyond simple invocation of members on literal
numerics. When PowerShell parses a script, if it finds a token that it
identifies as a number, whether that token is followed by a dot-reference of a
member or not, then PowerShell will treat it as a number. This means that if
you create a command whose name is recognized by the parser as a literal numeric
value or as a literal numeric value followed by a dot-reference to a member, then
you will not be able to invoke that command unless you use the call operator or
the dot-source operator. The reason for this is that the parser checks for
literal numeric values before it checks for commands. This is the case for every
format of literal numeric values, except for integers that are not followed by a
type suffix nor by a byte-size unit. This results in some unexpected behaviour
due to the inconsistency.

For example, you could create the following functions:

```
function 1a.LoadModule {...}
function 1b.CreateUser {...}
function 1c.AddUserToGroup {...}
function 1d.GetGroupMembers {...}
```

Once these are created, you can invoke each of these functions normally except
for the last one because the parser recognizes it as a literal decimal value
followed by a member invocation. With strict mode turned off, this invocation
would simply return nothing to the user. Regardless of where the command comes
from (batch file, executable program, PowerShell function or alias), if the
command is parsable as a literal number or as a literal number with a member
invocation, then invoking that command requires a call operator (unless the
literal number happens to be an integer -- then it works today due to the design
issue mentioned earlier).

This RFC is about correcting these inconsistencies.

## Motivation

As a PowerShell user, I can invoke members on every literal numeric value the
same way, with simplicity and elegance and without having to consider the type
of numeric value, so that my user experience remains consistent and reliable by
making sure that members on literal numeric values are easily discoverable and
accessible.

## Specification

The proposal is to fix the parser in PowerShell Core such that decimal literal
integer values followed by a dot-reference of a member are properly recognized
as just that, bringing the integer format in line with the rest of the numeric
formats that are used in PowerShell. With the appropriate changes in place,
users will be able to dot-reference any member after any literal numeric value,
regardless of the format of that literal numeric value.

Examples:

Invoking an ETS property on a decimal literal integer value:
```PowerShell
PS C:\> Update-TypeData -TypeName int -MemberName weeks -MemberType ScriptProperty -Value {New
-TimeSpan -Days ($this*7)} -Force
PS C:\> 2.weeks


Days              : 14
Hours             : 0
Minutes           : 0
Seconds           : 0
Milliseconds      : 0
Ticks             : 12096000000000
TotalDays         : 14
TotalHours        : 336
TotalMinutes      : 20160
TotalSeconds      : 1209600
TotalMilliseconds : 1209600000

PS C:\>
```

Invoking a command with a name that parses as a decimal literal integer value
that dot-references a member: 
```PowerShell
PS C:\> function 4.StartService {Start-Service wuauserv}
PS C:\> 4.StartService # Nothing happens because it's not parsed as a command
PS C:\>
```

Pros: 
 + Ensures consistency among literal numeric value use regardless of the format,
 whether you're invoking members or commands that look like numeric values
 followed by member invocations.  
 + Prefers simplicity and elegance over unnecessary syntax complexity.

Cons:
 + Would break commands whose name evaluates as a decimal literal integer value
 followed by a dot-reference of a member

## Alternate Proposals and Considerations

### Warn on command invocations that cannot be invoked without call/dot-source

As an addition to the proposal above, the parser could also identify commands
that are hidden by literal numeric values followed by a dot-reference of a
member and warn the user about the hidden commands.

Pros:
 + Improves discoverability and makes it easier for users to invoke commands
 that otherwise require invocation with a call operator or a dot-source
 operator (similar to how you are warned when you try to invoke a script
 without an absolute or relative path).

Cons:
 + Requires more parsing that may be unnecessary (how common is it for users to
 invoke commands whose name would be parsed as a numeric value or as a numeric
 value followed by a dot-reference of a member?)

### Move the consistency needle the other way

As an alternative of the proposal above, the consistency needle could be moved
the other way, requiring all literal numeric values to be wrapped in brackets in
order to dot-reference members on them.

Pros:
 + Keeps things consistent, but at a high cost.

Cons:
 + Risks breaking much more code.
 + Adds syntax complexity that shouldn't be necessary.

### Provide a way to define units for numeric values

This isn't really an alternative, because the proposed changes are required for
proper method invocation on a decimal literal integer value -- it's more of an
extension of the idea. It may be interesting to specifically allow the
definition of units that would be automatically supported on literal numeric
values that do not already have a unit added to them, where unit names (which
are simply a series of characters such as KB, MB, days, years, weeks, etc)
would be associated with a multiplier/converter script that would automatically
resolve the numeric value with the unit in the appropriate type (based on the
script).

Pros:
 + Eliminates the magic in KB, MB, GB, TB, and PB units and provides a unit
 extension point in PowerShell.

Cons:
 + Since this is an extension of the main idea, it would probably be best to
 consider it in a completely separate RFC.
  
## Additional Details

If this RFC gets accepted, as an added benefit to the PowerShell language it
would be possible to implement units on any numeric constant as a property of
the appropriate numeric type. Since PowerShell v1, there has been support for
units of KB, MB, GB, TB, and later PB was added as well. The way this support
is implemented has always been parser magic, meaning that the parser has extra
internal logic hardcoded to apply these units to literal numeric values that it
encounters. These are not the only units that are desirable in the PowerShell
scripting language. This change would allow for ETS extensions to define units
for numeric types that are then accessible as properties (invoked using a dot-
reference operator). For example, time units to define time spans (seconds,
minutes, hours, days, weeks, months, years). You can see these units defined in
the TypePx module. The existing limitation of having to wrap integers in
brackets hurts the end user experience when applying these units in a script.
Fixing this inconsistency would allow for easier invocation of these units, and
would encourage such units to be invoked as members (which you can do with KB,
MB, GB, TB, and PB today -- e.g. 2.MB). This would be a better way to implement
units in PowerShell, with less magic and more applied logic that can be used
in a repeatable, consistent manner. 

Also, as a learning exercise, I decided to figure out how to fix this in the
PowerShell Core project over the weekend. If you'd like to try the fix out,
or if you would like to see the changes required for this fix, visit
https://github.com/PowerShell/PowerShell/compare/master...KirkMunro:parsing-static-integers

---------------
## PowerShell Committee Decision

### Voting Results

Jason Shirk: Reject 

Joey Aiello: Reject

Bruce Payette: Absent

Steve Lee: Reject

Hemant Mahawar: Reject

### Majority Decision

It is not uncommon to have files starting with numbers to maintain a list of a sequence of files to execute/open.
For example:
```
1.Setup.ps1
2.Demo.ps1
3.Cleanup.ps1
```

On Linux, files may not have extensions.
Implementing this RFC will break this usage and can introduce other unintended effects without introducing significant improvement to the experience (removing need for parenthesis for calling integers as objects).

### Minority Decision

N/A
