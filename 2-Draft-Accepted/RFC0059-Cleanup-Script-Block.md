---
RFC: '0059'
Author: Joel Sallow
Status: Draft-Accepted
Area: Language
Comments Due: 7/29/2019
---

# Function / Script Cmdlet Disposal

Functions in PowerShell are highly versatile, but don't always have the capabilities to manage resources correctly.
For example, in .NET there are many types which implement `IDisposable`, advising that their `Dispose()` method should be used to clean them up when they're no longer needed.
In addition to these, there are some types which manage connections to remote hosts or databases which commonly include a `Close()` method which is often to be used prior to calling `Dispose()`.
Currently, PowerShell functions do not have an effective way of using these resources in the correct manner.

This is further complicated by the fact that any terminating errors or pipeline stops completely bypass the `end` named block.
As such, it cannot be relied upon for cleanup / disposal steps.

Notably, this includes any time a user includes the pipeline cmdlet `Select-Object` with the `-First` parameter.
Once the desired number of objects has been reached, the cmdlet throws an internal and deliberately hidden exception to halt the pipeline.
This is highly effective, but can lead to undesirable circumstances if essential cleanup steps are then skipped as a result.

Compiled cmdlets do not suffer this issue.
They can simply implement `IDisposable` and place the necessary steps in a `Dispose()` method of their own.
The pipeline processor will correctly handle **cmdlets** that require disposal steps, but currently not **functions** or scriptblocks in general.

## Motivation

As a developer,<br />
I want to be able to ensure I am not leaving resources or connections open when a user utilizes a function I have written.

As an administrator,<br />
I want to be able to ensure that any necessary logging or other necessary administrative tasks cannot be skipped accidentally or deliberately by a user invoking a function in a pipeline and using `Select-Object -First X` to bypass logging which would otherwise take place in the `end` block.

## Specification

A `cleanup {}` function block is required for these use cases.
It is intended to mirror the `Dispose()` member that can be implemented when creating a `PSCmdlet` class in a compiled library that implements `IDisposable`.

`IDisposable` can be implemented by PowerShell classes, but at the time of writing utilizing a PS-based class for a cmdlet is not feasible.
There are outstanding issues with PowerShell classes that complicate the possibility of creating a cmdlet with them, especially when working with modules.

In terms of documentation and recommendations for its use, `Cleanup {}` should be reserved for minimal and absolutely necessary operations, such as critical logging operations and disposal of `IDisposable` resources utilized by the script.

### Execution

The `cleanup {}` block will execute:

1. After `end{}` during normal command operation, and
1. During command disposal that occurs when a terminating exception is thrown during pipeline operations.
    - If the dispose block has already been called (as it is after the `end {}` step), it will simply return immediately.
    - This includes being executed after `StopProcessing()` is called, and not being interrupted/cancelled by `StopProcessing()` calls; if the command is being disposed, it has already completed processing.

### Behaviour

The `cleanup {}` block silently **discards** anything submitted to the success output stream from within it.

Documentation should advise that `cleanup {}` is _not intended_ for any standard output and will completely ignore it.
Additionally, all other PowerShell data streams will be accessible from `cleanup {}`, meaning there is no issue sending data using any of the following streams from within it:

- Error
- Warning
- Verbose
- Debug
- Information

### Implementation Details

- A new `cleanup` keyword, indicating a `NamedBlockAst` similarly to `begin`, `process`, and `end`, which is valid in **all the same contexts** as the existing named block keywords.
  - This includes new token types and appropriate entries in the `Tokenizer`'s keyword tables.
- Additional members and constructors for `ScriptBlockAst` to enable it to be recognised by the parser.
- Additional methods in `DlrScriptCommandProcessor` and `CompiledScriptBlock` classes to handle the block execution.
  - Both types implement or inherit from a type that implements `IDisposable`, and as such the existing `Dispose()` methods can generally be utilised to run this block.
- Implementing `IDisposable` for `PSScriptCmdlet` to handle the invocation for the `cleanup {}` block.
- Changes to the behaviour of command execution in `CommandProcessorBase` to recognise when a function or script cmdlet is being disposed, and dispose of these appropriately via a new `DisposeScriptCommands()` method.
  - Also required with this is an additional layer of `try/finally` during `Dispose()` events as we are opening up the possibility for users to throw terminating errors _during_ a command's disposal, and as such we must ensure that the engine-side disposal routines still execute appropriately.
- `PipelineProcessor` will `Dispose()` commands immediately after their `EndProcessing()` segments complete to ensure resources are disposed as soon as they're no longer needed.
- Changes will be required for `.ForEach{}` and `.Where{}` magic method implementations, which have some custom handling for named blocks that will need to be updated to account for `cleanup {}`.
- Improvements to `PipelineStopper` to be able to properly recognise critical regions of code (like `cleanup {}`) and prevent pipeline stops from being registered until that region of code is exited.
  - Ctrl+C will still be registered, but the pipeline stop will not begin until that critical code region is exited.

#### Error States

Terminating errors can be thrown from a `cleanup {}` block, which will skip the invocation of the rest of that `cleanup {}` block, without preventing other commands' `cleanup {}` blocks from being executed.

#### Ctrl+C Behaviour

While scripts usually terminate automatically after the StopProcessing signal is received, an exception will be made during the `cleanup {}` sequences, and the `PipelineStopper.IsStopping` flag will be temporarily cleared to permit script execution for the purposes of `cleanup {}`.
This should allow us to ensure the block executes correctly during the pipeline stopping sequence.

`cleanup {}` code should run regardless of `Ctrl+C` being pressed, whether before or during its operation, so that authors can ensure critical code can be run even when the user cancels normal processing.

#### Finalizer

As was raised during discussion with the committee, we may need to handle the edge case that a script cmdlet is not properly disposed by the time the finalizer is called.
This _should_ be an impossible state to reach since the PowerShell engine typically has full control over the lifecycle of a command, but since the implementation proposed includes having `PSScriptCmdlet` implement `IDisposable` directly, it's possible its `cleanup {}` block could be invoked on the finalizer thread and subsequently throw an error as there would not be a PowerShell runspace available to execute the code.

Similar to `InvokeWithPipe()` overloads, we can check if the current thread context matches the original context of the command.
If we find that the invocation is happening on the wrong thread, we can raise an event in the original context to have it executed there instead.

#### `Cmdlet` / `PSCmdlet`

Commands inheriting from `Cmdlet` or `PSCmdlet` can already implement a version of this behaviour.
To do so, such commands need also implement `IDisposable` and the `Dispose()` method.
This RFC would simply change when the dispose method is called.
Currently, the method is called during pipeline disposal, after **all** commands have completed their `EndProcessing()` steps.
As part of the changes proposed in this RFC, we would ensure each command's `cleanup {}` or `Dispose()` routines are called immediately after _each_ cmdlet's `EndProcessing()` execution completes during normal pipeline operation.
It will continue to be called in the case of a terminating error or pipeline stop as it is currently.

If this results in multiple calls to the `Dispose()` methods, the `cleanup {}` script will not be invoked after the first call.

### Examples

#### Pinging a Remote System

```powershell
using namespace System.Net.NetworkInformation

function Get-PingReply {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory, ValueFromPipeline)]
        [Alias('IPAddress', 'Destination')]
        [string]
        $Target
    )
    begin {
        $Ping = [Ping]::new()

        # Timeout is in ms
        $Timeout = 2500
        [byte[]] $Buffer = 1..32

        # 128 TTL and avoid fragmentation
        $PingOptions = [PingOptions]::new(128, $true)
    }
    process {
        $Ping.Send($Target, $Timeout, $Buffer, $PingOptions)
    }
    cleanup {
        if ($Ping) {
            $Ping.Dispose()
        }
    }
}

'1.2.3.4', 'www.google.com', '8.8.8.8' | Get-PingReply
```

#### Write to a File Using .NET Streams

```powershell
using namespace System.IO

function Write-File {
    [CmdletBinding(PositionalBinding = $false)]
    param(
        [Parameter(ValueFromPipeline)]
        [string] $InputObject,

        [Parameter(Position = 0, Mandatory)]
        [ValidateNotNullOrEmpty()]
        [string] $Path
    )
    begin {
        $fileStream = [FileStream]::new(
            <# path:   #> $Path,
            <# mode:   #> [FileMode]::Open,
            <# access: #> [FileAccess]::Write,
            <# share:  #> [FileShare]::Read)

        $streamWriter = [StreamWriter]::new($fileStream)
    }
    process {
        if ([string]::IsNullOrWhiteSpace($InputObject)) {
            return
        }

        $streamWriter.WriteLine($InputObject)
    }
    end {
        $StreamWriter.Flush()
    }
    cleanup {
        if ($streamWriter) {
            $streamWriter.Dispose()
        }

        if ($fileStream) {
            $fileStream.Dispose()
        }
    }
}

"Text" | Write-File -Path "C:\tmp.txt"
```

### Considerations

#### Breaking Change

This proposal is a breaking change, as it prevents users from directly calling a function, alias, or command named `cleanup` (since this will now parse as a keyword and not a command name).
However, such commands may still be invoked with the call operator (`& cleanup`).

It also prevents users from defining their own `cleanup` dynamic keyword, though use of dynamic keywords is uncommon, so this is unlikely to become a concern.

#### Behaviour with Ctrl+C During `Cleanup {}`

As PowerShell is primarily an administrative shell, it _typically_ (though not always) respects `Ctrl+C` and halts processing.
Due to the nature and purpose of the `cleanup {}` block, it's not suitable to permit `cleanup {}` blocks to be cancelled during their execution.
Allowing `cleanup {}` blocks to be cancelled may lead to memory leaks and other issues that can potentially arise from preventing proper disposal of `IDisposable` resources.

However, this means that actions taken within `cleanup {}` can potentially be disruptive, and it is the author's responsibility to ensure their `cleanup {}` actions do not indefinitely hang.
Compiled cmdlets may already implement `IDisposable` without restriction to similar effect, it is unlikely we have a pressing need to deviate from this behaviour.
Customer feedback may be necessary to determine if taking action towards this course is necessary.

To give a user appropriate feedback, we could emit a warning message that is displayed immediately when a pipeline stop is requested, before any command starts its dispose routines:

> ```code
> WARNING: Ctrl+C was pressed. Attempting to stop processing.
> ```

This message would be emitted as the first action in a pipeline's `StopProcessing()` implementation.

#### Possible Enhancements for `ForEach-Object`

`ForEachObjectCommand` already implements `IDisposable`, and we could add a `-Cleanup` parameter to maintain the parity between `ForEach-Object` and standard scriptblocks / functions.
If we added such a parameter, it would need to be specified explicitly by name, and not assumed based on number of provided scriptblock parameters as the cmdlet currently does with `begin`/`process`/`end`.

This is not critical to the main `cleanup {}` implementation.
During discussion with the committee, it was raised that it may not be strictly necessary / could be complex to implement correctly from the cmdlet itself.
As such, unless there is a need for it, it may be better to relegate `cleanup {}` functionality to functions and script cmdlets.
If a user wishes to implement a `cleanup {}` functionality, it's likely that the complexity of the function is sufficient to warrant a full function being defined rather than leveraging `ForEach-Object`.

### Possible Alternate Proposals

#### Naming Alternatives

Several different possible names for the block have been discussed; the two that keep coming back are `dispose` and `cleanup`.
`cleanup` is more desirable in some ways from a scripting perspective, as it does not come with any assumed knowledge about the `IDisposable` pattern in .NET.
However, given that the proposed behaviour is intended to be as close to a true `Dispose()` as is possible within the constraints of the PowerShell engine, it appears `dispose` is a little more appropriate. `cleanup` is also a slightly more nebulous and less strictly-defined term at present, although that could be remediated somewhat with documentation.

Additionally, while scripters unfamiliar with `IDisposable` objects would have to learn the new feature regardless of the name, programmers more familiar with the .NET ecosystem and the `IDisposable` pattern would be able to recognise the name and make reasonable assumptions about how it operates and should be used.
Of course, that in turn means that the operation of `cleanup {}` would need to be as close to the "real" `IDisposable` behaviour as possible, and equally as reliable.

#### Forcibly Cancel Processing on Multiple Presses of `Ctrl+C`

It was [suggested](https://github.com/PowerShell/PowerShell-RFC/pull/207#discussion_r393927153) that multiple presses of `Ctrl+C` could be used to forcibly exit disposal routines.

However, cmdlets can already choose to not ignore cancellation requests by:

1. Failing to implement `StopProcessing()` when needed, or
1. Simply call blocking methods during any part of their pipeline operation which inhibit or ignore `StopProcessing()` or `Dispose()` calls from the pipeline handler.

Script functions can also impede `StopProcessing()` calls simply by calling a blocking CLR method.
Given that inhibiting cancellation is not a new issue, it is not critical for the `cleanup {}` implementation to address this specific potential problem.
If it becomes an issue in future, it can be addressed separately.

#### Timeout for `Cleanup {}` Operations

In a similar vein to the above, an alternative may be to implement a hard timeout for command disposal, leaving it up to the runtime garbage collection routines beyond that point.
This could potentially be problematic in the same way as permitting `Ctrl+C` to cancel a dispose block, especially in cases where native interop is being performed either directly or by a type being utilised by a given script command.

As above, given that no such restriction is currently present for `IDisposable` cmdlets, it is not likely to be a pressing need.
We can reevaluate pending customer feedback.

If we later determine this is needed, a `$PSStopProcessingTimeout` preference variable could be created to allow the user to adjust their own timeout preferences in specific time-sensitive scenarios.

#### Reuse the `End {}` Block

This idea was explored briefly before being discarded.
The reason this cannot be the appropriate method is that pipeline behaviour somewhat relies upon there being a skippable `end{}` block.
For example, cmdlets such as `Sort-Object` only send output during `EndProcessing()`, and script functions place everything into the `end{}` block by default when no named blocks are specified.

If the `end {}` block were unskippable for logging purposes, pipeline and command disposal in the case of errors or `Select-Object -First X` would quickly become very sluggish compared to how they are now.
It is quite common for functions to be designed to do much of their processing in the `end {}` block / during `EndProcessing()`.
Even if we prevented output from occurring in that instance as we are with `cleanup {}`, the processing to produce said output would still be occurring.
This would significantly slow down intentional or necessary pipeline stops and reduce the utility of `Select-Object -First X` immensely.
Additionally, such a solution would break many existing commands which rely on the ability to pass output from `end{}` / `EndProcessing()`.
