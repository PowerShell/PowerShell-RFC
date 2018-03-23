---
RFC: RFC
Author: Robert Holt
Status: Draft
SupercededBy:
Version:
Area: Hash-requires
Comments Due: 2018-05-15
Plan to implement: Yes
---

# \#Requires Additions

Currently, PowerShell's `#requires` statement (or perhaps
pragma?) supports the following parameters:

* `-Version <N>[.<n>]`, where a minimum PowerShell version can be specified
* `-PSSnapin <PSSnapin-Name> [-Version <N><n>]`, where a required PowerShell Snapin can be specified
* `-Modules { <Module-name> | <Hashtable> }`, where PowerShell modules that are required can be specified
* `-ShellId <ShellId>`, where the required Shell ID can be specified
* `-RunAsAdministrator`, where the script is required to be run as administrator

These features are documented in [about_Requires](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_requires?view=powershell-6), along with the statement that:

> You can use #Requires statements in any script. You
> cannot use them in functions, cmdlets, or snap-ins.

Currently however, this is untrue, as `#requires`
statements are allowed by the parser/tokenizer anywhere in
a script and then effectively hoisted to the top of that
script to be checked before any part of the script is
executed, no matter where they are placed.

This RFC proposes the following changes:

* Only allow `#requires` at the top level of a script,
  before any lines that are not comments (i.e. with the
  intention that a hashbang can still work, just before
  any executable PowerShell code). Placing `#requires` anywhere
  after will cause a parse-time error. This would be a **breaking
  change**, albeit one that the documentation already claims to be 
  in force.
* Using `#requires` in the interactive console will cause
  a parse-time error. This could be a **minor breaking
  change**, since currently PowerShell throws a [pipeline
  creation error](https://github.com/PowerShell/PowerShell/issues/3803).
* Add support for the following new parameters (each
  independently up for discussion):
  * `-OS {Windows | Linux | MacOS}`, where an
    operating system (or possibly combination of them) can
    be specified as required. See [this PowerShell issue](https://github.com/PowerShell/PowerShell/issues/3751).
  * `-Assembly <Assembly-name>`, where a .NET assembly can
    be specified as required. See [this PowerShell issue](https://github.com/PowerShell/PowerShell/issues/5022).
  * `-MaxVersion <V>[.<v>]`, where a maximum PowerShell
     version can be specified as required. See [this PowerShell issue](https://github.com/PowerShell/PowerShell/issues/2846).

## Motivation

> As a PowerShell user, I can be sure that all the
> `#requires` statements in a script come before any
> PowerShell code, so that it's clear they always
> execute first, and so they can all be found easily.

> As a PowerShell user, I get feedback that `#requires`
> statements cannot be used in the interactive console,
> so that it's clear that they have no effect on an
> interactive session.

> As a PowerShell user, I can specify that my script
> `#requires` being run on a specific operating system,
> so that I can effectively, efficiently and declaratively
> guarantee that it is used only on systems I designed it
> for.

> As a PowerShell user, I can specify that my script
> `#requires` a given .NET Assembly to run in an
> efficient and declarative way, so that I don't have
> to do complex runtime logic to determine my script
> cannot run.

> As a PowerShell user, I can specify that my script
> `#requires` to be run in a version of PowerShell
> lower than a given version, so that I can declaratively
> prevent it from being run in an environment where
> changes to PowerShell would cause unintended behavior.

## Specification

1. `#requires` statements must appear in scripts
   above all executable PowerShell. Any `#requires`
   statement placed after any PowerShell code causes
   an unrecoverable parse-time error. This new restriction
   should not affect the usage of other comment-embedded
   directives or pragmas, such as `#sig`, linter directives or
   inline editor configurations. Specifically, the new `#requires`
   placement restriction must not interfere with Unix-style
   hashbangs (e.g. `#!/usr/bin/pwsh`).
2. Any use of `#requires` in an interactive session causes
   a specific parse-time error to be thrown, informing the
   user that `#requires` may not be used in the interactive
   console.
3. `#requires` can take an `-OS` parameter, with possible
   arguments being `Windows`, `Linux` and `MacOS` (and
   possibly some syntax for `or`-ing them). This check
   succeeds if-and-only-if the correspoding runtime
   PowerShell variable (`$IsWindows`, `$IsLinux` and
   `$IsMacOS` is true). Requiring a given OS when the
   corresponding runtime variable is false results in
   a pre-execution error with a specific error message
   stating that the script is required to be run on a
   different operating system.
4. `#requires` can take an `-Assembly` parameter, with
   possible arguments being exactly what a `using assembly`
   statement will accept. If a required assembly is not
   present on the executing system, a pre-execution error
   is raised.
5. `#requires` can take a `-MaxVersion` parameter, with
   a major version and optional minor version, to define
   the maximum (inclusive) version of PowerShell it should
   run on. The version given does not need to correspond to
   any version of PowerShell, but will just be compared in
   standard lexicographic tuple order. Executing a script
   required to be on a version of PowerShell strictly lower
   than the executing version results in a pre-execution
   error.

Finally, scripts with `#requires` in them should still
be editable in contexts that do not satisfy their 
requirements. For example, a script with `#requires -OS
MacOS` at the top should still allow a full editing user
experience on Windows or Linux. For context, see [this
PowerShell issue](https://github.com/PowerShell/PowerShell/issues/4549).

## Alternate Proposals and Considerations

* An `-Assembly` parameter may be unneccessary, given the
  possibility of using `using assembly <Assembly-name>`.
* Given the suite of proposed changes to `#requires`, any
  other proposed parameters for `#requires` are worth
  including and discussing in this RFC. Possible
  considerations are:
  * `-LanguageMode`, where a script must be run in a given
    PowerShell language mode.
  * `-Architecture`, where a script must be run on a
  machine with a given processor architecture.
  * `-Platform`, rather than trying to use combining
    logic with `-OS`.
* Another requires parameter, `-PSEdition`, also seems to have
  been added to the `#requires` functionality. However, it is
  currently [undocumented](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_requires?view=powershell-6) and
  there is an [open issue for it](https://github.com/PowerShell/PowerShell/issues/5908). It may
  be worth discussing in this RFC.
* Because of the pipeline-crash behavior of the interactive
  usage of `#requires`, there is already a PR open to change
  the behavior to what is described in this RFC.
