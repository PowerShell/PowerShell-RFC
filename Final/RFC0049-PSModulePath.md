---
RFC: 0049
Author: Steve Lee
Status: Final
SupercededBy: n/a
Version: 1.0
Area: Engine
Comments Due: 11/30
Plan to implement: Yes
---

# PSModulePath When Starting PowerShell within PowerShell

When starting PowerShell from a different version of PowerShell, the PSModulePath
should reflect the module search path and only include segments appropriate for
the version of PowerShell being started.

## Motivation

    As a PowerShell User,
    I can start a different version of PowerShell within PowerShell,
    so that I can do things required by that specific version of PowerShell.

## User Experience

### Starting Windows PowerShell from PowerShell 7

```powershell
PS7> $env:PSModulePath.split(';')
C:\Users\user\Documents\PowerShell\Modules
C:\Program Files\PowerShell\Modules
c:\program files\powershell\7-preview\Modules
C:\Program Files\WindowsPowerShell\Modules
C:\WINDOWS\system32\WindowsPowerShell\v1.0\Modules

PS7> powershell -noprofile
WinPS> $env:PSModulePath.split(';')
C:\Users\user\Documents\WindowsPowerShell\Modules
C:\Program Files\WindowsPowerShell\Modules
C:\WINDOWS\system32\WindowsPowerShell\v1.0\Modules
```

### Starting Windows PowerShell from PowerShell 7 with additions by user

Here a custom `C:\MyModules` is added

```powershell
PS7> $env:PSModulePath.split(';')
C:\Users\user\Documents\PowerShell\Modules
C:\Program Files\PowerShell\Modules
c:\program files\powershell\7-preview\Modules
C:\Program Files\WindowsPowerShell\Modules
C:\WINDOWS\system32\WindowsPowerShell\v1.0\Modules
C:\MyModules

PS7> powershell -noprofile
WinPS> $env:PSModulePath.split(';')
C:\Program Files\WindowsPowerShell\Modules
C:\WINDOWS\system32\WindowsPowerShell\v1.0\Modules
C:\MyModules
```

### Starting Windows PowerShell from PowerShell 7 with deletions by user

Here the `System32` path is removed

```powershell
PS7> $env:PSModulePath.split(';')
C:\Users\user\Documents\PowerShell\Modules
C:\Program Files\PowerShell\Modules
c:\program files\powershell\7-preview\Modules
C:\Program Files\WindowsPowerShell\Modules

PS7> powershell -noprofile
WinPS> $env:PSModulePath.split(';')
C:\Program Files\WindowsPowerShell\Modules
```

### Starting Windows PowerShell from PowerShell 7 with additions and deletions by user

Here the `System32` path is removed and the `C:\MyModules` path is added

```powershell
PS7> $env:PSModulePath.split(';')
C:\Users\user\Documents\PowerShell\Modules
C:\Program Files\PowerShell\Modules
c:\program files\powershell\7-preview\Modules
C:\Program Files\WindowsPowerShell\Modules
C:\MyModules

PS7> powershell -noprofile
WinPS> $env:PSModulePath.split(';')
C:\Program Files\WindowsPowerShell\Modules
C:\MyModules
```

## Specification

By default, PowerShell starts with (in order):

| Segment Description | Windows PowerShell example                                               | PowerShell 7 example                              |
|---------------------|--------------------------------------------------------------------------|---------------------------------------------------|
| User modules        | C:\Users\user\Documents\WindowsPowerShell\Modules                        | C:\Users\user\Documents\PowerShell\Modules        |
| System modules      | C:\Program Files\WindowsPowerShell\Modules                               | C:\Program Files\PowerShell\Modules               |
| $PSHOME modules     | C:\WINDOWS\system32\WindowsPowerShell\v1.0\Modules                       | c:\program files\powershell\7-preview\Modules     |
| Windows modules     |                                                                          | C:\WINDOWS\system32\WindowsPowerShell\v1.0\Module |
| App added modules   | C:\Program Files (x86)\Microsoft Azure Information Protection\Powershell |                                                   |

Currently, applications (or user) adding additional paths to `$env:PSModulePath` are dropped in PowerShell 7.

### Windows PowerShell PSModulePath construction

Windows PowerShell has logic at startup to construct the PSModulePath based on (simplified version):

- If PSModulePath doesn't exist: Combine `User` modules path, `System` modules path, and `$PSHOME` modules path
- If PSModulePath does exist:
  - If PSModulePath contains `$PSHOME` modules path:
    - `System` modules path is inserted before `$PSHOME` modules path if it's not there
  - else:
    - Just use `$env:PSModulePath` as-is as user had deliberately removed `$PSHOME`

`User` module path is prefixed only if User scope `$env:PSModulePath` doesn't exist.
Otherwise, it is assumed the user removed it if it's not there already.

There are no changes planned for Windows PowerShell 5.1, so this existing logic is in place.

### PowerShell 7 startup

Currently, PSCore6 doesn't use contents of `$env:PSModulePath` if it detects it was started from PowerShell
and simply overwrites it with `User` modules path + `System` modules path + `$PSHOME` modules path + `Windows` modules path.

Intent is to have `PSModulePath` behave how `Path` env var works on Windows.
`Path` on Windows is treated differently from other env vars.
When a process is started, Windows will combine the `User` `Path` env var with the `System` `Path` env var.
However, for other env vars if the `User` env var exists, a new process will have that value only even if a `Machine` env var
with the same name exists.
In this case, the `User` version of the env var is preferred.
In the changes detailed below, `PSModulePath` adopts the `Path` behavior to have a combined value from `User` and `System` versions of that env var.

Change would be on Windows:

- Retrieve user `PSModulePath` env var from registry
- Compare to process inherited `PSModulePath` env var
  - If the same:
    - Append the `System` `PSModulePath` to the end following the semantics of the `Path` env var
    - The Windows system32 path comes from the machine defined `PSModulePath` so does not need to be added explicitly
  - If different, treat as though user explicitly modified it and don't append `System` `PSModulePath`
- Prefix with PS7 user, system, and $PSHOME paths in that order
  - If `powershell.config.json` contains a user scoped `PSModulePath`, use that instead of the default for the user
  - If `powershell.config.json` contains a system scoped `PSModulePath`, use that instead of the default for the system

Unix systems don't have a separation of `User` and `System` env vars so `PSModulePath` is inherited and PS7 specific paths are prefixed if not
already existing.

### Starting Windows PowerShell from PowerShell 7 Implementation

Note that Windows PowerShell here means both `powershell.exe` and `powershell_ise.exe`.

Copy process `$env:PSModulePath` as `WinPSModulePath`:

- Remove PS7 `User` module path
- Remove PS7 `System` module path
- Remove PS7 `$PSHOME` module path

Use that `WinPSModulePath` when starting Windows PowerShell.

### Starting PowerShell 7 from Windows PowerShell

The PowerShell 7 startup continues as-is with the addition of inheriting paths Windows PowerShell have added.
Since the PS7 specific paths are prefixed, there is no functional issue.

### Starting PowerShell 6 from PowerShell 7

PowerShell Core 6 clobbers `$env:PSModulePath` and no changes will be made.

### Starting PowerShell 7 from PowerShell 6

The PowerShell 7 startup continues as-is with the addition of inheriting paths PowerShell Core 6 have added.
Since the PS7 specific paths are prefixed, there is no functional issue.

## Alternate Proposals and Considerations

There was a proposal to cache the inherited `$env:PSModulePath` on startup and pass it to Windows PowerShell when started.
However, this would not reflect any changes the user made and expect to be inherited by the child process.
