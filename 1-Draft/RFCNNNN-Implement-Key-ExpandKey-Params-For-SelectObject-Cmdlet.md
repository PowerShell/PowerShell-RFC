---
RFC:
Author: Brian G. Shacklett
Status: Draft
SupercededBy: n/a
Version: 0.1
Area: Utility Cmdlets
Comments Due:
Plan to Implement:
---

# Implement Key and ExpandKey Parameters for the Select-Object Cmdlet
This RFC proposes that the `Select-Object` Cmdlet be extended with two new
parameters: `Key` and `ExpandKey`. These parameters would allow the
Select-Object Cmdlet to index into Hashtables and would function similarly to
the `Property` and `ExpandProperty` parameters which currently exist for other
object types.  

## Motivation

The current implementation of the Select-Object command is not capable of
indexing into Hashtables, requiring either that the Hashtable be converted to a
`PSCustomObject` or that it be accessed via square bracket notation
(e.g.: `$myHashtable['MyKey']`).

When indexing into deeply nested objects, such as a Hashtable representation of
a fragment of an Azure Resource Manager template, one ends up needing either a
long chain of indexes which cannot be broken up onto separate lines, or a
number of explicit conversions into `PSCustomObject` types. Ignoring any performance considerations, this leads to some pretty disgusting code:

* Square Bracket Notation:
  ```
  $value = $myHashTable['properties']['httpListeners'][0]['properties']['FrontendIPConfiguration']['Id']
  ```

* Explicit Conversion:
  ```
  $value = [PSCustomObject]$myHashTable `
           | Select-Object -ExpandProperty properties `
           | ForEach-Object { [PSCustomObject]$_ } `
           | Select-Object -ExpandProperty httpListeners `
           | Select-Object -First 1 `
           | ForEach-Object { [PSCustomObject]$_ } `
           | Select-Object -ExpandProperty properties `
           | ForEach-Object { [PSCustomObject]$_ } `
           | Select-Object -ExpandProperty FrontendIPConfiguration `
           | ForEach-Object { [PSCustomObject]$_ } `
           | Select-Object -ExpandProperty Id
  ```

In the first case, this creates a very long line which can be difficult to read. In the second, readability is hampered by each of the conversions.

## Specification

### General
* The Select-Object Cmdlet is extended with two new parameters:
  * `Key`
  * `ExpandKey`

### Input
* Both the `Key` and `ExpandKey` parameters of the Select-Object Cmdlet accept
a type of `System.Object`.
* The current behavior of the incoming object being passed in via pipeline or
explicitly via the `InputObject` parameter remains the same.

### Output
* When called with the `Key` parameter, the Select-Object Cmdlet returns a
"slice" of the Hashtable including only the element for which the hash of the
`Key` matches the hash of the object passed to the `Key` parameter.
* When called with the `ExpandKey` parameter, the Select-Object Cmdlet returns
the value of the element for which the hash of the `Key` matches the hash of the
object passed to the `ExpandKey` parameter.

### Handling Unexpected Input
Sticking with the current behavior of square-bracket notation should be
sufficient and intuitive.

The current behavior when attempting to use square bracket notation to select
a key which does not exist as part of a given object is to return `$null`.
In the event that one attempts to index into a `$null` value, an error is thrown
stating that it is not possible to index into a null array. While the wording
could be improved in this case, both of these behaviors fit well with the
general behavior of PowerShell.

### Usage

#### Key:
```
PS> $myHashtable =
>> @{
>>   'color'  = 'blue'
>>   'shape'  = 'square'
>>   'number' = 1
>> }
PS> $myHashtable | Select-Object -Key 'color'


Name                           Value
----                           -----
color                          blue


PS>
```

#### ExpandKey:
```
PS> $myHashtable =
>> @{
>>   'color'  = 'blue'
>>   'shape'  = 'square'
>>   'number' = 1
>> }
PS> $myHashtable | Select-Object -ExpandKey 'color'
blue
PS>
```

### Re-implementing the original example:
```
$value = $myHashTable `
         | Select-Object -ExpandKey properties `
         | Select-Object -ExpandKey httpListeners `
         | Select-Object -First 1 `
         | Select-Object -ExpandKey properties `
         | Select-Object -ExpandKey FrontendIPConfiguration `
         | Select-Object -ExpandKey Id
```

Notice that each key lines up in a single column where the reader can easily
follow the path being accessed.

## Alternate Proposals and Considerations
