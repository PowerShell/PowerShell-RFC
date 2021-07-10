---
RFC: RFC0030
Author: Steve Lee
Status: Final
SupercededBy: N/A
Version: 1.0
Area: Engine-CommandCompleter
Comments Due: 12/4/2018
Plan to implement: Yes
---

# Abbreviation expansion based tab completion

PowerShell rules for cmdlet naming is to be descriptive.
Cloud services cmdlets tend to be granular and vast.

Azure PowerShell, for example, has a cmdlet called `New-AzSqlServerDisasterRecoveryConfiguration`.
If you start with `New-Az` and hit tab, you'll get 334 matches.
Then you continue typing `New-AzSql` and hit tab, you still get 19 matches.

These cmdlets typically do not have aliases since the aliases would be very long.

## Motivation

    As a PowerShell user,
    I can tab complete long cmdlet names easily and predictably,
    so that I can be more efficient using PowerShell interactively.

## Specification

Since PowerShell rule for cmdlet naming relies on Pascal Casing,
you can consistently predict a long cmdlet name used often by simply specifying
the uppercase letters:

```none
PS /Users/steve> n-assdrc<tab>

becomes

PS /Users/steve> New-AzSqlServerDisasterRecoveryConfiguration
```

The dash is still required to avoid short abbreviations from conflicting with aliases.
The abbreviation is not an alias as it does not work if specified on the command
line or script:

```none
PS /Users/steve> n-assdrc
n-assdrc : The term 'n-assdrc' is not recognized as the name of a cmdlet, function, script file, or operable program.
Check the spelling of the name, or if a path was included, verify that the path is correct and try again.
At line:1 char:1
+ n-assdrc
+ ~~~~~~~~
+ CategoryInfo          : ObjectNotFound: (n-assdrc:String) [], CommandNotFoundException
+ FullyQualifiedErrorId : CommandNotFoundException
```

This feature is intended only to be used interactively with tab completion.

Even shorter commands can benefit:

```none
PS /Users/steve> ct-j<tab>

becomes

PS /Users/steve> ConvertTo-Json
```

## Alternate Proposals and Considerations

PowerShell convention is for 2 letter abbreviations to entirely be in uppercase.
So you have cmdlets like `Set-AzVM`.
The abbreviated form is `s-avm`, not `s-av` which might be expected.

This feature is not dynamic aliases as readability of scripts would be significantly
impacted if `n-assdrc` was allowed in scripts.

Partial support where just verb or noun is abbreviated is not supported.
This could be added in the future based on customer feedback.

Fuzzy matching is not in scope of this RFC.
Fuzzy matching for [suggestions on CommandNotFoundException](https://github.com/PowerShell/PowerShell/pull/8458)
was implemented and could be used as a base for supporting tab completion.

If more variants/algorithms are supported, we may need a way to allow the user
to pick which ones they want to use.

---------------
## PowerShell Committee Decision

### Voting Results

Joey Aiello: Accept

Bruce Payette: Accept

Steve Lee: Accept

Hemant Mahawar: Accept

Jim Truher: Accept

### Majority Decision

Committee agrees that the RFC is complete for the targeted scenario and additional enhancement requires are out of scope and should be separate RFCs.

### Minority Decision

N/A
