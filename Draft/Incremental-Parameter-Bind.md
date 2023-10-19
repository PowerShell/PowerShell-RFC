---
RFC: RFC
Author: Darren Kattan, James Brundage
Status: Draft
Area: Dynamic Parameter Binding
Version: 1.0
Comments Due: 2023-12-19
Plan to implement: Yes
---

# Incremental Binding of RuntimeDefinedParameter in Dynamicparam

Facilitate PowerShell's dynamic parameter block by allowing `RuntimeDefinedParameter` objects to bind arguments incrementally. This will enhance the ability to use previously bound values to refine subsequent parameters dynamically.

## Motivation

    As a PowerShell script or module author,
    I can use the bound values of one parameter to filter the choices for subsequent parameters,
    so that I can offer dynamic and contextual choices to users and enhance the user experience.

## User Experience

Suppose a user wants to interactively install different versions of Microsoft Edge, filtered by Product, Platform, and Architecture. With this proposed change, once the user selects a Product, the subsequent options for Platform are filtered based on that selection. Once a Platform is selected, the Architecture choices are then filtered, and so forth. 

The sample code below demonstrates how this experience _might_ look with the proposed changes:

```powershell
$EdgeVersions = Invoke-RestMethod https://edgeupdates.microsoft.com/api/products

$productParamAttributes = [Collection[Attribute]]@( 
    ([ValidateSet]::new($EdgeVersions.Product)),
    ([Parameter]@{ Mandatory = $true })
)
[RuntimeDefinedParameter]::new('Product', [string], $productParamAttributes)

if($Product)
{
    $EdgeVersions = $EdgeVersions | ?{$_.Product -eq $Product}
    $Platforms = $EdgeVersions | %{ $_.Releases } | select -Unique -Expand Platform 

    $platformParamAttributes = [Collection[Attribute]]@( 
        ([ValidateSet]::new($Platforms)),
        ([Parameter]@{ Mandatory = $true })
    )
    [RuntimeDefinedParameter]::new('Platform', [string], $platformParamAttributes)

    if($Platform)
    {
        $EdgeVersions = $EdgeVersions | ?{$_.Releases.Platform -eq $Platform}                
        $Architectures = $EdgeVersions.Releases.Architecture | select -Unique | sort

        $architectureParamAttributes = [Collection[Attribute]]@( 
            ([ValidateSet]::new($Architectures)),
            ([Parameter]@{ Mandatory = $true })
        )
        [RuntimeDefinedParameter]::new('Architecture', [string], $architectureParamAttributes)

        if($Architecture)
        {
            $EdgeVersions = $EdgeVersions | ?{$_.Releases.Architecture -eq $Architecture}
            $versionToInstallValues = $EdgeVersions.Releases.ProductVersion | select -Unique

            $versionParamAttributes = [Collection[Attribute]]@( 
                ([ValidateSet]::new($versionToInstallValues)),
                ([Parameter]@{ Mandatory = $true })
            )
            [RuntimeDefinedParameter]::new('Version', [string], $versionParamAttributes)
        }
    }
}
```
Upon execution, users will be prompted incrementally:

```output
Product: [List of Products]
Platform: [Filtered List of Platforms]
Architecture: [Filtered List of Architectures]
Version: [Filtered List of Versions]
```

The syntax could be even more concise with user-supplied helper function `New-Parameter`
```powershell
function New-Parameter {
    param(
        [string]$Name,
        [Type]$Type = [object],
        [string[]]$ValidValues,
        [switch]$Mandatory
    )

    $attributes = [Collection[Attribute]]@(
        ([ValidateSet]::new($ValidValues)),
        ([Parameter]@{ Mandatory = $Mandatory })
    )

    return [RuntimeDefinedParameter]::new($Name, $Type, $attributes)
}

Function Get-EdgeUpdates
{
    param()
    dynamicparam
    {   
        $EdgeVersions = Invoke-RestMethod https://edgeupdates.microsoft.com/api/products
        New-Parameter -Name Product -ValidValues $EdgeVersions.Product -Type ([string]) -Mandatory      
        if($Product)
        {
            $EdgeReleases = $EdgeVersions | ?{$_.Product -eq $Product} | select -Expand Releases # Beta,Stable,Dev,Canary
            New-Parameter -Name Platform -ValidValues ($EdgeReleases.Platform | select -Unique) -Type ([string]) -Mandatory # Windows,Linux,MacOS,Android
            if($Platform)
            {
                $EdgeReleases = $EdgeReleases | ?{$_.Platform -eq $Platform}                
                New-Parameter -Name Architecture -ValidValues ($EdgeReleases.Architecture | select -Unique) -Type ([string]) -Mandatory # x86,x64,arm64,universal
                if($Architecture)
                {
                    $EdgeReleases = $EdgeReleases | ?{$_.Architecture -eq $Architecture}                
                    New-Parameter -Name Version -ValidValues ($EdgeReleases.ProductVersion | select -Unique) -Type ([string]) -Mandatory    
                }
            }
        }
    }
    end
    {
    }
}
```

## Specification

1. **Incremental Binding**: As `RuntimeDefinedParameter` objects are written to the output in the `dynamicparam` block, they are immediately bound if an argument is available. This behavior differs from the current state, where all parameters are bound after the entire `dynamicparam` block has executed.

2. **Variable Creation**: When a `RuntimeDefinedParameter` is bound, a variable with the name of that parameter should be created in the `dynamicparam` block scope. This makes the bound value immediately available for subsequent logic in the `dynamicparam` block.

3. **Backward Compatibility**: This change must not break scripts or modules that use the `dynamicparam` block but do not expect incremental binding. It's crucial to ensure the new behavior doesn't adversely affect existing implementations.

## Alternate Proposals and Considerations

1. **Explicit Flag for Incremental Binding**: Instead of changing the default behavior, introduce a. IncrementalBind flag to `CmdletBindingAttribute` or create an `IncrementalBindingAttribute` that enables incremental binding. This would allow script authors to opt-in to the new behavior explicitly.

2. **Dynamic Binding Callbacks**: Another approach might be to allow a scriptblock callback to be attached to a `RuntimeDefinedParameter`. This callback would execute when the parameter is bound, providing more granular control over the binding process.

3. **Performance Considerations**: Depending on the implementation, there could be performance implications. The exact performance impact should be assessed during the development phase, and optimizations should be considered to ensure that this feature doesn't introduce significant overhead.

Please note that the above proposal is a starting point and would benefit from feedback, especially concerning backward compatibility and potential pitfalls not identified in this draft.