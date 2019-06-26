---
RFC: RFCnnnn
Author: Kirk Munro
Status: Draft
SupercededBy: 
Version: 0.1
Area: Engine
Comments Due: July 26, 2019
Plan to implement: I'm willing to help
---

# Generalized Parallelism

The inspiration for this RFC came from [RFC 194](https://github.com/PowerShell/PowerShell-RFC/pull/194).

Recently PowerShell has received some enhancements targeted toward more
parallel invocation of PowerShell, such as the `&` background operator and the
`PowerShell.InvokeAsync` and `PowerShell.StopAsync` async methods. There have
also been an RFC for [parallelism in `foreach`](https://github.com/PowerShell/PowerShell-RFC/pull/174) and another RFC for [parallelism
in `ForEach-Object`](https://github.com/PowerShell/PowerShell-RFC/pull/194) (RFC 194).

[RFC 194](https://github.com/PowerShell/PowerShell-RFC/pull/194) focuses on adding parallelism support to one cmdlet: ForEach-Object.
While that is useful, with a little more effort and planning there may be a
better way to approach that problem such that any advanced function or cmdlet
could have parallel invocation support. This RFC is being put on the table for
consideration of the broader need for parallelism in PowerShell.

While it is somewhat in competition with [RFC 194](https://github.com/PowerShell/PowerShell-RFC/pull/194), the end result would be very
similar. `ForEach-Object` would have parallel invocation support, as would any
advanced function or cmdlet that was designed to have such support. By laying a
foundation that can be applied to any cmdlet, and then applying the new
capabilities that foundation brings to PowerShell to the `ForEach-Object`
cmdlet, we end up with a common framework for faster automation where
asynchronous invocation would be advantageous that can be applied to any
advanced function or cmdlet.

Note that [RFC 204](https://github.com/PowerShell/PowerShell-RFC/pull/204) is very closely related to this RFC and both should be
implemented at the same time, assuming that both are approved. If you haven't
already, have a quick look at that RFC and share feedback on it. It's very
straightforward.

Also note that 

## Motivation

    As an author of PowerShell commands,
    I can turn on parallelism support in my commands and implement the command logic accordingly,
    so that scripters using my command can build faster automated solutions that scale.

## User Experience

To demonstrate the user experience for command authors, let's take a closer
look at the `Invoke-Command` cmdlet. `Invoke-Command` has parallelism built-in
already, but when you inspect the metadata, you'll see that there are some gaps
in the implementation across its 17 parameter sets. These gaps result in
inconsistencies that could have been avoided completely by having parallelism
support for commands implemented as a feature that can be turned on in
parameter sets where it is needed.

> Please note that `Invoke-Command` creates remoting jobs which are background
jobs run in child processes, not thread jobs that are run in separate runspaces
on separate threads in the same process; however, that implementation predates
the PSThreadJob module, so don't let that distract you. The goal here is easier
parallelism in any command where it is appropriate, and in our cloud-centric
work environment, there are many commands where parallelism would be useful.
The `Invoke-Command` example is still a good one to use because it is already
familiar.

To understand what I'm talking about with `Invoke-Command`, invoke the
following in PowerShell:

```powershell
function HasParameter {
    param(
        [System.Management.Automation.CommandParameterSetInfo]$ParameterSet,
        [string]$ParameterName
    )
    @($ParameterSet.Parameters.where{$_.Name -eq $ParameterName}).Count -gt 0
}
$cmd = Get-Command Invoke-Command
$cmd.ParameterSets.foreach{
    [pscustomobject]@{
        ParameterSet = $_.Name
        AsJob = HasParameter $_ 'AsJob'
        JobName = HasParameter $_ 'JobName'
        ThrottleLimit = HasParameter $_ 'ThrottleLimit'
        TimeoutSecs = HasParameter $_ 'TimeoutSecs'
    }
} | ft
```

This outputs the following:

```none
ParameterSet         AsJob JobName ThrottleLimit TimeoutSecs
------------         ----- ------- ------------- -----------
InProcess            False   False         False       False
FilePathRunspace      True    True          True       False
Session               True    True          True       False
ComputerName          True    True          True       False
FilePathComputerName  True    True          True       False
Uri                   True    True          True       False
FilePathUri           True    True          True       False
VMId                  True   False          True       False
VMName                True   False          True       False
FilePathVMId          True   False          True       False
FilePathVMName        True   False          True       False
SSHHost               True   False         False       False
ContainerId           True    True          True       False
FilePathContainerId   True    True          True       False
SSHHostHashParam      True   False         False       False
FilePathSSHHost       True   False         False       False
FilePathSSHHostHash   True   False         False       False
```

That output highlights several limitations that come from the way parallelism
is implemented in `Invoke-Command`:

1. Only half of the parameter sets that support `-AsJob` allow you to name the
job that is created with a `-JobName` parameter, but they all should support
that.
1. Some of the parameter sets that support parallelism don't allow you to set a
throttle limit, but they all should.
1. `Invoke-Command` uses multi-process parallelism with the same parameter
names that are proposed for multi-threaded parallelism in [RFC 194](https://github.com/PowerShell/PowerShell-RFC/pull/194). This is
inconsistent and bound to cause some confusion.
1. None of the parameter sets that support parallelism allow you to set a
timeout, because these are implemented to run in background jobs, which don't
timeout. If they were implemented to run as threadjobs instead, timeouts become
possible and so they should consider timeouts in a multi-threaded
implementation.

If `Invoke-Command` was implemented instead using a setting in a `CmdletBinding`
attribute that automatically took care of the parameters used for parallel
invocation of certain parameter sets, then the parameters for parallelized
invocation would always be properly defined, and they would always work the
same way for this cmdlet and for every other advanced function or command that
supports parallelization.

With all of that explained, below you'll see a simplified example showing how
the `Invoke-Command` command might be defined to support parallelization with
common parameters. For that example, I only included a few parameter sets to
keep it simple (there is no need to include all 17 parameter sets here).

```powershell
function Invoke-Command {
    [CmdletBinding(
        DefaultParameterSetName='InProcess',
        # NEW! SupportsParallel defines the parameter sets that support
        # parallel execution of the process block in thread jobs that are
        # automatically created. All other parameter sets would result in
        # synchronous execution of the process block in the current runspace.
        # Each parameter set is named, separated by commas. If the command
        # should support parallel execution in only one of the parameter sets,
        # the array enclosures can be omitted. If the command should support
        # parallel execution in all parameter sets, the command author can
        # simply drop the equals operator and the values after it.
        SupportsParallel=@('FilePathRunspace','Session','VMId')
    )]
    param(
        # Define parameters here

        # NEW! Any parameter sets that support parallel invocation would
        # automatically receive additional common parameters. Like other common
        # parameters, these could be used in Intellisense and tab completion,
        # but they wouldn't be defined in a param block. They would be assigned
        # automatically to the appropriate parameter sets by PowerShell.
        # [-Parallel] : A switch used to run a command in parallel with the
        #               default configuration for parallel command execution.
        #               This parameter is not required if any of the following
        #               parameters are used in the invocation.
        # [-ThreadLimit <int>] : An int used to define the maximum number of
        #                        threads to use during the parallel command
        #                        execution (default value depends on system
        #                        configuration)
        # [-TimeoutSecs <int>] : An int used to define the maximum time allowed
        #                        for all threads to complete. This parameter
        #                        cannot be used in conjunction with the next
        #                        two parameters.
    )
    begin {
        # No changes here, this would be invoked synchronously in the current
        # session.
    }
    process {
        # NEW! The way this is invoked depends on whether or not the parameter
        # set that was used supports parallelization or not, and whether or not
        # the command was invoked with `-AsThreadJob` or `-ThreadJobPrefix`.
        #
        # For a parameter set that does not support parallelization, this would
        # be invoked synchronously.
        #
        # For a parameter set that supports parallelization, this would be
        # invoked in a thread job. If that 
    }
}
```

```output
Hello World
```

## Specification

## Alternate Proposals and Considerations

### Running a parallelized command asynchronously

Parallelization is about being able to perform multi-threaded processing of
objects in a pipeline. This is what is natively supported by the enhancements
outlined in this RFC. 

        # [-AsThreadJob] : A switch that tells PowerShell to create a thread
        #                  job and return it for each invocation of the process
        #                  block. This parameter is not required if the next
        #                  parameter is used.
        # [-ThreadJobPrefix <string>] : A string prefix that will be applied to
        #                               each thread job created. Thread jobs
        #                               are named Job<n> by default, where <n>
        #                               is an ordinal number. If a prefix is
        #                               provided, it is used in place of "Job"
        #                               in the job name.


### Multiple processes vs multiple threads

