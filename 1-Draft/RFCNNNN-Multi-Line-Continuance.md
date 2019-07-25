---
RFC: RFCnnnn
Author: Kirk Munro
Status: Draft
SupercededBy: N/A
Version: 0.3
Area: Parser/Tokenizer
Comments Due: June 16, 2019
Plan to implement: Yes
---

# Multi-line continuance

Consider this example of a New-ADUser command invocation:

```PowerShell
New-ADUser -Name 'Jack Robinson' -GivenName 'Jack' -Surname 'Robinson' -SamAccountName 'J.Robinson' -UserPrincipalName 'J.Robinson@enterprise.com' -Path 'OU=Users,DC=enterprise,DC=com' -AccountPassword (Read-Host -AsSecureString 'Input Password') -Enabled $true
```

By itself it's not too much to handle, but in a script commands with many
parameters like this can be difficult to manage.

To wrap this command across multiple lines, users can either use backticks or
they can use splatting. The former is a syntactical nuisance which should
really only be used in situations when no other option is available. The latter
is helpful, but it puts the parameters before the command, making it more
difficult for less experienced users to learn/use, and all scripters lose the
benefits of tab completion and Intellisense for parameters when they use
splatting.

As a workaround, they can work out the parameters they want to use for the
command first, and then convert it into a splatted command, but that's onerous.
Even though Visual Studio Code has an extension that makes splatting easier, as
can be seen [here](https://sqldbawithabeard.com/2018/03/11/easily-splatting-powershell-with-vs-code/), once you've converted to splatting you still lose Intellisense
for future updates unless you work from the command first and then add to your
splatted collection, and that's just in Visual Studio Code. Other editors may
or may not support that functionality, and users working in a standalone
terminal won't have that available to them either.

Instead, why not allow users to wrap commands across multiple lines in a more
intuitive way without having to deal with backticks on every line or splatting,
like this:

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

Of course, they could invoke external commands and pass through arguments this
way as well:

```PowerShell
& "./plink.exe" @
    --% $Hostname -l $Username -pw $Password $Command

cacls @
    c:\docs\work
    /E /T /C /G
    "FinanceUsers":F

```

Further, by generalizing multi-line continuance with a `@` character, we're
allowing users to apply line continuance the way they want to, which opens the
door to more C#-like line wrapping when you're working with multiple members or
methods in .NET, one after another. For example, this would work:

```PowerShell
$string @
    .ToUpper()
    .Trim()
    .Length
```

In each of these examples, the parser starts parsing the command as a
multi-line command when it encounters the `@` token as the last token on the
line, and in this mode command parsing stops once one of the following is
found:

* end of file
* two newlines (as opposed to the normal one)
* command-terminating token (i.e. all other ways of ending commands work the
same as usual, and this does not affect other elements of the PowerShell
syntax)

The pros/cons to this new syntax are as follows:

**Pros:**

* allows the scripter to wrap commands how they see fit, while still getting
Intellisense and tab completion, without using backticks.
* aside from the `@` character to initiate multi-line continuance, the rest of
the command is entered the exact same way it would be if it was entered on a
single line.
* ad hoc could support this syntax as well (PSReadline could wait for a
double-enter when in multi-line command parsing mode)
* no breaking changes (a standalone `@` is currently an unrecognized token in
PowerShell no matter where it is used).
* users can use a blank line to terminate the command, or they can opt to use
any valid command-terminating token instead, so it has a proper closing
character.

**Cons:**

* using a blank line as a statement terminator will be hard for some to accept
(if you're one of those folks, read below to the alternative proposals and
considerations section).

## Motivation

    As a script/module author,
    I can wrap commands across multiple lines easily and intuitively without backticks or splatting
    so that my scripts remain easy to write and maintain while still giving me the benefits of Intellisense and tab completion.

## Specification

* expand the command parser to accept multi-line commands after an at symbol
(`@`) is encountered at the end of a line
* terminate multi-line commands when the parser encounters two newlines
(rather than one), or when the parser encounters any other command-terminating
token

Note:
* for commands that do not use the stop-parsing sigil in their arguments,
command-terminating tokens include a pipe symbol, a redirection operator, a
closing enclosure, a semi-colon, or a `&` background operator.
* for commands that do use the stop-parsing sigil in their arguments,
command-terminating tokens include a pipe symbol or a redirection operator.

## Alternate Proposals and Considerations

### A different sigil

The original draft of this RFC included different options for the sigil that
could be used to enter multi-line parameter/argument parsing mode, and others
were presented in the discussion however none of the other sigils that were
presented could be used without breaking changes. When considering an alternate
sigil, it must be something that can be identified as a unique token without
breaking commands that accept multiple strings as positional parameters, such
as Write-Host (which can write many sigils to the console) or commands external
to PowerShell.

### Enclosures instead of a sigil

Instead of only requiring a single leading sigil, some users prefer the notion
of enclosures such that the multi-line command parsing mode would have a very
clear and well defined start and end. To meet that need, we could follow the
here-string syntax in PowerShell as an example, offering syntax like the
following:

```PowerShell
. {"./plink.exe" @`
    --% 
    $Hostname
    -l $Username
    -pw $Password
    $Command
`@}
```

All this would do is prevent newline tokens within the enclosures from being
treated as statement terminators in the current statement.

The closing closure could also be a recognized statement terminator even when
used after the stop parsing sigil, allowing those commands to be wrapped across
multiple lines as well.

Like here-string enclosures, we could require that the opening closure be the
last token on a line. Unlike here-string enclosures, however, it would be
preferable if the closing closure did not have to be at the start of a line,
since there is no need for it to be. It would simply have to be the first token
on line to close the statement, allowing for indentation within scripts.

This alternative also allows for blank lines to be used within the enclosures,
such that Scenario 3 in @dragonwolf83's comment could be supported and written
like this:

```PowerShell
New-ADUser @`
    -Name 'Jack Robinson'
    -GivenName 'Jack'
    -Surname 'Robinson'
    -SamAccountName 'J.Robinson'

    -UserPrincipalName (
        'J.Robinson@enterprise.com'
    )

   # Get the list of regions for where a user would reside in to put the user into the correct region
    -Path (
        $region = Get-Region -Name 'Jack Robinson'
        "OU=Users,OU=$region,DC=enterprise,DC=com"
    )

    -AccountPassword (
        Read-Host -AsSecureString 'Input Password'
    )

    -Enabled $true
`@

Get-ChildItem @`
    $rootFolder
    -File
    -Filter '*.ps*1'
`@
```

The best part is that it doesn't appear this syntax would introduce a breaking
change at all.

If people feel the backtick is still not visible enough here (and therefore not
desirable for this purpose), we should keep the `@` portion of the enclosures
so that we can avoid breaking changes, and simply replace the backtick with
something else. For example, we could do this instead:

```PowerShell
Get-ChildItem @-
    $rootFolder
    -File
    -Filter '*.ps*1'
-@
```

Backticks offer the advantage of continuing what they represent in PowerShell
already (line continuation), and they are syntactically very similar to here-
strings when used with the `@` symbol. That last point could be seen as a
disadvantage as well, because they may be visually harder to distinguish from
a single-quoted here-string.

The `@-|-@` alternative doesn't pick up on the backtick for line continuation,
but it is visually unique and easy to see in a script. The `-` character could
be seen as representing the line that makes up the statement as well.

### Inline splatting

There has also been some discussion about the idea of inline splatting, using a
format like `-@{...}` or `-@(...)`. Inline splatting has also been discussed
separately on [RFC0002: Generalized Splatting](https://github.com/PowerShell/PowerShell-RFC/blob/master/2-Draft-Accepted/RFC0002-Generalized-Splatting.md), but using the syntax `@@{...}` or
`@@(...)`.

Here is an example showing what that might look like:

```PowerShell
Get-ChildItem -@{
    LiteralPath = $rootFolder
    File = $true
    Filter = '*.ps*1'
}
```

Using inline splatting to be able to span a single command across multiple
lines like this has several limitations, including:

1. You cannot transition to/from the inline splatted syntax without a bunch of
manual tweaks to the command (either converting parameter syntax into hashtable
or array syntax or vice versa).
1. You're forced to choose between named parameters or positional
parameters/arguments for each splatted collection. i.e. You can splat in a
hashtable of named parameter/value pairs or an array of positional values, but
you can't mix the two (the example shown just above is also used earlier in
this RFC with positional parameters and switch parameters used without values,
matching the way it is often used as a single-line command).
1. There's no way to include unparsed arguments after the stop-parsing sigil in
splatting. You can add it afterwards, but not include it within.
1. Splatting requires a different syntax than typical parameter/argument input,
which is more to learn. In contrast, the proposal above only requires learning
about the `@` sigil (borrowed from splatting, but without specifying hashtables
or arrays -- just allow all content until a newline), reducing the learning
curve and allowing users to use parameters the same way in either case.
1. Inline splatting attempts to resolve the issue for commands with arguments,
but it does nothing for other scenarios where you want specific line wrapping
other than the defaults that PowerShell implicitly supports.

Further, unlike using a leading sigil such as `@`, which would work with
Intellisense and tab expansion as they are coded now, inline splatting would
require special work to make Intellisense and tab expansion work with it. That
is not a reason not to do it, but it is more code to write and maintain.

### Breaking changes

No known breaking changes in the original proposal, nor the alternative version
that uses enclosures.

All previous options from the original RFC and the discussion about it that
would have introduced breaking changes have been removed in favor of a syntax
that just works to the specification without any breaking changes, regardless
of how you use it.
