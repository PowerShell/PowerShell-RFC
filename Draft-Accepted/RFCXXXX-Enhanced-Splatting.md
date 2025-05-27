---
RFC: RFCXXXX
Author: Jordan Borean
Status: Draft
SupercededBy: N/A
Version: 1.0
Area: Language/Engine
Comments Due: 2025-09-30
Plan to implement: Yes
---

# PowerShell Splatting Enhancements

Splatting is a feature that allows users to build up parameters and values through a dictionary or array object to then bind directly as parameter or positional arguments respectively when invoking a command. The following example shows an equivalent way of specifying the `-Path` and `-Force` parameters with `Get-ChildItem` directly and with splatting as a hashtable.

```powershell
Get-ChildItem -Path C:\Windows -Force

$mySplat = @{
    Path = 'C:\Windows'
    Force = $true
}
Get-ChildItem @mySplat
```

The splat syntax is limited today to just a simple variable name to splat. This RFC is a proposal to extend support for splatting to include:

+ inline hashtable and array splatting
+ splatting a member value
+ splatting the result of a single expression `(...)`

What is not covered, but is still considered, in this RFC is the ability to:

+ splat .NET method arguments
+ inline filtering of hashtable values

## Motivation

    As a PowerShell scripter,
    I can use enhanced splatting,
    so that I can streamline my code and not have to define a variable just to splat and avoid long lines.

## User Experience

Some use cases behind splatting would be to:

+ split up long lines without relying on backticks

```powershell
# +140 chars long
Start-Process -FilePath C:\some\really\long\path.ps1 -ArgumentList 'some argument that is long' -WorkingDirectory C:\Windows -PassThru -Wait

# vs splatting can reduce the horizontal length
$mySplat = @{
    FilePath = 'C:\some\really\long\path.ps1'
    ArgumentList = 'some argument that is long'
    WorkingDirectory = 'C:\Windows'
    PassThru = $true
    Wait = $true
}
Start-Process @mySplat
```

+ passing through parameters in proxy functions

```powershell
Function Get-ProxyValue {
    [CmdletBinding()]
    param (
        ...
    )

    # Manually passing them through
    Get-Value -Param1 $Param1 -Param2 $Param2 -SomeSwitch:$SomeSwitch ...

    # Vs using a splat to do it automatically
    Get-Value @PSBoundParameters
}
```

+ making it easier to document or add/remove parameters when they are per line vs. embedding a string

```powershell
# If I wanted to remove -Param2 I would need to move the cursor to that position and delete the selection
Get-Value -Param1 $Param2 -Param2 $Param2 -SomeSwitch

# With a splat I can just remove the line which most editors have a shortcut or can Shift+End -> Delete
# I can also easily add a comment if need be
$mySplat = @{
    Param1 = $Param1
    # Some extra info providing more context to Param2
    Param2 = $Param2
    SomeSwitch = $true
}
Get-Value @mySplat
```

+ adding/remove values conditionally

```powershell
# Adds -File $somePath if the path exists
$mySplat = @{}
if (Test-Path $somePath) {
    $mySplat.File = $somePath
}
Test-Function @mySplat
```

The two main problems with splatting today are that they only work with a simple variable name and it is not possible to conditionally splat values without having to redefine a new `IDictionary`. For example:

```powershell
$myObjProperty = $myObj.Property
Test-Function @myObjProperty

$splatValue Get-SplatValue
Test-Function @splatValue

# This looks like it might work but it will pass the value positionally
# and not splat them
Test-Function @(Get-SplatValue)

Function Get-ProxyValue {
    [CmdletBinding()]
    param (
        [Parameter()]
        [string]
        $ProxyParameter,

        [Parameter()]
        [string]
        $InnerValue
    )

    # Makes a top level copy of the parameters and removes
    # ProxyParameter if present
    $innerSplat = [hashtable]$PSBoundParameters
    $innerSplat.Remove('ProxyParameter')

    if ($ProxyParameter) {
        # Do something in the proxy
    }

    Get-Value @innerSplat
}
```

For all these examples, we require a variable that is an `IDictionary` or `IList` to splat either through named parameters or positionally respectively. By expanding splatting functionality, we could avoid such scenarios by providing ways to do more complex splatting without the intermediate variable.

+ we need to name the variable something/add clutter to the code
+ we actually need a variable to be stored in the session state, takes up a slot in the function's tuple store
+ tab completion/prediction is more complicated as it needs to scan additional lines to try and find the command it is used for

## Specification

The proposed solution to this problem is to add a new pseudo parameter `-@` called the splatting parameter. This parameter is treated specially during the scriptblock compilation to mark the subsequent argument to be splatted. For example

```powershell
# Simple var
Test-Function -@ $ht
Test-Function -@ $array

# Member value
Test-Function -@ $obj.Property
Test-Function -@ $obj.Method()
Test-Function -@ $item['Key']
Test-Function -@ $item["Key"]
Test-Function -@ $item[$keyInVar]

# Inline hashtable
Test-Function -@ @{
    Path  = 'C:\path'
    Force = $true
}

# Inline array
Test-Function -@ @(
    'Positional 1'
    'Positional 2'
)

# Inline expression
Test-Function -@ (Get-SplatValue)

# Combination
Test-Function -@ $ht -@ @{
    Path = 'C:\path'
} -Force
```

In terms of backwards compatibility, it is not currently possible to use `-@` as a parameter without splatting:

```powershell
Function Test-Function {
    [CmdletBinding()]
    param (
        [Parameter()]
        ${@}
    )

    ${@}
}

Test-Function -@ value
# Test-Function: A positional parameter cannot be found that accepts argument 'value'
# '-@' is bound to the -@ parameter positionally and value is a second positional param

$splat = @{
    '@' = 'value'
}
Test-Function @splat
# value
# This works as only splatting can bind to '${@}'
```

As demonstrated above binding to an existing parameter `-@` does not work without splatting as `-@` is treated as a positional argument. This does have some potential backwards compatibility concerns for non-advanced functions or native commands where positional parameters are a bit more common. This is covered more under [considerations](#considerations).

The advantages of this approach are:

+ no Ast/Parsing changes are needed
+ `-@` cannot be used as a parameter today without cumbersome workarounds
+ the syntax is consistent across the various expression types; inline hashtables/arrays, function output, member properties/methods
+ `-@` can be used multiple times to splat multiple values
+ already precedence with "special" parameters, `-?` used for help

## Considerations

There are three main considerations with this approach:

+ behaviour change of bare `-@` as positional argument
+ behaviour when `-@` is used in a splatted value

```powershell
Test-Function -@ @{
    '@' = @{ Path = 'abc' }
}
```

+ behaviour when `-@` is used as a default parameter value

```powershell
$PSDefaultParameterValues['Test-Function:@'] = @{ Param = 'Value' }
Test-Function
```

### -@ as positional argument

The biggest concern with this change is that a bare `-@` is treated as a positional argument. For advanced functions this is less problematic, but for non-advanced functions or native commands it could be possible that the caller would want to pass in `-@` as a positional value, that is:

```powershell
# These work today with the positional args becoming @('-@',  'abc')
Invoke-MyFunction -@ abc
./native_command -@ abc
```

With the proposed changes the above code would have a behaviour change and the value `abc` will be treated as the value to splat. The only way to pass `-@` as a positional argument would be to either quote the value or splat it:

```powershell
Invoke-MyFunction '-@' abc
./native_command '-@' abc

Invoke-MyFunction -@ @('-@') abc
```

When looking through GitHub to find [using a bare -@ argument](https://github.com/search?q=-%40+language%3Apowershell&type=code) I found 4 examples:

+ https://github.com/helpimnotdrowning/PwshUtils/blob/c48f59dfa21789ad39ac4fc40fd6704fa81cdf64/ArchiveUtils.ps1#L68
+ https://github.com/TomasForte/ps_scripts/blob/0923c7bae7f8f257eb3bcedd75380ae44082d9d1/missingsource.ps1#L4
+ https://github.com/TomasForte/ps_scripts/blob/0923c7bae7f8f257eb3bcedd75380ae44082d9d1/load_images.ps1#L50
+ https://github.com/walnuts1018/Powershell/blob/5e45b20f0ca0892a673fd80d7c220e65c42df01f/Microsoft.PowerShell_profile.ps1#L52

All of them were using `-@` as a positional argument for a native command and not a PowerShell function.

There are 3 possible options we could do for this scenario:

+ document this might be a breaking change and encourage people to quote the `-@` which works on all older versions
+ put these new features behind an experimental feature so people can opt into it before making it on by default in a future release
  + this allows people to become aware that their code should change
  + telemetry could be used to see how common this occurrence is for native commands
+ do not support `-@` splatting for native commands
  + native commands are less likely to need splatting as inline arrays are already splatted
  + could possibly also be skipped for non-advanced functions but I would not recommend that

### -@ in splat value

With `-@` being treated as a parameter, it may be assumed that the value of `@` in a splat would also be splatted. This proposal instead treats `@` as just any other parameter to keep backwards compatibility with how `${@}` as a parameter can only be specified through a splat. The final implementation could potentially instead

+ emit an error if `@` is found in a splat and state it is not allowed, or
+ recursively splat each occurrence of `@` found in a splatted value

The first option should not be too hard to achieve but it would be different to how `@` as a key in a variable splat works today potentially causing confusion. The recursion option has the same problem but would also be quite hard to implement. As `-@` can be specified multiple times I do not see the value in trying to add recursive support for splatting `@` inside an existing splat.

### -@ as default parameter value

Like with `@` inside a splat, it is possible today to use `$PSDefaultParameterValues` to specify the default value for the `@` parameter:

```powershell
Function Test-Function {
    [CmdletBinding()]
    param (
        [Parameter()]
        ${@}
    )

    ${@}
}

$PSDefaultParameterValues['Test-Function:@'] = 'value'
Test-Function
# value
```

This behaviour in the RFC implementation is unchanged from how it works today where `:@` is bound to the `${@}` parameter. The problem is the same as using `@` inside a splat value so the chosen direction from that should also apply here.

## Alternative Proposals

See https://github.com/jborean93/PowerShell-RFC/pull/1 for a more in depth proposal with alternative options.

### Using @@{k='value'}

In summary there were 4 other choices:

|Option|Variable|Variable Member|Hashtable Literal|Array Literal|Expression|
|-|-|-|-|-|-|
|1. `@@<expr>`|`@var`|`@var.Member`|`@@{Key='Value'}`|`@@('Positional])`|`@&{...}`|
|2. `..<expr>`|`..$var`|`..$var.Member`|`..@{Key='Value'}`|`..@('Positional)`|`..(...)`|
|3. `@[<expr>]`|`@[$var]`|`@[$var.Member]`|`@[@{Key='Value'}]`|`@[@('Positional')]`|`@[...]`|
|4. `-splat` operator|`(-splat $var)`|`(-splat $var.Member)`|`(-splat @{Key='Value'})`|`(-splat @('Positional'))`|`(-splat (...))`|

This is how I would rank each option from 1-5 with the following categories:

+ Intuitiveness - How intuitive is the option
+ Usability - How much typing is needed
+ Consistency - How consistent is the splat options between all the scenarios
+ Impact - The risk (5 being lowest, 1 being highest) of this option breaking existing scripts

|Option|Intuitiveness|Usability|Consistency|Impact|Total|
|-|-|-|-|-|-|
|0. `-@` param|4|3|5|2|14|
|1. `@@<expr>`|5|4|1|3|13|
|2. `..<expr>`|1|5|3|1|10|
|3. `@[<expr>]`|3|2|2|5|12|
|4. `-splat` operator|2|1|4|4|11|

The most credible alternative would be option 2 which is to double up on the `@` to specify and inline hashtable or expression. While it somewhat of an evolution of the current splatting syntax I rejected it over the `-@` param due to a few reasons:

+ the syntax for splatting an expression result is not consistent
  + no double up on `@@` like the inline hashtable/array
  + cannot use `@$(...)` as expected because this is valid syntax today
+ it does not look great to double up on symbols in code
+ could lead to confusion where someone is wondering about the differences between `@{k='v'}` vs `@@{k='v'}`
+ it would require Ast/parsing changes making it harder to conditionally use with older pwsh versions, the below would not work

### Using -PSSplat/-Splat vs -@

Another alternative is to use a proper parameter name like `-PSSplat` or `-Splat` instead of `-@`. This is more explanatory of the parameter's purpose but has two main disadvantages:

+ it is longer to type, even more so when doing multiple splats
+ has a higher risk of impacting existing code which might use one of these parameters in their param block
