---
RFC: '0059'
Author: Joel Sallow, Dongbo Wang
Status: Draft-Accepted
Area: Language
Comments Due: 8/15/2021
---

# Resource cleanup for function and script cmdlet

Functions in PowerShell are highly versatile, but don't always have the capabilities to manage resources correctly.
For example, in .NET there are many types which implement `IDisposable`, advising that their `Dispose()` method should be used to clean them up when they're no longer needed.
In addition to these, there are some types which manage connections to remote hosts or databases which commonly include a `Close()` method which is often to be used prior to calling `Dispose()`.
Currently, PowerShell functions do not have an effective way of using these resources in the correct manner.

> **Note:** `IDisposable` can be implemented by PowerShell class, but at the time of writing utilizing a PS-based class for a cmdlet is not feasible.
There are outstanding issues with PowerShell class that complicate the possibility of creating a cmdlet with them, especially when working with modules.

This is further complicated by the fact that any terminating errors or pipeline stops could completely bypass the `end` named block.
As such, it cannot be relied upon for cleanup / disposal steps.

Notably, this includes any time a pipeline contains the cmdlet `Select-Object` with the `-First` parameter.
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

A `clean {}` named block is required for these use cases. Examples:

```powershell
using namespace System.Net.NetworkInformation

## Pinging a remote system
function Get-PingReply {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory, ValueFromPipeline)]
        [Alias('IPAddress', 'Destination')]
        [string] $Target
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
    clean {
        if ($Ping) { $Ping.Dispose() }
    }
}

'1.2.3.4', 'www.google.com', '8.8.8.8' | Get-PingReply
```

```powershell
using namespace System.IO

## Write to a file using .NET streams
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
        $fileStream = [FileStream]::new($Path, [FileMode]::Open, [FileAccess]::Write, [FileShare]::Read)
        $streamWriter = [StreamWriter]::new($fileStream)
    }
    process {
        if ([string]::IsNullOrWhiteSpace($InputObject)) { return }
        $streamWriter.WriteLine($InputObject)
    }
    end {
        $StreamWriter.Flush()
    }
    clean {
        if ($streamWriter) { $streamWriter.Dispose() }
        if ($fileStream) { $fileStream.Dispose() }
    }
}

"Text" | Write-File -Path "C:\tmp.txt"
```

The `clean` block is intended to be a convenient way for users to clean up resources that span across the `begin`, `process`, and `end` blocks.
It's supposed to be semantically similar to a `finally` block that covers all other named blocks of a script function or a script cmdlet,
so that the resource cleanup can be enforced in all the following scenarios:

1. when the pipeline execution finishes normally without terminating error;
2. when the pipeline execution is interrupted due to terminating error;
3. when the pipeline is halted by `Select-Object -First`;
4. when the pipeline is being stopped by <kbd>Ctrl+c</kbd> or `StopProcessing()`.

> **Warning:** adding the `clean` block is a breaking change,
> as it prevents users from directly calling a command named `clean`
> (since this will now be parsed as a keyword instead of a command name).
> However, such commands may still be invoked with the call operator (`& clean`).
> It also prevents users from defining their own `clean` dynamic keyword,
> though the use of dynamic keywords is uncommon, so this is unlikely to become a concern.

For a pipeline like `a | b | c`, the execution will look as follows:

```none
   start: a-begin, b-begin, c-begin
  inject: a-process
complete: a-end, b-end, c-end
   clean: a-clean, b-clean, c-clean
```

> **Note:** in `start`, `inject` and `complete` phases, any output to the pipeline will trigger the `process` block of the downstream command to run.

Comparing to the current pipeline processing, a new phase `clean` is added,
where the `clean` blocks from each command in the pipeline will run one by one,
similarly to how the `end` blocks execute.

The functional behaviors of the `clean` block are:

1. The `clean {}` block runs only if at least one of the other named blocks of the command gets executed.
1. The `clean {}` block runs in the same scope as the `begin`, `process`, and `end` blocks of the command.
1. The `clean {}` block runs when:
   - The pipeline execution finishes normally without terminating error.
   - The pipeline execution is interrupted due to terminating error
   - The pipeline is halted by `Select-Object -First`
   - The pipeline is being stopped by <kbd>Ctrl+c</kbd> or `PowerShell.Stop/StopAsync`
     - See the [Pipeline stopping behavior](#pipeline-stopping-behavior) section for more discussions.
1. The `clean {}` block silently **discards** any data written to the output stream.
   - `-OutVariable` cannot capture output from `clean {}` block.
1. The `clean {}` block supports writting to all other streams from within it.
   - Stream redirection works for all other streams.
   - `-WarningVariable` and `-InformationVariable` can capture warning and information output from the `clean {}` block.
1. The `clean {}` block does not propagate up any exception.
   - It does not override the terminating exception thrown from another named block.
   - Exception caught in `clean {}` block will be wrapped into an `ErrorRecord` and written to the error stream.
   - See the [Error Handling](#error-handling) section for the detailed error handling behaviors. 
1. The `clean {}` block follows the same semantics in a `SteppablePipeline`.

In terms of documentation and recommendations for its use,
`clean {}` should be reserved for minimal and absolutely necessary operations,
such as critical logging operations and disposal of `IDisposable` resources utilized by the script.

## Error Handling

Today, there are three kinds of errors in PowerShell:

1. Terminating error -- error that will be bubbled up and stop the execution of the rest script/function.
   - The error triggered by `ThrowTerminatingError`
   - The error triggered by `-ErrorAction Stop`
   - Exception stemmed from `throw` statement
      - Note that, the `throw` exception is special, in that it can be suppressed by `-ErrorAction SilentlyContinue/Ignore`
      ```powershell
      PS:1> function iii { [cmdletbinding()]param() throw 'aaa';  gcm blah -ErrorAction Stop }
      PS:2> iii
      Exception: aaa
      PS:3> iii -ErrorAction SilentlyContinue
      Get-Command: The term 'blah' is not recognized as a name of a cmdlet, function, script file, or executable program.
      ```
2. Non-terminating error -- error that doesn't stop the script/function execution.
   - The error triggered by `WriteError` (C#) or `Write-Error` (script)
3. General exception
   - Exception thrown from a .NET method invocation
   - Exception from expression like `1/0`
   - It honors the error `ActionPreference` setting
   - By default, whether this kind of exception will stop the script/function execution depends on whether the code that throws this exception is enclosed by `try/catch`, `try/finally`, or accompanied by a `trap` statement.
   ```powershell
   PS:1> function iii { 1/0; 1+2 }
   PS:2> iii
   RuntimeException: Attempted to divide by zero.
   3
   PS:3> try { iii } catch { 'catch' }
   catch
   ```

For the `finally` block, the error handling works the same with one exception:
A terminating error thrown from `finally` will override the terminating error thrown from `catch`.
This is the same behavior as in C#.

The `Clean` block is different from `finally` under the context of a pipeline.
Say we have the pipeline `"a | b | c"`, and each command have a `Clean` block implementation.
Then all three `Clean` blocks may need to run at the end of this pipeline execution.
So, what if they all throw terminating errors? How do we surface those errors to the user?

### Error handling proposal for 'Clean' block

Today, at the pipeline level, only the first terminating error is bubbled up.
This makes total sense, because the pipeline execution will stop at the first terminating error.
Therefore, if the pipeline is enclosed in a `try/catch`, then the first terminating error will be the one caught by the `catch` block.

With `Clean` block in the picture, it's now changed: the first terminating error doesn't completely stop the pipeline execution
because `Clean` blocks for commands in the pipeline will run.
So, allowing the `Clean` block to propagate exception (terminating error) means we may end up with multiple terminating errors.
What do we do with them? Which one to actually propagate further up? What about the rest? How do we present the rest terminating errors to the user?

> NOTE: today, any other terminating error that happen after the first terminating error
> (maybe an exception thrown from the `Dispose` implementation of a command, which is rare), are simply logged.
> Only the first terminating error is propagated up and eventually rendered in the console.

With all these considered, I think it makes most sense that **the `Clean` block never propagates up any exceptions**.
- All exceptions should be caught in the pipeline when executing the `Clean` block.
- Except for the internal ones like the `FlowControlException`,
  all other exception should be written to the current command's `ErrorOutput` pipe,
  so that they can be presented to the user.

    Example:
    ```powershell
    PS:1> function a { process { 'a-process' } clean { throw 'a-clean' } }
    PS:2> function b { process { throw 'b-process-throw' } clean { throw 'b-clean' } }
    ## Terminating error from 'Clean' blocks are written to error pipe
    ## The 'b-process-throw' thrown from b's 'Process' will be bubbled all the way up to
    ## the 'Executor' of console host, which will then create another pipeline to write
    ## to console. That's why it shows up in the end.
    PS:3> a | b
    Exception: a-clean
    Exception: b-clean
    Exception: b-process-throw

    ## It's more clear here that only 'b-process-throw' is bubbled up
    PS:4> try { a | b } catch { "caught: $_" }
    Exception: a-clean
    Exception: b-clean
    caught: b-process-throw
    ```

    Another example using PowerShell API:
    ```powershell
    ## Exception 'b-process-throw' will be bubbled all the way up, thrown from the method invocation
    PS:1> $ps = [powershell]::Create()
    PS:2> $ps.AddScript(@"
        function a { process { 'a-process' } clean { throw 'a-clean' } }
        function b { process { Write-Host 'b'; throw 'b-process-throw' } clean { throw 'b-clean' } }
        a | b
    "@).Invoke()
    MethodInvocationException: Exception calling "Invoke" with "0" argument(s): "b-process-throw"

    ## Terminating error from 'Clean' blocks were written to error pipe.
    PS:3> $ps.Streams.Error | % ToString
    a-clean
    b-clean

    ## 'b-process-throw' is bubbled up and will be caught.
    PS:4> $ps.Commands.Clear(); $ps.Streams.ClearStreams()
    PS:5> $ps.AddScript('try { a | b } catch { "caught: $_" }').Invoke()
    caught: b-process-throw
    PS:6> $ps.Streams.Error | % ToString
    a-clean
    b-clean
    ```

**Within the `Clean` block, terminating error and non-terminating errors should work just like how they work today**:
- Terminating error stops the `Clean` block execution
- Non-terminating errors act based on `$ErrorActionPreference`

    Example:
    ```powershell
    PS:1> function a { process { 'a-process' } clean { Write-Host 'a-before-throw'; throw 'a-clean-throw'; Write-Host 'a-after-throw' } }
    PS:2> function b { process { throw 'b-process-throw' } clean { Resolve-Path $null; Write-Host 'b-after-error' } }
    ## a's Clean block was stopped by 'throw';
    ## b's Clean block runs to the end by default.
    PS:3> a | b
    a-before-throw
    Exception: a-clean-throw
    Resolve-Path: Cannot bind argument to parameter 'Path' because it is null.
    b-after-error
    Exception: b-process-throw

    ## Non-terminating error in b's Clean will turn to terminating
    PS:4> $ErrorActionPreference = 'stop'
    PS:5> a | b
    a-before-throw
    Exception: a-clean-throw
    Resolve-Path: Cannot bind argument to parameter 'Path' because it is null.
    Exception: b-process-throw

    ## Exception by 'throw' statement will be suppressed
    PS:6> $ErrorActionPreference = 'ignore'
    PS:7> a | b
    a-before-throw
    a-after-throw
    b-after-error
    ```

For general exceptions, as mentioned above, they are by default non-terminating (subject to `$ErrorActionPreference`),
but will be bubbled up (_turned into terminating error_) if it's enclosed in `try/catch/finally`, or has `trap` statement.
- **This behavior should not change within the `Clean` block**.

    Example:
    ```powershell
    ## Behavior with 'try' statement in the 'End' block
    PS:1> function iii { end { try { 1/0; Write-Host 'end' } catch { Write-Host "caught: $_" } } }
    PS:2> iii
    caught: Attempted to divide by zero.

    ## Same behavior when moving the 'try' statement to 'Clean' block
    PS:3> function iii { end {} clean { try { 1/0; Write-Host 'clean' } catch { Write-Host "caught: $_" } } }
    PS:4> iii
    caught: Attempted to divide by zero.
    ```

    ```powershell
    ## Behavior with 'trap' statement in the 'End' block
    PS:1> function a { end { trap { Write-Host "caught: $_" } 1/0; Write-Host 'end' } }
    PS:2> a
    caught: Attempted to divide by zero.
    RuntimeException: Attempted to divide by zero.
    end

    ## Same behavior when moving the 'trap' statement to 'Clean' block
    PS:3> function b { end {} clean { trap { Write-Host "caught: $_" } 1/0; Write-Host 'clean' } }
    PS:4> b
    caught: Attempted to divide by zero.
    RuntimeException: Attempted to divide by zero.
    clean

    ## Overall, the 'trap' behavior is weird and less consistent comparing to 'try'.
    ## That's the existing behavior, so we just keep the consistency.
    ```

- **The 'try/catch/finally' and 'trap' outside a command should not turn a general exception that happens in the command's 'Clean' block to terminating error**.

    Given that 'Clean' block never propagate up any exception, the outer `try/catch` and `trap` won't be triggered by the general exception happens in `Clean` block anyway.
    So, I think it's probably better to make the general exception behavior consistent regardless of whether the command is enclosed in `try/catch` or `trap`.

    Example:
    ```powershell
    ## A general exception in 'End' block is non-terminating by default
    PS:1> function a { end { 1/0; Write-Host 'end' } }
    PS:2> a
    RuntimeException: Attempted to divide by zero.
    end
    ## The general exception gets turned into a terminating error by outer 'try' statement
    PS:3> try { a } catch { 'outer catch' }
    outer catch
    ## The general exception gets turned into a terminating error by outer 'trap' statement
    PS:4> & { trap { 'outer trap' } a }
    outer trap
    RuntimeException: Attempted to divide by zero.

    ## A general exception in 'Clean' block is non-terminating by default
    PS:5> function b { end {} clean { 1/0; Write-Host 'clean' } }
    PS:6> b
    RuntimeException: Attempted to divide by zero.
    clean
    ## Note that, outer 'try/trap' doesn't affect the general exception happens in 'Clean' block.
    ## so its behavior is consistent regardless of whether the command is enclosed by 'try/catch' or not.
    PS:7> try { b } catch { 'outer catch' }
    RuntimeException: Attempted to divide by zero.
    clean
    ## Its behavior is consistent regardless of whether the command is accompanied by 'trap' or not.
    PS:8> & { trap { 'outer trap' } b }
    RuntimeException: Attempted to divide by zero.
    clean
    ```

**`-ErrorVariable` should work for 'Clean' block, but `-OutVariable` should not**
- The 'Clean' block shall write nothing to the output pipe, so `-OutVariable` should not capture any output.
- The 'Clean' block shall write other streams to the corresponding pipes, so `-ErrorVariable` should work as expected (same to `-WarningVariable` and `-InformationVariable`).

    Example:
    ```powershell
    ## The 'Clean' block writes to output, error, warning, and information streams.
    PS:1> function a { [cmdletbinding()]param() end {} clean { 1/0; 'output'; Write-Warning 'warning'; Write-Information 'information' } }
    ## Capture all streams using the stream variables, also honor `ActionPreference` setting 
    PS:2> a -ErrorVariable err -OutVariable out -WarningVariable warn -InformationVariable info -ErrorAction SilentlyContinue -WarningAction SilentlyContinue
    ## Error, warning, and information variables captured the records as expected.
    PS:3> $err; $warn; $info
    RuntimeException: Attempted to divide by zero.
    warning
    information
    ## Out variable doesn't capture anything.
    PS:4> $out
    PS:5>
    ```

- I want to bring up the `dynamicparam` block here.
  Similarly, it doesn't write to the command's output pipe, but it does write to other streams of the command.
  However, all stream variables, such as `-ErrorVariable` and `-WarningVariable`, don't work for the `dynamicparam` block (see the below example).
  This seems to be inconsistent with the proposed `Clean` behavior right above,
  but I'm not concerned by this inconsistency because the `dynamicparam` is not really a peer of `Begin/Process/End`.
  It's special, and only used in parameter binding.

    ```powershell
    PS:1> function a { [cmdletbinding()]param() dynamicparam { 1/0; 'output'; Write-Warning 'warning'; Write-Information 'information' } end {} }
    ## Stream variables take no effect, and `ErrorAction/WarngingAction` take no effect too.
    PS:2> a -ErrorVariable err -OutVariable out -WarningVariable warn -InformationVariable info -ErrorAction SilentlyContinue -WarningAction SilentlyContinue
    RuntimeException: Attempted to divide by zero.
    WARNING: warning
    PS:3> $err; $warn; $info; $out
    PS:4>
    ```

**`Clean` block should stick to the rest of error handling as in other named blocks today**
- `ErrorAction` setting should be honored in `Clean` block
- Terminating error happened in `Clean` should set `$?` to false for the command
- Error handling for `Clean` should work consistently in PowerShell API

    Example:
    ```powershell
    ## 'throw' from 'Clean' block will mark '$?' to 'false'.
    PS:1> function a { [cmdletbinding()]param() end {} clean { throw 'throw-in-clean'; Write-Host 'clean' } }
    PS:2> function b { [cmdletbinding()]param() throw 'throw-in-end'; Write-Host 'end' }

    PS:3> a; $?  ## exception is not bubbled up from 'Clean', and thus `$?` gets evaluated.
    Exception: throw-in-clean
    False
    PS:4> b; $?  ## see the difference? it's because the exception is bubbled up from 'End', so `$?` doesn't get evaluated.
    Exception: throw-in-end

    ## 'throw' can be suppressed by 'SilentlyContinue', same behavior as in other named blocks today.
    PS:5> a -ErrorAction SilentlyContinue; $?
    clean
    True
    PS:6> $Error[0]
    Exception: throw-in-clean

    ## 'throw' is suppressed by 'SilentlyContinue' in 'End' block as well.
    PS:7> $Error.Clear()
    PS:8> b -ErrorAction SilentlyContinue; $?
    end
    True
    PS:280> $Error[0]
    Exception: throw-in-end
    ```

    ```powershell
    ## Use a true terminating exception in instead of 'throw', and try it out in both 'Clean' and 'End'.
    PS:1> function a { [cmdletbinding()]param() end {} clean { gcm blah -ErrorAction Stop; Write-Host 'clean' } }
    PS:2> function b { [cmdletbinding()]param() gcm blah -ErrorAction Stop; Write-Host 'end' }

    PS:3> a; $?  ## exception is not bubbled up from 'Clean', and thus `$?` gets evaluated.
    Get-Command: The term 'blah' is not recognized as a name of a cmdlet, function, script file, or executable program.
    Check the spelling of the name, or if a path was included, verify that the path is correct and try again.
    False
    PS:4> b; $?  ## exception is bubbled up from 'End', so `$?` doesn't get evaluated.
    Get-Command: The term 'blah' is not recognized as a name of a cmdlet, function, script file, or executable program.
    Check the spelling of the name, or if a path was included, verify that the path is correct and try again.

    ## 'SilentlyContinue' cannot suppress a terminating error thrown from 'ThrowTerminatingError' or '-ErrorAction Stop'.
    ## But again, exception is not bubbled up from 'Clean', and thus `$?` gets evaluated.
    PS:5> a -ErrorAction SilentlyContinue; $?
    Get-Command: The term 'blah' is not recognized as a name of a cmdlet, function, script file, or executable program.
    Check the spelling of the name, or if a path was included, verify that the path is correct and try again.
    False
    ## exception is bubbled up from 'End', so `$?` doesn't get evaluated.
    PS:6> b -ErrorAction SilentlyContinue; $?
    Get-Command: The term 'blah' is not recognized as a name of a cmdlet, function, script file, or executable program.
    Check the spelling of the name, or if a path was included, verify that the path is correct and try again.
    PS:7>
    ```

### Error handling proposal summary

The most important point of the proposal is that:
> **the `Clean` block never propagates up any exceptions**

All other parts of the proposal basically try to have the error handling for `Clean` block make sense under this design decision,
and also be consistent with the existing error handling as much as possible.

I think it's easy to communicate this design decision to our users:
> The `Clean` block is for resource clean-up only and is pretty self-contained.
> It won't propagate any terminating exception up, so as to not disrupt the script execution outside this `Clean` block.

## Pipeline Stopping Behavior

Today, the way how the `finally` clause behaves in case of pipeline stopping is inconsitent:
- If the pipeline stopping happens in the `try` clause, then the corresponding `finally` clause is guaranteed to run by suspending the stopping pipeline.
- If the pipeline stopping happens in the `finally` clause, then the `finally` clause will be terminated immediately.

Examples:
```powershell
## Press 'Ctrl+c' in 2 seconds
PS> try { Write-Host 'in try'; Start-Sleep -Seconds 10 } finally { Write-Host 'in finally' }
try
finally
PS>
```
```powershell
## Press 'Ctrl+c' in 2 seconds
PS> try { Write-Host 'try'; } finally { Start-Sleep -Seconds 10; Write-Host 'finally' }
try
PS>
```

As is shown, the `finally` clause does not reliably execute in the event of pipeline stopping.
So, when it comes to the `clean` block, we have a debate on two different behaviors:

1. Mimic the current behavior of the `finally` clause.
2. The `clean` block always runs to the end regardless of pipeline stopping, no matter it happens before or during its operation.

The pros and cons of each behavior is discussed below.

### Mimic `finally`

- Pros: consistent with the current behavior of `finally` clause.
- Cons: execution of the `clean` block is unreliable when user cancels the pipeline processing.

As already stated above, the current behavior of the `finally` clause in the event of pipeline stopping is inconsistent,
and it may be found surprising.
However, `finally` has been working in this way since it was introduced in PowerShell v3,
and as far as I know we never heard any complaints about this behavior yet.
So, it's hard to say this inconsistent behavior is really a problem practically,
to both the `finally` clause and the `clean` block.

### Reliably run to the end

- Pros: execution of the `clean` block is reliable when user cancels the pipeline processing.
- Cons: inconsistent with `finally`, and concerns about long running `clean` impacts the responsiveness to interactive users.

Due to the nature and purpose of the `clean` block, it may not be suitable to permit `clean` blocks to be cancelled during their execution.
Allowing `clean` blocks to be cancelled may lead to memory leaks and other issues that can potentially arise from preventing proper disposal of `IDisposable` resources.

However, this means that actions taken within a `clean` block can potentially be disruptive,
because a long running `clean` block cannot be cancelled and thus it would look like a hang to an interactive user.
So it would be the author's responsibility to ensure their `clean` actions finish fast.

A few ideas were brought up on how to mitigate a non-cancellable long running `clean` block.

1. Emit a warning message to give a user appropriate feedback.<br />
   A warning message can be displayed immediately when a pipeline stop is requested:
   ```code
   WARNING: Ctrl+C was pressed. Attempting to stop processing.
   ```
   This message would be emitted as the first action in a pipeline's `StopProcessing()` implementation.

2. Forcibly cancel processing on multiple presses of <kbd>Ctrl+c</kbd>, or multiple calls to `PowerShell.Stop/StopAsync`.<br />
   It was suggested in [this review comment](https://github.com/PowerShell/PowerShell-RFC/pull/207#discussion_r393927153).

3. Timeout for `Clean {}` Operations.<br />
   We could implement a hard timeout for `clean` block.
   Or alternatively, a `$PSStopProcessingTimeout` preference variable could be created to allow the user to adjust their own timeout preferences in specific time-sensitive scenarios.
   However, this could potentially be problematic in the same way as permitting pipeline stopping to cancel a `clean` block.

There are also arguments questioning the needs to concern about the potential hang issue:

1. Binary cmdlet can already choose to ignore cancellation requests by either not implementing `StopProcessing()` or calling blocking methods.
2. Script functions can also impede cancellation by calling a blocking .NET method.

Given that inhibiting cancellation is not a new issue, it may not be critical for the `clean` block implementation to address this specific potential problem.
But it's also recognized that the `clean` block would make this potential problem way easier to happen.

Being inconsistent with `finally` in how pipeline stopping is handled would be another concern.
Since the `clean` block is semantically similar to a `finally` clause,
their behaviors during pipeline stopping should be the same.
Therefore, making the `clean` block non-cancellable would require changing the `finally` clause to be non-cancellable as well,
which would be a breaking change that needs to be carefully reviewed.

## Other Considerations

### Possible Enhancements for `ForEach-Object` and `.ForEach` magic method

For `ForEach-Object`, we could add a `-Clean` parameter to maintain the parity.
If we added such a parameter, it would need to be specified explicitly by name, and not assumed based on number of provided scriptblock parameters as the cmdlet currently does with `begin`/`process`/`end`.

This is not critical to the main `clean` block implementation.
During discussion with the committee, it was raised that it may not be strictly necessary / could be complex to implement correctly from the cmdlet itself.
As such, unless there is a need for it, it may be better to relegate the `clean` block functionality to functions and script cmdlets.
If a user wishes to implement a `clean {}` functionality,
it's likely that the complexity is sufficient to warrant a full function being defined rather than leveraging `ForEach-Object`.

Similarly, as a mapping of the `ForEach-Object`,
the `.ForEach` magic method invokes the `begin`, `process`, and `end` blocks of a passed-in script block.
Ideally, it should be able to handle the `clean` block as well,
but again, this is not critical to the main `clean` block implementation.
Also, it may be desired to keep the parity between `ForEach-Object` and `.ForEach` method,
so both should be updated to support the `clean` block if it's proven to be needed.

## Possible Alternate Proposals

### Reuse the `End {}` Block

This idea was explored briefly before being discarded.
The reason this cannot be the appropriate method is that pipeline behavior somewhat relies upon there being a skippable `end {}` block.
For example, cmdlets such as `Sort-Object` only send output during `EndProcessing()`, and script functions place everything into the `end {}` block by default when no named blocks are specified.

If the `end {}` block were unskippable for logging purposes, pipeline and command disposal in the case of errors or `Select-Object -First X` would quickly become very sluggish compared to how they are now.
It is quite common for functions to be designed to do much of their processing in the `end {}` block / during `EndProcessing()`.
Even if we prevented output from occurring in that instance as we are with `clean {}`, the processing to produce said output would still be occurring.
This would significantly slow down intentional or necessary pipeline stops and reduce the utility of `Select-Object -First X` immensely.
Additionally, such a solution would break many existing commands which rely on the ability to pass output from `end {}` / `EndProcessing()`.
