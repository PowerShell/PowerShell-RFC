---
RFC: RFC0014
Author: Aditya Patwardhan
Status: Rejected
SupercededBy:
Version: 1.0
Area: LanguageAndParser
Comments Due: 02/01/2017
---

# Language enhancements for collections

System.Collections namespace has many collections that are frequently used in PowerShell scripts. To instantiate objects of these classes, New-Object and the type name has to be used. With Powershell v5 and above the ```[typename]::new() ``` can be used, by still needs the full type name. This RFC proposes a way to simplify the instantiation of types in System.Collections namespace.

## Motivation

The various ways user can instantiate these classes currently is as follows:

```PowerShell
$list = New-Object System.Collections.ArrayList
$list = [System.Collections.ArrayList]::new()
```
Collections are used frequently in PowerShell scripts, we can make the instantiating collections easier.

Performing a search on GitHub for usage of collections we know which collection type is used more frequently.

| ClassName | Usage on GitHub | Search Uri
--- |---:| ---
ArrayList | 2352 | [ArrayList Usage](https://github.com/search?l=PowerShell&q=%22New-Object+System.Collections.ArrayList%22+language%3APowerShell&ref=searchresults&type=Code&utf8=%E2%9C%93)
BitArray | 41 | [BitArray Usage](https://github.com/search?l=PowerShell&q=%22New-Object+System.Collections.BitArray%22+language%3APowerShell&ref=searchresults&type=Code&utf8=%E2%9C%93)
Queue | 1260 | [Queue Usage](https://github.com/search?l=PowerShell&q=%22New-Object+System.Collections.Queue%22+language%3APowerShell&ref=searchresults&type=Code&utf8=%E2%9C%93)
SortedList | 52 | [SortedList Usage](https://github.com/search?l=PowerShell&q=%22New-Object+System.Collections.SortedList%22+language%3APowerShell&ref=searchresults&type=Code&utf8=%E2%9C%93)
Stack | 71051 | [Stack Usage](https://github.com/search?l=PowerShell&q=%22New-Object+System.Collections.stack%22+language%3APowerShell&ref=searchresults&type=Code&utf8=%E2%9C%93)


## Specification

This RFC proposes adding type accelerators to support types from Sytem.Collections namespace. Looking at the usage on GitHUb, ArrayList, Queue and Stack are proposed to be added.

The definitions of the proposed accelerators are:

| Accelerator | Type Name
--- | ---
[list] | System.Collections.ArrayList
[queue] | System.Collections.Queue
[stack] | System.Collections.Stack

The usage of the proposed accelerators will be as follows;

```PowerShell
$list = [list]::new()
$queue = [queue]::new()
$stack = [stack]::new()
```

As the type accelerators can be used before the array operator the collection can be initialized using the supplied values.

```PowerShell
$list = [list] @(1,2,3)
$queue = [queue] @(1,2,3)
$stack = [stack] @(1,2,3)
```

## Alternate Proposals and Considerations

### New operator for list, queue and stack

A new operator can be proposed which creates a list, say @[] / @<> etc. This would mean we would need to make changes to the Parser to support the new operator. Since all the collections are single dimensional and an array operator is very familiar to users, adding a new operator would have a learning curve and discoverability issue.

### Alternative names for collections

The RFC proposes to use list, queue and stack for System.Collections.ArrayList, System.Collections.Queue and System.Collections.Stack respectively. Type accelerator names for Queue and Stack are same as class names.
Using 'list' is proposed for System.Collections.ArrayList instead of 'ArrayList' for brevity, 'ArrayList' is too long.


### Scope

Type accelerators for generic types for collections are out of scope for this RFC. They might be addressed in future RFCs.

---------------
## PowerShell Committee Decision

### Voting Results

Jason Shirk: Reject 

Joey Aiello: Reject

Bruce Payette: Reject

Steve Lee: Reject

Hemant Mahawar: Reject

### Majority Decision

The committee reviewed this and rejected this RFC because:

- `using namespace System.Collections` would solve most of the examples in this RFC
- The parser already has syntax for `using type list = System.Collections.ArrayList` for user defined accelerators although the implementation is not complete currently

Because there is an existing solution to save from typing the fully qualified typename and an existing design to enable user defined type accelerators, we decided to not accept this RFC.
We should continue to look at usage data to determine if in the future we add new default accelerators.

### Minority Decision

N/A
