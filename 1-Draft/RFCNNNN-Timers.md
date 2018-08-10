---
RFC: RFCNNNN
Author: Rob Pleau
Status: Draft
Version: 1.0
Area: Microsoft.PowerShell.Utility / Commands
Comments Due:
Plan to implement: Yes
---

# Timer Functionality in PowerShell

Many users of PowerShell have a need to create a timer.  Typically users doing this are doing so by manually instantiating an instance of `System.Diagnostics.Stopwatch`.

## Motivation

As a *PowerShell User*
I want to *Create timers for automation jobs*
So that *I can have logic stop a job that runs to long*

PowerShell should have built-in functionality to start and stop timers.  Users of the language who are automating around long running jobs are already implementing this using `System.Diagnostics.Stopwatch`, this would build into the language a more direct way to create timers.

## Specification

The following functions should be added.  Multiple instances could be created, so that a session can have more than one timer running if needed.  These could be referenced by an `Id` property, similar to Jobs or PSSessions

| Function | Usage | OutPut |
| --- | --- | --- |
| New-Timer| Instantiate a new timer without starting it. (`IsRunning` would be `false`) | `[System.Diagnostics.Stopwatch]` |
| Start-Timer | Instantiate a new timer and automatically start it. (`IsRunning` would be `true`) | `[System.Diagnostics.Stopwatch]` |
| Stop-Timer | Pass an `-Id` to stop a timer. (`IsRunning` would be `false`) | `[System.Diagnostics.Stopwatch]` |
| Remove-Timer | Pass an `-Id` to remove a timer completely. | `void` |
| Reset-Timer | Pass an `-Id` to reset a timer to **0**. (`IsRunning` would be `false`) | `[System.Diagnostics.Stopwatch]` |
| Restart-Timer | Pass an `-Id` to reset a timer to **0**. (`IsRunning` would be `true`) | `[System.Diagnostics.Stopwatch]`  |
| Get-Timer | Pass an `-Id` to get data on a timer, omit `-Id` to get all current timers. | `[System.Diagnostics.Stopwatch[]]` |
| Get-TimerElapsed |  Pass an `-Id` to get a single timers elasped time. | `[System.TimeSpan]` |

For inline/pipeline usage, an optional script block parameter could be built in which would start a timer and hold the session until...
A. The timer expires
B. The script block completes

## Alternate Proposals and Considerations

Alternatively, instead of "Timer" for the noun, "Stopwatch" might be more direct.