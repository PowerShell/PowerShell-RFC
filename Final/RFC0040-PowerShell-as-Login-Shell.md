---
RFC: RFC0040
Author: Steve Lee
Status: Final
SupercededBy: N/A
Version: 1.0
Area: Console
Comments Due: July 15, 2019
Plan to implement: Yes, PS7
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

`-l` (expanded form is `-Login`) will:

- On Windows, do nothing as the login shell concept doesn't exist.
  No error, this param is ignored.
- On Unix based systems, exec `pwsh` using `sh -l` so that the normal login
  shell processing occurs and the environment is pre-populated as expected.

>[!NOTE]
> The actual path to `sh` would depend on the operating system.
> For Linux, the path would be `/usr/bin`.
> For Snap, the path would be `/snap/bin`.
> For macOS, there is an issue with `sh` not loading the login profile with
> our usage, so `pwsh` will use `zsh` with `sh` emulation mode to enable
> proper initialization of the environment.

Since `pwsh` is simply starting an instance of itself, it should work the same
regardless if a stable or preview release.

Initial performance tests indicate that `pwsh -l` will incur a 6% startup hit.
Compared to a pure Bourne shell script which has a 4% startup hit.
Most of that is simply processing the Bourne shell scripts to setup the environment.
As such, there is a relative 2% additional perf hit with this proposal.

The impact seems acceptable given that the login shell is typically the first
shell you create and subsequent use of `pwsh` would not be a login shell.

In addition, `pwsh` will need to be updated to detect if it was started with
a hyphen (for example, `-pwsh`) as that is another way that Unix based operating
systems start the default login shell and doesn't necessarily use `-l`.

On Linux, this will be implemented by inspecting `/proc` on the filesystem.
On macOS, this will be implemented by calling `sysctl()` getting the process
information for the current process and inspecting `argv[0]`.

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

### `pwsh-login` script

This proposal would be to include a Bourne shell script called `pwsh-login`
that would contain:

```sh
#!/bin/sh -l
exec /usr/local/bin/pwsh "$@"
```

The location of `sh` would depend on the specific operating system.

The perf impact of this compared to starting pwsh as a non-login shell is
approximately 4%.

### Supporting different default login shells

In the future, if relying on `sh` (and thus `/etc/profile`) is not sufficient,
we can allow an optional argument to `-login` to indicate the shell to use
to process the script that creates the environment.

### macOS moving to Zsh by default

Expectation is that macOS intends to switch to `zsh` as the default shell so
it may make sense to revisit this in the future to default to using `zsh` so
that `zprofile` gets processed.
