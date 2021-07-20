---
RFC:  RFC0023
Author:  Bruce Payette
Status:  Rejected
SupercededBy:  N/A
Version: 0.1
Area: CMDLETS
Comments Due: 4/10/2017
---

# Native Support for Concurrent Programming in PowerShell

This RFC proposes a new built-in facility for concurrent programming in PowerShell. This facility will be implemented as a set of cmdlets on top of the existing runspace facility.

## Motivation

There are many scenarios in system administration where concurrent operations would be useful (see issue https://github.com/PowerShell/PowerShell/issues/3008). PowerShell workflow provides for concurrent operations but only within the context of a workflow thus entailing the constraints and overhead of the workflow runtime. There are a number of existing 3rd party implementations of similar functionality (e.g. https://github.com/proxb/PoshRSJob) The goal here is to provide a common in-box solution for concurrent execution in PowerShell scripts. 

### Goals

Simplicity of use
-	Doesn’t use the jobs pattern which requires explicit cleanup for the job, Tasks can be fire and forget
    -	Tasks will clean up after themselves, closing the runspace if appropriate and disposing the powershell object
    -	Auto-dispose means no need for the equivalent to Remove-Job
-	No hierarchy as is the case with jobs/childjobs – one task object per call to Start-PSTask
-	No separate wait and receive commands. A single Wait-PSTask command subsumes both.
Reliability:
-	Variables in task scriptblocks will need to either be explicitly initialized or have the $using: qualifier. Use of uninitialized variables will result in a parse-time error.
-	Stream (error, verbose, etc.) actions (-OnError, -OnVerbose, etc.) are supported to make sure output is not missed.

## Specification

New cmdlets Start-PSTask, Wait-PSTask, Stop-PSTask

Start-PSTask [-ScriptBlock] <scriptblock> [-Arguments <Object[]>] [-Name <string>] [-OnError <scriptblock>] [-OnOutput <scriptblock>] [-OnVerbose <scriptblock>] [-OnInformation <scriptblock>] [-OnWarning <scriptblock>] [-OnDebug <scriptblock>] [<CommonParameters>]
Start-PSTask [-FunctionName] <string> [-ArgumentList <Object[]>] [-Name <string>] [-OnError <scriptblock>] [-OnOutput <scriptblock>] [-OnVerbose <scriptblock>] [-OnInformation <scriptblock>] [-OnWarning <scriptblock>] [-OnDebug <scriptblock>] [<CommonParameters>]

-	This cmdlet starts a task instance and returns a PSTaskInfo object for that task.
-	Tasks will be identified by an integer ID which increases monotonically for each new task created
-	The maximum number of concurrently executing tasks will be controlled by a preference variable $PSTaskMaximumInstanceCount
-	Parameter -ScriptBlock
    -	Specifies the scriptblock to use for the task’s actions. This scriptblock will be subject to strict variable assignment: variables must either be explicitly assigned in the scriptblock of have the `$using:` modifier. Use of unassigned variables will result in an error.
-	Parameter -FunctionName
    -	Specifies the name of a function to use as the task’s action. Since $using: can’t be used in a function body, the -ArgumentList parameter should be used to pass arguments to the function.
-	Parameter -ArgumentList
    -	Allows arguments to be passed to functions or parameterized scriptblocks.
-	Parameter -NoThrottle 
    -	allows a task to be run that doesn’t count against the maximum allowed number of concurrent tasks.
?	Example Scenario: background task colorizing the console that runs forever
-	Parameter -Name 
    -	if specified, the task id will use the name as the stem of the task id. E.g. if the name if “foo” then tasks will be named “foo.1”. “foo.2”, etc.
-	Parameter -On* 
    -	These parameters specify actions will be executed in the originating runspace, not in the task runspace. If there are any records in the associated stream, then the action scriptblock will be executed, receiving the PSTaskInfo object as $PSTaskInfo
Stop-PStask -TaskList <PSTaskInfo[]>
-	Takes an array of PSTaskInfo objects and stops any tasks in the QUEUED or RUNNING states. Tasks in the STOPPED or COMPLETED state will not be impacted.
-	Returns nothing on success, but the task is placed in the STOPPED state.
-	If the task doesn’t exist, returns nothing
-	If the task fails to stop within as certain time window, returns an error
-	-Parameter -TaskList
    -	Takes a list of PSTaskInfo objects to stop.
    -	Supports ValueFromPipeline
Wait-PSTask -Task <PSTaskInfo[]> [-Timeout NN]
-	Waits until the task object has completed, then writes content of all streams returned by the task to the corresponding output streams
-	Any pending task actions (OnError, On*) will be invoked prior to the task output being written.
-	Adds PSTaskID property to each object output 
-	Parameter -TaskList
    -	Takes a list of PSTaskInfo objects to wait for.
    -	Supports ValueFromPipeline
-	Parameter -Timeout
    -	Optional timeout for the wait in milliseconds

A new class – PSTaskInfo – will be added. The PSTaskInfo class represents an instance of a task. A task can be in one of four states: QUEUED, RUNNING or COMPLETED or STOPPED. Tasks are queued when there aren’t enough runspaces to run all tasks concurrently (i.e. the value in $PSTaskMaximumInstanceCount has been reached). This class is defined as follows:

```csharp
    public enum TaskState
    {
        QUEUED    = 0,
        RUNNING   = 1,
        COMPLETED = 2,
        STOPPED   = 3,
    }

    public class PSTaskInfo
    {
        /// <summary>
        /// The current state of the task
        /// </summary>
        public TaskState State { get; set; }
        
        /// <summary>
        /// True if there were any errors during execution
        /// </summary>
        public bool HadErrors {get { … }}

        /// <summary>
        /// Wait for the task to complete then update the collections
        /// </summary>
        /// <returns></returns>
        public PSDataCollection<PSObject> Wait() { … }

        /// <summary>
        /// Identifier for the current task
        /// </summary>
        public string TaskID { get; set; }

        /// <summary>
        /// Content of the streams saved as lists so they can be inspected without
        /// draining the collection. Must be typed as PSObject to allow the PSTaskID to
        /// be added to the result object.                        
        /// </summary>
        public List<PSObject> Output { get; set; }
        public List<PSObject> Error { get; set; }
        public List<PSObject> Verbose { get; set; }
        public List<PSObject> Warning { get; set; }
        public List<PSObject> Debug { get; set; }
        public List<PSObject> Information { get; set; }

    }
```

### New Preference variable
```powershell
	$PSTaskThrottleLimit
```
This preference variable controls the maximum number of concurrently running tasks. This limit can be bypassed by using the -NoThrottle parameter on Start-PSTask.

### Execution Environment

Tasks will be executed in the same process/appdomain as the parent. The execution environment for a throttled task will be equivalent to runspace pool runspace. This implies that there is no shared state between the parent and task runspaces. Any modules required by the task will either need to be explicitly loaded or be loaded through autoloading. The current working directory for the runspace will be whatever it was set to when last used.
Tasks started with -NoThrottle will execute in process in a new runspace created by the equivalent of RunspaceFactory.CreateRunspace().

### Examples

```powershell
 #===================================
 # Simple single task execution
 #
 Start-PSTask { "Hello" } | Wait-PSTask

 #===================================
 # Use with Foreach-Object
 #
 1..20 | foreach {Start-PSTask { "Hello $using:_" }} | Wait-PSTask

 #====================================
 # Use ForEach-Object with -Arguments
 #
 1..10 | foreach {Start-PSTask {param($x) "x is $x"} -Arguments $_} | wait-pstask
 
 #===================================
 # Example using foreach statement to get sizes of files into a hashtable
 #
 $tl = foreach ($f in Get-ChildItem *.ps1) { 
    Start-PSTask {
        # Return a hashtable mapping leaf name to count
        @{ (Split-Path -Leaf $using:f) = (cat $using:f).Count }
    }
 }

 $result = @{}
 # Add all of the hashtables together...
 foreach ($r in Wait-PSTask $tl)
 {
    $result += $r
 }
 # And return the aggregate hash table...
 $result

 #====================================
 # launch parallel tasks in a foreach loop, computing the 
 # factorial of the iteration number
 #
 $tasks = foreach ($iteration in 1..10) {
            Start-PSTask {
		   
                function fact ($x)
                {
                    (1..$x).foreach{
                        begin {$result = 1} 
                        process {$result *= $_ } 
                        end { $result }}
                }

                if ($using:iteration -eq 3)
                {
                    Write-Error "Error in iteration 3"
                }
   # Write directly to console for immediate display

                [console]::WriteLine(
"Fact $using:iteration is $(fact $using:iteration)")
            }
          }

 # Handle the results of the tasks...
 foreach ($t in $tasks)
 {
    Wait-PSTask $t
    if ($t.HadErrors)
    {
        $t.Error
    }
 }

#====================================
# start three individual tasks
$t1 = start-pstask { "Hello from task1" }
$t2 = start-pstask { "Hello from task2" }
$t3 = start-pstask { "Hello from task3" }
# wait for the results of all three
Wait-PSTask $t1,$t2,$t3
``` 

## Alternate Proposals and Considerations

None.

