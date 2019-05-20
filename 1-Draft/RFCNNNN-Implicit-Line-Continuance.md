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

# Support implicit line continuance when using named parameters, splatting, or the stop parsing sigil in commands

Consider this example of a New-ADUser command invocation:

```PowerShell
New-ADUser -Name 'Jack Robinson' -GivenName 'Jack' -Surname 'Robinson' -SamAccountName 'J.Robinson' -UserPrincipalName 'J.Robinson@enterprise.com' -Path 'OU=Users,DC=enterprise,DC=com' -AccountPassword (Read-Host -AsSecureString 'Input Password') -Enabled $true
```

By itself it's not too much to handle, but in a script commands with many parameters like this can be difficult to manage. To wrap this command across multiple lines, users can either use backticks, or they can use splatting. The former is a syntactical nuisance which should really only be used in situations when PowerShell cannot implicitly intuit how lines are wrapped. The latter is helpful, but users lose the benefits of tab completion and Intellisense for parameters when they use splatting. As a workaround, they can work out the parameters they want to use for the command first, and then convert it into a splatted command, but that's onerous. Even though Visual Studio Code has an extension that makes splatting easier, as can be seen [here](https://sqldbawithabeard.com/2018/03/11/easily-splatting-powershell-with-vs-code/), once you've converted to splatting you still lose Intellisense for future updates unless you work from the command first and then add to your splatted collection, and that's just in Visual Studio Code. Other editors may or may not support that functionality, and users working in a standalone terminal won't have that available to them either.

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

And lastly, if users have a bunch of arguments they want to pass after the stop parsing sigil (`--%`), let them do this also (thanks @Jaykul for the suggestion):

```PowerShell
&"./plink.exe"
    --% $Hostname -l $Username -pw $Password $Command
```

For the last one, `--%` stops the parsing of that command, so no further implicit line continuance may be used because that would require the parser which the scripter opted out of using for that command.

## Motivation

    As a script/module author,
    I can wrap long commands across multiple lines at any named parameter, splatted collection, or the stop-parsing sigil
    so that my scripts are easier to write and maintain while I still get the benefits of Intellisense and tab completion.

## Specification

Note: This RFC is already implemented and submitted as [PR #9614](https://github.com/PowerShell/PowerShell/pull/9614) in case you want to try it out early and see what it's like.

Since this RFC includes some breaking changes (see below), it will be initially implemented with the `PSImplicitLineContinuanceForNamedParameters` experimental feature flag. If the PowerShell Team and the community agree that the risk of this breaking change is low enough, and the workaround is sufficient for those low frequency use cases where it does become an issue, I would much prefer not using an experimental feature flag at all.

For the implementation, this RFC builds on the work done in [PR #8938](https://github.com/PowerShell/PowerShell/pull/8938), evolving the logic used in the tokenizer that supports implicit line continuance for lines starting with a pipe such that it can be used to check for implicit line continuance for named parameters and splatted collections as well. To keep performance optimal, the parser logic that identifies command parameters where the implicit line continuance checks will be invoked will recognize if a pipe is found on a subsequent line as well, to ensure the lookahead logic is only invoked once per line in commands/pipelines.

## Alternate Proposals and Considerations

While implicit line continuance for splatted collections is not a breaking change (PowerShell syntax does not support `@variableName` at the start of a command), implicit line continuance for named parameters is a breaking change in some scenarios, because PowerShell syntax currently supports commands that start with dash, and PowerShell supports unary operators whose name starts with a single dash.

For example, consider this script:

```PowerShell
function -dash {
    'run'
}

Get-Process -Id $PID
-dash
```

In PowerShell 6.x and earlier, two commands will be executed: `Get-Process`, and `-dash`. With this PR in place and the experimental feature enabled, only one command would be executed: `Get-Process`. The reason is that the parser would identify `-dash` as being intended as a parameter on the command on the previous line. That's how the parser needs to work to make this implicit line continuance functional.

Similarly, consider this script:

```PowerShell
Get-Process -Id $PID
-split 'a b c d'
```

Similar to the previous example, in PowerShell 6.x and earlier, one command and one statement will be executed: `Get-Process`, and the unary `-split` statement. With this PR in place and the experimental feature enabled, only one command would be executed: `Get-Process`, because again, the parser would identify `-split` as being intended as a parameter on the command on the previous line. The same applies to the `-join` unary operator as well. It does not apply to the `--` prefix arithmetic operator because the parser knows parameter names cannot start with a dash.

To fix this, users can do one of the following:

1. For either example, insert a blank line between `Get-Process` and the next command that starts with dash, as shown here:

    ```PowerShell
    function -dash {
        'run'
    }

    Get-Process -Id $PID

    -dash # Runs the -dash command because implicit line continuance stops looking when it encounters a blank line

    Get-Process -Id $PID

    -split 'a b c d' # splits the string 'a b c d' into an array with four items
    ```

2. In the example with a command that starts with a dash, invoke the command using the call operator, as shown here:

    ```PowerShell
    function -dash {
        'run'
    }

    Get-Process -Id $PID
    & -dash # Runs the `-dash` command because implicit line continuance sees the call operator and recognizes that the line does not continue further
    ```

    Unfortunately this workaround does not work for the -split or -join unary operators, because you cannot invoke statement that starts with a unary operator with the call operator (the call operator is only for invoking commands).

In practice, I believe that there are not many commands out there that start with a dash. While researching this, I couldn't find a single command on Windows or Linux that starts with a dash, so that's not a very risky scenario. However, since the `-split` and `-join` unary operators exist in PowerShell, there is a good likelihood that those may be used on a line following a command, and that is where this breaking change would have the most potential impact. While I don't feel that risk is enough to reject this proposal entirely, because it adds significant value to script authors, it is worth considering what the best approach would be.

Special casing the named unary operators could work (among the thousands of commands on my system, none of them have split or join parameters, but some may exist somewhere), but that would mean future unary operators be special cased as well, so that approach isn't desirable because it risks future breaking changes.

If the risk is deemed to be too high because of the risk with named unary operators, at a bare minimum I feel this feature offers enough significant value to the community that it should not be rejected, but instead offered as an optional feature so that users wanting the benefit can opt-into the functionality, and mark it as enabled for their scripts/modules. I also suspect the majority of scripters would want it turned on and would then simply write their scripts accordingly. It's really too bad though that unary operators with string names use the same single first character as parameter names because they get in the way here. In hindsight, named operators should probably have been prefixed with something other than a single dash to differentiate them from named parameters (something to consider if PowerShell ever comes out with a version with significant breaking changes plus conversion scripts).

### Some Non-Breaking Alternatives

1. Require users to opt-in on any command where they want this wrapping by ending the first line (and only the first line) with a sigil.

    This suggestion was proposed by @jaykul. We could add a token at the end of a command and from that point on, treat subsequent lines as part of the same command if they begin with named parameters, splatted collections, or the stop-parsing sigil (`--%`). For example, this could be a multi-line command that follows these rules:

    ```PowerShell
    New-ADUser @
        -Name 'Jack Robinson'
        -GivenName 'Jack'
        -Surname 'Robinson'
        -SamAccountName 'J.Robinson'
        -UserPrincipalName 'J.Robinson@enterprise.com'
        -Path 'OU=Users,DC=enterprise,DC=com'
        -AccountPassword (Read-Host -AsSecureString 'Input Password')
        -Enabled $true
    ```

    **Pros:**

    * the @ symbol is familiar since it is used for splatting
    * PSReadline could recognize when you're opting in to use this at the command prompt, only running the command if you hit enter twice (although this wouldn't be used nearly as often at the prompt -- it's really for scripts).

    **Cons:**

    * Users lose the benefit of not having to use any characters for line continuance.

    Here's another alternative sigil to do the same thing that is a bit more literal:

    ```PowerShell
    New-ADUser ...
        -Name 'Jack Robinson'
        -GivenName 'Jack'
        -Surname 'Robinson'
        -SamAccountName 'J.Robinson'
        -UserPrincipalName 'J.Robinson@enterprise.com'
        -Path 'OU=Users,DC=enterprise,DC=com'
        -AccountPassword (Read-Host -AsSecureString 'Input Password')
        -Enabled $true
    ```

    **Pros:**

    * ... is literal and familiar
    * the same PSReadline benefits listed for the previous example apply.

    **Cons:**

    * the same cons listed in the previous example apply.

1. Add support for optional features to PowerShell, and make this feature optional, rather than experimental.

    The experimental feature is designed to be a temporary holding place until features are fully developed. There are advantages to having optional features as well, which users can opt into if they want the benefit. Ideally such functionality would have support for enabling optional features in both scripts and modules, so that the features could be used by script/module authors without impacting the experience of individual consumers of those scripts/modules. This alone (support for optional features) warrants another RFC, but assuming it is there and assuming such features can be turned on for scripts (via `#requires`?) and/or modules (via the manifest), scripters who want this capability could have it automatically turned on for new scripts/modules without risk to existing scripts/modules because they wouldn't have that feature enabled, thus preventing a breaking change. Anyone who wants this for existing scripts/modules could manually enable it and take on the responsibility of making their code work appropriately.

    **Pros:**

    * allows the feature to be used without any breaking changes, and without extra continuance characters.

    **Cons:**

    * requires optional feature work in PowerShell (an RFC that I'd be happy to write).

1. Add argument enclosure support to PowerShell.

    One reader of this RFC who expressed their dislike for this functionality was pointing out that they don't like the fact that there is no terminator to the wrapped command parameters, so PowerShell needs to determine when the command ends based on lookahead. If preferable, new syntax could be added to allow for argument enclosures so that there is a defined terminator. For example, consider this syntax:

    ```PowerShell
    New-ADUser -{
        -Name 'Jack Robinson'
        -GivenName 'Jack'
        -Surname 'Robinson'
        -SamAccountName 'J.Robinson'
        -UserPrincipalName 'J.Robinson@enterprise.com'
        -Path 'OU=Users,DC=enterprise,DC=com'
        -AccountPassword (Read-Host -AsSecureString 'Input Password')
        -Enabled $true
    }
    ```

    What's interesting about this syntax is that these don't need to be just (named) parameter enclosures. They can be defined as argument enclosures, allowing scripters to put whatever they want on whichever lines they want, wrapping where it makes sense for them to do so, while using named parameters, positional parameters, or arguments (for external commands that don't have parameters).

    **Pros:**

    * allows the feature to be used without any breaking changes (script blocks do not support negate or subtraction operators, so this new enclosure could be added without risk)
    * allows the scripter to wrap how they see fit, while still getting Intellisense and tab completion. P
    * parameters and arguments are entered the same way they would be if they were all placed on one line (e.g. unlike splatting, parameter names are entered with dashes, no equals signs are required, etc.).

    **Cons:**

    * requires a little extra syntax to make it work properly.

