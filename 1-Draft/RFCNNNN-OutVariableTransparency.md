---
RFC:
Author: Robert Holt
Status: Draft
SupercededBy: N/A
Version:
Area: Language/Common Variables
Comments Due: 2018-05-01
Plan to implement: Yes
---

# OutVariable Transparency

PowerShell supports OutVariables, which allow users to send the output of a command to the variable rather than to the pipeline.

Since this feature was included in PowerShell v1, the implementation detail of the ArrayList used to collect the output into the OutVariable has been exposed to the user, so that rather than getting the `object` or `object[]` that they would get with pipeline output, an `ArrayList` is always received.

This RFC floats the idea that OutVariable usage should be transparent to the user, so output is agnostic between OutVariable and the pipeline stream. As a corner case in PowerShell's implementation, OutVariables currently represent extra caveats for new users.

## Motivation

    As a PowerShell user, I can use OutVariable just like pipeline output, so that my language experience is simpler and more consistent.

## Specification

This table summarises the changes proposed in this RFC:

| Input                                            | Non-OutVar Result Type       | Old Result Type              | New Result Type              |
| :----------------------------------------------: | :--------------------------: | :--------------------------: | :--------------------------: |
| `'Hello'`                                        | System.String                | System.Collections.ArrayList | System.String                |
| `@(1, 2)`                                        | object[]                     | System.Collections.ArrayList | object[]                     |
| `[System.Collections.ArrayList]::new(@(1,2))`    | object[]                     | System.Collections.ArrayList | object[]                     |
| `@(,[System.Collections.ArrayList]::new(@(1,2))` | System.Collections.ArrayList | System.Collections.ArrayList | System.Collections.ArrayList |
| `$null`                                          | ⊥                            | System.Collections.ArrayList | ⊥                            |

## Alternate Proposals and Considerations

This RFC would consitute a breaking change, and would break any
scripts depending on the output of OutVariable being a collection.

The alternative here is to leave OutVariables as they are, as an element of the language that users are used to, depend on and like.