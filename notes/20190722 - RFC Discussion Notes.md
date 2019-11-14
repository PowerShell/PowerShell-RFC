# RFC Meeting - July 22, 2019

## Attendees

* Kenneth Hansen
* Steve Lee
* Jim Truher
* Dongbo Wang
* Joey Aiello

## Notes

* Reviewing line continuation all up, including the merged [PS/PS PR #8938](https://github.com/PowerShell/PowerShell/pull/8938/files)
  and the open RFC [#179](https://github.com/PowerShell/PowerShell-RFC/pull/179/files)
  * Jim to write response on #179 about whitespace concerns vs. backticks
  * General agreement that #8938 should not have been merged without RFC, but we're okay with the approach today,
    and will leave it
    * As a lesson, we should be more vigilant on PRs, especially as the affect the language/parser
    * This breaks right-click paste to terminal (Ctrl+V still works because PSReadline)
  * Line continuation likely needs to be addressed all up in a single RFC (regarding all other operators)
    * There's a concern that the risk is high, requiring a significant amount of time to make sure we get right,
      and the benefit is only for a small number of users
    * We've discussed this at length, and the current agreement is that this is not a widespread or urgent problem:
      the risk is high, and we shouldn't continue spending time on it
* Desire to write and review RFC process blog at Wed's meeting