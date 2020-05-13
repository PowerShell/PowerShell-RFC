---
RFC: RFC<four digit unique incrementing number assigned by Committee, this shall be left blank by the author>
Author: Ilya Sazonov
Status: Draft
SupercededBy: <link to another RFC>
Version: 1.0
Area: Engine
Comments Due: 7/1/2020
Plan to implement: Maybe
---

# Automatic resource management

## Rationale

Currently PowerShell has limited capabilities compared with C# to excecute a mandatory code.

PowerShell users can benefit from `finally` block of `try - catch` statement. It is very limited experience because resources in PowerShell can be distributed across scopes, cmdlets, pipelines and runspaces.

If an user created a temporary file one should worry about deleting it, while PowerShell could register a mandatory code to delete this file and execute it automatically at the right time.

PowerShell makes many smart things for users and could offer smart resource management.

## Motivation

As a script writer, I can use some well-known API-s which grabs resources
and I want PowerShell understands this and releases the resources automatically at the right time
so that avoids resorce leaks.

As an advanced module and script writer, I can use some API-s which grabs resources
and I want to register a code PowerShell must run to release the resources automatically at the right time so that avoids resorce leaks.

As an advanced module and script writer, I want to make sure that a mandatory (cleanup, logging) code is always executed even if a pipeline / script / cmdlet is unexpectedly interrupted.

As binary module developer I want to benefit from this in binary modules too.

## User Experience

User has no needs explicitly dispose an object when out of the script block:

```powershell
# some script commands
...
    # new script block
    {
        # PowerShell detects and registers new IDisposable object for the script block scope
        $fs = [System.IO.File]::Open("file", [System.IO.FileMode]::Open)
        ...
        # Here the script block is closed and
        # PowerShell Engine calls implicitly $fs.Dispose()
        # before destroying the scope.
    }
```

User has no needs explicitly close a connection:

```powershell
# some script commands
...
    # new script block
    {
        # PowerShell finds the type in ETS and registers new object for the scope
        # because ETS prescribes to comply Close() to destroy an object of the type.
        $db = [Database]::Open("name")
        ...
        # Here the script block is closed and
        # PowerShell Engine calls implicitly $db.Close()
        # before destroying the scope.
    }
```

User has no needs explicitly run a mandatory code at cmdlet destroy time:

```powershell
function Get-PingReply {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory, ValueFromPipeline)]
        [string]
        $Target
    )
    begin {
        # PowerShell detects and registers new IDisposable object for the cmdlet scope
        $Ping = [Ping]::new()
    }
    process {
        $Ping.Send($Target, $Timeout, $Buffer, $PingOptions)
    }
}

Get-PingReply '1.2.3.4'
# PowerShell run implicitly $Ping.Dispose() before destroy the cmdlet.
```

User has the ability to explicitly assign a mandatory code:

```powershell
# some script commands
...
    # new script block
    {
        $tempFile = New-TemporaryFile

        # Explicitly say to PowerShell Engine how release a resource before the runspace close
        $tempFile.MakeDisposable([DisposableKind]::AtRunspaceClose, { script to remove the temp file }
        ...
    }
```

## Specification

General statement is to teach `PowerShell Engine` to execute a mandatory (finally) code.

> Notice, .Net Runtime does not ensure finalizer/dispose code execution at application shutdown time.

This covers the main scenarios:

- call `Dispose()` for objects with `IDisposable` to avoid resource leaking
- call mandatory code for objects without `IDisposable` similar to the one that closes a database connection or deletes a temporary file.

_Our approach should be resource-centric._
It was expressed by @BrucePay in [related issue](https://github.com/PowerShell/PowerShell/issues/6673), then in another form @lzybkr, then @alx9r.

We need to __register__ resources for which it is necessary to execute a mandatory (finally) code.

This covers all scenarios described in this RFC and all other scenarios.
It makes the `Engine` smart and brings great UX.
Old scripts and modules will benefit without any change.
No changes is needed in tools like [PowerShell EditorSyntax](https://github.com/PowerShell/EditorSyntax).

Try to beat any scenario as part of this resource-centric approach and you will see that the approach covers this scenario even `ForEach-Object -Parallel`) automatically (including dot sourcing).
(In fact, there can certainly be scenarios where this is impossible in principle, like remoting and serialization, but there may be tradeoffs and workarounds)

### Context / Scope

The `Engine` should take into account Context / Scope.

Since variables can be copied to other scopes, the `Engine` must maintain a _usage counter_ during variable assignment / creation operations.
This adds a bit of complexity, but it should not affect performance, as in most cases, the `Engine` will only have to check a flag and only in very rare cases do extra work.
Even an `IDisposable` check needs to be done only once.
Moreover, we can even make a cache for all known types using our `TypeGen.ps1` utility.

Another feature is that native objects can turn around in `PSObject` and the `Engine` should understand this too.

### Annotating types / objects

There are four options for annotating:

- make a cache for all known types using our `TypeGen.ps1` utility
- use ETS to enhance some types with this feature with default options

    For example, if we define a TempFile type then users will not have to worry about deleting temporary files created by New-TemporaryFile cmdlet at all
- embed this in the cmdlets so that they issue an already annotated object into the pipeline

    For example, the same New-TemporaryFile or Open-Database cmdlet could do the annotation internally so that users have no need do this explicitly
- explicitly annotate like `$tempFile.MakeDisposable(DisposableKind.AtRunspaceClose, { script to remove the temp file })`

Note that there are many APIs that use disposable classes and now developers will have the opportunity to freely write objects of these types in the pipeline without fear of resource leakage.

### Kinds of annotation

Mentioned `DisposableKind` is based on context / scope and could be:

- `Unknown`, it is default for new object. The `Engine` should annoutate the object. The default allow the `Engine` to process old code.
- `Auto`, the `Engine` annoutates based on current context/scope
- `AtGlobal`, dispose at default runspace close
- `AtModuleUnload`
- `AtRunspaceClose`
- `AtCmdletClose/AtPipelineClose`
- `AtScriptBlockClose`, dispose at current script block scope close

Thus, a developer is always able to set a desired behavior.

Moreover, we can always call MakeDisposable() in the script to adjust the behavior if the `Engine` cannot do this automatically.
In this case, we will also need a method to force the mandatory code to execute like `$tempFile.PSDispose()`.
It may also be required if we want to free an expensive resource before an end of the script block.

### Implementtation details

The main difficulty is that this is a generalized solution and users will expect it to work everywhere and as expected.
Thus, it is possible the implementation will be adjusted based on feedback for a long time.

From @lzybkr
> I don't think PSVariable should implement IDisposable - but as an implementation detail, maybe SessionStateScope would.

and
> you need two things - better disposable support and a way to register arbitrary cleanup code - this would be analogous to C# using and finally statements.

## Alternate Proposals and Considerations

Add new `Dispose{}` script block to cmdlets.

This approach:

- only partially solves the problem of releasing up resources
- completely assigns responsibility to an user implying his high qualifications
- requires changes in Parser and tools
