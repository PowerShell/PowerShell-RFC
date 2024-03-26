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

cat ./testfile.sh

```bash

#!/bin/bash
echo "Without set -u : No output - No error is produced"
echo "$firstname"
echo "With set -u : Equivalent to Set-StrictMode -Version 2.0"
set -u
echo "$firstname"
```

./testfile.sh

```output
Without set -u : No output - No error is produced

With set -u : Equivalent to Set-StrictMode -Version 2.0
./testfile.sh: line 7: firstname: unbound variable
```

Get-Content ./testfile.ps1

```powershell

Write-Output "Without Set-StrictMode -version 2.0 : No output - No error is produced"
Write-Output "$firstname"
Write-Output "With Set-StrictMode -version 2.0 : Equivalent to set -u"
Set-StrictMode -version 2.0
Write-Output "$firstname"
```

./testfile.ps1

```output
Without Set-StrictMode -version 2.0 : No output - No error is produced

With Set-StrictMode -version 2.0 : Equivalent to set -u
InvalidOperation: /Users/jasonhelmick/natcmdbash/testfile.ps1:5
Line |
   5 |  Write-Output "$firstname"
     |                ~~~~~~~~~~
     | The variable '$firstname' cannot be retrieved because it has not been set.
```

### set -e/ $ErrorActionPreference

In the example below, `set -e` is not equivalent to `$ErrorActionPreference` for native commands.

cat ./testfile.sh

```bash
#!/bin/bash
echo "Without set -e : Will receive message after failure"
cat ./nofile
echo "Message After failure"
echo "With set -e : Will NOT continue script execution after failure"
set -e
cat ./nofile
echo "Message After failure"
```

./testfile.sh

```output
Without set -e : Will receive message after failure
cat: ./nofile: No such file or directory
Message After failure
With set -e : Will NOT continue script execution after failure
cat: ./nofile: No such file or directory
```

Get-Content ./testfile.ps1

```powershell
Write-Output "Without `$ErrorActionPreference : Will receive message after failure"
Get-Content -path ./nofile
Write-Output "Message After failure"
/bin/echo "With (Bash) `$ErrorActionPreference = 'Stop' : SHOULD NOT continue script execution after failure - but does"
$ErrorActionPreference = "Stop"
/bin/cat ./nofile
/bin/echo "Message After failure"
Write-Output "With (PowerShell) `$ErrorActionPreference = 'Stop' : SHOULD NOT continue script execution after failure"
$ErrorActionPreference = "Stop"
Get-Content -path ./nofile
Write-Output "Message After failure"
```

./testfile.ps1

```output
Without $ErrorActionPreference : Will receive message after failure
Get-Content: /Users/jasonhelmick/natcmdbash/testfile.ps1:2
Line |
   2 |  Get-Content -path ./nofile
     |  ~~~~~~~~~~~~~~~~~~~~~~~~~~
     | Cannot find path '/Users/jasonhelmick/natcmdbash/nofile' because it does not exist.

Message After failure
With (Bash) $ErrorActionPreference = 'Stop' : SHOULD NOT continue script execution after failure - but does
cat: ./nofile: No such file or directory
Message After failure
With (PowerShell) $ErrorActionPreference = 'Stop' : SHOULD NOT continue script execution after failure
Get-Content: /Users/jasonhelmick/natcmdbash/testfile.ps1:10
Line |
  10 |  Get-Content -path ./nofile
     |  ~~~~~~~~~~~~~~~~~~~~~~~~~~
     | Cannot find path '/Users/jasonhelmick/natcmdbash/nofile' because it does not exist.
```

### set -o pipefail

In the example below, PowerShell has no equivalent to `set -o pipefail` for native commands.

cat ./testfile.sh

```bash
#!/bin/bash
echo "Without set -o pipefail : returns zero (true)"
cat ./nofile | echo "pipe statement after failure"
echo "returns $?"
echo "With set -o pipefail : returns non-zero (false)"
set -o pipefail
cat ./nofile | echo "pipe statement after failure"
echo "returns $?"
```

./testfile.sh

```output
Without set -o pipefail : returns zero (true)
pipe statement after failure
cat: ./nofile: No such file or directory
returns 0
With set -o pipefail : returns non-zero (false)
pipe statement after failure
cat: ./nofile: No such file or directory
returns 1
```

Get-Content ./testfile.ps1

```powershell
Write-Output "Without `$ErrorActionPreference = 'Stop' : should return zero (true)"
Get-Content -path ./nofile | Write-Output "pipe statement after failure"
Write-Output "returns $?"
Write-Output "With set -o equiv. `$ErrorActionPreference = 'Stop' : should return non-zero (false)"
$ErrorActionPreference = 'Stop'
Get-Content -path ./nofile | Write-Output "pipe statement after failure"
Write-Output "returns $?"
```

./testfile.ps1

```output
Without $ErrorActionPreference = 'Stop' : should return zero (true)
Get-Content: /Users/jasonhelmick/natcmdbash/testfile.ps1:2
Line |
   2 |  Get-Content -path ./nofile | Write-Output "pipe statement after failu …
     |  ~~~~~~~~~~~~~~~~~~~~~~~~~~
     | Cannot find path '/Users/jasonhelmick/natcmdbash/nofile' because it does not exist.

returns False
With set -o equiv. $ErrorActionPreference = 'Stop' : should return non-zero (false)
Get-Content: /Users/jasonhelmick/natcmdbash/testfile.ps1:6
Line |
   6 |  Get-Content -path ./nofile | Write-Output "pipe statement after failu …
     |  ~~~~~~~~~~~~~~~~~~~~~~~~~~
     | Cannot find path '/Users/jasonhelmick/natcmdbash/nofile' because it does not exist.
```

> [!NOTE] A common configuration command for POSIX shells `set -euo pipefail` includes the `set -u`
> configuration which returns an error if any variable has not been previously defined. This is
> equivalent to the existing PowerShell `Set-StrictMode` and does not need to be addressed in this
> RFC.

### $PSNativeCommandUseErrorActionPreference

The `$PSNativeCommandUseErrorActionPreference` preference variable will be implemented as a boolean.

- When enabled (1), `$ErrorActionPreference` will affect native commands with described behavior.
- When disabled (0), `$ErrorActionPreference` will have no effect to the original behavior.
- The default value is disabled (0) to maintain backward compatibility.
- For non-zero exit codes and except for the value `Ignore`, an `ErrorRecord` will be added to
  `$Error` that wraps the exit code and the command executed that returned the exit code.

### Error object

The reported error record object will be the new type: `NativeCommandException` with the following details:

| Property        | Definition
----------------  | -------------------
| ExitCode:       |  The exit code of the failed command.
| ErrorID:        | `"Program {0} ended with non-zero exit code {1}"`, with the command name and the exit code, from resource string `ProgramFailedToComplete`.
| ErrorCategory:  | `ErrorCategory.NotSpecified`.
| object:         | Exit code
| Source:         | The full path to the application
| ProcessInfo     | Details of failed command including path, exit code, and PID
| TargetObject    | Specifies the object that was being processed when the error occurred.

### Preference variable resolves error handling conflicts

In cases where an existing script already handles non-zero native command errors, the preference
variable `$PSNativeCommandUseErrorActionPreference` may be set to `$false`. Error handling behavior
of the PowerShell cmdlets is handled separately with `$ErrorActionPreference`.

## Alternative Approaches and Considerations

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
