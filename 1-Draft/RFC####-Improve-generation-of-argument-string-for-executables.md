---
RFC: RFC
Author: Timo Schwarte
Status: Draft
Version: 1.0
Area: Calling external executables
Comments Due: 2017-06-31
---

# Improve generation of argument-string for executables

When Powershell calls an external executable, the
[NativeCommandParameterBinder](https://github.com/PowerShell/PowerShell/blob/master/src/System.Management.Automation/engine/NativeCommandParameterBinder.cs)
takes all parameters of the command and generates the argument-string (also called "CommandLine") that is sent to the executable (as [ProcessStartInfo.Arguments](https://msdn.microsoft.com/en-us/library/system.diagnostics.processstartinfo.arguments.aspx)).
It seems as if the NativeCommandParameterBinder trys to generate an argument-string, such that `argv[]` contains the arguments, that were given as parameters to the command. This works well, if an argument contains spaces. In this case the NativeCommandParameterBinder adds quotes arround the argument, so it reaches `argv[]` as one argument (not split into parts).
However if the argument itself contains quotes, those are not escaped. Therefor the corrosponding element in `argv[]` has no quotes and (depending on the actual argument) the argument can be splitted into multiple `argv[]` elements and it can even occur that the following arguments are not handled correctly.

This RFC suggests to make the NativeCommandParameterBinder compatible to
[the typical CommandLine escaping rules](https://msdn.microsoft.com/en-us/library/17w5ykft.aspx).
Additionally it suggests to add a verbose-symbol, to add a custom formated string to the CommandLine (for executables, that don't follow the typical escaping rules.)

This is basically a bugfix for https://github.com/PowerShell/PowerShell/issues/1995 and https://github.com/PowerShell/PowerShell/issues/3049 and on [stackoverflow](http://stackoverflow.com/a/21334121/2770331) (see `and I think this is a bug` within that answer).
However, as this is a very longstanding bug, there are workarounds for some cases. These will no longer work if this is corrected, therefor this document suggests to add a preference variable to get the old behaviour back if somebody needs it.

## Motivation

    As a powershell user who wants to call external executables,
    I can pass any arguments to those executable,
    so that these arguments reach the `argv[]` array unchanged.

## Specification

## Alternate Proposals and Considerations
