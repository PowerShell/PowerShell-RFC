---
RFC:  RFC0000
Author:  Staffan Gustafsson
Status:  Draft
SupercededBy:  N/A
Version: 0.1
Area: LANGUAGE
Comments Due: 2017-04-20
---

# Support calling .NET extension methods in PowerShell

This RFC proposes changes to language syntax and runtime binding to enable the calling of generic .NET methods, and finding appropriate extension methods based on the type of the extension methods first parameter.

## Motivation

Extension methods enable you to "add" methods to existing types without creating a new derived type, recompiling, or otherwise modifying the original type. Extension methods are a special kind of static method, but they are called as if they were instance methods on the extended type. For client code written in PowerShell, there would be no apparent difference between calling an extension method and the methods that are actually defined in a type.

There are lots of useful extension methods in .NET already written, many of which are in the area of processing sets of objects and where the expressiveness/performance ratio, from a PowerShell perspective, is quite favorable.

### Goals

Simplicity of use
-	Should be as easy as possible to call from PowerShell.
    -	Type parameters will be deduced to the most specific type possible
```powershell
    # System.Linq.Enumerable.Select[TInput, TResult] takes two generic type parameters
    using namespace System.Linq    
    $a.Select({param([int] $i) $i.ToString()}) # [int,object] will be deduced.

    [func[int,string]] $f = {param([int]$i) ($i * 2).ToString()}
    $a.Select($f) # [int,string] will be deduced.
    
    $a.Select(param($i) $i * 2) #  [object,object] will be deduced.
```

Simplication of 'Magic' methods
The `Foreach` and `Where` methods that are added to collections are a hack which could be solved more elegantly by making extension methods of the EnumerableOps methods.
By doing this, they wouldn't need to be special cased. Normal method resolution would make it work, and would also make `List[T].Foreach` not hide the extension methods.

```csharp
public static class EnumerableOps {
    static IEnumerable<T> Foreach<T>(this IEnumerable<T> c, ScriptBlock sb){}
    static IEnumerable<T> Foreach<T>(this IEnumerable<T> c, ScriptBlock sb, WhereOperatorSelectionMode mode){}
    static IEnumerable<T> Foreach<T>(this IEnumerable<T> c, ScriptBlock sb, WhereOperatorSelectionMode mode, int count){}
    static IEnumerable<T> Foreach<T>(this IEnumerable<T> c, object expression, object[] args){}

    static IEnumerable<T> Where<T>(this IEnumerable<T> c, ScriptBlock sb){}
    static IEnumerable<T> Where<T>(this IEnumerable<T> c, ScriptBlock sb, WhereOperatorSelectionMode mode){}
    static IEnumerable<T> Where<T>(this IEnumerable<T> c, ScriptBlock sb, WhereOperatorSelectionMode mode, int count){}
}

```


## Specification

Extension methods are choosen from all methods on all loaded types where the methods has the ExtensionAttribute.
From these, the resulting set is limited to the types whose namespace are in use in the context where the call is made.
From these, a best fit is selected based on parameters.

The binding to an extension methods should not depend on the namespaces in use in other contexts. Each file/script or interactive
session maintains its own set of namespaces.

If multiple methods are an equally good match, an error is produced. To call such a method anyway, the standard static method approach can be used, i.e.

```powershell
[func[int,string]] $f = {param([int]$i) ($i * 2).ToString()}
[Linq.Enumerable]::Select($a, $f)

```

## Alternate Proposals and Considerations

Do not require namespaces to be specified, and determine how conflicts should be resolved by some other means.

Do not allow type parameter deduction. Require type arguments to be specified.

## See also

https://github.com/PowerShell/PowerShell/issues/2226 - Add support for extension methods (LINQ)
https://github.com/PowerShell/PowerShell/issues/2864 - Add conversion from PSMethodInfo to delecate/Func[]