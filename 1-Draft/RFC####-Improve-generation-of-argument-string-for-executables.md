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

When the parser detects the 'Verbatim-Argument-Symbol' (`--=`), the next argument is copied to the argument-string exactly as it is, without escaping or adding quotes around the argument. The symbol is detected in the parser, so the string literal `'--='` or any other object that expands to that string is not interpreted as 'Verbatim-Argument-Symbol'.

### Preference Variable

If the Preference Variable `$PsUseLegacyInconsistentArgumentStringGeneration` is set to true, the quoting is done as it is now. (Only for compatibility with old Scripts). The Variable is false by default.

## Alternate Proposals and Considerations

### Other shells

Maybe this is not the strongest argument ever, but many other modern Commandshells on windows create the argument-string compatible to [these rules](https://msdn.microsoft.com/en-us/library/17w5ykft.aspx):
- https://www.hamiltonlabs.com/Cshell.htm (not documented, but escapes correctly)
- [Cygwin](https://cygwin.com/git/gitweb.cgi?p=newlib-cygwin.git;a=blob;f=winsup/cygwin/winf.cc;hb=HEAD) (see `linebuf::fromargv`)
- [Python](https://svn.python.org/projects/python/trunk/Lib/subprocess.py) (see `list2cmdline`)
- [Tcl](https://github.com/tcltk/tcl/blob/master/win/tclWinPipe.c) (see `BuildCommandLine`, Comment: "N backslashes followed a quote -> insert N * 2 + 1 backslashes then a quote.")

### [These rules](https://msdn.microsoft.com/en-us/library/17w5ykft.aspx) are widely accepted

Most Compilers on Windows generate executables, that split the CommandLine compatible to [those rules](https://msdn.microsoft.com/en-us/library/17w5ykft.aspx), for example the compiler shiped with VisualStudio as well as the `mingw` compiler suite.
The .Net runtime also splits the CommandLine in a way that is compatible to those rules. The Windows API function [`CommandLineToArgvW`](https://msdn.microsoft.com/en-us/library/windows/desktop/bb776391.aspx) is (although incorrectly documented) also compatible to those rules.
Therefore Powershell should build the Commandline in a way, that matches those rules instead of just sometimes adding quotes around arguments.

### Linux

While this change is important on windows, it's absolutely necessary on Linux: On Windows the CommandLine is splitted by the next executable and most executables follow the described rules. On Linux the CommandLine is not splitted by the called executable -- the .Net Core runtime splits the string. Therefor on Linux the described rules do not only apply to many calls of external executables, they apply to ALL calls of external executables. When the proposed changes are implemented, the arguments from within powershell always arive -- as expected -- as the `argv[]` array in called executables.

### Batch files

Sadly, one of the few exeptions to those typical parsing rules is `cmd.exe`. Because of this, one cannot reliably call batch files with arbitrary arguments. (This is no problem of powershell, it's a cmd design problem.) Some arguments are impossible -- an uneven number of double quotes can only be sent in the last argument. To my knowledge there is no clean way to deal with this, therefor I think using the typical escaping rules is still the way to go. In many cases this is the correct way, as many batch files simply redirct their arguments to other executable and in those cases [these rules](https://msdn.microsoft.com/en-us/library/17w5ykft.aspx) are applying. 

### Considerations outside of the scope of this RFC

Maybe on Linux `--%` and `--=` can be depricated, as they dont make much sense on Linux -- the CommandLine always needs to be escaped acording to [these rules](https://msdn.microsoft.com/en-us/library/17w5ykft.aspx) (see paragraph "Linux").

As the main purpose of `--%` was (as far as I know) not the possibility to influence the CommandLine directly but instead a way to disable many special characters in a powershell command, in the future one could possibly add a `--$` token that disables all special characters exept quotes and `$` -- a clean way of what `--%` was originally meant for.
