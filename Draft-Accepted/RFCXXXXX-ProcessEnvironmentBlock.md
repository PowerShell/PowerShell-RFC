---
RFC: RFCNNNN-ProcessEnvironmentBlock.md
Author: Jordan Borean
Status: Draft
SupercededBy:
Version: 1.0
Area: engine
Comments Due: March 1, 2025
Plan to implement: Yes
---

# Process Environment Blcok

## Motivation

    As a powershell user,
    I want to be able to specify environment variables to set when running a sub process like in bash,
    so that I don't have to manually unset them after or impact parallel pipelines running.

## User Experience

A common scenario on sh based shells is to specify environment variables inline with the command so that those environment variables are only set on the new process.
For example the following will set the env vars `FOO` and a modified `PATH` for the `command` subprocess without affecting the current sh shell.

```bash
FOO=bar PATH=abc:$PATH command
```

There is no equivalent syntax in PowerShell and the only way to achive something similar is to manually set the environment variables and unset them after:

```powershell
$oldPath = $env:PATH

$env:FOO = 'bar'
$env:PATH = "abc:$env:PATH"
command
$env:FOO = $null
$env:PATH = $oldPath
```

There are some downsides with this approach, for example:

+ Environment variables are process wide so will affect parallel pipelines
+ You need to ensure you unset the env vars after running the command
  + This is complicated further when dealing with `PATH` like env vars as now you need to save the old value

The first is problematic when you start using `ForEach-Object -Parallel` or other multithreading mechanisms in PowerShell because each environment set/unset action will affect the other pipelines running at the same time causing unexpected results.

A way around this is to spawn a subshell to set the env vars but this is wasteful as now a new intermediary process is needed and the user also needs to find a way to smuggle the environment variables to set and command arguments to that intermediary process.
A very simple example would be something like this

```powershell
pwsh {
    $env:FOO = 'bar'
    $env:PATH = "abc:$env:PATH"
    command
}
```

It gets more coplicated when you start having to deal with using variables in the subscope which is a common pattern.

## Specification

A solution to this problem is to take advantage of the call operator and to check if the first argument is a hashtable that specifies the environment block to set.
For example, the bash command above can now be called in PowerShell in a few different ways:

```powershell
# All in one line
& @{FOO = 'bar'; PATH = "abc:$env:PATH"} command

# Span multiple lines
& @{
    FOO = 'bar'
    PATH = "abc:$env:PATH"
} command

# Using var
$envBlock = @{
    FOO = 'bar'
    PATH = "abc:$env:PATH"
}
& $envBlock command

# Dynamically build env block
$envBlock = @{}
if ($IsWindows) {
    $envBlock.ENV_VAR_ONLY_FOR_WINDOWS = 'foo'
}
& $envBlock command
```

The advantages of this approach is that:

+ No Ast/parsing changes are needed
+ The hashtable can be built at runtime and not parse time
+ It is simple to spread this across multiple lines without dealing with backticks or splatting
+ The syntax for builtin the hashtable is a common pattern in PowerShell so no special patterns are needed here
+ It is possible to use variables as part of the key/variable or dynamically build the hashtable at runtime
+ The current behaviour of `& @{FOO = 'bar'} command` would require a user having a command called `System.Collections.Hashtable` to work which is very unlikely
  + If we were really concerned we could check the count of the hashtable entries as `@{}` and `@{FOO = 'bar'}` with the latter only being the one to use this new mode

```powershell
Function System.Collections.Hashtable { "Function: $args" }
& @{FOO = 'bar'} test

# Function: test
```

The actual implementation would require some special logic to:

+ See if the command type is a `Hashtable` and to use the second argument as the actual command to run
+ Fail if there is no second argument to specify the command to run
+ Potentially fail if the command is not an `Application` command type (native binary)
+ Implementation on Windows and non-Windows would be to set the specified env vars on the [ProcessStartInfo.Environment](https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.processstartinfo.environment?view=net-9.0) block
  + The `NativeCommandProcessor` currently uses this functionality for modifying the `PSModulePath` on Windows when calling `powershell.exe`
  + The same mechanism works on all platforms so there isn't any platform specific code

There are some potential downsides to this approach:

+ There is no way to achieve this without the call operator without Ast changes
  + I think this to be an upside but could be annoying from an interactive shell perspective
+ The syntax is a bit verbose compared to the sh equivalent
  + The tradeoff is added flexibility with support for dynamic hashtables, multiline spanning
+ It can only apply to native commands, we cannot set env vars for cmdlets/functions running in process which may confuse users

## Alternate Proposals and Considerations

An alternative proposal would be to support a similar syntax of sh based shells `FOO=bar command`.
Ast and parsing wise this is currently valid syntax so it could have a potential impact on existing scripts as show by the below example:

```powershell
Function FOO=abc { "Function: $args" }

FOO=abc BAR=def test
# Function: BAR=def test
```

While a similar concern exists for `& @{}`, the conflict with `FOO=abc` is based on the value being specified whereas the hashtable command would have always been `System.Collections.Hashtable` reducing the chances of someone actually using that.
For example both `& @{}` and `& @{FOO='abc'}` will attempt to run `System.Collections.Hashtable` whereas `FOO=abc ...` will attempt to find the command `FOO=abc` whereas `BAR=abc ...` would attempt to find `BAR=abc`.
The other advantage is that using a hashtable restricts this usage to being used by the call operator only, this should reduce the chances of an existing conflict and reserve it for more specialised examples.

Other considerations to think off with this proposal are:

+ What should happen if the command being run with this hashtable syntax is not an `Application` type but a function or cmdlet.

Should it fail explicitly, should it ignore the values, should it splat the values as parameters to achieve inline splatting.
My recommendation would be to have it fail at least in the first implementation and reserve it for future purposes.

+ We need to support both empty values and case sensitive key conflicts

A hashtable by itself allows the value to be an empty string but case sensitive key conflicts are a bit more difficult.
It is possible to have a case sensitive key lookup for a hashtable when using the `[Hashtable]::new()` constructor to build it.

```powershell
$ht = [Hashtable]::new()
$ht.foo = '1'
$ht.FOO = '2'

$ht

# Name                           Value
# ----                           -----
# FOO                            2
# foo                            1
```

I would think that this scenario is quite rare and typically goes against the PowerShell ethos but as shown above it is possible to achieve with a hashtable if really neeeded.

+ Whether this would apply only to a `Hashtable` type or any subclasses of that type, or anything that implements `IDictionary`.

My recommendation would be to keep it just for a hashtable to keep the implementation simple.
