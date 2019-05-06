---
RFC: RFCnnnn
Author: Steve Lee
Status: Draft
SupercededBy: N/A
Version: 1.0
Area: Console
Comments Due: June 6, 2019
Plan to implement: Yes
---

# Enable PowerShell as Login Shell on POSIX-based systems

POSIX shells on Unix-based systems use a `-l` switch to start the shell as a login shell.
This means the shell is not being started from an existing configured environment and
needs to execute `/etc/profile` to populate environment variables along with other
environment changes.

## Motivation

    As a PowerShell user,
    I can use PowerShell as my login shell on Unix-based systems,
    so that I can use PowerShell everywhere.

## Specification

`/etc/profile` is a Bourne shell (sh) script that is executed to configure the
environment and used with login shells.
On Linux systems, this script will typically call out to other scripts (`rc` files)
to perform their own configuration.
On macOS, this script executes `path_helper` which sets up the `$env:PATH`
environment variable and then if the shell is `bash` executes `bashrc`.

Many existing tools expect `-l` to be accepted by a shell and to act as a login
shell.
Without supporting `/etc/profile`, some environment variables will be missing
in PowerShell rendering it limited as a login or default shell.

If `-l` (or `-LoadProfile`) is specified, PowerShell will execute `/etc/profile`
using `sh` in a new process and dump out the resulting environment variables.
It will then set those variables in the current PowerShell process.

On Windows, this switch will only explicitly load the PowerShell profiles.

Since PowerShell is starting a new process to process `/etc/profile` which itself
may start child processes like `path_helper`, there should not be more than 100ms
impact when `-l` is specified.

## Alternate Proposals and Considerations

This RFC is not intended to address the default behavior of PowerShell to load
the PowerShell profile.
`-NoProfile` is still required to have PowerShell not execute PowerShell profiles.

The implementation is intended to be limited to just environment variables.
`umask` settings, for example, will not be inherited to PowerShell.
