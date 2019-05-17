---
RFC: RFCnnnn
Author: Kirk Munro
Status: Draft
SupercededBy: N/A
Version: 1.0
Area: Parser/Tokenizer
Comments Due: June 16, 2019
Plan to implement: Yes
---

# Support implicit line continuance when using named parameters or splatting in commands

Nobody likes having to use the backtick to wrap commands in PowerShell, yet many users still use it to get the style they prefer in their scripts.

For example, PowerShell has long supported explicit line continuance when you end a pipelined command line with a pipe symbol, like this:

```PowerShell
Get-Process -Id $PID |
    Stop-Process
```

Even though that is available, when you have a pipeline with many stages (commands) in it, the style is not as desirable as it could be if you could place the pipeline on the beginning of the line instead, like this:

```PowerShell
Get-Process -Id $PID
    | Stop-Process
```

Historically that syntax would not be possible unless you ended each line before the last line in the pipeline with a backtick. That shouldn't be necessary though, and PowerShell should be smart enough to recognize implicit line continuance when pipelines are placed at the beginning of the line. This was discussed in detail in [Issue #3020](https://github.com/PowerShell/PowerShell/issues/3020), and [PR #8938](https://github.com/PowerShell/PowerShell/pull/8938) that was recently merged added this support to PowerShell for version 7.

While that is helpful, there is another potential improvement where PowerShell could support implicit line continuance: when using named parameters or splatting in commands, and that's what this RFC is about.

For example, consider this example of a New-ADUser command invocation:

```PowerShell
New-ADUser -Name 'Jack Robinson' -GivenName 'Jack' -Surname 'Robinson' -SamAccountName 'J.Robinson' -UserPrincipalName 'J.Robinson@enterprise.com' -Path 'OU=Users,DC=enterprise,DC=com' -AccountPassword (Read-Host -AsSecureString 'Input Password') -Enabled $true
```

By itself it's not too much to handle, but in a script commands with many parameters like this can be difficult to manage it in a script. To wrap this command across multiple lines, users can either use backticks, or they can use splatting. The former is a nuisance which should really only be used in situations when PowerShell cannot implicitly intuit how lines are wrapped. The latter is helpful, but users lose the benefits of tab completion and Intellisense for parameters when they use splatting. As a workaround, they can work out the parameters they want to use for the command first, and then convert it into a splatted command, but that's generally onerous. Even though Visual Studio Code has an extension that makes splatting easier, as can be seen [here](https://sqldbawithabeard.com/2018/03/11/easily-splatting-powershell-with-vs-code/), once you've converted to splatting you still lose Intellisense for future updates unless you work from the command first and then add to your splatted collection, and that's just in Visual Studio Code. Other editors may or may not support that functionality, and users working in a standalone terminal won't have that available to them either.

Instead, why not allow users to do this by supporting implicit line continuance when using named parameters:

```PowerShell
New-ADUser
    -Name 'Jack Robinson'
    -GivenName 'Jack'
    -Surname 'Robinson'
    -SamAccountName 'J.Robinson'
    -UserPrincipalName 'J.Robinson@enterprise.com'
    -Path 'OU=Users,DC=enterprise,DC=com'
    -AccountPassword (Read-Host -AsSecureString 'Input Password')
    -Enabled $true
```

Further, if they have some parameters they want to splat in (because splatting is not just about shortening lines -- it's very useful for users to apply common parameters/values to multiple commands), let them do this as well:

```PowerShell
$commonParams = @{
    Path = 'OU=Users,DC=enterprise,DC=com'
    Enabled = $true
}
New-ADUser
    -Name 'Jack Robinson'
    -GivenName 'Jack'
    -Surname 'Robinson'
    -SamAccountName 'J.Robinson'
    -UserPrincipalName 'J.Robinson@enterprise.com'
    -AccountPassword (Read-Host -AsSecureString 'Input Password')
    @commonParams
```

## Motivation

    As a script/module author,
    I can wrap long commands across multiple lines at any named parameter or splatted collection,
    so that my scripts are easier to maintain while I still get the benefits of Intellisense and tab completion.

## Specification

Note: This RFC is already implemented and submitted as [PR #9614](https://github.com/PowerShell/PowerShell/pull/9614).

Since this RFC includes some breaking changes (see below), it will be initially implemented with the `PSImplicitLineContinuanceForNamedParameters` experimental feature flag. If the PowerShell Team and the community agree that the risk of this breaking change is low enough, and the workaround is sufficient for those low frequency use cases where it does become an issue, I would much prefer not using an experimental feature flag at all.

For the implementation, this RFC builds on the work done in [PR #8938](https://github.com/PowerShell/PowerShell/pull/8938), evolving the logic used in the tokenizer that supports implicit line continuance for lines starting with a pipe such that it can be used to check for implicit line continuance for named parameters and splatted collections as well. To keep performance optimal, the parser logic that identifies command parameters where the implicit line continuance checks will be invoked will recognize if a pipe is found on a subsequent line as well, to ensure the lookahead logic is only invoked once per line in commands/pipelines.

## Alternate Proposals and Considerations

While implicit line continuance for splatted collections is not a breaking change (PowerShell syntax does not support `@variableName` at the start of a command), implicit line continuance for named parameters is a breaking change, because PowerShell syntax currently supports commands that start with dash.

For example, consider this script:

```PowerShell
function -dash {
    'run'
}

-dash
```

In practice, I believe this to be a very low risk breaking change, because it is unlikely that (m)any users have defined functions or aliases that start with a dash, and in researching commands on Windows and Linux I was not able to find any commands with names that start with dash. However, in the event that someone is using such a command, they can still invoke it even if this breaking change is applied, by using the call operator as follows:

```PowerShell
& -dash
```
