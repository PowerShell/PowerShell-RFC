---
RFC: RFC
Author: Joel Francis
Status: Draft
SupercededBy:
Version: 1.0
Area: Engine
Comments Due: 2018-12-31
Plan to implement: Yes
---

# Granular Usage for ConfirmImpact

Currently, `ConfirmImpact` is implemented _purely_ at the cmdlet level. This is an extremely useful,
though entirely under-used, framework. A majority of the default cmdlets retain the default
`ConfirmImpact` rating of `Medium` &mdash; even harmless cmdlets like `ConvertTo-Json`. For a full
list and discussion of that, see
[PowerShell/PowerShell#8157](https://github.com/PowerShell/PowerShell/issues/8157).

A more granular approach, where the cmdlet author can specify different impact ratings according to
the severity of an action requested by the user, is needed. For example, using `Invoke-Command`
to take an action on the local machine is of no particular impact beyond the given severity of the
contained commands themselves, whereas invoking the same command sequence on 50 or more remote
machines on the domain may lead to undesirable results.

Currently, such a distinction is impossible to provide with `ConfirmImpact`'s present
implementation. Additionally, use of `$ConfirmPreference` to modify the currently-set preference is
at best a subpar solution as it does not address the underlying problem, and the preference
variables _do not_ cross module boundaries, so this method is limited to intra-module use-cases;
even attempting to apply this methodology to a standard cmdlet that is sourced from a module like
`Get-ChildItem` whilst acting in the scope of another module will fail.

## Motivation

    As a user,
    I can utilise ConfirmImpact as a measure of potential harm of a given action,
    so that I can be informed of potentially dangerous actions of a command, while
    being able to ignore less-severe impacts according to my selected
    $ConfirmPreference.

## Specification

The ShouldProcess method and all of its overloads shall be given an optional additional parameter,
which takes its default value from the advanced function or cmdlet's chosen `ConfirmImpact` setting.
This value defaults to a `ConfirmImpact` of `Medium` just the same as a cmdlet that sets the
`SupportsShouldProcess` setting without explicitly specifying its `ConfirmImpact`.

## Alternate Proposals and Considerations

* Potentially support another method to modify the current ConfirmImpact, similarly to how setting $ConfirmPreference would work (although this cannot directly be inherited with inter-module calls.)