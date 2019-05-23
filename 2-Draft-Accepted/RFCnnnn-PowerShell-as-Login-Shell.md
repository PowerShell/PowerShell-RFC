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

Many existing tools expect `-l` to be accepted by a shell and to act as a login
shell.

`-l` (expanded form is `-LoadProfile`) will explicitly have pwsh load the PowerShell
profile.
This is effectively a no-op as if this switch is not specified, pwsh will load
the PowerShell profile and only doesn't load it if `-noprofile` is specified.

This will allow tools that expect `-l` to be accepted to work.
There is no additional work to process `/etc/profile` when `-l` is used.

For cases where you do need `/etc/profile` to be processed,
such as using pwsh as your default shell,
the proposal is to include a Bourne shell script that can be specified as the
shell:

```sh
#!/bin/sh -l
exec /usr/local/bin/pwsh "$@"
```

This script will be called `pwsh-login` and should be used whenever you require
a specific login shell.

## Alternate Proposals and Considerations

### Process /etc/profile using sh and copy the env vars

This proposal would be that if `-l` is specified, pwsh would run `/bin/sh -l -c export`
which would create a new shell process that does all the processing needed for
a login shell and export the environment variables that would be exported.
Additional code is needed to take those environment variables and copy them to
the parent pwsh process.

The downsides to this approach is additional code to maintain,
but more importantly not getting a complete login shell environment as things
like ulimit and umask would not be inherited into pwsh.

### pwsh to start `/bin/sh -l -c "exec pwsh"`

This proposal would have pwsh when given the `-l` switch to start Bourne shell
as a login shell and start pwsh within that process.

This would result in a complete login shell environment, however, would
incur the performance penalty of starting pwsh twice.
