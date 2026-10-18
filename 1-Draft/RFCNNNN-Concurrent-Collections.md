---
RFC: RFCnnnn
Author: Kirk Munro
Status: Draft
SupercededBy: 
Version: 0.1
Area: Parser
Comments Due: July 25, 2019
Plan to implement: Yes
---

# Closures for concurrent collections

PowerShell is getting more and more support for multi-threaded use, such as:

* the PSThreadJob module
* `S.M.A.PowerShell.InvokeAsync` and `S.M.A.PowerShell.StopAsync`
* parallelization in `ForEach-Object` ([see related RFC 194](https://github.com/PowerShell/PowerShell-RFC/pull/194))

With these either available now or being made available soon, it would be
helpful if scripters could easily create concurrent alternatives to arrays
(`@()`) and hashtables (`@{}`).

## Motivation

    As a scripter,
    I can easily define and work with a ConcurrentBag<PSObject>,
    so that I can collect unordered objects from multiple threads in thread-safe manner.

    As a scripter,
    I can easily define and work with a ConcurrentDictionary<PSObject, PSObject>,
    so that I can store objects relative to keys in a dictionary in thread-safe manner.

## User Experience

```powershell
# Define a ConcurrentBag<PSObject>
$collection = ~@()

# Define a ConcurrentDictionary<PSObject,PSObject>
$dictionary = ~@{}

# Retrieve a bunch of logs from remote computers
$computers = 's1','s2','s3'
$computers | Start-ThreadJob {
    # Retrieve the data and store it in a thread-safe collection
    $using:collection += Get-LogData -ComputerName $_
    # Retrieve the data and store it in a thread-safe dictionary
    $dictionary[$_] = Get-LogData
}

# Process the collection as an array
$collection.ToArray()

# Process items in the dictionary
foreach ($key in $dictionary.Keys) {
    # Do something with each value
    $dictionary[$key]
}
```

```output
The data gathered in the $collection collection.

The data gathered in each key in the $dictionary collection.
```

## Specification

* define `~@()` enclosures as `System.Collections.Concurrent.ConcurrentBag<PSObject>`
* define `~@{}` enclosures as `System.Collections.Concurrent.ConcurrentDictionary<PSObject,PSObject>`
* ensure that the `+=` operator adds data to a `ConcurrentBag` (right now using
that operator with `ConcurrentBag` results in it being converted to an array of
objects instead)
* show a warning (suggestion) users if they use `[]` index operators with a
`ConcurrentBag` indicating that the `ConcurrentBag` collection is unordered and
therefore cannot be used with index operators, and suggesting that users should
use the `.ToArray()` method of `ConcurrentBag` once the collection has been
populated if they want to convert it to an array and process items using their
index.

## Alternate Proposals and Considerations

### Breaking changes

There are technically two breaking changes in this RFC:

1. Adding items to `ConcurrentBag` with `+=` results in the collection being
converted into an array in current PowerShell versions.
1. You can create a command named `~@` in PowerShell, so anyone using that
command name will break if we add `~@()` as enclosures.

I believe both of these are low-risk breaking changes that are worth making so
that we can have easier concurrent collection support in PowerShell moving
forward. Further, the first one could be identified as a bug and is likely not
what you want, and the second one is only breaking if you happen to be invoking
your `~@` command using parentheses with at least one value inside (e.g. `@(1)`).

### Alternative syntaxes

To simplify the syntax somewhat, we could implement `~()` for `ConcurrentBag`
and `~{}` for `ConcurrentDictionary`.

On the plus side, these are shorter and therefore easier to type. Also, by
removing the `@` character from `~@()`, it becomes a little less like an array
(which it is not, since arrays are ordered and you cannot index into a
`ConcurrentBag`, which is unordered).

On the downside, the potential for the second breaking change increases
because it is more likely that someone has created an alias of `~` than `~@`.
That would only break, however, if they were invoking that command with round
brackets with at least one value inside (e.g. `~('something')`), so it would
still be a very low risk breaking change.

We could also flip the enclosures, defining `@~()` as a `ConcurrentBag` and
`@~{}` as a `ConcurrentDictionary`. The advantage here is that they result in
parser errors, so wouldn't be a breaking change. The disadvantage is that the
syntax is obscure and really doesn't present itself very well at all.

### Don't add the enclosures because we have using

Since scripters can use the `using` statement to make it easier to work with
namespaces, they could do the following:

```PowerShell
using namespace System.Collections.Concurrent

$x = [ConcurrentBag[PSObject]]::new()
```

That works, but it lacks discovery of enclosures that should be used when
working with threads. One of the advantages of new enclosures is not simply
syntactic sugar. It's also beneficial for discovery. Users could learn about
different types of collections in one place, and learn when it is appropriate
to use one type over another.

The other key advantage to using enclosures, particularly for
`ConcurrentDictionary`, is ease of use in pre-population of the data. This
syntax is familiar and very straightforward:

```PowerShell
$x = ~@{
    PropOne = 1
    PropTwo = 2
}
```

The alternative is having to use the `Add` method on the collection to populate
it. You could use the index operator to create keys and assign values, but when
you are assigning multiple values at once, that is much less PowerShell-like
than simply defining the dictionary and assigning values to it in one step.

### Related discussion

There is a long running thread where some community members have been
[discussing a new enclosure for `List<PSObject>`](https://github.com/PowerShell/PowerShell/issues/5643). Those thoughts should be
reviewed and considered when looking at this proposal so that we come up with
a unified strategy to move forward with.
