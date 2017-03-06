---
RFC: 0002
Author: Jason Shirk
Status: Draft
Area: Splatting
Comments Due: 3/31/2016
---

# Generalized Splatting

Splatting was introduced in PowerShell v2
as a way of passing named arguments to a command
using an expression (e.g. `@PSBoundParameters`)
instead of explicit parameter syntax (e.g. `-Param1 $value`).
The expression syntax introduced was limited
to variable references and is more restrictive than necessary.

Proxy functions are much easier to write using splatting.
A proxy function might add or remove parameters to another command,
but might not care about all of the parameters to the proxied command.
The proxy still needs to pass those parameters through to the proxy,
and it does this with splatting.

Scripters have found splatting useful beyond proxy functions,
but the current syntax limits broader use of splatting.

## Motivation

One common use of splatting is to provide an alternate syntax for calling a command with many parameters.
For example, consider a command like the following

```PowerShell
Update-TypeData -TypeName MyCustomType `
             -MemberName MyCustomProperty `
             -MemberType NoteProperty `
             -Value 42
```

In the above example, note the use of backticks at the end of every line.
Those backticks are difficult to read and easy to forget.
Splatting can help some here:

```PowerShell
$addTypeParams = @{
    TypeName = 'MyCustomType'
    MemberName = 'MyCustomProperty'
    MemberType = 'NoteProperty'
    Value = 42
}
Update-TypeData @addTypeParams 
```

This works, but feels a bit messy because of the need for a variable,
when the hashtable could start on the same line as the command name.

Another common use of splatting is to remove a specific parameter when splatting:

```PowerShell
$PSBoundParameters.Remove('SomeExtraParam')
Command @PSBoundParameters
```  

This proposal suggesst a syntax that improves this scenario as well.

## Specification

This RFC proposes a more generalized syntax for splatting to support:

- Splatting general expressions (as opposed to simple variables)
- Splatting in method invocations
- Splatting in switch cases

Today, the syntax for splatting is a sigil used on a variable name.
This proposal expands the use of the sigil to be allowed on an expression.
For example:

```PowerShell
# Existing usage:
command @PSBoundParameters

# Equivalent - but splatting an expression (the value of a variable) 
command @$PSBoundParameters

# Splat another expression - a hashtable
Update-TypeData @@{
    TypeName = 'MyCustomType'
    MemberName = 'MyCustomProperty'
    MemberType = 'NoteProperty'
    Value = 42
}
```

Note the two '@' characters in the hashtable example above.
If the second '@' was omitted, the example would pass a hashtable (no splatting)
in all versions of PowerShell, even V1, which means splatting without the second '@'
would be a breaking change.

We can use more general expressions as well:

```PowerShell
# Call a command that returns a hashtable or array to splat
Get-ChildItem @$(Get-ChildItemArgs)

# a method that returns a hashtable or array to splat
Invoke-Something @$($obj.GetInvokerArgs())
```

### Relaxed splatting

In some cases, it is desirable to splat only the arguments that are available,
and ignore the others.
We will call this relaxed splatting.
For example:

```PowerShell
$myArgs = @{ Path = $pwd; ExtraStuff = 1234 }
Get-ChildItem @$myArgs
```

The above example would fail with a "parameter not found" because of the 'ExtraStuff' key.
Here is a possible syntax to allow the above without resulting in an error: 

```PowerShell
$myArgs = @{ Path = $pwd; ExtraStuff = 1234 }
Get-ChildItem @?$myArgs
```

We can think of '@?' as the 'relaxed splatting' operator.
If '@' is the splatting operator,
adding the '?' is suggestive of being more permissive,
much like the C# '?.' member access operator.

### Splatting in method invocations

Today, named arguments are only supported when calling commands,
named arguments do not work when calling methods.
PowerShell could adopt a C# like syntax to name arguments,
but splatting provides a similar capability with some additional flexibility.
The obvious syntax would mirror command invocation:

```PowerShell
# Splat a hashtable defined outside the method call
$subStringArgs = @{startIndex = 2}
$str.SubString(@$subStringArgs)

# Splat a hashtable defined inline in the method call
$str.SubString(@@{startIndex = 2; length=2})
```

Mixing splatting and positional arguments is supported.

```PowerShell
$str.SubString(2, @@{length=2})
```

A runtime check (or parse time if possible) is necessary to make sure an argument is not specified positionally
and via splatting in the same invocation:

```PowerShell
# Must be an error, parse time or runtime, because startIndex
# is specified positionally and via splatting.
$subStringArgs = @{startIndex = 2}
$str.SubString(2, @$subStringArgs)
```

Multiple splatted arguments are not allowed:

```PowerShell
# Error, multiple splatted arguments
$str.SubString(@@{startIndex = 2}, @@{length=2})
```

The splatted argument must be last.

```PowerShell
# Error, splatted argument is not last
$str.SubString(@@{length=2}, 2)
```

### Splatting in switch cases

It can be awkward to match multiple conditions with a single switch statement.
For example:

```PowerShell
switch ($color) {
  { $_ -eq 'Red' -or $_ -eq 'Blue' -or $_ -eq 'Green' } { $true }
  default { $false }
}
```

With splatting, this can be simplified:

```PowerShell
switch ($color) {
  @@('Red','Blue','Green') { $true }
  default { $false }
}
```

### Modifying hashtables for splatting

Sometimes it is useful to provide a 'slice' of a hashtable,
e.g. you want to remove or include specific keys.
The Add/Remove methods on a hashtable work, but can be verbose.
This proposal suggests overloading the '+' and '-' operators to provide a hashtable 'slice':

```PowerShell
Get-ChildItem @$($PSBoundParameters - 'Force') # Splat all parameters but 'Force'
Get-ChildItem @$($PSBoundParameters - 'Force','WhatIf') # Splat all parameters but 'Force' and 'WhatIf'
Get-ChildItem @$($PSBOundParameters + 'LiteralPath','Path') # Only splat 'LiteralPath' and 'Path'
```

Today, PowerShell supports "adding" two hashtables with the '+' operator,
the result is a new hashtable with all of the key value pairs from both hashtables.
It is an error for a key to be specified in both operands.

This proposal adds semantics for adding and subtracting other values.
If the value is enumerable, the effect is to treat each value in the collection as a key,
otherwise the value is treated as a single key.
The result is always a new hashtable,
the left hashtable operand is unmodified.

When using '+', the result will only include keys found in the right hand side.
When using '-', the result will exclude all keys from the right hand side.

In either case,
it is not an error to specify a key in the right hand side operand that is not present in the left hand side.  

## Alternate Proposals and Considerations

### Slicing operators

The suggested use of '+' and '-' is perhaps surprising
even though they correspond to Add and Remove, respectively.
The actual operation is also similar to a union or intersection,
so other operators should be considered, perhaps bitwise operators
like '-bnot' and '-bor', or maybe new general purpose set operators. 

### Postfix operator

The use of a sigil is not always well received.
This proposal nearly considers '@' a prefix unary operator,
but it doesn't quite specify it as such.

A postfix operator is another possiblity and would look less like a sigil.
This idea was rejected because, when reading a command invocation from left to right,
it's important to understand how a hash literal is to be used.
The sigil makes it clear a hash literal is really specifying command arguments.
Furthermore, the sigil simplifies the analysis required for good parameter completion,
and does not require a complete expression to begin providing parameter name completion.