---
RFC: RFCnnnn
Author: Kirk Munro
Status: Draft
SupercededBy: 
Version: 0.1
Area: Engine
Comments Due: September 15, 2019
Plan to implement: Yes
---

# Make terminating errors terminate the right way in PowerShell

By default in PowerShell, terminating errors do not actually terminate. For
example, if you invoke this command in global scope, you will see the output
"Why?" after the terminating error caused by the previous command:

```PowerShell
& {
    $string = 'Hello'
    $string.Substring(99)
    'Why?'
}
```

PowerShell has upheld this behaviour since version 1.0 of the language. You can
make the terminating error actually terminate execution of the command by
wrapping the command in try/catch, like this:

```PowerShell
try {
    $string = 'Hello'
    $string.Substring(99)
    'Why?'
} catch {
    throw
}
```

You can also convert the terminating-yet-handled-as-a-non-terminating error
into an actual terminating error like this:

```PowerShell
& {
    $ErrorActionPreference = 'Stop'
    $string = 'Hello'
    $string.Substring(99)
    'Why?'
}
```

In those last two examples, the exception raised by the .NET method terminates
execution of the running command.

The difference between the first example and the workarounds poses a risk to
scripters who share scripts or modules with the community.

In the first workaround, the risk is that end users using a shared resource
such as a script or module may see different behaviour from the logic within
that module depending on whether or not they were inside of a `try` block when
they invoked the script or a command exported by the module. That risk is very
undesirable, and as a result many community members who share scripts/modules
with the community wrap their logic in a `try/catch{throw}` (or similar)
scaffolding to ensure that the behavior of their code is consistent no matter
where or how it was invoked.

In the second workaround, if the shared script does not use
`$ErrorActionPreference = 'Stop'`, a caller can get different behaviour by
manipulating their `$ErrorActionPreference`. The caller should not be able to
manipulate terminating error behavior in commands that they invoke -- that's up
to the command author, and they shouldn't have to use extra handling to make
terminating errors terminate.

Now consider this code snippet:

```PowerShell
New-Module -Name ThisShouldNotImport {
    # The next line uses "Generics" in the type name, when it should use "Generic"
    $myList = [System.Collections.Generics.List[string]]::new()

    function Test-RFC {
        [CmdletBinding()]
        param()
        'Some logic'
        1 / 0 # Oops
        'Some more logic'
    }
} | Import-Module
```

There are several things wrong with this snippet:

1. If you invoke that snippet, the `ThisShouldNotImport` module imports
successfully because the terminating error
(`[System.Collections.Generics.List[string]]` is not a valid type name) does
not actually terminate the loading of the module. This could cause your module
to load in an unexpected state, which is a bad idea.
1. If you loaded your module by invoking a command defined with that module,
you won't see the terminating error that was raised during the loading of the
module (the terminating error that was raised during the loading of the module
is not shown at all in that scenario!), so you could end up with some
undesirable behaviour when that command executes even though the loading of the
module generated a "terminating" error, and not have a clue why.
1. The Test-RFC command exported by this module produces a terminating error,
yet continues to execute after that error.
1. If the caller either loads your module or invokes your command inside of a
`try` block, they will see different behaviour.

Any execution of code beyond a terminating error should be intentional, not
accidental like it is in both of these cases, and it most certainly should not
be influenced by whether or not the caller loaded the module or invoked the
command inside of a `try` block. Binary modules do not behave this way. Why
should script modules be any different?

Now have a look at the same module definition, this time with some extra
scaffolding in place to make sure that terminating errors actually terminate:

```PowerShell
New-Module -Name ThisShouldNotImport {
    trap {
        break
    }
    $myList = [System.Collections.Generic.List[string]]::new()

    function Test-RFC {
        [CmdletBinding()]
        param()
        $callerEAP = $ErrorActionPreference
        try {
            'Some logic'
            1 / 0 # Oops
            'Some more logic'
        } catch {
            Write-Error -ErrorRecord $_ -ErrorAction $callerEAP
        }
    }
} | Import-Module
```

With this definition, if the script module generates a terminating error, the
module will properly fail to load (note, however, that the mispelled type name
has been corrected in case you want to try this out). Further, if the command
encounters a terminating error, it will properly terminate execution and the
error returned to the caller will properly indicate that the `Test-RFC`
command encountered an error. This scaffolding is so helpful that members of
the community apply it to every module and every begin, process and end block
within every function they define within that module, just to get things to
work the right way in PowerShell.

All of this is simply absurd.

Any script module that generates a terminating error in the module body should
fail to import without extra effort, with an appropriate error message
indicating why it did not import.

Any advanced function defined within a script module that encounters a
terminating error should terminate gracefully, such that the error message
indicates which function the error came from, while respecting the callers
`-ErrorAction` preference, without requiring extra scaffolding code to make it
work that way.

Between the issues identified above, and the workarounds that include
anti-patterns (naked `try/catch` blocks and `trap{break}` statements are
anti-patterns), the PowerShell community is clearly in need of a solution that
automatically resolves these issues in a non-breaking way.

## Motivation

As a script, function, or module author,<br/>
I can write scripts with confidence knowing that terminating errors will
terminate those commands the right way, without needing to add any scaffolding
to correct inappropriate behaviours in PowerShell<br/>
so that I can keep my logic focused on the work that needs to be done.

## User experience

The way forward for this issue is to add an optional feature (see: #220) that
makes terminating errors terminate correctly. The script below demonstrates
that a manifest can be generated with the `ImplicitTerminatingErrorHandling`
optional feature enabled, and with that enabled the module author can write the
script module and the advanced functions in that module knowing that
terminating errors will be handled properly. No scaffolding is required once
the optional feature is enabled, because it will correct the issues that need
correcting to make this just work the right way, transparently.

```powershell
$moduleName = 'ModuleWithBetterErrorHandling'
$modulePath = Join-Path `
    -Path $([Environment]::GetFolderPath('MyDocuments')) `
    -ChildPath PowerShell/Modules/${moduleName}
New-Item -Path $modulePath -ItemType Directory -Force > $null
$nmmParameters = @{
    Path = "${modulePath}/${moduleName}.psd1"
    RootModule = "./${moduleName}.psm1"
    FunctionsToExport = @('Test-ErrorHandling')
}

#
# Create the module manifest, enabling the optional ImplicitTerminatingErrorHandling
# feature in the module it loads
#
New-ModuleManifest @nmmParameters -OptionalFeatures ImplicitTerminatingErrorHandling

$scriptModulePath = Join-Path -Path $modulePath -ChildPath "${moduleName}.psm1"
New-Item -Path $scriptModulePath -ItemType File | Set-Content -Encoding UTF8 -Value @'
    # If the next command is uncommented, Import-Module would fail when trying to load
    # this module due to the terminating error actually terminating like it should
    # 1/0 # Oops!
    function Test-ErrorHandling {
        [cmdletBinding()]
        param()
        'Some logic'
        # The next command generates a terminating error, which will be treated as
        # terminating and Test-ErrorHandling will fail properly, with the error text
        # showing details about the Test-ErrorHandling invocation rather than details
        # about the internals of Test-ErrorHandling.
        Get-Process -Id 12345678 -ErrorAction Stop
        'Some more logic'
    }
'@
```

Module authors who want this behaviour in every module they create can use
`$PSDefaultParameterValues` to add the optional feature to the
`-OptionalFeatures` parameter of `New-ModuleManifest` by default.

```PowerShell
$PSDefaultParameterValues['New-ModuleManifest:OptionalFeatures'] = 'ImplicitTerminatingErrorHandling'
```

Scripters wanting the behaviour in their scripts can use #requires:

```PowerShell
#requires -OptionalFeatures ImplicitTerminatingErrorHandling
```

## Specification

Implementation of this RFC would require the following:

### Implementation of optional feature support

See #220 for more information.

### Addition of the `ImplicitTerminatingErrorHandling` optional feature definition

This would require adding the feature name and description in the appropriate
locations so that the feature can be discovered and enabled.

### PowerShell engine updates

The PowerShell engine would have to be updated such that:

* scripts invoked with the optional feature enabled treat terminating errors as
terminating
* scripts and functions with `CmdletBinding` attributes when this optional
feature is enabled treat terminating errors as terminating and gracefully
report errors back to the caller (i.e. these commands should not throw
exceptions)

## Alternate proposals and considerations

### Related issue

[PowerShell Issue #9855](https://github.com/PowerShell/PowerShell/issues/9855)
is very closely related to this RFC, and it would be worth considering fixing
that issue as part of this RFC if it is not already resolved at that time.
