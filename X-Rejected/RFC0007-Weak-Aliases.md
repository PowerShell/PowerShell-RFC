---
RFC: 0007
Author: Jason Shirk
Status: Rejected
Area: Command Resolution
---

# Weak Aliases

This RFC is an attempt to address the communities desire to [remove the aliases `curl` and `wget`]((https://github.com/PowerShell/PowerShell/pull/1901)) from Windows PowerShell.

## Motivation

The introduction of the aliases `curl` and `wget` happened in PowerShell V3 and was an unintentional breaking change.
Regretfully, removing these aliases now would also be a breaking change.

An analysis of scripts from various sources was performed to understand the usage patterns of `curl` in PowerShell scripts.
The source of scripts included the Windows code base, GitHub and a corpus of scripts collected by the PowerShell team from various sources like poshcode.org.

There is a [gist](https://gist.github.com/lzybkr/3cd091334355f381d1d6ee7acfad5a48) with the code I used to analyze the scripts from GitHub.

My analysis concludes that `curl` calls `Invoke-WebRequest` approximately twice as often as `curl.exe`.

In the Windows code base, usage of `curl` was not common, but was primarily used as an alias for `Invoke-WebRequest`.

I analyzed 667 scripts on GitHub that contained the string `curl`.

In 465 scripts, `curl` was not used as a command name.

In 124 scripts, my analysis concluded that `curl.exe` was the intended command, but 3 specific scripts appeared many times in the data set. Eliminating those duplicates, we end up with 45 scripts using `curl.exe`.

The remaining scripts are assumed to use `Invoke-WebRequest` - approximately 75 scripts.

Removing the aliases breaks scripts in another unexpected way as well.
Approximately 10 or so scripts remove the alias, presumably to call `curl.exe`, e.g.
```powershell
rm alias:curl
```
If the alias does not exist, this will result in an error, possibly breaking the script in an unexpected way.

There are other scenarios where removal of the aliases cause undue difficulties for our customers.

* PowerShell usage in compiled code - more difficult to discover
* Windows PowerShell updates as part of Windows, possibly breaking *users* of scripts that don't know how to fix the script

The conclusion of this analysis is that removing the aliases would cause enough problems that we should prefer another solution.

## Specification

This proposal is inspired by the notion of a [weak symbol](https://en.wikipedia.org/wiki/Weak_symbol) used by C/C++ compilers in the ELF and COFF object formats.

In PowerShell, an alias has the highest order of command precedence.
This proposal introduces a weak alias which would have the lowest order of command precedence.

A weak alias can be created in various ways - both in C# and PowerShell:
```C#
    var alias = new SessionStateAliasEntry("curl", "Invoke-webRequest");
    alias.Weak = true;
    // Add alias to InitialSessionState
```

```powershell
New-Alias -Name curl -Value Invoke-WebRequest -Weak
```

Command resolution changes as follows:

1. During the search for aliases, if a weak alias is found, it is saved and the search resumes with looking for functions, cmdlets, and external commands.
2. If no other commands are found, the saved weak alias is used.

Note that this lookup describes a long standing confusing feature where `Get-*` commands may be invoked without the `Get-` prefix.
With the introduction of weak aliases, we can codify this quirk of command resolution by creating a weak alias for all `Get-*` cmdlets, e.g.

```powershell
New-Alias -Name Process -Value Get-Process -Weak
New-Alias -Name ChildItem -Value Get-ChildItem -Weak
# etc
```

This also helps with discoverability when `Get-Command` returns these weak aliases.

## Alternate Proposals and Considerations

### Considerations

Adding weak aliases for `curl` and `wget` could still be considered a breaking change.
A script that means to use `Invoke-WebRequest` may stop working correctly if `curl.exe` is installed on a system.

### Alternate Proposals

Another way to implement a weak alias is to allow multiple definitions, e.g.

```powershell
New-Alias -Name curl -Definition 'curl.exe','Invoke-WebRequest'
```

Resolution would try each definition in the order specified.
This design suffers a small problem of knowing the exact extension for the external command, which might be undesirable if the external command is often wrapped in another script, e.g. `curl.cmd`.

There are some suggestions to choose the appropiate target command based on parameters.
This suggestion has some significant flaws as in some cases, it is impossible to distinguish which target command was desired.   

---------------
## PowerShell Committee Decision

### Voting Results

Jason Shirk: Reject 

Joey Aiello: Reject

Bruce Payette: Reject

Steve Lee: Reject

Hemant Mahawar: Absent

### Majority Decision

The [feedback](https://github.com/PowerShell/PowerShell-RFC/issues/16) was extremely helpful as lead to some great discussion and insights.
After discussion of the RFC and the feedback with the PowerShell team, the committee voted unanimously to reject this RFC.
We will continue to develop a better solution to address this problem.

The Weak Alias RFC as it's currently written does not solve the overall problem with aliases that collide with existing tools.
Although it resolves some specific use cases, it introduces other problems which may lead to unpredictable and inconsistent results.
The wget/curl aliases were already removed from PowerShell Core so the problem was limited to Windows PowerShell.

The PowerShell team have discussed several ideas to help mitigate this issue on Windows PowerShell and over time educate users to not rely on aliases in scripts.
PSScriptAnalyzer already contains a rule to discourage the use of aliases in scripts and more work may be done to enforce the rule.
A new RFC will be drafted introducing the capability for deprecation in PowerShell to inform users of potential future breaking changes.

### Minority Decision

N/A
