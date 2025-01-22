---
RFC: RFCNNNN-PipelineStopToken.md
Author: Jordan Borean
Status: Draft
SupercededBy:
Version: 1.0
Area: engine
Comments Due: January 1, 2025
Plan to implement: Yes
---

# PipelineStopToken for PSCmdlet

## Motivation

    As a powershell developer/user,
    I want to be able to call .NET methods but still respond to ctrl+c/stop requests,
    so that my script does not hang or take long to be responsive again.

## User Experience

PowerShell can stopped through the [PowerShell.Stop Method](https://learn.microsoft.com/en-us/dotnet/api/system.management.automation.powershell.stop?view=powershellsdk-7.4.0) which in a REPL environment like the console is triggerd when `CTRL+C` is pressed.
When a stop signal is received the running command will trigger the [Cmdlet.StopProcessing Method](https://learn.microsoft.com/en-us/dotnet/api/system.management.automation.cmdlet.stopprocessing?view=powershellsdk-7.4.0) which gives the cmdlet the ability to cancel whatever action that may be running in the main thread so that the PowerShell engine can stop.
Most of the time no special action is required but one area that is problematic is calling a long running .NET method which blocks the ability for PowerShell to stop processing.

An trivial example of how this is a problem is shown in the below function.
This ignores the fact that `Start-Sleep` will respond to stop requests as it has implemented `StopProcessing`, this example is meant to demonstrate that .NET methods do not respond to the stop requests.

```powershell
Function Test-ScriptStopProcessing {
    [CmdletBinding()]
    param()

    [System.Threading.Tasks.Task]::Delay(10000).GetAwaiter().GetResult()
}
```

If you run `Test-ScriptStopProcessing` then press `CTRL+C`, the pipeline will block for the full 10 seconds before freeing up.
There are three ways to avoid this problem today:

1. Call a PowerShell cmdlet that implements `StopProcessing` properly
2. Implement a compiled cmdlet with your own `StopProcessing` implementation
3. Use a spinlock loop to constantly free up the engine to respond to events

Option 1 is not always possible, not every .NET method has been implemented as a binary cmdlet with `StopProcessing` implemented properly.
Option 2 is only feasible for compiled modules, the vast majority of users would be writing PowerShell script functions and are more familiar with PowerShell itself.
It also requires a lot of boilerplate to be added to properly implement this, for example:

```powershell
Add-Type -TypeDefinition @'
using System;
using System.Management.Automation;
using System.Threading;
using System.Threading.Tasks;

[Cmdlet(VerbsDiagnostic.Test, "CmdletStopProcessing")]
public class TestCmdletStopProcessing : PSCmdlet, IDisposable
{
    private readonly CancellationTokenSource _cancellationTokenSource = new();

    protected override void EndProcessing()
    {
        Task.Delay(10000, _cancellationTokenSource.Token).GetAwaiter().GetResult();
    }
    
    protected override void StopProcessing() => _cancellationTokenSource.Cancel();

    public void Dispose() => _cancellationTokenSource.Dispose();
}
'@ -PassThru | Select-Object -First 1 | ForEach-Object { Import-Module -Assembly $_.Assembly }

Test-CmdletStopProcessing
```

The developer needs to account for:

+ Making the class `IDisposable`
+ Ensuring the cancellation token is disposed
+ Overriding `StopProcessing` and setting it to cancel

Option 3 is only possible if the .NET method exposes a way to start the task and poll the result, for example the `async/await` `Task` pattern can be achieved with.

```powershell
& {
    $task = [System.Threading.Tasks.Task]::Delay(10000)
    while (-not $task.IsCompleted) { Start-Sleep -Milliseconds 300 }
    $task.GetAwaiter().GetResult()
}
```

It won't be immediately apparent why someone would write code like this over just awaiting the task without the loop.

While there are a myriad of patterns and ways of trying to cancel a .NET function the most common ones that are part of the Base Class Library (BCL) is the `CancellationToken` type.
Most tasks add an overload that accepts a `CancellationToken` so exposing a ready to go property of a `CancellationToken` we can use in our functions and cmdlets will cover the majority of ways to cancel a .NET method invocation.

## Specification

To solve this problem we should add a public property to `PSCmdlet` that contains a CancellationToken that is already hooked into the stop processing action:

```csharp
public partial class PSCmdlet
{
    public CancellationToken PipelineStopToken { get; }
}
```

This allows both a compiled cmdlet to just do `Task.Delay(10000, PipelineStopToken);` and not have to worry about all the other setup code required but it also allows advanced script functions to access the same token through `$PSCmdlet.PipelineStopToken`.
For example a compiled cmdlet can now be simplified to:

```powershell
Add-Type -TypeDefinition @'
using System;
using System.Management.Automation;
using System.Threading;
using System.Threading.Tasks;

[Cmdlet(VerbsDiagnostic.Test, "CmdletStopProcessing")]
public class TestCmdletStopProcessing : PSCmdlet
{
    protected override void EndProcessing()
    {
        Task.Delay(10000, PipelineStopToken).GetAwaiter().GetResult();
    }
}
'@ -PassThru | Select-Object -First 1 | ForEach-Object { Import-Module -Assembly $_.Assembly }

Test-CmdletStopProcessing
```

An advanced script function can now be written like:

```powershell
Function Test-ScriptStopProcessing {
    [CmdletBinding()]
    param()

    [System.Threading.Tasks.Task]::Delay(10000, $PSCmdlet.PipelineStopToken).GetAwaiter().GetResult()
}

Test-ScriptStopProcessing
```

There also supports a very limited way of having a function or cmdlet register it's own action to handle cases when an API does not support the `CancellationToken` overload.

For a compiled cmdlet this is easy to achieve by calling the [CancellationToken.Register](https://learn.microsoft.com/en-us/dotnet/api/system.threading.cancellationtoken.register?view=net-9.0#system-threading-cancellationtoken-register(system-action((system-object))-system-object)) method:

```powershell
Add-Type -TypeDefinition @'
using System;
using System.Management.Automation;
using System.Threading;
using System.Threading.Tasks;

[Cmdlet(VerbsDiagnostic.Test, "CmdletStopProcessing")]
public class TestCmdletStopProcessing : PSCmdlet
{
    protected override void EndProcessing()
    {
        using PowerShell ps = PowerShell.Create();
        ps.AddScript("Start-Sleep -Seconds 10");

        PipelineStopToken.Register((p) => ((PowerShell)p).Stop(), ps);
        ps.Invoke();
    }
}
'@ -PassThru | Select-Object -First 1 | ForEach-Object { Import-Module -Assembly $_.Assembly }

Test-CmdletStopProcessing
```

It is also techincally possible to do the same thing with a script function but the registered cancel action must be defined as a .NET delegate as it will run in another thread.

```powershell
Add-Type -TypeDefinition @'
using System;
using System.Management.Automation;

public static class CancellationAction
{
    public static void StopPowerShell(object powershell)
        => ((PowerShell)powershell).Stop();
}
'@

Function Test-ScriptStopProcessing {
    [CmdletBinding()]
    param()

    $ps = [PowerShell]::Create()
    $null = $ps.AddScript('Start-Sleep -Seconds 10')

    $null = $PSCmdlet.PipelineStopToken.Register([CancellationAction]::StopPowerShell, $ps)
    $ps.Invoke()
}

Test-ScriptStopProcessing
```

The fact the delegate must be implemented in a .NET method limits the functionality here but it is still something that was not possible to achieve before.
It is also not the primary use case of adding the `PipelineStopToken` which is to provide an already configured `CancellationToken` to .NET methods that accept it.

## Alternate Proposals and Considerations

Some alternate possibilities are:

+ Expose `-CancellationToken` as a common parameter - https://github.com/PowerShell/PowerShell/issues/19685
+ Expose a `stop` block to advanced functions

The `-CancellationToken` option seems somewhat tangentally related to this RFC which aims to provide an easy to use cancellation token that is hooked into `StopProcessing` for scripts and cmdlet to use.
This may be useful in the future for exposing a way for callers to be able to cancel a PowerShell script or cmdlet but I don't think it fits well into the PowerShell model and there is no easy way to really call the cancel token created by the caller.
By integrating the cancellation token as part of the `PSCmdlet` instance we have a token that's fully managed by PowerShell, scoped to the command being run, and is automatically triggered for the specific scenario when the engine is set to stop.

The `stop` block could look something like

```powershell
Function Test-Function {
    [CmdletBinding()]
    param()

    begin {
        $cancelSource = [System.Threading.CancellationTokenSource]::new()
    }
    end {
        [System.Threading.Tasks.Task]::Delay(10000, $cancelSource.Token).GetAwaiter().GetResult()
    }
    stop {
        $cancelSource.Cancel()
    }
}
```

While this is flexible and allows the caller to do whatever they want with `stop` it has a few drawbacks:

+ Would also need an explicit `begin`/`process`/`end` block for the main bit of code, it cannot be used by the implicit `end` block example
+ The `stop` call is run in another thread, trying to sync the variables across is either impossible or very expensive
+ The `stop` branch may be blocking itself, the `PipelineStopToken` method should not block the stop call and is very limited in scope
+ More Ast changes needed to implement this new block
+ The caller needs to deal with the setup of the `CancellationTokenSource` instance
+ This does not help the compile cmdlet simplification achieved by the `PipelineStopToken` approach

Putting aside the question of whether it will be possible to share the variable state across Runspace instances to actually respond to the stop event, I think this approach is just too flexible and more complex to implement than the proposed method in the RFC.
