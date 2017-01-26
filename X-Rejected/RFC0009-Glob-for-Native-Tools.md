---
RFC: 0009
Author: Andrew Schwartzmeyer
Status: Rejected
Version: 0.1
Area: Globbing
---

# Extend globbing to native tools

This is a request for comment on the extension of PowerShell globbing to native tools.
Currently, PowerShell supports globbing of file path arguments to cmdlets;
but the support does not extend to the use of native tools instead of cmdlets.

## Motivation

For example, `Get-ChildItem *.md`, will list all files ending with `.md`.
After parameter binding has occurred, the `LocationGlobber` is invoked,
which expands the wildcard `*` and resolves the expanded paths.
However `/bin/ls *.md` fails with:

    /bin/ls: cannot access '*.md': No such file or directory

because the native Linux tool `ls` expects the calling shell
(in this case PowerShell) to have expanded `*` to a space separated list of relative paths.

This problem is not as prevalent on Windows because Windows command-line tools,
in the general case, perform their own globbing when needed,
since neither the Windows command prompt nor PowerShell have performed it for them.

Conversely, Linux command-line tools were developed in an ecosystem
where all calling shells perform globbing of wild cards in arguments.
Since we can expect Linux users of PowerShell to use Linux command-line tools,
we are highly motivated to support this expected shell behavior for those tools.

This author first ran into the problem when attempting to compile a small C program while in PowerShell:

    gcc *.c

which is use case similar to many others,
and the lack of globbing by PowerShell ended my attempt.

An additional motivation is the fact that the expansion of `~` to the user's home directory
is provided by PowerShell's glob expansion.
Without globbing, it is sent to native tools literally.

## Specification

Extending this globbing to native tools is conceptually easy.
While the native command parameter binder does not support advanced PowerShell binding mechanisms
(since native tools are not cmdlets),
we can call the `LocationGlobber` on each argument individually,
and pass the expanded arguments to the native tool.

However, this comes with multiple caveats:

First, we cannot expect to use any of our own common parameters,
because the native tool handles its own options and arguments.
So we lose the ability to specify, for instance, `-Hidden`,
and instead must default to expanding all possible files.

Second, we must not fail. If the argument cannot be expanded,
we have to pass it unchanged to the native tool.
This is because, without PowerShell's parameter bindings,
we cannot distinguish `--option`, `some_arbitrary_argument`,
and `a/file/path/to/glob*`.
So we have to attempt the globbing on each argument,
only replace the globbed argument if it succeeds;
and pass the original argument if it fails.

Third, we should continue to respect the verbatim marker,
`--%`, so that all globbing can be turned off entirely.

Fourth, PowerShell's globbing does not support escape characters.
So ``Get-ChildItem `*`` is still expanded.
This is because the cmdlet supports the option of `-LiteralPath`
which disables globbing; but this parameter is not available to native tools.
Supporting escape characters in the `LocationGlobber` would be a major breaking change,
and so both out of scope, and likely out of consideration.
The suggested solution for those needing to use glob characters literally
is to use the supported verbatim marker.
Thus point number three is even more important.

With these caveats in mind, the following implementation is proposed:

This entire extension of globbing should be compiled only for the
Linux configuration of PowerShell, so as to not break existing Windows behavior.

If the verbatim marker is specified;
remove it and do no globbing (keep existing behavior).

Else, for each argument, attempt to expand it using the existing API:

    LocationGlobber.GetGlobbedProviderPathsFromMonadPath

Specifically set `allowNonexistingPaths = false` so that commands such as `git reflog`
continue to work (where `reflog` would otherwise be a non-existing path to glob to `/pwd/reflog`).
This unfortunately prevents `mkdir ~/newdir` from working as expected,
since `~/newdir` should be globbed to `/home/user/newdir` even though it does not yet exist;
but there is no way to differentiate it from a random argument.

Do not check if the argument contains glob characters using the API:

    LocationGlobber.StringContainsGlobCharacters

Since `~` is not treated as a glob character, but is expanded by the globbing system.

If the expansion succeeds, replace the expanded argument with the
space-separated concatenation of the resulting expansions.
If any error in the expansion occurs (exception or no results),
keep the original argument.

Before expanding, setup the context to include both hidden and normal files;
existing Linux shells do not differentiate these types when expanding globs.
This author cannot think of a scenario where glob expansion should prefer only one type or the other,
so this does not need to be configurable.

## Alternate Proposals and Considerations

### Absolute versus relative paths

The default behavior of PowerShell's globber is to resolve to the absolute path of the match;
but existing Linux environments expect globbing to return relative paths.
(However, the major exception to this rule is expansion of `~` to
the absolute path of the user's home directory.)

Common globbing behavior on Linux looks like these examples:

- The argument `*.md` expands to `README.md CHANGELOG.md`.

- The argument `~/bin` expands to `/home/user/bin`.

- The argument `~/sub/../*` expands to `/home/user/sub/../one /home/user/sub/../two`,
  (i.e. the `~` is expanded but the `..` is left intact).

But using PowerShell's default behavior, globbing looks like this:

- The argument `*.md` expands to `/path/to/README.md /path/to/CHANGELOG.md`.

- The argument `~/bin` expands to `/home/user/bin`
  (this will not need to be changed).

- The argument `~/sub/../*` expands to `/home/user/one /home/user/two`,
  (i.e. the `~` is expanded but the `..` is resolved away).

The differences between these two behaviors is mostly cosmetic.
While there are a few edge cases where relative paths will need to be kept intact,
for instance, when creating relative symlinks with `ln -s ../ previous_dir`,
the majority of use cases will see no difference between absolute and relative paths,
(the former are simply noisier).
There is an additional edge case where a subcommand shares
the same name as file existing in the current directory.

A contrived example:

```powershell
> mkdir status
> git status
git: '/home/andrew/src/PowerShell/status is not a git command. See 'git --help'.
> git --% status
On branch globbing
Your branch is up-to-date with 'origin/globbing'.
```

These edge cases can be handled with the use of the verbatim marker to disable globbing.

With the additional consideration that disabling path resolution within
PowerShell's globber would introduce lots of extra (potentially breaking) changes,
the initial extension of globbing to native tools should operate with resolved (absolute) paths.

However, if we could remove the absolute path resolution,
then `status` would be expanded to `status`, resolving the above edge case.
Likewise we could set `allowNonexistingPaths = true`,
and enable the use of `mkdir ~/newdir`.
So if these change can be made with minimal chance of breaking changes,
it should be made.

### Configurability

It may be desirable for the extension of native globbing to be togglable,
perhaps by a new PowerShell preference variable, `$NativeGlobbingPreference`.
If the extension is brought back to the Windows build of PowerShell,
this will be necessary, so as to not break existing Windows PowerShell behavior.
However, it would be sufficient for the initial implementation to exist only on Linux,
where a breaking change such as this is not breaking anything,
and togglability is available via the verbatim marker.

#### Cross-platform inconsistency

For tools that exist on both Windows and Linux, such as Git and GCC,
we should avoid breaking scripts which use these tools.
But this might not be that difficult.

Git, for instance, implements its own globbing on wildcard characters passed
to it literally. This was done so that `git add file*` works *as expected by Git users*
on Windows where globbing is not normally done.
This is also the case for Git on Linux in PowerShell;
it does not differentiate between platforms,
but it handles both cases by globbing when there is a glob character,
and accepting pre-globbed arguments when the shell performed the globbing.

GCC through MinGW (i.e. not Cygwin), behaves *differently* on Windows.
GCC on Linux, per the example above, does not perform its own globbing.
But GCC on Windows through MinGW performs globbing.
Regardless, if the shell starts to perform globbing for GCC through MinGW,
it will not break as it will receive file paths,
which are an accepted input.

Thus, at least for the initial implementation,
there does not seem to be a requirement to implement globbing on Windows.

---------------
## PowerShell Committee Decision

### Voting Results

Jason Shirk: Reject 

Joey Aiello: Reject

Bruce Payette: Absent

Steve Lee: Reject

Hemant Mahawar: Reject

### Majority Decision

We want to optimize for the future and not the past.
Create an environment where native commands can assume globbing will be handled for it on both Windows and Linux in a consistent way.
Current Windows policy is that cmd.exe is for heritage scripts and therefore we only need to solve this for PowerShell.
Recommendation is for PowerShell to implement globbing consistent with man glob(7).
To deal with native commands that use globbing characters, we provide an exclusion list that is user extensible.

Globbing must work for PowerShell Core 6.0.
The gcc and git use cases must work.

### Minority Decision

N/A
