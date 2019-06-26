---
RFC: RFCnnnn
Author: Kirk Munro
Status: Draft
SupercededBy: 
Version: 0.1
Area: Parser,Compiler
Comments Due: July 26, 2019
Plan to implement: Yes
---

# `~&` ThreadJob Operator

In PowerShell 6.0, the `&` background operator was added to PowerShell to
facilitate easy invocation of a pipeline in a background job. This is useful,
but there are also scenarios when you want each invocation of a pipeline in a
thread job.

For example, in [PR 123](LINK) there is an RFC to easily add parallel processing to any
advanced function or cmdlet in PowerShell. With that capability plus a `~&`
ThreadJob operator, scripters would be able to easily leverage parallel
processing in a pipeline, and additionally run the entire pipeline in a thread
so that the monitoring and processing of the results can be deferred to later
in their script or command.

## Motivation

    As a scripter,
    I want an operator that allows me to create a thread job in which a single pipeline is executed
    so that I can run pipelines on threads more easily in PowerShell.

## User Experience

```powershell
# Run a command in a threadjob
$job = Get-Process -Id 12345678 2>&1 ~&
# Get the results from the threadjob
$job | Receive-Job
```

```output
Get-Process : Cannot find a process with the process identifier 12345678.
At line:1 char:1
+ Get-Process -Id 12345678 2>&1
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
+ CategoryInfo          : ObjectNotFound: (12345678:Int32) [Get-Process], ProcessCommandException
+ FullyQualifiedErrorId : NoProcessFoundForGivenId,Microsoft.PowerShell.Commands.GetProcessCommand
```

## Specification

The implementation of this operator would mirror what was done for the `&`
background operator, but launching a thread job instead of a background job.

## Alternate Proposals and Considerations

@BrucePay suggested that he always figured they would support dispatching
commands on a runspace or runspace pool using syntax something like the
following:

```PowerShell
& $runspacePool invoke-stuff -arg1 -arg2 abc
```

That syntax is familiar (it's similar to `& (gmo ModuleName) Do-Stuff`), but it
lacks consistency with the new `&` background operator, and may not be as
discoverable. On the other hand, supporting `~&` would allow both of those
operators to be discussed in the same documentation, and since they would have
the same use scenarios, it will be easy to use one if you have used the other.

I believe supporting runspace pool management is interesting and useful, not
necessarily for a single pipeline like this, but moreso for commands that are
configured to support parallel processing (as described in the RFC behind [PR
123](link)), where common parameters can be augmented to support passing in a
specific runspace or a runspace pool.
