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

In POSIX shells, terminating execution when a command has an error is enabled via executing `set -e`
in the session. In addition, to ensure that an error is returned if any command in a pipeline fails,
POSIX shells address this via executing `set -o pipefail` in the session.

This specification proposes a similar idea, but adapted to the PowerShell conventions of preference
variables and catchable, self-describing, terminating error objects. This proposal adds the
equivalent functionality of `set -e -o pipefail` (abbreviated to `set -eo pipefail`) to stop
execution and return an error if any command in a pipeline fails.

 The specification and alternative proposals are based on the
 [Equivalent of bash `set -e` #3415](https://github.com/PowerShell/PowerShell/issues/3415)
 committee review of the associated
 [pull request](https://github.com/PowerShell/PowerShell/pull/3523), and
 [implementation plan](https://github.com/PowerShell/PowerShell-RFC/pull/88#issuecomment-613653678)

## Specification

This RFC proposes a preference variable to enable errors produced by native commands to be
PowerShell errors, so that failures will produce error objects that are added to the error stream
and may terminate execution of the script without added boilerplate.

The specification proposes similar functionality to the common POSIX shell configuration
`set -eo pipefail`.

- `set -u` - returns an error if any variable has not been previously defined.
- `set -e` - instructs to immediately exit if any command has a non-zero exit status.
- `set -o pipefail` - prevents errors in the pipeline from being masked. The return code for the
  non-zero error is returned for the entire pipeline.

### set -u/ Set-StrictMode

In the example below, `set -u` behavior of `bash` is shown followed by the proposed behavior for
PowerShell. This is not part of this proposal, but added for clarity.

```bash
bash-3.2$ cat file
#!/bin/bash
/bin/echo "Without set -u : No output - No error is produced"
/bin/echo "$firstname"
/bin/echo ""
/bin/echo "With set -u : Equivalent to Set-StrictMode -Version 2.0"
set -u
/bin/echo "$firstname"
```

```output
bash-3.2$ file
Without set -u : No output - No error is produced

With set -u : Equivalent to Set-StrictMode -Version 2.0
/Users/jasonhelmick/natcmdbash/strict: line 10: firstname: unbound variable
```

```powershell
PS> cat ./file.ps1
/bin/echo "Without Set-StrictMode -version 2.0 : No output - No error is produced"
/bin/echo "$firstname"
/bin/echo ""
/bin/echo "With Set-StrictMode -version 2.0 : Equivalent to set -u"
Set-StrictMode -version 2.0
/bin/echo "$firstname"
```

```output
PS> ./file.ps1
Without set -u : No output - No error is produced

With Set-StrictMode -version 2.0 : Equivalent to set -u
InvalidOperation: /Users/jasonhelmick/natcmdbash/psstrict.ps1:9
Line |
   9 |  /bin/echo "$firstname"
     |             ~~~~~~~~~~
     | The variable '$firstname' cannot be retrieved because it has not been set.
```

### set -e/ $ErrorActionPreference

In the example below, `set -e` is not equivalent to `$ErrorActionPReference` for native commands.

```bash
bash-3.2$ cat file
#!/bin/bash
/bin/echo "Without set -e : Will receive message after failure"
/bin/cat ./nofile
/bin/echo "Message After failure"
/bin/echo ""
/bin/echo "With set -e : Will NOT receive message after failure"
set -e
/bin/cat ./nofile
/bin/echo "Message After failure"
```

```output
bash-3.2$ file
Without set -e : Will receive message after failure
cat: ./nofile: No such file or directory
Message After failure

With set -e : Will NOT receive message after failure
cat: ./nofile: No such file or directory
```

```powershell
PS> cat ./file.ps1
/bin/echo "Without $ErrorActionPreference : Will receive message after failure"
/bin/cat ./nofile
/bin/echo "Message After failure"
/bin/echo ""
/bin/echo "With `$ErrorActionPreference = 'Stop' : SHOULD NOT receive message after failure - but does"
$ErrorActionPreference = "Stop"
/bin/cat ./nofile
/bin/echo "Message After failure"
/bin/echo ""
Write-Host "With cmdlet's - `$ErrorActionPreference = 'Stop' : Will NOT receive message after failure"
$ErrorActionPreference = 'Stop'
Get-Content ./nofile
Write-Host "Message After failure"
```

```output
PS> ./file.ps1
Without $ErrorActionPreference : Will receive message after failure
cat: ./nofile: No such file or directory
Message After failure

With $ErrorActionPreference = 'Stop' : SHOULD NOT receive message after failure - but does
cat: ./nofile: No such file or directory
Message After failure

With cmdlet's - $ErrorActionPreference = 'Stop' : Will NOT receive message after failure
Get-Content: /Users/jasonhelmick/natcmdbash/psstop.ps1:17
Line |
  17 |  Get-Content ./nofile
     |  ~~~~~~~~~~~~~~~~~~~~
     | Cannot find path '/Users/jasonhelmick/natcmdbash/nofile' because it does not exist.
```

### set -o pipefail/ PSNativeCommandErrorAction

In the example below, PowerShell has no equivalent to `set -o pipefail` for native commands.

```bash
bash-3.2$ cat file
#!/bin/bash
/bin/echo "Without set -o pipefail : returns 0"
/bin/cat ./nofile | /bin/echo "pipe statement after failure"
/bin/echo "returns $?"
/bin/echo ""
/bin/echo "With set -o pipefail : returns non-zero"
set -o pipefail
/bin/cat ./nofile | /bin/echo "pipe statement after failure"
/bin/echo "returns $?"
```

```output
bash-3.2$ file
Without set -o pipefail : returns 0
pipe statement after failure
cat: ./nofile: No such file or directory
returns 0

With set -o pipefail : returns non-zero
cat: ./nofile: No such file or directory
pipe statement after failure
returns 1
```

```powershell
PS> cat ./file.ps1
/bin/echo "Without `$PSNativeCommandErrorAction = 'Stop' : should return 0 (true)"
/bin/cat ./nofile | /bin/echo "pipe statement after failure"
/bin/echo "returns $?"
/bin/echo ""
/bin/echo "With set -o equiv. `$PSNativeCommandErrorAction = 'Stop' : should return non-zero (false)"
$PSNativeCommandErrorAction = 'Stop'
/bin/cat ./nofile | /bin/echo "pipe statement after failure"
/bin/echo "returns $?"
```

```output
PS> ./file.ps1
Without `$PSNativeCommandErrorAction = 'Stop' : should return 0 (true)
cat: ./nofile: No such file or directory
pipe statement after failure
returns False

With set -o equiv.  = 'Stop' : should return non-zero (false)
cat: ./nofile: No such file or directory
pipe statement after failure
returns False
```

> [!NOTE] A common configuration command for POSIX shells `set -euo pipefail` includes the `set -u`
> configuration which returns an error if any variable has not been previously defined. This is
> equivalent to the existing PowerShell `Set-StrictMode` and does not need to be addressed in this
> RFC.

### $PSNativeCommandErrorAction

The `$PSNativeCommandErrorAction` preference variable will implement a version of the
`$ErrorActionPreference` variable for native commands.

- The value will default to `Continue` for compatibility with existing behavior.
- For non-zero exit codes and except for the value `Ignore`, an `ErrorRecord` will be added to
  `$Error` that wraps the exit code and the command executed that returned the exit code.
- Initially, only the existing values of `$ErrorActionPreference` will be supported in
  `$PSNativeCommandErrorAction` as described in the table below.

### Valid values for `$PSNativeCommandErrorAction`

| Value           | Definition
----------------  | -------------------
| Break           | Enter the debugger when an error occurs or when an exception is raised.
| Continue        | Displays the error message and continues executing.
| Ignore          | Suppresses the error message and continues to execute the command.
| Inquire         | Displays the error message and asks you whether you want to continue.
| SilentlyContinue| The error message isn't displayed and execution continues without interruption.
| Stop            | Displays the error message and stops executing. In addition to the error generated, the Stop value generates an ActionPreferenceStopException object to the error stream. stream
| Suspend         | Automatically suspends a job to allow for further investigation.

### Preference variable resolves error handling conflicts

In cases where an existing script already handles non-zero native command errors, the preference
variable `$PSNativeCommandErrorAction` may be set to `Continue`. Error handling behavior of the
PowerShell cmdlets is handled separately with `$ErrorActionPreference`.

### Error object

The reported error record object will be the new type: `NativeCommandException` with the following details:

| Property        | Definition
----------------  | -------------------
| ExitCode:      |  The exit code of the failed command.
| ErrorID:       | `"Program {0} ended with non-zero exit code {1}"`, with the command name and the exit code, from resource string `ProgramFailedToComplete`.
| ErrorCategory: | `ErrorCategory.NotSpecified`.
| object:         | Exit code
| Source:        | The full path to the application
| ProcessInfo    | Details of failed command including path, exit code, and PID
| TargetObject   | Specifies the object that was being processed when the error occurred.

## Alternative Approaches and Considerations

### Native commands should respect $ErrorActionPreference

Native commands should use the $ErrorActionPreference setting and not need
$PSNativeCommandErrorAction. This could be released as an experimental feature.

### Extending $PSNativeCommandErrorAction

For users that prefer to set a single preference variable that affects both PowerShell and native
commands, The values of `$PSNativeCommandErrorAction` may be extended to include
`MatchErrorActionPreference`, which should apply the `$ErrorActionPreference` setting to native
commands.

- If `$PSNativeCommandErrorAction` is set to `MatchErrorActionPreference` then the value of
  `$ErrorActionPreference` will define the behavior for native command errors.

### Explicit invocation logic

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

A common configuration command for POSIX scripts `set -euo pipefail` includes the `set -u`
configuration which returns an error if any variable has not been previously defined. This is
equivalent to the existing PowerShell `Set-StrictMode` and is not needed to be addressed in this RFC.
