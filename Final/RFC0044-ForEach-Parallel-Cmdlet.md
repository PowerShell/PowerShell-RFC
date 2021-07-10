---
RFC: RFC0044
Author: Paul Higinbotham
Status: Final
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

- `-Parallel` parameter takes a script block that is executed in parallel for each piped input variable

- `-ThrottleLimit` parameter takes an integer value that determines the maximum number of script blocks running at the same time

- `-TimeoutSeconds` parameter takes an integer that specifies the maximum time to wait for completion before the command is aborted

- `-AsJob` parameter switch indicates that a job is returned, which represents the command running asynchronously

The `ForEach-Object -Parallel` command will stream output to the console until all piped input has been processed.
If the `-AsJob` switch is used then a job object is returned and remains in the running state while input is being processed.
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

The 'TimeoutSeconds' parameter will attempt to halt all script block executions after the timeout time has passed, however it may not be immediately successful if the running script is calling a native command or API, in which case it needs for the call to return before it can halt the running script.

### Variable passing

ForEach-Object -Parallel will support the PowerShell `$_` current piped item variable within each script block.
It will also support the `$using:` directive for passing variables from script scope into the parallel executed script block scope.  
If the passed in variable is a value type, a copy of the value is passed to the script block.  
If the passed in variable is a reference type, the reference is passed and each running script block can modify it.
Since the script blocks are running in different threads, modifying a reference type that is not thread safe will result in undefined behavior.  

ScriptBlock variables are a special case because they have runspace affinity, and cannot be safely passed to other runspace script blocks for parallel execution.
Consequently, an error will be generated if a ScriptBlock object is directly passed through the input pipeline, or if passed to the parallel script block via the `$using:` directive.
However, it is still possible to pass in a ScriptBlock object indirectly such as through an object method returning a ScriptBlock.
This is not recommended and will result in undefined behavior.

### Exceptions

For critical exceptions, such as out of memory or stack overflow, the CLR will crash the process.
Since all parallel running script blocks run in different threads in the same process, all running script blocks will terminate, and queued script blocks will never run.
This is different from PowerShell jobs (Start-Job) where each job script runs in a separate child process, and therefore has better isolation to crashes.
The lack of process isolation is one of the costs of better performance while using threads for parallelization.  

For all other catchable exceptions, PowerShell will catch them from each thread and write them as non-terminating error records to the error data stream.
If the `ErrorAction` parameter is set to 'Stop' then cmdlet will attempt to stop the parallel execution on any error.

### Stop behavior

Whenever a timeout, a terminating error (-ErrorAction Stop), or a stop command (Ctrl+C) occurs, a stop signal will be sent to all running script blocks, and any queued script block iterations will be dequeued.
This does not guarantee that a running script will stop immediately, if that script is running a native command or making an API call.
So it is possible for a stop command to be ineffective if one running thread is busy or hung.  

We can consider including some kind of 'forcetimeout' parameter that would kill any threads that did not end in a specified time.  

If a job object is returned (-AsJob) the child jobs that were dequeued by the stop command will be at 'NotStarted' state.

### Data streams

Warning, Error, Debug, Verbose data streams will be written to the cmdlet data streams as received from each running parallel script block.  
Progress data streams will not be supported, but can be added later if desired.

### Supported scenarios

```powershell
# Ensure needed module is installed on local system
if (! (Get-Module -Name MyLogsModule -ListAvailable)) {
    Install-Module -Name MyLogsModule -Force
}
```

```powershell
$computerNames = 'computer1','computer2','computer3','computer4','computer5'
$logs = $computerNames | ForEach-Object -ThrottleLimit 10 -TimeoutSeconds 1800 -Parallel {
    Get-Logs -ComputerName $_
}
```

```powershell
$computerNames = 'computer1','computer2','computer3','computer4','computer5'
$job = ForEach-Object -ThrottleLimit 10 -InputObject $computerNames -TimeoutSeconds 1800 -AsJob -Parallel {
    Get-Logs -ComputerName $_
}
$logs = $job | Wait-Job | Receive-Job
```

```powershell
$computerNames = 'computer1','computer2','computer3','computer4','computer5'
$logNames = 'System','SQL'
$logs = ForEach-Object -InputObject $computerNames -Parallel {
    Get-Logs -ComputerName $_ -LogNames $using:logNames
}
```

```powershell
$computerNames = 'computer1','computer2','computer3','computer4','computer5'
$logNames = 'System','SQL','AD','IIS'
$logResults = ForEach-Object -InputObject $computerNames -Parallel {
    Get-Logs -ComputerName $_ -LogNames $using:logNames
} | ForEach-Object -Parallel -ScriptBlock {
    Process-Log $_
}
```

```powershell
$threadSafeDictionary = [System.Collections.Concurrent.ConcurrentDictionary[string,object]]::new()
Get-Process | ForEach-Object -Parallel {
    # This works because the passed in object is a concurrent dictionary that is thread safe
    $dict = $using:threadSafeDictionary
    $dict.TryAdd($_.ProcessName, $_)
}
```

### Unsupported scenarios

```powershell
# Variables must be passed in via $using: keyword
$LogNameToUse = "IISLogs"
$computers | ForEach-Object -Parallel {
    # This will fail because $LogNameToUse has not been defined in this scope
    Get-Log -ComputerName $_ -LogName $LogNameToUse
}
```

```powershell
# Passed in reference variables should not be assigned to
$MyLogs = @()
$computers | ForEach-Object -Parallel {
    # Throws error, cannot assign to using variable
    $using:MyLogs += Get-Logs -ComputerName $_
}
At line:3 char:5
+     $using:MyLogs += Get-Logs -ComputerName $_
+     ~~~~~~~~~~~~~
The assignment expression is not valid. The input to an assignment operator must be an object that is able to accept assignments, such as a variable or a property.
+ CategoryInfo          : ParserError: (:) [], ParentContainsErrorRecordException
+ FullyQualifiedErrorId : InvalidLeftHandSide

$dict = [System.Collections.Generic.Dictionary[string,object]]::New()
$computers | ForEach-Object -Parallel {
    $dict = $using:dict
    $logs = Get-Logs -ComputerName $_
    # Not thread safe, undefined behavior
    $dict.Add($_, $logs)
}
```

```powershell
# Value types not passed by reference
$count = 0
$computers | ForEach-Object -Parallel {
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
