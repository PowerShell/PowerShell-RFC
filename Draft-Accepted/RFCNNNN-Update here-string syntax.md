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

The here-string syntax should be updated to support the following 3 requests:
1. Allow single line here-strings like `Write-Host @'It's nice being able to use contractions'@` so the user doesn't have to worry about escaping quote characters.  
2. Allow a variable number of delimiter characters like `Write-Host @@'@'Nested here string'@'@@` so users can create nested here-strings.  
3. Allow indentation of the end delimiter of here-strings so the indentation level stays consistent like in this example:

```
function MyFunction
{
    Write-Host @@'
    This here-string spans multiple lines
    Isn't it cool?
    '@@
}
```

See also this related PR: https://github.com/PowerShell/PowerShell/pull/17483 and these related issues: https://github.com/PowerShell/PowerShell/issues/2337 https://github.com/PowerShell/PowerShell/issues/13204


## Motivation

    As a user,
    I can write/paste strings with quote characters without having to escape them,
    which makes the shell easier to use.

    As a developer,
    I can generate strings meant for PowerShell consumption (tab completion, code snippets, etc.) without having to escape special characters.

    As a developer,
    I can move code with here-strings into a function and indent it without manually removing the indentation for the string terminator.

## User Experience

Example of user experience with example code/script.
Include example of input and output.

```powershell
@'It's a beautiful day'@
```

```output
It's a beautiful day
```

```powershell
@'It's a
beautiful day
'@
```

```output
Parser error
```

```powershell
@@'
    ^Line starts at '^'
        This line has 4 leading whitespaces
    '@@
```

```output
^Line starts at '^'
    This line has 4 leading whitespaces
```

## Specification
Here-strings start with a variable amount of `@` characters, followed by either a double or single quote character to signify if it's expandable or not.  
Single line here-strings start immediately after the chosen quote character.  
Multi line here-strings start on the line that follows the starting sequence. Whitespace characters on the same line as the starting sequence are ignored, non-whitespace characters produce a syntax error.

Single line here-strings end when a matching quote character is found if it's also followed by the same amount of `@` characters that the string started with.  
Multi line here-strings follow the same ending rules as single line here-strings, except they also require the line with the string terminator characters to be empty, or only consist of whitespace.  
For backwards compatibility, multi line here-strings that start with a single `@` character cannot have whitespace before the string terminator characters.

Multi line here-strings that start with 2 or more `@` characters can be indented, when this is done each line will be trimmed for leading whitespaces. The amount of whitespaces removed depends on the lowest indentation level used in the here-string. For example, if line 1 is indented by 5 spaces and line 2 is indented by 4 spaces then the parsed string will include one whitespace on line 1 and no whitespaces on line 2.

## Alternate Proposals and Considerations
Instead of using multiple `@` it could use multiple quote characters instead. There are 3 reasons why I think `@` is the better choice:
1. It's easier to tell at a glance how many characters there are with @@@@@' VS @''''' especially when you also consider double quotes and all the different single/double quote variants that exist.
2. If you decide to swap between a single quote and double quote here-string it's easier to just change 1 character in each end instead of multiple characters in each end.
3. In the case of single line strings it allows the string to start with any character. It quotes were used you wouldn't be able to start the string with a quote character because it would be counted as part of the "header" section of the here-string.

The special backwards compatibility behavior when using a single `@` could be removed to make it easier for new users to learn the syntax and to simplify the code.  
Unfortunately it was declined the last time a PR was made: https://github.com/PowerShell/PowerShell/issues/2337#issuecomment-452380310