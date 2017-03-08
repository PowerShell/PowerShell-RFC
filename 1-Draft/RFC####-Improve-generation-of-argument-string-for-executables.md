---
RFC: RFC
Author: Timo Schwarte
Status: Draft
Version: 1.0
Area: Calling external executables
Comments Due: 2017-XX-XX
---

# Improve generation of argument-string for executables

When Powershell calls an external executable, the
[NativeCommandParameterBinder](https://github.com/PowerShell/PowerShell/blob/master/src/System.Management.Automation/engine/NativeCommandParameterBinder.cs)
takes all parameters of the command and generates the argument-string (also called "CommandLine") that is sent to the executable (as [ProcessStartInfo.Arguments](https://msdn.microsoft.com/en-us/library/system.diagnostics.processstartinfo.arguments.aspx)).
It seems as if the NativeCommandParameterBinder trys to generate an argument-string, such that `argv[]` of the called executable contains the arguments, that were given as parameters to the command. This works well, if an argument contains spaces. In this case the NativeCommandParameterBinder adds quotes arround the argument, so it reaches `argv[]` as one argument (not split into parts).
However if the argument itself contains quotes, those are not escaped. Therefor the corrosponding element in `argv[]` has no quotes and (depending on the actual argument) the argument can be splitted into multiple `argv[]` elements and it can even occur that the following arguments are not handled correctly.

This RFC suggests to make the NativeCommandParameterBinder compatible to
[the typical CommandLine escaping rules](https://msdn.microsoft.com/en-us/library/17w5ykft.aspx).
Additionally it suggests to add a verbose-symbol, to add a custom formated string to the CommandLine (for executables, that don't follow the typical escaping rules.)

This is basically a bugfix for https://github.com/PowerShell/PowerShell/issues/1995 and https://github.com/PowerShell/PowerShell/issues/3049 and on stackoverflow (["This seems like a bug to me. If I am passing the correct escaped strings to PowerShell, then PowerShell should take care of whatever escaping may be necessary for however it invokes the command."](http://stackoverflow.com/questions/6714165/powershell-stripping-double-quotes-from-command-line-arguments)
and ["... and I think this is a bug Powershell doesn't escape any double quotes that appear inside the arguments."](http://stackoverflow.com/a/21334121/2770331)).
However, as this is a very longstanding bug, there are workarounds for some cases. These will no longer work if this is corrected, therefor this document suggests to add a preference variable to get the old behaviour back if somebody needs it.

## Motivation

    As a powershell user who wants to call external executables,
    I can pass any arguments to those executable,
    so that these arguments reach the `argv[]` array unchanged.

## Specification

### Quoting

The decision if an argument needs to be quoted, will be simplified: Any Argument, that contains `"`, `'` or a character that matches `char.IsWhiteSpace` will be quoted. To quote an argument (compatible to [MSVC rules](https://msdn.microsoft.com/en-us/library/17w5ykft.aspx) and `CommandLineToArgvW`):
- Every occurrence of N times `\` followed by `"` will be replaced by (2*N+1) times `\` followed by `"`. (N &#x2208; {0,1,2,...})
- N times `\` at the end of the string is replaced by (2*N) times `\`. (N &#x2208; {0,1,2,...})
- `"` is added to the beginning and to the end of the string

### Verbatim Argument

TODO:
`--=` next argument is copied to cmdline as is, symbol detected in parser

### Preference Variable

## Alternate Proposals and Considerations

### Other shells

TODO:

Therefore Powershell should build the Commandline in a way, that matches those rules instead of just sometimes adding quotes around arguments.
Almost every other modern Commandshell on windows does this:
- https://www.hamiltonlabs.com/Cshell.htm (not documented)
- [Cygwin](https://cygwin.com/git/gitweb.cgi?p=newlib-cygwin.git;a=blob;f=winsup/cygwin/winf.cc;hb=HEAD) (see `linebuf::fromargv`)
- [Python](https://svn.python.org/projects/python/trunk/Lib/subprocess.py) (see `list2cmdline`)
- [Tcl](https://github.com/tcltk/tcl/blob/master/win/tclWinPipe.c) (see `BuildCommandLine`, Comment: "N backslashes followed a quote -> insert N * 2 + 1 backslashes then a quote.")
### Accepted

TODO:
msvc, mingw, .net, winapi, .netcore on linux

### Batch files

### Linux
TODO:
Not Part of this RFC:
- might depricate `--%` and `--=` on linux
- might add `--$` to only evaluate `$...` and string literals, but treat everything else as verbose text arguments
