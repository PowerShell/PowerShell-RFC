---
RFC: 0003
Author: Jason Shirk
Status: Draft
Area: Strict Mode
Comments Due: 3/31/2016
---

# Strict Mode

Strict mode is a feature of PowerShell designed to catch scripting errors at runtime.
It has been present since V1.
In V1, it was a runspace wide setting controlled by the command `Set-PSDebug`.
V2 added the command `Set-StrictMode` which is dynamically scoped,
so it is possible to turn on strict mode is more limited fashion.

Scripts and modules that enable strict mode often must disable strict mode
when calling a ScriptBlock or command that was not tested with strict mode.
Strict may report errors in the called commands and these errors are not always bugs.
It is possible to write good PowerShell that reports an error in strict mode.

## Motivation

This proposal suggests a way to make strict mode more useful so that more scripts and modules
can turn it on and leave it on without worrying about the commands or ScriptBlocks that they call.

## Specification

Global or dynamic scoping of strict mode causes problems for disciplined script authors
who attempt to leave strict mode on for their scripts/modules and call scripts written by less disciplined authors.
Today, a disciplined script author must typically disable strict mode when releasing the scripts.

This RFC proposes a way of specifying strict mode to apply lexically:

```PowerShell
using strict

param([ScriptBlock]$sb)

# Invoked script block not run in strict mode.
& $sb

# Error when property doesn't exist
$sb.MissingProperty 
```

Like all other using statements, `using strict` must appear before any code in the file.
The syntax maybe be extended to provide finer grained control over which strict mode diagnostics are enabled.
As proposed, it would enable all strict mode errors (like `Set-StrictMode -v Latest`).

### Alternate Proposals and Considerations

#### Change Set-StrictMode

We could change `Set-StrictMode` to work lexically.
This option is attractive as it requires no new syntax.

It would "break" people who rely on it working from the global scope,
for example when they set it in their profile.
 
This could also cause problems when running a script or module on an older
version of PowerShell where `Set-StrictMode` uses dynamic scoping instead of lexical scoping.

#### Open Questions

* Should `Set-StrictMode -Off` have any effect?
* Should there be an easy way to use lexical strict mode to apply to a module that consists of multiple ps1 files?
Maybe a module manifest entry?  