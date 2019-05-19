---
RFC: 0037
Author: Tyler Leonhardt
Status: Draft
Area: Cmdlet-ErrorPassThru
Comments Due: 6/18/2019
---

# New ErrorPassThru parameter for Cmdlets and advanced functions

This RPC proposes the addition of ad
`-ErrorPassThru`
parameter to help better catch and handle terminating and non-terminating errors.

The usage would look something like this:

```pwsh
Get-Foo -ErrorPassThru | % {
    # $_ is the error record just like in a try/catch
    $_
}
```

The items that would get emitted to the pipeline are ONLY the ErrorRecords caused by non-Terminating and Terminating Errors.

This will also "catch" any exceptions allowing the user to not have to use a try/catch block at all.

## Motivation

### Handling non-terminating and terminating errors is verbose

It's very annoying handling non-terminating and terminating errors.
Often it looks something like this:

```pwsh
try {
    Get-Foo -ErrorAction Stop
} catch {
    $_
}
```

The
`ErrorAction`
is needed to make sure non-terminating errors actually go into the
`catch`.
Then the try/catch is needed to handle error.

This will get verbose quickly if you have multiple statements that you want to handle differently:

```pwsh
try {
    Get-Foo -ErrorAction Stop
} catch {
    $_
}

try {
    Get-Bar -ErrorAction Stop
} catch {
    $_
}

try {
    Get-Baz -ErrorAction Stop
} catch {
    $_
}
```

### There's not an easy way to run something per-non-terminating error

Let's say that
`Get-Foo`
in the previous example contained 3 non-terminating errors.

With
`-ErrorAction Stop`
specified,
the pipeline will stop at the first non-terminating error and never get to the other 2.

But what if you wanted all 3 non-terminating errors to be emitted,
but you wanted to run a script on every error
(i.e. I want to log all of my non-terminating errors to Splunk)?

This is possible,
but it's a bit annoying:

```pwsh
Get-Foo 2>&1 | % {
    if($_.GetType -eq [ErrorRecord]) {
        # Handle Errors
    } else {
        # Handle non-errors
    }
}
```

This leverages the core concepts of the pipeline and redirection of the Error stream into the Output stream.

Leveraging the pipeline here is good,
but the problem is that the redirection merges Error into Output/Success stream leaving the user to sort through the pipeline themselves.

## Specification

Let's take a look at the example above again:

```pwsh
Get-Foo -ErrorPassThru | % {
    # $_ is the error record just like in a try/catch
    $_
}
```

As mentioned above, this will do two things:

* Not require a try/catch for terminating errors giving the user the ability to handle both terminating and non-terminating errors in the same fashion.
* Allow the user to simply handle all non-terminating errors by leveraging the existing pipeline concept.

This would allow users to log errors for reporting,
or if a non-terminating error is actually bad and you need to stop the pipeline,
that can all be handled in the
`ForEach-Object`
just as you would with anything else
(i.e use the `throw` statement).

### Alternate Proposals and Considerations

The motivation of this RFC comes from the issue https://github.com/PowerShell/PowerShell/issues/6010.

The proposal there was a parameter that you pass a ScriptBlock to:

```
Stop-Process foo* -OnError {
     Write-Host "Had an error."
}
```

This would then:

* run for every non-terminating error
* run once for the terminating error

However,
it's a bit confusing how this plays well with the pipeline and is suggesting changing the meaning of
`continue`
and
`break`
statements by adding them to a new context.

The
`OnError`
proposal reminds me of a JavaScript callback which isn't really a well adopted pattern in the PowerShell world,
which is why I chose against it.

The current proposal
`ErrorPassThru`,
leverages one of the core components of PowerShell - the pipeline.

#### Open Questions

With an
`ErrorPassThru`
added,
I wonder if a
`VerbosePassThru`,
`WarningPassThru`,
etc are useful?
