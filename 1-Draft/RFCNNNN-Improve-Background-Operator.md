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
I can launch jobs with a new `&!` background operator that will show job stream data in my console<br/>
So that watch the concurrent progress of multiple background jobs in my host when needed without blocking my console.

Also, for completeness/functional parity with cmdlets:

As a user,<br/>
I can see the output of jobs launched with `Start-Job` or `Start-ThreadJob` in my current host by using a new `-Watch` switch<br/>
So that I can launch multiple background jobs and watch their concurrent progress in my host without blocking my console.

As a user,<br/>
I can show or hide the output of running jobs by using a new `Watch-Job` cmdlet (with `Watch-Job -Off` to stop watching)<br/>
So that I can control the display and monitoring of running jobs without having to muck around with `Receive-Job -Keep`.

As a PowerShell contributor,<br/>
I can see the process ID associated with any job by looking at the job members<br/>
So that I can more easily attach the Visual Studio debugger to that job when I need to.

Last, one describing the existing motivation for the `&` background operator
(this is already in place today):

As a user,<br/>
I can launch jobs silently with the `&` background operator<br/>
So that I can easily run pipelines as jobs in my automation solutions without producing output.

## User Experience

### Launching a background job with the output visible in the host

```powershell
# This command:
Get-Process &!
# Would return the same output as this command:
Start-Job {Get-Process} -Watch
```

```output
Id     Name            PSJobTypeName   State         HasMoreData     Location             Command
--     ----            -------------   -----         -----------     --------             -------
1      Job1            BackgroundJob   Running       True            localhost            Microsoft.PowerShell.Man…
```

In addition to the output above, which is the job object created by the command
that was invoked, PowerShell will monitor any jobs launched this way and show
their stream output in the current console, with the job output from any stream
prefixed with "[JobName]:" (e.g. [Job1]: ). Progress messages from jobs would
appear in textual format in the console.

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

### Monitoring the output from one or more jobs

This allows job output monitoring to be turned on or off for any job.

```powershell
# Monitor the output from job ids 1-5
Watch-Job -Id 1..5
# Turn off the monitor for the job named Job2
Watch-Job -Off -Name Job2
# Monitor the output for the last job (or jobs) we launched
Watch-Job -Job $!
```

```output
# As described in the comments above, with the output showing in the console.
```

### Viewing the process ID for a job

This shows how you can access the read-only process ID for a job.

```powershell
$!.ProcessId
```

```output
# The process id for the background job(s)
```

In the case of `ThreadJob` instances, `ProcessId` would contain the ID of the
current process.

## Specification

1. Define a `Watch-Job` cmdlet with the following syntax:

    ```powershell
    Watch-Job [-Name] <string[]> [-Off] [<CommonParameters>]

    Watch-Job [-InstanceId] <guid[]> [-Off] [<CommonParameters>]

    Watch-Job [-Id] <int[]> [-Off] [<CommonParameters>]
    ```

    This cmdlet would turn monitoring on/off for jobs. See the next item for
    details on what happens when monitoring is on.

1. For jobs invoked with `&!` or `Start-Job -Watch`, or for jobs referenced
in `Watch-Job`, hook up event handlers on the output stream collections for
the job objects to capture the output from the jobs as it is added in real
time, and show that output in the current terminal with the job name or ID in
front of each output record. Records for all stream types will be shown using
their string representation when monitored this way.
1. Add a `&!` operator that starts a job with monitoring turned on.
1. Add a boolean property to `BackgroundJob` objects called `Watch` that
identifies if the job is currently configured to show output or not.
1. Add an integer property to `BackgroundJob` objects called `ProcessId` that
identifies the ID of the process where the job is running.
1. Add a `-Watch` parameter to `Start-Job` that sets `Watch` to true
and hooks up event handlers as described in the first item in this list.
1. Add a `$!` variable to store the most recently run job (or jobs if multiple
jobs are launched at the same time, e.g. `cmd1 & cmd2 & cmd3 &`).
1. Add the `ProcessId` property to the default output for all jobs.

## Alternate Proposals and Considerations

All alternative proposals and considerations identified so far have either been
dismissed or incorporated into the RFC.
