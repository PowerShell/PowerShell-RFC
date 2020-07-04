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
to use error handling features like those used by cmdlets.

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

The Bourne again shell provides a `set -eo pipefail` exit code handling mode
that acts as though this boilerplate were included in certain contexts. 
Like exception-based error handling, this can exit a stack of functions, 
scripts and shells. This allows robust scripts to be written with a minimal
amount of boilerplate.

To support a similar style of error handling with native commands, this
RFC proposes converting native command errors into PowerShell errors.
The Bourne shell compatible `set -e` mode (without `set -o pipefail`) is 
_not_ supported.

The specification and alternative proposals are based on the
[Equivalent of bash `set -e` #3415](https://github.com/PowerShell/PowerShell/issues/3415)
committee review of the associated
[pull request](https://github.com/PowerShell/PowerShell/pull/3523), and 
[implementation plan](https://github.com/PowerShell/PowerShell-RFC/pull/88#issuecomment-613653678)

## Specification

This RFC proposes including native commands in the error handling
framework when the feature is enabled by adding an error to the error
stream if the exit status of a native command is false. 

The `$PSNativeCommandErrorAction` preference variable will implement a version
of the `$ErrorActionPreference` variable for native commands. The value will 
default to `Ignore` for compatibility with existing behaviour. 
- For all values except `Ignore`, an `ErrorRecord` will be added to `$Error` that wraps
the exit code and the command executed that returned the exit code. 
- Initially, only the existing values of `$ErrorActionPreference` will be supported. 
- The set of values may be extended later to include `MatchErrorActionPreference`, 
which should apply the `$ErrorActionPreference` setting to native commands also.
- An enum converter will convert between `$PSNativeCommandError` and 
`$ErrorActionPreference` values, where `MatchErrorActionPreference` is converted to the 
current value of `$ErrorActionPreference`

The reported error record should be created with the following details:
- exception: `ExitCode`, with the exit code of the failed command.
- error id: `"Program {0} failed with unhandled exit code {1}"`, with the command
name and the exit code, from resource string `ProgramFailedToComplete`.
- error category: `ErrorCategory.NotSpecified`.
- object: the (boxed) exit code.

This does not provide the actual semantics of bash `set -eo pipefail` as Bourne shell 
style integration with existing existing status handling syntax is not implemented. 
As a result, PowerShell script logic for handling native command exit codes would need 
to use either `try`..`catch` or existing exit status handling language constructs, 
according to the setting of this preference variable.

One way of overriding`$ErrorActionPreference` for a single native command and handling
its exit status explicitly would be to put this logic into a script block and call it 
with the invocation operator (`&`).

## Alternative Approaches and Considerations

A number of tweaks to existing behaviours could be combined to give equivalent
functionality to bash `set -eo pipefail`

### Add "strict" native command option.

The `$PSStrictNativeCommand` preference should treat creation of an `ErrorRecord` 
for native commands in the same way as this is treated elsewhere.
Possible values are:
- `$false`: (the default) ignore non-zero exit codes. 
This is the same as existing Powershell treatment of this case.
- `$true`: Populate the error stream of the native command with an `ErrorRecord` 
associated with an `ExitException` exception. 

### Modifying existing semantics to consider exit code and exit status 

The error will throw an exception, potentially terminating the session, in the same 
situation as other non-terminating errors will do this, i.e. where 
`$ErrorActionPreference` is set to `"stop"`. This would not be the desired behaviour on
commands where the script already handles a non-zero exit code, which would require
the addition of extra boilerplate to use multiple native commands in combination.
There are a number of ways that this could be made more flexible by integrating exit 
code and exit status handling into the language syntax.

#### Convert non-terminating errors to terminating where the command output is used.

This approach implements semantics equivalent to bash `set -eo pipefail`
in the runtime layer.

The `$PSStrictPipeLine` preference variable would govern promotion of a non-terminating
error to a terminating error on getting an object from the pipeline output stream.
Possible values would be:
- `$false`: (the default) an object can be collected from the pipeline output stream 
regardless of the command exit value.
This is the same as existing PowerShell treatment of this case.
- `$true`: where the exit status of a native command is `$false`, trying to get an
object from its output stream will create a terminating error from the non-terminating 
errors in its error stream. 
Conversion to boolean would be structured to ensure that this returns `$false` if the 
output pipeline does not contain anything without trying to get an actual value from it.

This would allow syntax like `if`, `while` and pipeline chain operators to be usefully 
combined with native commands.

#### Sanitise semantics of treating a native command as a conditional value.

This option in combination with the above enables functionality analogous to
bash `set -eo pipefail`

PowerShell converts the output stream to a boolean value where a native command is used 
as a "condition", i.e. the `-not` operator, or an `if`, `elseif` or `while` statement. 
This is not particularly useful with native commands, which would tend to produce no 
output on success, at least when executed in batch as opposed to interactive mode.
 
The `$PSUseNativeExitStatus` variable would govern whether exit status is used 
in determining the boolean value of a native command  for a conditional context.
Possible values would be:
- `$false`: (the default) the boolean value of native command is defined as whether
 or not the length of the output stream is non-zero.
This is the same as existing PowerShell treatment of this case.
- `$true`: the boolean value of a native command is it's exit status (`$?`), and 

This would allow syntax like `if` and `while` to be usefully combined with
native commands.

#### Add strict pipeline chain failure semantics.

Treat only ignored exit statuses as exceptions.

The `$PSStrictPipeLineChain` preference variable would govern the exit 
status in the last command of a pipeline.
Possible values would be:
- `$false`: (the default) would ignore `$false` exit status on the last command.
This is the same as existing PowerShell treatment of this case.
- `$true`: for a pipeline that is being used in a conditional context (see above), 
and where the exist status of the last command in the pipeline is `$false`, 
would create a terminating error from the non-terminating errors in the command error 
stream.

This should improve on the basic specification by allowing the idiom to be usefully 
combined with pipeline chaining.

### Use dynamic scope/Set-StrictMode

This approach implements dynamic scoping for native command error management using a 
cmdlet. 

Some of the above would be enabled with `Set-StrictMode -version 6`
instead of with boolean preference variables.

The dynamic scoping approach would improve on earlier listed approaches
by limiting the scope of error handling configuration so that script
functions could not have an effect on the error handling mode in the
calling scope.

If `Set-StrictMode` is used for this, it would need to enable some combination
of enhancements that will not raise errors on scripts that have already implemented 
strong native command error handling.


### Use lexical scope/Exception handling extensions

Dynamic scoping approach would have side-effects on called scripts. 
This is analogous to the behaviour of Bourne type shells, which maintain 
this behaviour for [historic compatibility](http://austingroupbugs.net/view.php?id=52)
reasons.

Lexical scoping of native command error handling would improve on
earlier options by integrating native command error handling fully into
the existing exception handling without out any side-effects in calling 
or called scripts. 

With this approach, native command error handling mode would be used through 
language syntax instead of preference variables or cmdlets. A possible syntax might be
to add a strictness option to the try statement which applies to the lexical scope of
the try statement.

Implementing this functionality only as a lexically scoped extension could 
be the preferred option because it becomes a kind of syntactic sugar which is 
implemented by the parser and compiler, improving testability, and keeping complexity 
out of the runtime layer. 