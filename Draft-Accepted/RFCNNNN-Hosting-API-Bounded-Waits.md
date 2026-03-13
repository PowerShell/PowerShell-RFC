---
RFC: RFCNNNN
Author: Taha Ahmad
Status: Draft
SupercededBy: N/A
Version: 1.0
Area: Engine
Comments Due: 2026-05-12
Plan to implement: Yes
---

# Bounded-Wait Timeout Support for PowerShell Hosting API

This RFC proposes adding two public API members to the PowerShell hosting API
and adding bounded alternatives to the critical unbounded `WaitOne()`/`Wait()` calls in the pipeline path
with bounded alternatives. The changes are fully opt-in and backwards
compatible.

## Motivation

    As a developer hosting PowerShell in a C# application,
    I can set a timeout on PowerShell.Invoke() and PowerShell.Stop(),
    so that a runaway or deadlocked script cannot hang my host process
    indefinitely.

The PowerShell hosting API (`System.Management.Automation.PowerShell`) is used
by VS Code's PowerShell extension, Azure Functions, Azure Automation, Jupyter
notebooks, and thousands of custom C# applications. When a script hangs or
deadlocks, every `WaitOne()` call in the pipeline path blocks indefinitely.
The host has no way to recover short of killing its own process.

Since `Thread.Abort` does not exist in .NET Core, there is currently no
mechanism for a hosting application to impose a deadline on script execution
through the public API.

### Real-world scenarios

| Consumer | Problem today |
|----------|--------------|
| VS Code PowerShell Extension | Must `Process.Kill()` when the integrated console hangs |
| Azure Functions | Stuck scripts hold RunspacePool slots forever, requiring app pool recycle |
| Custom C# hosts (CI/CD, management tools) | No timeout mechanism; stuck invocations require process restart |
| Jupyter / Polyglot Notebooks | Hung cell execution means killing the entire kernel |

## User Experience

### Setting a timeout on Invoke

```csharp
using var rs = RunspaceFactory.CreateRunspace();
rs.Open();

using var ps = PowerShell.Create();
ps.Runspace = rs;
ps.AddScript("Start-Sleep -Seconds 300");

var settings = new PSInvocationSettings
{
    Timeout = TimeSpan.FromSeconds(10)
};

try
{
    // Returns normally if the script completes within 10 seconds.
    // Throws TimeoutException if the deadline elapses.
    Collection<PSObject> results = ps.Invoke(null, settings);
}
catch (TimeoutException ex)
{
    Console.WriteLine($"Script timed out: {ex.Message}");
    // The runspace is still usable for subsequent invocations.
}
```

```output
Script timed out: The operation did not complete within the allotted time of 00:00:10.
```

### Bounded Stop

```csharp
ps.AddScript("Start-Sleep -Seconds 300");
ps.BeginInvoke();

try
{
    ps.Stop(TimeSpan.FromSeconds(5));
}
catch (TimeoutException)
{
    Console.WriteLine("Stop did not complete in 5 seconds.");
}
```

### Default behavior (unchanged)

```csharp
// No Timeout set — default is InfiniteTimeSpan.
// Behaves identically to previous versions.
var settings = new PSInvocationSettings();
ps.Invoke(null, settings);  // Same-thread, no Task.Run, no overhead.
```

## Specification

### New public API surface

#### `PSInvocationSettings.Timeout` property

```csharp
/// <summary>
/// Maximum time to wait for synchronous operations (Invoke, Stop, Close).
/// Default is Timeout.InfiniteTimeSpan which preserves backwards compatibility.
/// </summary>
public TimeSpan Timeout { get; set; } = System.Threading.Timeout.InfiniteTimeSpan;
```

| Attribute | Value |
|-----------|-------|
| Type | `System.TimeSpan` |
| Default | `System.Threading.Timeout.InfiniteTimeSpan` |
| Valid range | Any `TimeSpan >= TimeSpan.Zero`, or `InfiniteTimeSpan` |
| Exception on set | None |

#### `PowerShell.Stop(TimeSpan)` overload

```csharp
/// <summary>
/// Stop the currently running command with a bounded timeout.
/// </summary>
/// <param name="timeout">Maximum time to wait for stop to complete.</param>
/// <exception cref="TimeoutException">Thrown if stop does not complete within timeout.</exception>
public void Stop(TimeSpan timeout);
```

- Initiates stop via `CoreStop(true, null, null)` and waits on the returned
  `AsyncWaitHandle` with a bounded timeout
- Waits on `asyncResult.AsyncWaitHandle.WaitOne(timeout)`
- Throws `TimeoutException` if the deadline elapses
- Swallows `ObjectDisposedException` (calling after `Dispose()` is safe)

### Invoke with finite Timeout (single Runspace path)

When `settings.Timeout` is finite and the PowerShell instance is bound to a
single `Runspace` (not a `RunspacePool`):

1. The invocation is dispatched to a `ThreadPool` thread via `Task.Run`.
2. The calling thread joins with `invokeTask.Wait(timeout)`.
3. **If the task completes:** results are returned normally. Any exception from
   the invocation is unwrapped from `AggregateException` and rethrown as-is.
4. **If the deadline elapses:** `CoreStop(true, null, null)` is called
   (best-effort pipeline stop), then `TimeoutException` is thrown.

When `settings.Timeout` is `InfiniteTimeSpan` (default), the invocation runs
directly on the calling thread with zero overhead -- identical to existing
behavior.

### Invoke with finite Timeout (RunspacePool path)

When the PowerShell instance uses a `RunspacePool`:

1. `pool.BeginGetRunspace(null, null)` is called.
2. `AsyncWaitHandle.WaitOne(settings.Timeout)` bounds the pool slot wait.
3. If the slot is not available in time: `TimeoutException` is thrown.

### Internal bounded waits (not public API)

These are correctness fixes that prevent indefinite hangs during cleanup:

| Location | What changed | Bound |
|----------|-------------|-------|
| `ConnectionBase.CoreClose` | Runspace-opening event wait | 30 s |
| `ConnectionBase.StopPipelines` | Now parallel (`Task.Run` + `Task.WaitAll`); per-pipeline bound via `LocalPipeline.Stop` | per-pipeline (30 s) |
| `LocalConnection.Close/Dispose` | Job and close waits | 30 s |
| `LocalConnection.Dispose` | Catches `TimeoutException`, forces `Broken` state | -- |
| `LocalPipeline.Stop` | `PipelineFinishedEvent.WaitOne` | 30 s |

### STA COM caveat

When `Timeout` is finite, `Invoke()` uses `Task.Run` which dispatches to an
MTA thread. Scripts that depend on STA COM apartment state (legacy ActiveX,
Windows Forms `ShowDialog`) must leave `Timeout` at `InfiniteTimeSpan`.

### Runspace reuse after timeout

After a `TimeoutException` from `Invoke()`, the `Runspace` remains usable. The
caller should allow a brief moment for the stopped pipeline to drain before
starting a new invocation (poll `InvocationStateInfo.State` or use a short
`Thread.Sleep`).

## Alternate Proposals and Considerations

### CancellationToken-based API

A `CancellationToken` parameter on `Invoke()` would be more idiomatic for
modern .NET async patterns. However:

- `Invoke()` is synchronous. A `CancellationToken` parameter on a sync method
  is unusual and the cancellation semantics (cooperative vs. forced) would need
  careful design.
- `Task.Run` + `Wait(timeout)` + `CoreStop` provides deterministic forced
  shutdown, which is what hosting applications need.
- A `CancellationToken`-based API can be added in a future RFC without
  conflicting with this design.

### Configurable internal caps

The 30-second internal caps on `LocalPipeline.Stop()`, `StopPipelines()`, and
`LocalConnection.Close()` are hardcoded. A system-wide or per-runspace cap
setting could be added in a follow-up RFC. The 30-second default was chosen as
a conservative bound that is generous enough for any well-behaved pipeline but
prevents indefinite hangs.

### Experimental feature gating

The `[Experimental]` attribute system targets cmdlet parameters and visibility
toggles. `PSInvocationSettings.Timeout` is a plain property on a POCO class,
not a cmdlet parameter. Runtime gating (`if (!IsEnabled) throw`) is possible
but adds friction for a feature that is already opt-in by design. If the
Committee prefers experimental gating, we can add a `PSHostingAPITimeout`
feature flag.

### No-op Stop after Dispose

`Stop(TimeSpan)` silently swallows `ObjectDisposedException`. This matches the
.NET pattern where calling cleanup methods on a disposed object should not
throw. The alternative (throwing) would force callers to wrap every `Stop()`
call in try/catch after `using` blocks, which is boilerplate with no benefit.

## Implementation

A reference implementation is available at
PowerShell/PowerShell#27027 (6 modified files, 2 new test files, +913/-20
lines).

An interactive documentation site with full specification, execution flow
diagrams, and 8 adversarial scenario walkthroughs is available at:
https://ps-bounded-wait-docs.pages.dev/
