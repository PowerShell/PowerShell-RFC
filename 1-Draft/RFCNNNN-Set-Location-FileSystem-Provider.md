---
RFC: RFC<four digit unique incrementing number assigned by Committee, this shall be left blank by the author>
Author: Glenn Sarti
Status: Draft
SupercededBy: N/A
Version: 0.1
Area: <Area within the PowerShell language>
Comments Due: <Date for submitting comments to current draft (minimum 1 month)>
Plan to implement: <Yes | No>
---

# Set-Location on the FileSystem provider should use the actual name on disk

Currently when using `Set-Location` (or it's cd alias) the FileSystem provider will use the literal text that is entered.  For example:

Given a filesystem with following layout:

``` text
C:\
  |
  + Users
  + Windows
```

Issuing the following PowerShell commands results in:

``` powershell
PS C:\> cd Users
PS C:\Users> cd ..
PS C:\> cd users
PS C:\users>
```

Note - The last command used the path `users` instead of the _actual_ name `Users`

This contrary to:

* Using tab completion
Tab completion resolves to the name on disk

* "magic" paths like tilde (`~`)

``` powershell
PS C:\> cd ~
PS C:\Users\glenn.sarti>
```

* 8.3 Alternate names

``` powershell
PS C:\> cd C:\progra~1
PS C:\Program Files>
```

This can cause issues with child process creation, as the working directory passed to the child process, strictly speaking, does not exist.  This is proved simply by calling cmd.exe:

Given the filesystem layout example above (`C:\Users` is the correct name, not `C:\users`)

``` powershell
C:\users> cmd.exe /c cd
C:\users
```

Note that cmd.exe outputs the "invalid" directory of `C:\users`

Note that, while strictly the fault of PowerShell, I have seen other cross-platform tools behave in strange ways (for example Ruby and NodeJS) where they are performing file path operations and raise errors because the current working directory is not what is on disk.

## Motivation

> As a PowerShell user,
> I can spawn child processes
> so that the spawned child process has a valid Working Directory that actually exists


> As a PowerShell user,
> I can set location on a case-insensitive filesystem
> And PowerShell will resolve the correct name for me
> so that I don't have to remember that the directory I'm in may not be _actual_ name

> As a PowerShell user,
> I can set the location on a filesystem
> so that I get a consistent experience whether I use "magic" aliases, filesystem alternate names or case-insensitivity

## User Experience

Example of user experience with example code/script.
Include example of input and output.

Given the filesystem layout above, I would expect the FileSystem provider to behave like

```powershell
PS C:\>cd users
PS C:\Users


PS C:\>Set-Location users
PS C:\Users
```

The existing behaviour of tab completion, magic aliases and alternate names is to remain the same

## Specification

The implementation of this would probably be inside the FileSystem Provider. This will probably only affect Windows based filesystems (NTFS) as Linux and OSX ones tend to be case sensitive.

## Alternate Proposals and Considerations

This may already be implemented in PowerShell 7.x [Pull Request 9250](https://github.com/PowerShell/PowerShell/pull/9250) however I would also suggest this be backported to 6.x.

This is _probably_ not a breaking change, but it may be unexpected because this is diverging from the existing behavior from, at least, as far back as PowerShell 4.0.
