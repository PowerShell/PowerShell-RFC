---
RFC: RFCnnnn
Author: Prasoon Karunan
Status: Draft
Version: 1.0
Area: Language
---

# Support for Foreach-Else statement

PowerShell foreach statment always works with empty collections which is known as a gotcha from begining (AFAIK). This RFC is to propose a solution to that gotcha without changing the existing behavior as I see it as breaking change.

## Motivation

This syntax is something similar to what python has as a for-else loop (with a completely different behavior)
```
for item in collection:
    if do_something(item):
        # Done it!
        process(item)
        break
else:
    # Didn't doanything..
    cannotDo()
```

## Syntax

Implementing an else statement which will avoid having an if-else to check the collection before calling foreach.

### Current

```
if($Null -ne $EmptyCollection){
    Foreach($Element in $EmptyCollection){
        DoStuff -Element $Element
    }
}
else{
    Write-Output -InputObject "The collections is empty"
}
```

```
Foreach($Element in $EmptyCollection){
    DoStuff -Element $Element
}
else{
    Write-Output -InputObject "The collections is empty"
    # Or/and Do something else
}
```

This is one use case I see withe this approach. I hope my fellow PowerShell peeps can find more usecase/cons of this approach.


## Alternate proposal

When having collections containing empty/null object, else statement can be used to handle the case.

```
$EmptyCollection = @('Element1',$null,'Element2')
Foreach($Element in $EmptyCollection){
    DoStuff -Element $Element
}
else{
    # Do something else
}
```

In this case, the else will be part of the loop and will behave like

```
$EmptyCollection = @('Element1',$null,'Element2')
Foreach($Element in $EmptyCollection){
    if($Element -ne $Null){
        DoStuff -Element $Element
    }
    else{
        # Do something else
    }
}
```
