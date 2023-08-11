---
RFC: 
Author: MartinGC94
Status: Draft
SupercededBy: 
Version: 1.0
Area: Language
Comments Due: 23-02-2023
Plan to implement: Yes
---

# Update here-string syntax

The here-string syntax should be updated to support the following 2 requests:
1. Allow a variable number of delimiter characters like `Write-Host @''@'Nested here string'@'@'` so users can create nested here-strings.  
2. Allow indentation of the end delimiter of here-strings so the indentation level stays consistent like in this example:

```
function MyFunction
{
    Write-Host @@'
    This here-string spans multiple lines
    Isn't it cool?
    '@@
}
```

See also this related PR: https://github.com/PowerShell/PowerShell/pull/17483 and these related issues: https://github.com/PowerShell/PowerShell/issues/2337


## Motivation

    As a developer,
    I can move code with here-strings into a function and indent it without manually removing the indentation for the string terminator.

## User Experience

Example of user experience with example code/script.
Include example of input and output.

```powershell
@'It's a
beautiful day
'@
```

```output
Parser error due to the characters after the header.
```

```powershell
@@'
    ^This line starts at '^'
        This line has 4 leading whitespaces
    '@@
```

```output
^This line starts at '^'
    This line has 4 leading whitespaces
```

## Specification
Here-strings start with a "header" that consists of an `@` symbol, followed by a variable amount of single/double quote characters.  
White space characters are allowed after the header, but they are not included in the actual content. Any non-whitespace characters after the header will cause a parser error.  
Double quoted strings are expandable, while single quoted strings are literal.  
Expansion/escape rules are the same as the standard double/single quoted strings, except quote characters don't need to be escaped.  


Here-strings end with a "footer" line that consists of a matching amount (and kind) of single/double quote characters as the header, followed by an `@` symbol.  
If there is more than one single/double quote character in the header, the footer can be indented with whitespaces (this requirement exists to preserve backwards compatibility with the old here-string syntax which did not allow any characters before the footer).  
Any non-whitespace characters before the footer will cause a parser error.

The content of a here-string consists of everything between the header and footer lines.  
If the footer is indented, then the amount of whitespaces it's indented with determines how many whitespaces to trim from the beginning of each line in the here-string.  
Lines with less whitespace indentation than the footer will cause a parser error unless they are empty or only consists of whitespace characters.  
Lines that are empty or only consists of whitespaces that have less than or equal to the amount of whitespaces as the footer are trimmed so they only consist of the line break character(s).  
If the whitespace indentation of a line is greater than the footer then the additional whitespaces will be included as is in the content.

## Alternate Proposals and Considerations
Instead of using multiple quote characters it could use multiple `@` characters instead. `@` has the following 3 advantages:
1. It's easier to tell at a glance how many characters there are with @@@@@' VS @''''' especially when you also consider double quotes and all the different single/double quote variants that exist.
2. If you decide to swap between a single quote and double quote here-string it's easier to just change 1 character in each end instead of multiple characters in each end.
3. If single line here-strings are ever added it will allow the string to start with any character. Quotes will not allow you to start the string with a quote character because it would be counted as part of the "header" section of the here-string.

Despite these advantages, the use of multiple quote characters were choosen due to familiarity with the raw string literals in C#.

The special backwards compatibility behavior when using a single `@` could be removed to make it easier for new users to learn the syntax and to simplify the code.  
However it was declined the last time a PR was made: https://github.com/PowerShell/PowerShell/issues/2337#issuecomment-452380310
