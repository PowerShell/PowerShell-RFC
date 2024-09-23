---
RFC: RFCNNNN-Using-IDisposables.md
Author: Jordan Borean
Status: Draft
SupercededBy:
Version: 1.0
Area: engine
Comments Due: July 1, 2024
Plan to implement: Yes
---

# Using Statement Blocks for IDisposables

## Motivation

    As a powershell developer/user,
    I can enclose my disposable instances in a block,
    so that they are disposed automatically when they are no longer needed.

## User Experience

PowerShell can interact with many instance types that might hold onto system resources which are tied to the lifetime of that instance.
Examples of such types are the [FileStream](https://learn.microsoft.com/en-us/dotnet/api/system.io.filestream?view=net-8.0), [RegistryKey](https://learn.microsoft.com/en-us/dotnet/api/microsoft.win32.registrykey?view=net-8.0), [Process](https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.process?view=net-8.0) which contain operating system (OS) handles/identifiers.
On Windows, some of these handles could place a lock which can impact future operations, like trying to open a new handle on a file locked for reading, unload a registry hive with a key still open, etc.

A safe way to deal with these objects is to wrap the code that creates the instance in a `try/finally` block.
For example the below PowerShell script interacts with various streams that should be disposed when no longer needed to free up OS resources or internal memory buffers:

```powershell
$reader = [System.IO.File]::OpenText("numbers.txt")
try {
    [Console]::WriteLine($reader.ReadLine())
}
finally {
    $reader.Dispose()
}
```

The drawbacks here are

+ The user needs to use a `try/finally` block to ensure it is cleaned up
+ The `finally` block can be interrupted with an engine stop request
+ The user need to explicitly write code not only to pre-define the vars, set the vars, then dispose of them with a null check

C# developers can use the [using statement](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/using) that allows them to scope an `IDisposable` instance in an automatic `try/finally` block like the above:

```csharp
using (StreamReader reader = File.OpenText("numbers.txt"))
{
    Console.WriteLine(reader.ReadLine());
} // reader is disposed here
```

This RFC proposes adding an equivalent `using` statement in PowerShell that can do the same thing as C#.
For example the same C# `StreamReader` example in PowerShell can now be:

```powershell
using ($reader = [System.IO.File]::OpenRead("numbers.txt")) {
    [Console]::WriteLine($reader.ReadLine())
}
```

Another more complex example where the user might have multiple disposable objects that should be disposed is:

```powershell
$fs = $cryptoStream = $sr = $null
try {
    $fs = [System.IO.File]::OpenRead($Path)
    $cryptoStream = [System.Security.CryptoGraphy.CryptoStream]::new(
        $fs,
        [System.Security.Cryptography.ToBase64Transform]::new(),
        [System.Security.Cryptography.CryptoStreamMode]::Read)
    $sr = [System.IO.StreamReader]::new($cryptoStream, [System.Text.Encoding]::ASCII)
    $sr.ReadToEnd()
}
finally {
    ${sr}?.Dispose()
    ${cryptoStream}?.Dispose()
    ${fs}?.Dispose()
}
```

This is a more complicated version as now the finally block needs to check that each of the variables to be disposed are actually set and they are disposed in the opposite order they were defined.
The proposed syntax for supporting this scenario would be:

```powershell
using (
    $fs = [System.IO.File]::OpenRead($Path)
    $cryptoStream = [System.Security.CryptoGraphy.CryptoStream]::new(
        $fs,
        [System.Security.Cryptography.ToBase64Transform]::new(),
        [System.Security.Cryptography.CryptoStreamMode]::Read)
    $sr = [System.IO.StreamReader]::new($cryptoStream, [System.Text.Encoding]::ASCII)
) {
    $sr.ReadToEnd()
}
```

In this scenario the `using ()` statement supports multiple `AssignmentStatementAst` statements allowing it to track multiple variables to dispose.
The equivalent syntax in C# would be this example:

```csharp
using (FileStream fs = File.OpenRead(path))
using (CryptoStream cryptoStream = new CryptoStream(...))
using (StreamReader sr = new StreamReader(cryptoStream, Encoding.ASCII))
{
    sr.ReadToEnd();
}
```

## Specification

The syntax for the proposed `Using` statement would be:

```
using ($<disposable>; ...) {
    <Statement list>
}
```

The part inside the parenthesis is a variable to dispose after the `<Statement list>` ends.
It can contain multiple variables as well as the code used to define the variable.
Some examples of this would be:

```powershell
# Creates the instance separately from the using statement
$key = Get-Item 'HKLM:\System\CurrentControlSet\Control\Session Manager\Environment'
using ($key) {
    $key.GetValue('PATH', '', 'DoNotExpandEnvironmentNames')
}

# Creates the variable inside the parenthesis
using ($key = Get-Item 'HKLM:\System\CurrentControlSet\Control\Session Manager\Environment') {
    $key.GetValue('PATH', '', 'DoNotExpandEnvironmentNames')
}

# Disposes multiple values at the end of the block
$machineKey = Get-Item 'HKLM:\System\CurrentControlSet\Control\Session Manager\Environment'
$userKey = Get-Item 'HKCU:\Environment'

using ($machineKey; $userKey) {
    $m = $machineKey.GetValue('PATH', '', 'DoNotExpandEnvironmentNames')
    $u = $userKey.GetValue('PATH', '', 'DoNotExpandEnvironmentNames')
    "${m};${u}"
}

# Same as the above but with the vars defined in the parenthesis
using (
    $machineKey = Get-Item 'HKLM:\System\CurrentControlSet\Control\Session Manager\Environment'
    $userKey = Get-Item 'HKCU:\Environment'
) {
    $m = $machineKey.GetValue('PATH', '', 'DoNotExpandEnvironmentNames')
    $u = $userKey.GetValue('PATH', '', 'DoNotExpandEnvironmentNames')
    "${m};${u}"
}
```

## Implementation Details

I propose adding a new AST type `UsingDisposableStatementAst` that contains

+ `PipelineBaseAst[]` - Disposables
+ `StatementBlockAst` - Body

When visited the Ast will visit the `Disposables` to find any `AssignmentStatementAst` or `VariableExpressionAst` which it can use to keep track of variables to dispose.
The compiler will then emit an Expression that wraps the `Body` with the below for each `Disposables` (in definition order):

```powershell
$v = ...  # Value from assignment/variable
try {
    ...
}
finally {
    ${v}?.Dispose()
}
```

## Alternate Proposals and Considerations

Some questions that would need consensus from the maintainers are:

+ The name of the statement
+ Whether to support multiple disposables in one block
+ Should it dispose the variable or actual instance captured before the body is run
+ Support only `IDisposable` vs duck typing with `Dispose()` method
+ What to do in case of an error in the dispose
+ Future support for `using` declarations without the block/braces

### Name of the Statement

This RFC proposes using `using` as the statement name to try and provide some familiarity with how this type of operation works in C#.
The downside to `using` is that PowerShell already has a [using statement](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_using?view=powershell-7.4).
The existing `using` statements are handled at parse time and support the following:

```powershell
using assembly <.NET-assembly-path>
using module <module-name>
using namespace <.NET-namespace>
```

It should be possible to differentiate between these three parse time statements (and future ones) by checking to see if the next token after `using` starts with `(` rather than `assembly`, `module`, `namespace`, and future additions.
I believe this is a more complex topic for more advanced developers so it would be ideal to align with how C# works.

There is also a `$using:varName` "scope" used in remoting situations but as that is part of the variable name itself rather than a statement I do not think it should be an issue.

Alternate names could be:

+ `clean` - Aligns with the [clean block](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_functions_advanced_methods?view=powershell-7.4#clean) introduced in 7.3
+ `dispose` - Associated with the `Dispose()` method and the action being done
+ Something else to be proposed

A con to the above options over `using` is that this is currently valid syntax in PowerShell:

```powershell
function dispose {}

dispose ($var = 'abc') { 'foo' }
```

If a keyword that is not reserved is chosen, it could potentially break scripts that have defined that keyword as a command.

### Multiple Disposables

The current proposal aims to support disposing multiple variables either specified as a `VariableExpressionAst` or `AssignmentStatementAst` and the variables will be disposed in the reverse order they were defined in the `using` statement.
It could be desirable to only allow one variable or assignment in the statement to simplify the definition.
The downside of this is increased indentation of the code when trying to dispose of multiple objects, for example:

```powershell
using ($var1 = ...) {
    using ($var2 = ...) {
        ...
    }
}
```

If only 1 disposable could be defined in each block it might be nice to support a `using ()` expression without an expression body like so that the subsequent vars do not need to be indented:

```powershell
using ($var1 = ...)
using ($var2 = ...) {
    ...
}
```

I feel that it is more PowerShell like to support multiple statements in the `using (...)` rather than multiple `using (...)` calls in the same line.
By allowing multiple statements the parser should also be simpler to implement.

### Variable or Instance

The C# implementation is set to dispose the instance value as it was at the start of the using statement rather than at the end.
Take for example this C# code:

```csharp
Foo v = new Foo();
using (v)
{
    v = new Foo();
    Console.WriteLine(v);
}
```

When compiled this code is treated as:

```csharp
Foo v = new Foo();
Foo vTemp = v;
try
{
    v = new Foo();
    Console.WriteLine(v);
}
finally
{
    if (vTemp != null)
    {
        ((IDisposable)vTemp).Dispose();
    }
}
```

Only the original value of `v` when the `using (v)` block starts will be the one disposed.
This could be confusing for end users who might expect that the newly assigned `v` is disposed and vice versa.
This implementation would have to make the choice of;

+ Should it follow the C# behaviour and dispose the instance set at the start of the block - my recommendation
+ Should it dispose the result of the variable/assignment as it is set at the end of the block
+ Should it attempt to stop variable reassignment inside the block
  + This may not be possible to stop with `Set-Variable` but assignments could potentially be tracked/stopped at parse time

### IDisposable vs Duck typing

Should any variables specified by the using statement only support `IDisposable` types or should it also support any method with a well defined method like `Dispose()`.
The C# implementation only works with `IDisposable` types but with the dynamic nature of PowerShell it might be useful to support anything with a `Dispose()` method.
For example:

```powershell
# Dispose() is met as part of the IDisposable contract
class MyDisposableClass : IDisposable {
    [void] Dispose() {}
}

# Dispose() is just a method on the type
class MyDuckTypedClass {
    [void] Dispose() {}
}

# Dispose() is part of an ETS member
$obj = [PSCustomObject]@{}
$obj | Add-Member -MemberType ScriptMethod -Name Dispose -Value { ... }

# Dispose() is overridden by ETS member
$fs = [System.IO.File]::OpenRead($path)
$fs | Add-Member -MemberType ScriptMethod -Name Dispose -Value { ... }
```

Should this new implementation work with all the above examples or should it try and limit itself to specific scenarios.
Should PowerShell attempt to support `IEnumerable`, `Array` types (or not at all) and enumerate the variable values to see what ones to dispose.
From an implementation perspective it would be simpler to support only `IDisposable` types as the expression tree would not have to go through the binder to support the ETS members but it is less flexible.

### Error Handling

How should exceptions that are thrown in a potential `Dispose()` call be treated, should they just be written as any normal `MethodInvocationException` in that statement or should they be ignored.
Having the exception be raised like normal makes the most sense as callers can still wrap the `using ()` block in their own `try/catch` and silently ignoring errors is not ideal.

```powershell
try {
    # $var.Dispose() in this scenario throws an exception.
    using ($var = Get-Disposable) {
        ...
    }
}
catch {
    $_  # MethodInvocationException ErrorRecord
}
```

In the case when multiple using statements are used and multiple `Dispose()` calls raise an exception, the outermost block is the one that will be in the catch block.
This follows the convention in C# where the output of the below is `first` from the outermost `using` block dispose.

```csharp
using System;

public class Program
{
    public static void Main()
    {
        try
        {
            using (D a = new D("first"))
            using (D b = new D("second"))
            {
            }
        }
        catch (Exception e)
        {
            Console.WriteLine(e.Message);
        }
    }
}

public class D : IDisposable
{
    private string _msg;

    public D(String msg) => _msg = msg;

    public void Dispose()
    {
        throw new Exception(_msg);
    }
}
```

It would make sense for PowerShell to follow along with this convention but one possible alternative is to raise an [AggregateException](https://learn.microsoft.com/en-us/dotnet/api/system.aggregateexception?view=net-8.0) instead.

Another error condition that needs to be considered is how should non disposable objects be treated.
As the parser has no knowledge of the instance type or whether it has a `Dispose()` method, should this be silently ignored, or should an error/exception (which one) at runtime.

### Using Without Block

While not part of this RFC, C# supports a using statement without a block, for example:

```csharp
public void Main(bool flag)
{
    if (flag)
    {
        using Foo foo = new();
        // Foo is disposed here as it is no longer in scope
    }

    Console.WriteLine("end");
}
```

As PowerShell has a more complex scoping setup it is most likely not feasible to implement in PowerShell but it should be considered in case someone wants to revisit it in the future.
How should this syntax be done in PowerShell, will it have any impact on the syntax proposed in this RFC.
Possible options that should work with this RFC is to support an explicit `using $var = ...` without the parenthesis.
The presence of `$` after `using` should be able to differentiate between the existing `using ...` statements and the new one proposed in this RFC.
