---
RFC: NNNN
Author: Joel Sallow
Status: Draft
Area: Language
Comments Due: 7/29/2019
---

# Function / Script Cmdlet Disposal

Functions in PowerShell are highly versatile, but don't always have the capabilities to manage resources correctly.
For example, in .NET there are many types which implement `IDisposable`, advising that their `Dispose()` method should be used to clean them up when they're no longer needed.
In addition to these, there are some types which manage connections to remote hosts or databases which commonly include a `Close()` method which is often to be used prior to calling `Dispose()`.
Currently, PowerShell functions do not have an effective way of using these resources in the correct manner.

This is further complicated by the fact that any terminating error in the pipeline completely bypasses the `end` named block.
As such, it cannot be relied upon for cleanup / disposal steps.

This includes any time a user includes the pipeline cmdlet `Select-Object` with the `-First` parameter.
Once the desired number of objects has been reached, the cmdlet throws an internal and deliberately hidden exception to halt the pipeline.
This is highly effective, but can lead to undesirable circumstances if essential cleanup steps are then skipped as a result.

Compiled cmdlets do not suffer this issue, as they can simply implement `IDisposable` and place the necessary steps in a `Dispose()` method of their own.
The pipeline processor will correctly handle **cmdlets** that require disposal steps, but currently not **functions**.

## Motivation

As a developer,<br />
I want to be able to ensure I am not leaving resources or connections open when a user utilizes a function I have written.

As an administrator,<br />
I want to be able to ensure that any necessary logging or other necessary administrative tasks cannot be skipped accidentally or deliberately by a user invoking a function in a pipeline and using `Select-Object -First X` to bypass logging which would normally take place in the `end` block.

## Specification

A `dispose{}` block is required for these use cases.

### Execution

The `dispose{}` block will execute:

1. After `end{}` during normal command operation, and
2. During command disposal that occurs when a terminating exception is thrown during pipeline operations.
    - If the dispose block has already been called (as it is after the `end{}` step), it will simply return immediately.

### Behaviour

The `dispose{}` block _does not_ guarantee output, but neither does it prevent it.

- If output is received from the `dispose{}` block during normal command operation, it will be sent along the pipeline as per usual.
- If output is received from the `dispose{}` block during a halting pipeline, it will be **discarded**.

As a result, documentation should advise that `dispose{}` is _not intended_ for any standard output and the limitations of doing so.
Additionally, all other PowerShell data streams are accessible from `dispose{}`, meaning there is no issue using any of the following streams from within `dispose{}`:

- Error
- Warning
- Verbose
- Debug
- Information

Terminating errors can also be thrown from a `dispose{}` block, which will skip the invocation of the rest of that `dispose{}` block, without preventing other `dispose{}` blocks from being executed.

### Implementation Details

- A new `dispose` keyword, indicating a `NamedBlockAst` similarly to `begin`, `process`, and `end`, which is valid in the same contexts as the existing named block keywords.
  - This includes new token types and appropriate entries in the `Tokenizer`'s keyword tables.
- Additional members and constructors for `ScriptBlockAst` to enable it to be recognised by the parser.
- Additional methods in `DlrScriptCommandProcessor` and `CompiledScriptBlock` to enable the block to be executed at the correct time.
  - Both types implement or inherit from a type that implements `IDisposable`, and as such the existing `Dispose()` methods can generally be utilised to run this block.
- Changes to the behaviour of command execution in `CommandProcessorBase` to recognise when a function or script cmdlet is being executed, and appropriately call their `Dispose()` methods to invoke the dispose block.
  - This can include changes to the existing command disposal behaviour, to enable commands to dispose their resources as soon as their tasks in the pipeline are completed, instead of waiting for the entire pipeline to complete.
  - Also required with this is an additional layer of `try/finally` during `Dispose()` events as we are opening up the possibility for users to throw terminating errors _during_ a command's disposal, and as such we must ensure that disposal still completes appropriately.

### Alternate Proposals and Considerations

#### Reuse the `End{}` Block

This idea was explored briefly before being discarded.
The reason this cannot be the appropriate method is that pipeline behaviour somewhat relies upon there being a skippable `end{}` block.
For example, cmdlets such as `Sort-Object` only send output during `EndProcessing()`, and script functions place everything into the `end{}` block by default when no named blocks are specified.

If the `end{}` block were unskippable for logging purposes, pipeline and command disposal in the case of errors or `Select-Object -First X` would quickly become very sluggish compared to how they are now.
It is quite common for functions to be designed to do much of their processing in the `end{}` block.
Even if we prevented output from occurring in that instance as we are with `dispose{}`, the processing to produce said output would still be occurring.
This would significantly slow down intentional or necessary pipeline stops and reduce the utility of `Select-Object -First X` immensely.