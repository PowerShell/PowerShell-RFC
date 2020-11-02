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
benefit when expecting non-terminating errors in normal execution such as non-responsive computers
from a list. This default behavior is controlled with the preference variable
`$ErrorActionPreference` default of `Continue`.

In production, often customers prefer that script execution stops when a non-terminating error
occurs. This is particularly true in CI where the preference is to fail fast. PowerShell currently
supports customers with this ability by setting the `$ErrorActionPreference` variable in the script
to `Stop`.

```Powershell
$ErrorActionPreference = 'Stop'
```

Native commands usually return an exit code to the calling application which will be zero for
success or non-zero for failure. However, native commands currently do not participate in the
PowerShell error stream. Redirected `stderr` output is not interpreted the same as the PowerShell
error stream as many native commands use `stderr` as information/verbose stream and thus only the
exit code matters. Users working with native commands in their scripts will need to check the
execution status after each call using a helper function similar to below:

```Powershell
if ($LASTEXITCODE -ne 0)
{
    throw "Command failed. See above errors for details"
}
 ```

Simply relaying the errors through the error stream isn't the solution. The example itself doesn't
support all cases as `$?` can be false from a cmdlet or function error, making `$LASTEXITCODE`
stale.

In POSIX shells, this need to terminate on command error is addressed by the `set -e` configuration,
which causes the shell to exit when a command fails. In addition, to ensure that an error is
returned if any command in a pipeline fails, POSIX shells address this need with `set -o pipefail`
configuration.

This specification proposes a similar idea, but adapted to the PowerShell conventions of preference
variables and catchable, self-describing, terminating error objects. This proposal further adds to
the functionality of `set -e` with `set -o pipefail` to return an error if any command in a pipeline
fails. This is planned to be equivalent to `set -eo pipefail`.

 The specification and alternative proposals are based on the
 [Equivalent of bash `set -e` #3415](https://github.com/PowerShell/PowerShell/issues/3415)
 committee review of the associated
 [pull request](https://github.com/PowerShell/PowerShell/pull/3523), and
 [implementation plan](https://github.com/PowerShell/PowerShell-RFC/pull/88#issuecomment-613653678)

## Specification

This RFC proposes a preference variable to configure the elevation of errors produced by native
commands to first-class PowerShell errors, so that native command failures will produce error
objects that are added to the error stream and may terminate execution of the script without added
boilerplate.

The specification proposes similar functionality to the common POSIX shell configuration `set -eo pipefail`.

- `set -e` - terminates on command error
- `set -o pipefail` - returns an error if any command in the pipeline fails.

> [!NOTE] A common configuration command for POSIX shells `set -euo pipefail` includes the `set -u`
> configuration which returns an error if any variable has not been previously defined. This is
> equivalent to the existing PowerShell `Set-StrictMode` and is not addressed in this RFC.

The `$PSNativeCommandErrorAction` preference variable will implement a version of the
`$ErrorActionPreference` variable for native commands.

- The value will default to `Ignore` for compatibility with existing behavior.
- For non-zero exit codes and except for the value `Ignore`, an `ErrorRecord` will be added to
  `$Error` that wraps the exit code and the command executed that returned the exit code.
- Initially, only the existing values of `$ErrorActionPreference` will be supported.
- The set of values may be extended later to include `MatchErrorActionPreference`, which should
  apply the `$ErrorActionPreference` setting to native commands.
- A conversion will occur between `$PSNativeCommandErrorAction` and `$ErrorActionPreference`
  values, where `MatchErrorActionPreference` is converted to the current value of
  `$ErrorActionPreference`

Valid values for `$PSNativeCommandErrorAction`

| Value           | Definition
----------------  | -------------------
| Break           | Enter the debugger when an error occurs or when an exception is raised.
| Continue        | (Default) - Displays the error message and continues executing.
| Ignore          | Suppresses the error message and continues to execute the command.
| Inquire         | Displays the error message and asks you whether you want to continue.
| SilentlyContinue| No effect. The error message isn't displayed and execution continues without interruption.
| Stop            | Displays the error message and stops executing. In addition to the error generated, the Stop value generates an ActionPreferenceStopException object to the error stream. stream
| Suspend         | Automatically suspends a workflow job to allow for further investigation.

The reported error record object will be the new type: `NativeCommandException` with the following details:

| Property        | Definition
----------------  | -------------------
| ExitCode:      |  The exit code of the failed command.
| ErrorID:       | `"Program {0} ended with non-zero exit code {1}"`, with the command name and the exit code, from resource string `ProgramFailedToComplete`.
| ErrorCategory: | `ErrorCategory.NotSpecified`.
| object:         | exit code
| Source:        | The full path to the application
| ProcessInfo    | details of failed command including path, exit code, and PID

## Alternative Approaches and Considerations

One way of checking for a single native command and handling its exit
status explicitly would be to put this logic into a script block and call it with the invocation
operator (`&`).

```Powershell
if ($LASTEXITCODE -ne 0)
{
    throw "Command failed. See above errors for details"
}
 ```

### Set-StrictMode

A common configuration command for POSIX shells `set -euo pipefail` includes the `set -u`
configuration which returns an error if any variable has not been previously defined. This is
equivalent to the existing PowerShell `Set-StrictMode` and is not needed to be addressed in this RFC.

### Preference variable resolves error handling conflicts

In cases where an existing script already handles non-zero native command errors, the preference
variable `$PSNativeCommandErrorAction` may be set to `Ignore`. Error handling behavior of the
PowerShell cmdlets is handled separately with `$ErrorActionPreference`.

