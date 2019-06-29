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

For example, in [PR 206](https://github.com/PowerShell/PowerShell-RFC/pull/206) there is an RFC to easily add parallel processing to any
advanced function or cmdlet in PowerShell. With that capability plus a `~&`
ThreadJob operator, scripters would be able to easily leverage parallel
processing in a pipeline, and additionally run the entire pipeline in a thread
so that the monitoring and processing of the results can be deferred to later
in their script or command.

## Motivation

    As a scripter,
    I want an operator that allows me to create a thread job in which a single pipeline is executed
    so that I can discover it easily (when learning about the & background operator) and run my pipelines on threads more easily in PowerShell.

## User Experience

```powershell
$twoSessions = @($session1, $session2)

# Run a command in a threadjob
$job = Invoke-Command -Parallel -Session $twoSessions -ScriptBlock {'ThreadJob data'} ~&

# Get the results from the threadjob
$job | Wait-Job | Receive-Job
```

```output
ThreadJob data
ThreadJob data
```

## Specification

The implementation of this operator would mirror what was done for the `&`
background operator, but launching a thread job instead of a background job.

## Alternate Proposals and Considerations

### Dispatching commands on a runspace/runspace pool

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

### Using `&~` instead of `~&` for the operator

I had originally considered using `&~` for the operator for this, but ultimately
decided to flip it around. This decision was made in consideration for the [RFC
in PR 193](https://github.com/PowerShell/PowerShell-RFC/pull/193), which proposes additional background operators that control how job
data is displayed in a PowerShell terminal. If that RFC gets traction, it would
be better for both `&` and `~&` to have an equivalent where the `!` follows the
ampersand in both operators for consistency. That is why the operator that is
proposed for thread jobs has the `~` placed in front of the `&`.

### Breaking changes

If there is a script today that uses the `&` background operator after a command
whose name finishes with a `~` or that ends with a `~` argument with no spaces
between the two, this change will break that command. I believe this is very,
very low risk, but wanted to call it out so that we're aware of the potential
for a breaking change.

### Eliminating any need for an `-AsJob` parameter

Some proposals come up from time to time that suggest adding an `-AsJob`
parameter to a PowerShell cmdlet. This is a bad design approach for a language
that already has the `&` background operator and that has both background jobs
and thread jobs, for many reasons:

* the operator approach works for _all pipelines_, vs. `-AsJob` only working on
the command where it is a parameter
* the `-AsJob` name does not indicate if it creates a thread job or a
background job, which is an important detail to be aware of, and which should
be consistent across commands
* using `-AsJob` as a parameter changes what is output by the command, which
causes challenges for thread jobs if the resources should be released in an end
block

For these reasons, a better design that promotes consistency, discoverability,
and applicability anywhere in the language regardless of the commands being
used is either the `Start-*Job` cmdlets or, for simplicity and ease of use, the
`&` background operator and the `~&` threadjob operator.
