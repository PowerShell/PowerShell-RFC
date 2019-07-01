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
    I can execute foreach-object piped input in script blocks running in parallel threads, either synchronously or asynchronously, while limiting the number of threads running at a given time.

## Specification

A new `-Parallel` parameter set will be added to the existing ForEach-Object cmdlet that supports running piped input concurrently in a provided script block.  

- `-Parallel` parameter switch specifies parallel script block execution

- `-ScriptBlock` parameter takes a script block that is executed in parallel for each piped input variable

- `-ThrottleLimit` parameter takes an integer value that determines the maximum number of script blocks running at the same time

- `-TimeoutSecs` parameter takes an integer that specifies the maximum time to wait for completion before the command is aborted

- `-AsJob` parameter switch indicates that a job is returned, which represents the command running asynchronously

The 'ForEach-Object -Parallel' command will return only after all piped input have been processed.
Unless the '-AsJob' switch is used, in which case a job object is returned immediately that monitors the ongoing execution state and collects generated data.
The returned job object can be used with all PowerShell cmdlets that manipulate jobs.

### Implementation details

Implementation will be similar to the ThreadJob module in that thread script block execution will be contained within a PSThreadChildJob object.
The jobs will be run concurrently on separate runspaces/threads up to the ThrottleLimit value, and the remainder queued to wait for an available runspace/thread to run on.  
Initial implementation will not attempt to reuse threads and runspaces when running queued items, due to concerns of stale state breaking script execution.
For example, PowerShell uses thread local storage to store per thread default runspaces.
And even though there is a runspace 'ResetRunspaceState' API method, it only resets session variables and debug/transaction managers.
Imported modules and function definitions are not affected.
A script that defines a constant function would fail if the function is already defined.  
The initial assumption will be that runspace/thread creation time is insignificant compared to the time needed to execute the script block, either because of high compute needs or because of long wait times for results.
If this assumption is not true then the user should consider batching the work load to each foreach-object iteration, or simply use the sequential/non-parallel form of the cmdlet.

The 'TimeoutSecs' parameter will attempt to halt all script block executions after the timeout time has passed, however it may not be immediately successful if the running script is calling a native command or API, in which case it needs for the call to return before it can halt the running script.

### Variable passing

ForEach-Object -Parallel will support the PowerShell `$_` current piped item variable within each script block.
It will also support the `$using:` directive for passing variables from script scope into the parallel executed script block scope.  
If the passed in variable is a value type, a copy of the value is passed to the script block.  
If the passed in variable is a reference type, the reference is passed and each running script block can modify it.
Since the script blocks are running in different threads, modifying a reference type that is not thread safe will result in undefined behavior.

### Supported scenarios

```powershell
# Ensure needed module is installed on local system
if (! (Get-Module -Name MyLogsModule -ListAvailable)) {
    Install-Module -Name MyLogsModule -Force
}
```

```powershell
$computerNames = 'computer1','computer2','computer3','computer4','computer5'
$logs = $computerNames | ForEach-Object -Parallel -ThrottleLimit 10 -TimeoutSecs 1800 -ScriptBlock {
    Get-Logs -ComputerName $_
}
```

```powershell
$computerNames = 'computer1','computer2','computer3','computer4','computer5'
$job = ForEach-Object -Parallel -ThrottleLimit 10 -InputObject $computerNames -TimeoutSecs 1800 -AsJob -ScriptBlock {
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

```powershell
$computerNames = 'computer1','computer2','computer3','computer4','computer5'
$logNames = 'System','SQL','AD','IIS'
$logResults = ForEach-Object -Parallel -InputObject $computerNames -ScriptBlock {
    Get-Logs -ComputerName $_ -LogNames $using:logNames
} | ForEach-Object -Parallel -ScriptBlock {
    Process-Log $_
}
```

### Unsupported scenarios

```powershell
# Variables must be passed in via $using: keyword
$LogNameToUse = "IISLogs"
$computers | ForEach-Object -Parallel -ScriptBlock {
    # This will fail because $LogNameToUse has not been defined in this scope
    Get-Log -ComputerName $_ -LogName $LogNameToUse
}
```

```powershell
# Passed in reference variables should not be assigned to
$MyLogs = @()
$computers | ForEach-Object -Parallel -ScriptBlock {
    # Not thread safe, undefined behavior
    # Cannot assign to using variable
    $using:MyLogs += Get-Logs -ComputerName $_
}

$dict = [System.Collections.Generic.Dictionary[string,object]]::New()
$computers | ForEach-Object -Parallel -ScriptBlock {
    $dict = $using:dict
    $logs = Get-Logs -ComputerName $_
    # Not thread safe, undefined behavior
    $dict.Add($_, $logs)
}
```

```powershell
# Value types not passed by reference
$count = 0
$computers | ForEach-Object -Parallel -ScriptBlock {
    # Can't assign to using variable
    $using:count += 1
    $logs = Get-Logs -ComputerName $_
    return @{
        ComputerName = $_
        Count = $count
        Logs = $logs        
    }
}
```

## Alternate Proposals and Considerations

Another option (and a previous RFC proposal) is to resurrect the PowerShell Windows workflow script `foreach -parallel` keyword to be used in normal PowerShell script to perform parallel execution of foreach loop iterations.
However, the majority of the community felt it would be more useful to update the existing ForeEach-Object cmdlet with a -parallel parameter set.
We may want to eventually implement both solutions.

There are currently other proposals to create a more general framework to support running arbitrary scripts and cmdlets in parallel, by marking them as able to support parallelism (see RFC #206).
That is outside the scope of this RFC, which focuses on extending just the ForEach-Object cmdlet to support parallel execution, and is intended to allow users to do parallel script/command execution without having to resort to PowerShell APIs.
