---
RFC: RFCnnnn
Author: Paul Higinbotham
Status: Draft
SupercededBy: N/A
Version: 1.0
Area: Engine
Comments Due: July 1, 2019
Plan to implement: Yes
---

# Implement PowerShell language foreach -parallel

Windows PowerShell currently supports the foreach language keyword with the -parallel switch flag, but only for workflow scripts.

```powershell

workflow wf1 {
    $list = 1..5
    foreach -parallel -throttlelimit 5 ($item in $list) {
        Start-Sleep -Seconds 1
        Write-Output "Output $item"
    }
}

```

This will run the script block with each value in the `$list` array, in parallel using workflow jobs.
However, workflow is not supported in PowerShell Core 6, partly because it is a Windows only solution but also because it is cumbersome to use.
In addition the workflow implementation is very heavy weight, using lots of system resources.  

This is a proposal to re-implement `foreach -parallel` in PowerShell Core, using PowerShell's support for concurrency via Runspaces.
It is similar to the [ThreadJob module](https://www.powershellgallery.com/packages/ThreadJob/1.1.2) except that it becomes part of the PowerShell language via `foreach -parallel`.

## Motivation

    As a PowerShell User,
    I can do simple fan-out concurrency from within the language, without having to obtain and load a separate module or deal with PowerShell jobs.

## Specification

The PowerShell `foreach -parallel` language keyword will be re-implemented to perform invoke script blocks in parallel, similar to how it works for workflow functions except that script blocks will be invoked on threads within the same process rather than in workflow jobs running in separate processes.
The default behavior is to fan-out script block execution to multiple threads, and then wait for all threads to finish.
However, a `-asjob` switch will also be supported that returns a PowerShell job object for asynchronous use.
If the number of foreach iterations exceed the throttle limit value, then only the throttle limit number of threads are created at a time and the rest are queued until a running thread becomes available.

### Supported foreach parameters

- `-parallel`
- `-throttlelimit`
- `-timeout`
- `-asjob`

### P0 Features

- `foreach -parallel` fans out script block execution to threads, along with a bound single foreach iteration value

- `-throttlelimit` parameter value specifies the maximum number of threads that can run at one time

- `-timeout` parameter value specifies a maximum time to wait for all iterations to complete, after which 'stop' will be called on all running script blocks to terminate execution

- `-asjob` switch causes foreach to return a PowerShell job object that is used to asynchronously monitor execution

- When a job object is returned, it will be compatible with all relevant job cmdlets

- All script blocks running in parallel will run isolated from each other.
Only foreach iteration objects will be passed to the parallel script block.

### Data stream handling

`foreach -parallel` will use normal PowerShell pipes to return various data streams.
Data will be returned in order received.  
Except when `-asjob` switch is used, in which case a single job object is returned.
The returned job object will contain an array of child jobs that represent each iteration of the foreach.

### Examples

```powershell
$computerNames = 'computer1','computer2','computer3','computer4','computer5'
$logs = foreach -parallel -throttle 10 -timeout 300 ($computer in $computerNames)
{
    Get-Logs -ComputerName $computer
}
```

```powershell
$computerNames = 'computer1','computer2','computer3','computer4','computer5'
$job = foreach -parallel -asjob ($computer in $computerNames)
{
    Get-Logs -ComputerName $computer
}
$logs = $job | Wait-Job | Receive-Job
```

```powershell
$params += @{
    $argTitle = "Title1"
    $argValue = 102
}
foreach -parallel ($param in $params)
{
    c:\scripts\ToRun.ps1 @param
}
```

## Alternate Proposals and Considerations

One alternative is to create a `ForEach-Parallel` cmdlet instead of re-implementing the `foreach -parallel` keyword.
This would work well but would not be as useful as making it part of the PowerShell language.
But if re-implementing the foreach keyword becomes problematic, it would be a good fallback solution.  
