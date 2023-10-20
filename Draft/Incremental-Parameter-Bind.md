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

Suppose a user wants to write a Get-LatestAvailableMicrosoftEdgeVersion function using data available from https://edgeupdates.microsoft.com/api/products

The data is JSON but is structured as followed

1. Product: Stable
   - Platform: Windows
     - Architecture: x64
       - msi: 118.0.2088.57
     - Architecture: arm64
       - msi: 118.0.2088.57
     - Architecture: x86
       - msi: 118.0.2088.57
   - Platform: Linux
     - Architecture: x64
       - rpm: 118.0.2088.57
       - deb: 118.0.2088.57
   - Platform: MacOS
     - Architecture: universal
       - pkg: 118.0.2088.57
       - plist: 118.0.2088.57
   - Platform: Android
     - Architecture: arm64
       - Version: 118.0.2088.58
   - Platform: iOS
     - Architecture: arm64
       - Version: 118.0.2088.60

2. Product: Beta
   - Platform: iOS
     - Architecture: arm64
       - Version: 118.0.2088.36
   - Platform: Linux
     - Architecture: x64
       - rpm: 119.0.2151.12
       - deb: 119.0.2151.12
   - Platform: Windows
     - Architecture: arm64
       - msi: 119.0.2151.12
     - Architecture: x64
       - msi: 119.0.2151.12
     - Architecture: x86
       - msi: 119.0.2151.12
   - Platform: MacOS
     - Architecture: universal
       - plist: 119.0.2151.12
       - pkg: 119.0.2151.12
   - Platform: Android
     - Architecture: arm64
       - Version: 119.0.2151.11


To get the version number the function would require the following parameters:
- Product
- Platform
- Architecture (Not Applicable for Android and iOS)
- PackageType (Applicable for Linux to disambiguate rpm/deb and Windows to disambiguate msi/exe)


With this proposed change, once the user selects a Product and Platform, Architecture and PackageType options are conditionally required based on the API data.

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

1. **Explicit Flag for Incremental Binding**: Instead of changing the default behavior, introduce an `IncrementalBind` flag to `CmdletBindingAttribute` or create an `IncrementalBindingAttribute` that enables incremental binding. This would allow script authors to opt-in to the new behavior explicitly.

2. **Expose BindParameter method in dynamicparam block**: Syntax could look like
```powershell
dynamicparam
{   
    $ProductParam = [RuntimeDefinedParameter]::new('Product', [string], $productParamAttributes)
    
    $_.BindParameter($ProductParam)
    $Product = $PSBoundParameters['Product']
    # or
    $Product = $this.BindParameter($ProductParam)
    # or
    if($_.TryBindParameter($ProductParam, out $Product))
    {
        ...
    }
}
```

1. **Performance Considerations**: Depending on the implementation, there could be performance implications. The exact performance impact should be assessed during the development phase, and optimizations should be considered to ensure that this feature doesn't introduce significant overhead.

2. **Discoverability and Documentation Considerations**: Perhaps the most reasonable objection to this RFC is discoverability of incrementally bound parameters by Get-Help. For this I propose we either:
   - Do nothing as this is already a problem with dynamic parameters that are conditionally present based off the bound value of static parameters
   - `Get-Help` could append `This function has dynamic parameters and may require additional parameters not documented here, supply -ArgumentList parameter for additional details` to commands that implement dynamic/incrementally bound parameters

   The syntax could look like `Get-Help MyIncrementallyBindableFunction -Product Edge`

   This is similar to --help or /h options for many command line tools where you can get details about sub-commands.

   This would be relatively easy to wire up as
       1. Get-Help doesn't have any parameters with ValueFromRemainingArguments
       2. We could pass -ArgumentList to Get-Command to within Get-Help so the incrementally bound parameters become available

Please note that the above proposal is a starting point and would benefit from feedback, especially concerning discoverability and performance.