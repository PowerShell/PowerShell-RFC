---
RFC: RFCnnnn
Author: Prasoon Karunan
Status: Draft
Version: 1.0
Area: Language
---

# Support for Foreach-Else statement

This RFC is to add an else clause to the foreach keyword which gives a better way to handle empty collections provided to foreach without using a `if/else` statement inside it.

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

## Proposed Syntax

Implementing an else statement which will avoid having an if-else to check the collection before calling foreach.

### Without the else clause

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


### With else clause
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


## Alternate proposal 1

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

## Alternate proposal 2

Rather than an alternate proposal, this will acheive one more use case of handling the coditional statement with in the foreach loop.

```
$Collection


Name   Type
----   ----
Group1 Group
User1  User
Group2 Group
User2  User

# How we currently handle this situation, this will handle only for Group type
Foreach($Element in $Collection.Where({$_.Type -eq 'Group'})){
    DoForGroup -Element $Element
}

# How we currently handle this situation for both the types
Foreach($Element -like in $Collection){
    if($Element.Type -eq 'Group'){
        DoForGroup -Element $Element
    }
    else{
        DoForUser -Element $Element
    }
}

# Proposal for above two scenarios

Foreach($Element.Type -eq 'Group' in $Collection){
    DoForGroup -Element $Element
}
else{
    DoForUser -Element $Element
}
```