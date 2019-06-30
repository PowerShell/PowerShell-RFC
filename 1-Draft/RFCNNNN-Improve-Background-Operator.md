---
RFC: RFCnnnn
Author: Kirk Munro
Status: Draft
SupercededBy: 
Version: 0.1
Area: PSEngine,Host,Jobs
Comments Due: July 20, 2019
Plan to implement: Yes
---

# Improved the `&` Background Operator

This RFC was born from some side comments made on [PowerShell Issue #9873](https://github.com/PowerShell/PowerShell/issues/9873#issuecomment-501289658).

The `&` background operator has great potential, and it could be encouraging
for users who come to PowerShell from bash since it is already familiar to
them; however, the initial implementation of this operator was simply to
invoke `Start-Job` in a manner familiar with bash users, and the current
implementation doesn't work like the same operator does in bash. It also left
several opportunities for a more feature-rich PowerShell-based version of that
operator on the table. The end result is a feature that feels more like a
checklist item that is partially checked off than a feature that adds
significant, tangible value to the PowerShell community. In a nutshell, we can
and should do better.

Note that there is a [related RFC about adding a `~&` ThreadJob operator](https://github.com/PowerShell/PowerShell-RFC/pull/205).

## Motivation

As a user,<br/>
I can reference the most recent job(s) launched with `Start-Job` or the `&` background operator in a `$!` variable<br/>
So that I can write my handling of those jobs in a consistent way without the job object or id, using the same variable that is used in bash for the same purpose.

As a user,<br/>
I can launch jobs with a new `&!` background operator that will show job progress in an overlay<br/>
So that watch the concurrent progress of multiple background jobs in my host without blocking my console.

As a user,<br/>
I can use the `&` operator (and the proposed `&!` operator) after the stop parsing sigil (`--%`)<br/>
So that I can launch legacy commands in the background more easily, without having to use `Start-Job`.

Also, for completeness/functional parity with cmdlets:

As a user,<br/>
I can see the output of jobs launched with `Start-Job` or `Start-ThreadJob` in my current host by using a new `-ShowData` switch<br/>
So that I can launch multiple background jobs and watch their concurrent progress in my host without blocking my console.

As a user,<br/>
I can show or hide the output of running jobs by using new `Show-JobData` and `Hide-JobData` cmdlets<br/>
So that I can control the display and monitoring of running jobs without having to muck around with `Receive-Job -Keep`.

As a PowerShell contributor,<br/>
I can see the process ID associated with any background job by looking at the job members<br/>
So that I can more easily attach the Visual Studio debugger to that job when I need to.

Last, one describing the existing motivation for the `&` background operator
(this is already in place today):

As a user,<br/>
I can launch jobs silently with the `&` background operator<br/>
So that I can easily run pipelines as jobs in my automation solutions without showing output.

## User Experience

### Launching a background job with the output visible in the host

```powershell
# This command:
Get-Process &!
# Would return the same output as this command:
Start-Job {Get-Process} -ShowData
```

```output
Id     Name            PSJobTypeName   State         HasMoreData     Location             Command
--     ----            -------------   -----         -----------     --------             -------
1      Job1            BackgroundJob   Running       True            localhost            Microsoft.PowerShell.Man…
```

In addition to the output above, which is the job object created by the command
that was invoked, PowerShell would show the output from the job (and any other
job) launched this way in an overlay much like the output from Write-Progress,
with the job data from any stream appearing as it is output in the overlay,
prefixed with "[JobName]:" (e.g. [Job1]: ). This overlay would be slightly
larger than Write-Progress so that users could see more output from the job in
the overlay, and when all jobs sharing output complete the overlay would
disappear after a delay (5 seconds). Progress messages from the jobs would
appear above the output.

### Launching a background job quietly and referencing the job with the `$!` variable

```powershell
# These commands:
$null = Get-Process &
$!
# Would return the same output as these commands:
$null = Start-Job {Get-Process}
$!
```

```output
Id     Name            PSJobTypeName   State         HasMoreData     Location             Command
--     ----            -------------   -----         -----------     --------             -------
1      Job1            BackgroundJob   Running       True            localhost            Microsoft.PowerShell.Man…
```

The `&` operator permits launching a job using an operator without any of the
messages from the various streams appearing by default in the console (this is
how this operator works today).

The `$!` variable in the subsequent command returns the job that was created
with the command before it.

### Fanning out to multiple background jobs and referencing the jobs with the `$!` variable

```powershell
$null = cmd1 & cmd2 & cmd3 &
$!
```

```output
Id     Name            PSJobTypeName   State         HasMoreData     Location             Command
--     ----            -------------   -----         -----------     --------             -------
1      Job1            BackgroundJob   Running       True            localhost            Microsoft.PowerShell.Man...
2      Job1            BackgroundJob   Running       True            localhost            Microsoft.PowerShell.Man...
3      Job1            BackgroundJob   Running       True            localhost            Microsoft.PowerShell.Man...
```

The `$!` variable in those two commands returns the jobs that were created with
the command before it.

### Launching a legacy command with the stop-parsing sigil and either the `&` or `&!` background operator

This corrects a limitation with the background operator(s), which currently
treat `&` (and `&!`) as an argument instead when used after the stop-parsing
sigil.

```powershell
./plink.exe" --% $Hostname -l $Username -pw $Password $Command &
```

```output
Id     Name            PSJobTypeName   State         HasMoreData     Location             Command
--     ----            -------------   -----         -----------     --------             -------
1      Job1            BackgroundJob   Running       True            localhost            Microsoft.PowerShell.Man...
```

### Showing or hiding the data from one or more jobs

This allows job data to be shown or hidden on demand for any jobs.

```powershell
# Show the data from job ids 1-5
Show-JobData -Id 1..5
# Hide the data from job name Job2
Hide-JobData -Name Job2
# Show the data for the last job (or jobs) we launched
Show-JobData -Job $!
```

```output
# As described in the comments above, with the data showing in an overlay
```

### Viewing the process ID for a job

This shows how you can access the read-only process ID for a job.

```powershell
$!.ProcessId
```

```output
# The process id for the background job(s)
```

## Specification

1. For jobs invoked with `&!` or `Start-Job -ShowData`, or for jobs referenced
in `Show-JobData`, hook up event handlers on the data stream collections for
the job objects to capture the data from the jobs as it is added in real time,
and show that data in the current terminal with the job name or ID in front of
that data. Mirror how `ProgressInformation` is rendered when it comes to
showing job data this way.
1. Add a `&!` operator that starts a job with the data display turned on.
1. Add a boolean property to `BackgroundJob` objects called `ShowData` that
identifies if the job is currently configured to show data or not.
1. Add an integer property to `BackgroundJob` objects called `ProcessId` that
identifies the ID of the process where the job is running.
1. Add a `-ShowData` parameter to `Start-Job` that sets `ShowData` to true
and hooks up event handlers as described in the first item in this list.
1. Add a `$!` variable to store the most recently run job (or jobs if multiple
jobs are launched at the same time, e.g. `cmd1 & cmd2 & cmd3 &`).

## Alternate Proposals and Considerations

### Add the process ID to the default output for a background job

It may seem like a good idea to show the process ID for a background job in the
default tabular output. While this would be convenient, we also have thread
jobs that don't have a process ID, so it is probably better to leave it out for
consistency across all jobs. It is always accessible via the `ProcessID` member
of background jobs regardless.

### Flip the silent/noisy approach with the operators

This approach is proposed because we've already shipped the `&` background
operator, with jobs running silent by default. In bash, if you run a command in
the background using the `&` control operator, the output is displayed in the
current terminal. This discrepancy may be confusing to users. If we feel that
the PowerShell `&` background operator should be more consistent with the bash
equivalent, we could make the `&` background operator show output by default
and have the new `&!` background operator run silent. This wouldn't be a
breaking change, but it would change the behavior for any scripts that are
already using the `&` background operator today.
