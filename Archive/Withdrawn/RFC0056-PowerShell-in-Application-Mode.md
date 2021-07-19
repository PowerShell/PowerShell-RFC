---
RFC: '0056'
Author: Bruce Payette
Status: Withdrawn
SupercededBy: 
Version: 6.?
Area: Hosting
Comments Due: 01/03/2019
Plan to implement: Yes
---

#PowerShell Application Mode

##Motivation

There is increasing need for the ability to deploy stand-alone non-interactive (embedded) PowerShell “applications”. These applications would be consumed by higher-layer services which, for various reasons, can’t run PowerShell in-process with the existing PowerShell API.
Examples:

-	Azure Functions/AWS lambdas
-	Azure Stack
-	PowerShell language service for VSCode, etc.
-	Azure resource extension scripts/configurations
-	Puppet/Chef

##Specifcation

This type of PowerShell deployment differs from traditional interactive applications in a number if ways:
-	The functionality is likely predefined – the application is composed of a known set of scripts and modules.
-	This implies that only a subset of the PowerShell functionality is required. For example, the help and formatting subsystems might be unnecessary. The net result would be faster startup, reduced footprint and consequently a reduced security Surface.
-	Interactive debugging is not as applicable; logging is required for postmortem debugging.

###Logging

PowerShell provides a number of “streams” for logging style behavior however they are mostly focused on the interactive experience (verbose, error, etc.). By default, these streams are simply written to the console. Without a console, there are two things that might be done with the messages:

-	Embed them in the output stream. This requires the calling application to demultiples the messages (which is what the PowerShell console host does now.) Returning them in a single stream has the advantage that the messages are in the proper context.
-	Log the messages, with to a buffer in memory or to a file.

###Packaging

-	Similar to ‘dotnet package’ – bundle all required resources into a directory which can be copied.
-	Bundle into a ‘self-installing’ exe with resources (e.g. modules) read from memory rather than disk files. (or a .zip file)

###Servicing & Versioning

-	tbd

###Security

-	tbd

##Basic Host model (non-HTTP)

-	use stdin/stdout for host connection
-	JSON commands sent to std in, JSON objects returned from stdout.
-	Asynchronous “jobs” model for communication
-	Commands start/stop jobs or retrieve results
-	Results are retrieved in batches, formatted as JSON objects.

An example command might look like:

```JSON
{
    "Action" = "StartPipeline",
    "PipeLine" = "get-process | sort -descending | select -first 5"
}
```

This command, when processed by the host will immediately return a job id for the running pipeline as follows:

```JSON
{
    "JOBID" = <id>
    "STATUS" = “Running”
}
```
 To stop a running job, the parent will send a command as follows

```JSON
{
    "Action" = "StopPipeline"
    "ID" = <id>
}
```
The host will then return a result object as follows:

```JSON
{
    "JOBID" = <id>
    "STATUS" = “Succeeded” | “Failed”
    "ERROR"  = “error string”
}
```
To retrieve results from a job, the parent application will send a message as follows:

```JSON
{
    "Action" = "RecieveJob"
    "ID" = <id>
    "MaxRecords" = 500  # optional
}
```
If a large amount of data is expected, the parent application can specify the maximum number of records to return. Records will be buffered by the PAM host. Subsequence `ReceiveJob` calls will retrieve the next set of records from the PAM host.

##Asynchronous Notification of State Changes

On job state changes the PAM host will write a message to stdout

```JSON
{
    "JOBSTATE" = “DataReady" | "Completed" | "Error"
    "ID" = <id>
}
```
##HTTP Host

The HTTP host will run as a standalone HTTP application. Communication will use the same JSON messages as the basic host but over HTTP instead of stdio.

