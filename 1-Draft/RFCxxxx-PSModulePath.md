---
RFC:
Author: Steve Lee
Status: Draft
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
    C:\Users\slee\Documents\PowerShell\Modules
    C:\Program Files\PowerShell\Modules
    c:\program files\powershell\7-preview\Modules
    C:\WINDOWS\system32\WindowsPowerShell\v1.0\Modules

    PS7> powershell -noprofile
    WinPS> $env:PSModulePath.split(';')
    C:\Users\slee\Documents\WindowsPowerShell\Modules
    C:\Program Files\WindowsPowerShell\Modules
    C:\WINDOWS\system32\WindowsPowerShell\v1.0\Modules
    C:\Program Files (x86)\Microsoft Azure Information Protection\Powershell
    ```

### Starting Windows PowerShell from PowerShell 7 with additions by user

    ```powershell
    PS7> $env:PSModulePath.split(';')
    C:\Users\slee\Documents\PowerShell\Modules
    C:\Program Files\PowerShell\Modules
    c:\program files\powershell\7-preview\Modules
    C:\WINDOWS\system32\WindowsPowerShell\v1.0\Modules
    C:\MyModules

    PS7> powershell -noprofile
    WinPS> $env:PSModulePath.split(';')
    C:\Users\slee\Documents\WindowsPowerShell\Modules
    C:\Program Files\WindowsPowerShell\Modules
    C:\WINDOWS\system32\WindowsPowerShell\v1.0\Modules
    C:\Program Files (x86)\Microsoft Azure Information Protection\Powershell
    C:\MyModules
    ```

### Starting Windows PowerShell from PowerShell 7 with deletions by user

    ```powershell
    PS7> $env:PSModulePath.split(';')
    C:\Users\slee\Documents\PowerShell\Modules
    C:\Program Files\PowerShell\Modules
    c:\program files\powershell\7-preview\Modules

    PS7> powershell -noprofile
    WinPS> $env:PSModulePath.split(';')
    C:\Users\slee\Documents\WindowsPowerShell\Modules
    C:\Program Files\WindowsPowerShell\Modules
    C:\WINDOWS\system32\WindowsPowerShell\v1.0\Modules
    C:\Program Files (x86)\Microsoft Azure Information Protection\Powershell
    ```

### Starting Windows PowerShell from PowerShell 7 with additions and deletions by user

    ```powershell
    PS7> $env:PSModulePath.split(';')
    C:\Users\slee\Documents\PowerShell\Modules
    C:\Program Files\PowerShell\Modules
    c:\program files\powershell\7-preview\Modules
    C:\MyModules

    PS7> powershell -noprofile
    WinPS> $env:PSModulePath.split(';')
    C:\Users\slee\Documents\WindowsPowerShell\Modules
    C:\Program Files\WindowsPowerShell\Modules
    C:\WINDOWS\system32\WindowsPowerShell\v1.0\Modules
    C:\Program Files (x86)\Microsoft Azure Information Protection\Powershell
    C:\MyModules
    ```

## Specification

By default, PowerShell starts with (in order):

| Segment Description | Windows PowerShell example                                               | PowerShell 7 example                              |
|---------------------|--------------------------------------------------------------------------|---------------------------------------------------|
| User modules        | C:\Users\slee\Documents\WindowsPowerShell\Modules                        | C:\Users\slee\Documents\PowerShell\Modules        |
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

### PowerShell 7 startup

Currently, PS7 doesn't use contents of `$env:PSModulePath` if it detects it was started from PowerShell
and simply overwrites it with `User` modules path + `System` modules path + `$PSHOME` modules path + `Windows` modules path.

Change would be to use `$env:PSModulePath`, but prefix with `User` modules path + `System` modules path +
`$PSHOME` modules path if any of those paths are not already there.

`Windows` modules path will already be there along with any additional paths added by the user or applications.

### Starting Windows PowerShell from PowerShell 7 Implementation

Note that Windows PowerShell here means both `powershell.exe` and `powershell_ise.exe`.

Copy process `$env:PSModulePath` as `PS7PSModulePath`:

- Remove PS7 `User` module path
- Remove PS7 `System` module path
- Remove PS7 `$PSHOME` module path

Get fresh `$env:PSModulePath` from `Machine` scope if exists, otherwise `User` scope
as `WinPSModulePath`.

If any of the Windows PowerShell path segments was removed in `PS7PSModulePath`, then
remove it from `WinPSModulePath`.

Any segments in `WinPSModulePath` that exists in `PS7PSModulePath` are removed from
`PS7PSModulePath`.

Append remainder of `PS7PSModulePath` to end of `WinPSModulePath`.
Use that `WinPSModulePath` when starting Windows PowerShell.

### Starting PowerShell 7 from Windows PowerShell

The PowerShell 7 startup covers this case and should just work.

### Starting PowerShell 6 from PowerShell 7

PowerShell Core 6 clobbers `$env:PSModulePath` and no changes will be made.

### Starting PowerShell 7 from PowerShell 6

The PowerShell 7 startup covers this case and should just work.

## Alternate Proposals and Considerations
