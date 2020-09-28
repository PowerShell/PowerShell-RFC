---
RFC: RFC00XX
Author: Jason Helmick
Status: Draft
Area: Core
Comments Due: 10/31/2020
---

# Native Command Error Handling

PowerShell scripts using native commands would benefit from being able to use error handling
features like those used by cmdlets.

## Motivation

In PowerShell by default, script processing continues when non-terminating errors occur. This is a
benefit when expecting non-terminating errors in normal execution such as non-responsive computers from
a list. This default behavior is controlled with the preference variable
`$ErrorActionPreference` default of `Continue`.

During development and debugging, often customers prefer that script execution stops when a
non-terminating error occurs. PowerShell currently supports customers with this ability by setting
the preference variable in the script.

```Powershell
$ErrorActionPreference = 'Stop'
```

Native commands return an exit code to the calling application which will be zero for success or
non-zero for failure. However, native commands currently do not participate in the PowerShell error
stream. Customers working with native commands in their scripts will need to check the execution
status after each call using a helper function similar to below:

```Powershell
if( ! $? )
{
    exit $lastexitcode;
}
 ```

Simply relaying the errors through the error stream isn't the solution. The example itself doesn't
support all cases as `$?` can be false from a cmdlet or function error, making `$LASTEXITCODE`
stale.

To support a similar style of error handling with native commands, this specification proposes
converting native command errors into PowerShell errors similar to the `sh`-compatible
`set -e` mode.

 The specification and alternative proposals are based on the
 [Equivalent of bash `set -e` #3415](https://github.com/PowerShell/PowerShell/issues/3415)
 committee review of the associated
 [pull request](https://github.com/PowerShell/PowerShell/pull/3523), and
 [implementation plan](https://github.com/PowerShell/PowerShell-RFC/pull/88#issuecomment-613653678)

## Specification

This RFC proposes including native commands in the error handling framework when the feature is
enabled by adding an error to the error stream if the exit code of a native command is non-zero.

The `$PSNativeCommandErrorAction` preference variable will implement a version of the
`$ErrorActionPreference` variable for native commands.

- The value will default to `Ignore` for compatibility with existing behavior.
- For non-zero exit codes and except for the value 'Ignore', an `ErrorRecord` will be added to
  `$Error` that wraps the exit code and the command executed that returned the exit code.
- Initially, only the existing values of `$ErrorActionPreference` will be supported.
- The set of values may be extended later to include `MatchErrorActionPreference`, which should
  apply the `$ErrorActionPreference` setting to native commands.
- A conversion will occur between `$PSNativeCommandErrorAction` and `$ErrorActionPreference`
  values, where `MatchErrorActionPreference` is converted to the current value of
  `$ErrorActionPreference`

Valid values for `$ErrorActionPreference`

| Value           | Definition
----------------  | -------------------
| Break           | Enter the debugger when an error occurs or when an exception is raised.
| Continue        | (Default) - Displays the error message and continues executing.
| Ignore          | Suppresses the error message and continues to execute the command. The Ignore value is intended for per-command use, not for use as saved preference. Ignore isn't a valid value for the $ErrorActionPreference variable.
| Inquire         | Displays the error message and asks you whether you want to continue.
| SilentlyContinue| No effect. The error message isn't displayed and execution continues without interruption.
| Stop            | Displays the error message and stops executing. In addition to the error generated, the Stop value generates an ActionPreferenceStopException object to the error stream. stream
| Suspend         | Automatically suspends a workflow job to allow for further investigation.

The reported error record should be created with the following details:

| Property        | Definition
----------------  | -------------------
| exception:      | `ExitCode`, with the exit code of the failed command.
| error id:       | `"Program {0} failed with unhandled exit code {1}"`, with the command name and the exit code, from resource string `ProgramFailedToComplete`.
| error category: | `ErrorCategory.NotSpecified`.
| object:         | exit code

 This does not provide the actual semantics of bash `set -eo pipefail` as Bourne shell-style
 integration with existing status handling syntax is not implemented.

`Set -eo pipefail` is a combination of `Set -u` and `Set -o pipefail`:

- `Set -u`, a reference to any variable not previously defined results in an error. Similar to
  Set-StrictMode in PowerShell.
- `Set -o pipefail`, by default, pipeline success is determined by the last command executed. `Set
  -o pipefail` returns an error if any command in the pipeline fails.

 As a result of not including the semantics of `Set -eo pipefail`, PowerShell script logic for
 handling native command exit codes would need to use either `try`..`catch` or existing exit status
 handling language constructs, according to the setting of this preference variable.

 One way of overriding `$ErrorActionPreference` for a single native command and handling
 it's exit status explicitly would be to put this logic into a script block and call it
 with the invocation operator (`&`).

## Alternative Approaches and Considerations

We could extend this existing behavior to give equivalent functionality to bash `Set -eo pipefail`.
This includes:

- `Set -u`, a reference to any variable not previously defined results in an error. Similar to
  `Set-StrictMode` in PowerShell.
- `Set -o pipefail`, by default, pipeline success is determined by the last command executed. `Set
  -o pipefail` returns an error if any command in the pipeline fails.

Today there exists a workaround for handling native command exit codes using a `try`..`catch` block.
This alternative may be considered in the future.

### Add "strict" native command option

The `$PSStrictNativeCommand` preference should treat creation of an `ErrorRecord` for native
commands in the same way as this is treated elsewhere. Described here as a Boolean, could be
considered as an enum to allow for future expansion. Possible values are:

- `$false`: (the default) ignore non-zero exit codes. This is the same as existing PowerShell
  treatment of this case.
- `$true`: Populate the error stream of the native command with an `ErrorRecord` associated with an
  `ExitException` exception.

### Modifying existing semantics to consider exit code and exit status

The error will throw an exception, potentially terminating the session, in the same situation as
other non-terminating errors will do this, i.e. where `$ErrorActionPreference` is set to `"stop"`.
This would not be the desired behavior on commands where the script already handles a non-zero exit
code, which would require the addition of extra boilerplate to use multiple native commands in
combination. There are a number of ways that this could be made more flexible by integrating exit
code and exit status handling into the language syntax.

#### Convert non-terminating errors to terminating where the command output is used

This approach implements semantics equivalent to bash `set -eo pipefail` in the runtime layer.

The `$PSStrictPipeLine` preference variable would govern promotion of a non-terminating error to a
terminating error on getting an object from the pipeline output stream. Possible values would be:

- `$false`: (the default) an object can be collected from the pipeline output stream regardless of
  the command exit value. This is the same as existing PowerShell treatment of this case.
- `$true`: where the exit status of a native command is `$false`, trying to get an object from its
  output stream will create a terminating error from the non-terminating errors in its error stream.
  Conversion to boolean would be structured to ensure that this returns `$false` if the output
  pipeline does not contain anything without trying to get an actual value from it.

This would allow syntax like `if`, `while` and pipeline chain operators to be usefully combined with
native commands.

#### Sanitize semantics of treating a native command as a conditional value

This option in combination with the above enables functionality analogous to bash `set -eo pipefail`

PowerShell converts the output stream to a boolean value where a native command is used as a
"condition", i.e. the `-not` operator, or an `if`, `elseif` or `while` statement. This is not
particularly useful with native commands, which would tend to produce no output on success, at least
when executed in batch as opposed to interactive mode.

The `$PSUseNativeExitStatus` variable would govern whether exit status is used in determining the
boolean value of a native command for a conditional context. Possible values would be:

- `$false`: (the default) the boolean value of native command is defined as whether or not the
  length of the output stream is non-zero. This is the same as existing PowerShell treatment of this
  case.
- `$true`: the boolean value of a native command is it's exit status (`$?`), and

This would allow syntax like `if` and `while` to be usefully combined with native commands.

#### Add strict pipeline chain failure semantics.

Treat only ignored exit statuses as exceptions.

The `$PSStrictPipeLineChain` preference variable would govern the exit status in the last command of
a pipeline. Possible values would be:

- `$false`: (the default) would ignore `$false` exit status on the last command. This is the same as
  existing PowerShell treatment of this case.
- `$true`: for a pipeline that is being used in a conditional context (see above), and where the
  exist status of the last command in the pipeline is `$false`, would create a terminating error
  from the non-terminating errors in the command error stream.

This should improve on the basic specification by allowing the idiom to be usefully combined with
pipeline chaining.

### Use dynamic scope/Set-StrictMode

This approach implements dynamic scoping for native command error management using a cmdlet.

Some of the above would be enabled with `Set-StrictMode -version 6` instead of with boolean
preference variables.

The dynamic scoping approach would improve on earlier listed approaches by limiting the scope of
error handling configuration so that script functions could not have an effect on the error handling
mode in the calling scope.

If `Set-StrictMode` is used for this, it would need to enable some combination of enhancements that
will not raise errors on scripts that have already implemented strong native command error handling.

### Use lexical scope/Exception handling extensions

Dynamic scoping approach would have side-effects on called scripts. This is analogous to the
behavior of Bourne type shells, which maintain this behavior for
[historic compatibility](http://austingroupbugs.net/view.php?id=52) reasons.

Lexical scoping of native command error handling would improve on earlier options by integrating
native command error handling fully into the existing exception handling without any
side-effects in calling or called scripts.

With this approach, native command error handling mode would be used through language syntax instead
of preference variables or cmdlets. A possible syntax might be to add a strictness option to the try
statement which applies to the lexical scope of the try statement.

Lexical scope requires more runtime overhead; a mapping must be maintained at runtime between each
runtime scope and its corresponding lexical scope that existed at parse-time. In addition, lexical
scope may not apply well to errors. Errors produce stack traces at runtime and should be handled in
a runtime-facing way. For these reasons, this alternative is not under consideration.

