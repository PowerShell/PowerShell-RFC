---
RFC: RFC0XXX
Author: Tyler Leonhardt
Status: Draft
Area: Traps
Comments Due: 
---

# Trap should handle non-terminating errors

This RFC proposes that we change the behavior of the Trap statement to also trap non-terminating exceptions.

To do this,
we will also add a property on ErrorRecord to determine if it was a terminating or non-terminating exception.

With this change, the following code:

```pwsh
trap {
    "hello"
    continue
}
Get-Process IDontExist
```

will output the same as the following code that works today:

```pwsh
trap {
    "hello"
    continue
}
Get-Process IDontExist -ErrorAction Stop
```

If you wanted to handle both terminating and non-terminating errors in your
`trap`
you can use the
`ErrorType`
property:

```pwsh
trap {
    if($_.ErrorType -eq "Terminating") {
        "oh no!"
    } else {
        "this is ok"
        continue
    }
}
```

## Motivation

The Trap statement has fallen out of favor in recent years to its error handling cousin -
the try/catch block.

This gives us an opportunity to revamp the Trap statement so that it can serve a purpose that the try/catch could not serve.

The purpose would be to offer a
*single solution*
to handle both terminating and non-terminating issues.

## Specification

`ErrorRecord`
will get an additional property called
`ErrorType`
this will come from an
`enum`
called
`ErrorRecordType`
which has 2 values:

* `Terminating`
* `NonTerminating`

Traps will then also be passed non-terminating errors to be handled. 

## Alternate Proposals and Considerations

### Introduce a `trapall`

Instead of changing the behavior of
`trap`,
we would add a
`trapall`
that does the above behavior.
This might get tricky if a
`trap`
and a
`trapall`
were defined.
This also means that code written using
`trapall`
would not run at all in older versions of PowerShell.

### Add a -OnError parameter to every cmdlet

The motivation of this RFC comes from the issue
[PowerShell#6010](https://github.com/PowerShell/PowerShell/issues/6010).

 The proposal there was a parameter that you pass a ScriptBlock to:

 ```pwsh
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
