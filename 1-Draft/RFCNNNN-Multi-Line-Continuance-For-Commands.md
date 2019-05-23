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

# Multi-line continuance for commands

Consider this example of a New-ADUser command invocation:

```PowerShell
New-ADUser -Name 'Jack Robinson' -GivenName 'Jack' -Surname 'Robinson' -SamAccountName 'J.Robinson' -UserPrincipalName 'J.Robinson@enterprise.com' -Path 'OU=Users,DC=enterprise,DC=com' -AccountPassword (Read-Host -AsSecureString 'Input Password') -Enabled $true
```

By itself it's not too much to handle, but in a script commands with many parameters like this can be difficult to manage. To wrap this command across multiple lines, users can either use backticks or they can use splatting. The former is a syntactical nuisance which should really only be used in situations when no other option is available. The latter is helpful, but it puts the parameters before the command, making it more difficult for less experienced users to learn/use, and all scripters lose the benefits of tab completion and Intellisense for parameters when they use splatting. As a workaround, they can work out the parameters they want to use for the command first, and then convert it into a splatted command, but that's onerous. Even though Visual Studio Code has an extension that makes splatting easier, as can be seen [here](https://sqldbawithabeard.com/2018/03/11/easily-splatting-powershell-with-vs-code/), once you've converted to splatting you still lose Intellisense for future updates unless you work from the command first and then add to your splatted collection, and that's just in Visual Studio Code. Other editors may or may not support that functionality, and users working in a standalone terminal won't have that available to them either.

Instead, why not allow users to wrap commands across multiple lines in a more intuitive way without having to deal with backticks on every line or splatting, like this:

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

Get-ChildItem @
    $rootFolder
    -File
    -Filter '*.ps*1'

```

Of course, they could invoke external commands and pass through arguments this way as well:

```PowerShell
& "./plink.exe" @
    --% $Hostname -l $Username -pw $Password $Command

cacls @
    c:\docs\work
    /E /T /C /G
    "FinanceUsers":F

```

In each of these examples, the parser starts parsing the command as a multi-line command when it encounters the `@` token as the last token on the line, and in this mode command parsing stops once one of the following is found:

* end of file
* two newlines (as opposed to the normal one)
* command-terminating token (i.e. all other ways of ending commands work the same as usual, and this does not affect other elements of the PowerShell syntax)

The pros/cons to this new syntax are as follows:

**Pros:**

* allows the scripter to wrap commands how they see fit, while still getting Intellisense and tab completion, without using backticks.
* parameters and arguments are entered the exact same way they would be if they were all placed on one line.
* ad hoc could support this syntax as well (PSReadline could wait for a double-enter when in multi-line command parsing mode)
* no breaking changes (a standalone @ is currently an unrecognized token in PowerShell no matter where it is used).
* users can use a blank line to terminate the command, or they can opt to use a semi-colon or some other valid command-terminating token instead.

**Cons:**

* no known cons at this time

## Motivation

    As a script/module author,
    I can wrap commands across multiple lines easily and intuitively without backticks or splatting
    so that my scripts remain easy to write and maintain while still giving me the benefits of Intellisense and tab completion.

## Specification

* expand the command parser to accept multi-line command parameters/arguments after an at symbol (`@`) is encountered at the end of a line
* terminate multi-line commands when the parser encounters either a blank line or a command-terminating token (a command-terminating token will be a pipe symbol, redirection operator, closing enclosure, or semi-colon when the command does not have the stop-parsing sigil in its arguments, or simply pipe symbol or redirection operator the command does have the stop-parsing sigil in its arguments)

## Alternate Proposals and Considerations

### A different sigil

The original draft of this RFC included different options for the sigil that could be used to enter multi-line parameter/argument parsing mode, and others were presented in the discussion however none of the other sigils that were presented could be used without breaking changes. When considering an alternate sigil, it must be something that can be identified as a unique token without breaking commands that accept multiple strings as positional parameters, such as Write-Host (which can write many sigils to the console) or commands external to PowerShell.

### Enclosures instead of a sigil

The original draft also included some proposals for enclosures instead of a leading sigil. While it may seem up front that enclosures make good sense, some thought needs to be given to the stop-parsing sigil, which causes PowerShell to treat closing enclosures as arguments for a command.

For example, consider this syntax:

```PowerShell
. {"./plink.exe" @
    --% 
    $Hostname
    -l $Username
    -pw $Password
    $Command}
```

That command will not parse because there is no closing brace for the script block. What appears to be a closing brace is placed after the stop-parsing sigil, and therefore treated as an argument to the plink command. To correct this, the closing brace must be placed on a separate line, but in a multi-line command you cannot do that (because the command is multi-line, so where would the parser terminate after a stop-parsing sigil) and therefore, unless the sigil used to identify the end of the multi-line parameter/arguments was one that could be respected by the stop-parsing sigil, and safely be introduced without risk to breaking changes, enclosures simply cannot be used.

### Inline splatting

There has also been some discussion about the idea of inline splatting, using a format like `-@{...}` or `-@(...)`. Inline splatting has also been discussed separately on [RFC0002: Generalized Splatting](https://github.com/PowerShell/PowerShell-RFC/blob/master/2-Draft-Accepted/RFC0002-Generalized-Splatting.md), but using the syntax `@@{...}` or `@@(...)`. Here is an example showing what that might look like:

```PowerShell
Get-ChildItem -@{
    LiteralPath = $rootFolder
    File = $true
    Filter = '*.ps*1'
}
```

Using inline splatting to be able to span a single command across multiple lines like this has several limitations, including:

1. You cannot transition to/from the inline splatted syntax without a bunch of manual tweaks to the command (either converting parameter syntax into hashtable or array syntax or vice versa).
1. You're forced to choose between named parameters or positional parameters/arguments for each splatted collection. i.e. You can splat in a hashtable of named parameter/value pairs or an array of positional values, but you can't mix the two (the example shown just above is also used earlier in this RFC with positional parameters and switch parameters used without values, matching the way it is often used as a single-line command).
1. There's no way to include unparsed arguments after the stop-parsing sigil in splatting. You can add it afterwards, but not include it within.
1. Splatting requires a different syntax than typical parameter/argument input, which is more to learn. In contrast, the proposal above only requires learning about the `@` sigil (borrowed from splatting, but without specifying hashtables or arrays -- just allow all content until a newline), reducing the learning curve and allowing users to use parameters the same way in either case.

Further, unlike using a leading sigil such as `@`, which would work with Intellisense and tab expansion as they are coded now, inline splatting would require special work to make Intellisense and tab expansion work with it. That's not a reason not to do it, but it is more code to write/maintain.

### Breaking changes

All previous options from the original RFC and the discussion about it that would have introduced breaking changes have been removed in favor of a syntax that just works to the specification without any breaking changes, regardless of how you use it.
