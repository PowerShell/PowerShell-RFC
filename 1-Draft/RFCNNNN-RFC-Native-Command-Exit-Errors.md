---
RFC: RFCNNNN-RFC-Native-Command-Exit-Errors
Author: Micheal Padden
Status: Draft
Area: Engine, Language
Version: 1.0.0
---

# Native Command Error Handling

Although Powershell has an exception-based error handling framework, it
does not currently apply to native commands. 

Exception-based error handling makes it easier to write robust code
as less boilerplate code is needed to check and handle errors.

Powershell scripts using native commands would benefit from being able
to error handling features like those used by cmdlets.

## Motivation

Native commands return an exit code to the calling application which
will be zero for success or non-zero for failure. A robust script will
not assume success, instead checking the exit code after any statement
calling a native command, using boilerplate like the following:
```Powershell
 if( ! $? ) 
 { 
     exit $lastexitcode; 
 }
```

Bourne type shells provide a `set -e` error handling mode that acts as 
if this boilerplate were included in certain contexts. Like exception
based error handling, this can exit a stack of functions, scripts and
shells. This allows robust scripts to be written with a minimal amount
of boilerplate.

To support a similar style of error handling with native commands, this
RFC proposes including native command errors in Powershell's exception
handling framework in a similar way.

The specification and alternative proposals are based on the
[Equivalent of bash `set -e` #3415](https://github.com/PowerShell/PowerShell/issues/3415)
and committee review of the associated
[pull request](https://github.com/PowerShell/PowerShell/pull/3523).


## Specification

This RFC proposes including native commands in the error handling
framework, by allowing an error to be reported to the error stream
when a native command exits with a non-zero exit code, similar to 
the `set -e` option in bourne type shells.

The `$PSIncludeNativeCommandInErrorActionPreference` preference
variable should govern treatment of non-zero exit codes on native
commands. Possible values are:
- `$false`: (the default) ignore non-zero exit codes. 
The effect is the same as existing Powershell treatment of this case.
- `$true`: report an error for non-zero exit codes on a native command.

The reported error record should be created with the following details:
- exception: `ExitException`, with the exit code of the failed command.
- error id: `"Program {0} failed with exit code {1}"`, with the command
name and the exit code, from resource string `ProgramFailedToComplete`.
- error category: `ErrorCategory.NotSpecified`.
- object: the (boxed) exit code.

The existing `$ErrorActionPreference` variable should govern how native
command errors are handled. The `$ErrorActionPreference` value `"stop"`
will treat such errors as terminating errors, allowing the error to
terminate the session, or be caught as an exception.

Alternative approaches outlined below try to address limitations in the
approach taken with this base specification.

## Alternate Approaches and Considerations

### Control native command error management with ActionPreference value

Another preference option could be added to `$ErrorActionPreference`.

With this approach, an `$ErrorActionPreference` value of
`"StopIncludingNativeCommand"` would cause an exception to be thrown
for non-zero exit codes on native commands, as well as any error in
cmdlets.

This approach is limited to managing errors in native commands when
this option is set, so integrates less well with overall Powershell 
error handling features. For example, where the preference is "Inquire",
a non-zero exit code on a native command would not generate an inquiry.

### Treat non-zero exit codes as errors only on untested native commands

A similar approach that ignores the exit code in native commands where
the output is tested should give useful results.

The `$PSManageNativeCommandErrors` preference variable would govern
treatment of non-zero exit codes for "untested" native commands.
Possible values would be:
- `$false`: (the default) value would ignore any non-zero exit codes.
- `$true`: would report and error for non-zero exit codes on an
untested native command.
Commands would be considered "tested" if used in the lexical scope of
an `if` or loop condition, the operand of a logical operator
(`!`,`||` or `&&`).

This should improve on the basic specification by allowing the idiom to
be combined with normal control flow statement usage.

A common idiom with native commands is to return success or failure in
the exit code rather than the output stream. Constructs such as if and
while test the exit code rather than the command output in Bourne type
shells, so treating non-zero exit code as an exception in such contexts
would not make sense. Thus `set -e` is applied to "untested" commands
only, where a "tested" command is any command in the scope of an `if`
or `while` condition, a logical operator (`!`,`||` or `&&`) or any
command before the last in a pipeline (see output of pipeline below).

Although PowerShell `if` and loop conditions check the output of the
command rather than checking the exit code, a native command may be
able to accept arguments that produce suitable output only on success,
but still indicate the failure reason using the exit code. Where the
native command does not provide such an option, it is probably not
useful to test it's output in an `if` or loop condition.

Note: with bourne type shells, "tested" commands are those in the
dynamic scope of a tested context rather than the lexical scope of the
context. This limits the usefulness of `set -e`, but is needed for
[historic compatibility](http://austingroupbugs.net/view.php?id=52)
reasons.

### Treat errors as exceptions only on untested commands

This approach varies the basic specification so that errors in all
commands can be treated exceptions where the command result is not
tested.

As well as the new boolean preference to enable reporting errors for
non-zero native command exit codes, a new preference value option would
be added for `$ErrorActionPreference`.

A `"StopOnDiscarded"` value for `$ErrorActionPreference` would have the
effect of ignoring non-terminating command errors in a context where
the result is tested, and throwing an exception in other contexts.

This should improve on the basic specification by allowing the idiom to
be combined with normal control flow statement usage and applied to
cmdlets also.

Treating non-zero exit codes on native commands as errors only in
untested contexts as with earlier listed approaches gives inconsitent
error handling between native commands and cmdlets. The parameter
`-ErrorAction SilentlyContinue` would need to be given to a cmdlet to
prevent an exception being thrown where `$ErrorActionPreference` is
`"stop"` and a non-terminating error should not cause an exception
because the output is being tested.

### Allow output of pipeline to be treated as tested

This approach varies the above approaches to "untested" commands so
that the output of a pipeline can be treated as tested. 

A `$PSIncludeNativeCommandInErrorActionPreference` variable with three
possible values would be used instead of a boolean variable. 
Possible values would be:
`"none"`:(default) do not report an error on a non-zero exit code
`"command"`: report an error on a non-zero exit code after a
native command in an untested pipeline.
`"pipeline"`: report an error on a non-zero exit code after an
native command which is the last command in an untested pipeline.

This should add some flexibility for handling errors in multiple
command pipelines to deal with different expectations.

In Powershell, the status of a pipeline is false if any command in the
pipeline fails. Thus treating a command that outputs to a pipeline as
"untested" is arguably be closer to this existing Powershell idiom.
Korn compatible shells like bash provide similar behaviour with 
`set -o pipefail`. This makes the exit code of the pipeline zero where
all exit codes were zero, or otherwise the exit code of the last
command with a non-zero exit code.

However this might not be the expected behaviour in all circumstances,
as `set -o pipefail` is not enabled on bourne type shells, and commands
that output into a pipe might sometimes be considered to have their 
success or failure "tested" for error handling purposes.

### Use dynamic scope/Set-StrictMode

This approach implements dynamic scoping for native command error
management using a cmdlet. 

One of the approaches above would be enabled with
`Set-StrictMode -version 6`, instead of a boolean preference variable.
For example,
- An error would be eported for native commands with a non-zero exit
code.
- An exception would be thrown for "untested" native commands with a
non-zero exit code.

The dynamic scoping approach would improve on earlier listed approaches
by limiting the scope of error handling configuration so that script
functions could not have an effect on the error handling mode in the
calling scope.

There might be difficulties using `Set-StrictMode` for this. Other 
strict mode checks tend to identify coding errors based on the internal
Powershell context, not than the wider OS native environment. Use of 
the latest strict mode with existing scripts that already implement
robust error handling on native commands would be more difficult, as 
normal control flow statements involving native commands would have to
be restuctured into `try`/`catch` statements.

The dynamic scoping approach would have side-effects on called scripts.

### Use lexical scope/Exception handling extensions

With the lexical scoping approach, native command error handling mode
would be used through language syntax instead of preference variables
or cmdlets. A possible syntax might be to add parameters to the try
statement.

For example:
- `WithNativeCommandExitError` parameter, to report an error where a
native command exits with a non-zero exit code
- `WithIgnoredErrorException` parameter, to throw an exception for
reported command errors where the result of the command is not tested.

Lexical scoping of native command error handling would improve on
earlier listed approaches in a number of ways. Native command error
handling would sit alongside overall exception handling. There would be
no side-effects in calling or in called scripts. It could also minimize
runtime overhead as the facility should be mostly handled by the parser
and compiler. 