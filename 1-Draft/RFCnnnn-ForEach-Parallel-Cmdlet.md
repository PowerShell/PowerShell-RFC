---
RFC: RFCnnnn
Author: Paul Higinbotham
Status: Draft
SupercededBy: N/A
Version: 1.0
Area: Engine
Comments Due: July 18, 2019
Plan to implement: Yes
---

# PowerShell ForEach-Object -Parallel Cmdlet

This RFC proposes a new parameter set for the existing ForEach-Object cmdlet to parallelize script block executions, instead of running them sequentially as it does now.  

## Motivation

    As a PowerShell User,
    I can do simple fan-out concurrency with the PowerShell ForEach-Object cmdlet, without having to obtain and load a separate module, or deal with PowerShell jobs unless I want to.

## Specification

There will be two new parameter sets added to the existing ForeEach-Object cmdlet to support both synchronous and asynchronous operations for parallel script block execution.
For the synchronous case, the `ForEach-Object` cmdlet will not return until all parallel executions complete.
For the asynchronous case, the `ForEach-Object` cmdlet will immediately return a PowerShell job object that contains child jobs of each parallel execution.

### Implementation details

Implementation will be similar to the ThreadJob module.
Script block execution will be run for each piped input on a separate thread and runspace.
The number of threads that run at a time will be limited by a `-ThrottleLimit` parameter with a default value.
Piped input that exceeds the allowed number of threads will be queued until a thread is available.
For synchronous operation, a `-Timeout` parameter will be available that terminates the wait for completion after a specified time.
Without a `-Timeout` parameter, the cmdlet will wait indefinitely for completion.

### Synchronous parameter set

Synchronous ForEach-Object -Parallel returns after all script blocks complete running or timeout

```powershell
ForEach-Object -Parallel -ThrottleLimit 10 -TimeoutSecs 1800 -ScriptBlock {} 
```

- `-Parallel` : parameter switch specifies fan-out parallel script block execution

- `-ThrottleLimit` : parameter takes an integer value that determines the maximum number threads

- `-TimeoutSecs` : parameter takes an integer that specifies the maximum time to wait for completion in seconds

### Asynchronous parameter set

Asynchronous ForEach-Object -Parallel immediately returns a job object for monitoring parallel script block execution

```powershell
ForEach-Object -Parallel -ThrottleLimit 5 -AsJob -ScriptBlock {}
```

- `-Parallel` : parameter switch specifies fan-out parallel script block execution

- `-ThrottleLimit` : parameter takes an integer value that determines the maximum number threads

- `-AsJob` : parameter switch returns a job object

### Variable passing

ForEach-Object -Parallel will support the PowerShell `$_` current piped item variable within each script block.
It will also support the `$using:` directive for passing variables from script scope into the parallel executed script block scope.

### Examples

```powershell
$computerNames = 'computer1','computer2','computer3','computer4','computer5'
$logs = $computerNames | ForEach-Object -Parallel -ThrottleLimit 10 -TimeoutSecs 1800 -ScriptBlock {
    Get-Logs -ComputerName $_
}
```

```powershell
$computerNames = 'computer1','computer2','computer3','computer4','computer5'
$job = ForEach-Object -Parallel -ThrottleLimit 10 -InputObject $computerNames -AsJob -ScriptBlock {
    Get-Logs -ComputerName $_
}
$logs = $job | Wait-Job | Receive-Job
```

```powershell
$computerNames = 'computer1','computer2','computer3','computer4','computer5'
$logNames = 'System','SQL'
$logs = ForEach-Object -Parallel -InputObject $computerNames -ScriptBlock {
    Get-Logs -ComputerName $_ -LogNames $using:logNames
}
```

## Alternate Proposals and Considerations

Another option (and a previous RFC proposal) is to resurrect the PowerShell Windows workflow script `foreach -parallel` keyword to be used in normal PowerShell script to perform parallel execution of foreach loop iterations.
However, the majority of the community felt it would be more useful to update the existing ForeEach-Object cmdlet with a -parallel parameter set.
We may want to eventually implement both solutions.
But the ForEach-Object -Parallel proposal in this RFC should be implemented first since it is currently the most popular.
