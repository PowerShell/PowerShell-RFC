---
RFC: RFC<four digit unique incrementing number assigned by Committee, this shall be left blank by the author>
Author: Anam Navied
Status: Draft
SupercededBy: N/A
Version: 1.0
Area: Microsoft.PowerShell.PSResourceGet
Comments Due: 2026-06-20
Plan to implement: Yes
---

# Title

Cross-feed Dependency Support when Microsoft Artifact Registry is Default Trusted Repository for PSResourceGet

This RFC addresses how cross-feed dependencies will be supported when Microsoft Artifact Registry (MAR) is registered as a default repository. Today, MAR hosts 213 unique PowerShell modules - including the `Az` module family - and is expected to grow to include additional Microsoft owned modules.

Starting with PSResourceGet version `1.3.0-preview1`, MAR will be registered as the default repository with highest priority. The next step, in future 1.3.x releases, is to shift retrieval of Microsoft-owned modules from PSGallery to MAR, if the module is a dependency. This concept of cross-feed dependency arises when a module, such as `Aztools` (community-owned, hosted on PSGallery), takes a dependency on some `Az.*` modules (Microsoft-owned, hosted on both PSGallery and MAR). In this situation, PSResourceGet should install `Aztools` from PSGallery, but resolve and install its `Az.*` dependency from MAR to support adoption of MAR.

The user should face minimal disruption in their workflow, while attaining the benefit of getting the `Az.*` dependency modules from a trusted repository source.

## Motivation

    As a user consuming PowerShell community-owned and Microsoft-owned modules,
    I can install a module and if it takes a dependency on a Microsoft owned module which is hosted on Microsoft Artifact Registry (MAR)
    then the dependency is seamlessly installed from MAR with minimal interruption to the overall install flow and minimal extra work required from the user,
    so that the user can get Microsoft-owned modules from the trusted repository.

## User Experience

In this example, the PSResourceGet 1.3.x release containing the cross-feed dependency solution is imported.

The user installs `aztools` and it is installed from `PSGallery`, but the `Az.*` dependencies, which are Microsoft-owned modules hosted on MAR, are installed from MAR.

```powershell
Import-Module "Microsoft.PowerShell.PSResourceGet" -RequiredVersion "1.3.0" # This includes prerelease 1.3.x versions
Install-PSResource -Name "aztools" -Repository PSGallery -PassThru
```

```output
Name                   Version Prerelease Repository Description
----                   ------- ---------- ---------- -----------
Az.Monitor             7.0.0              MAR        Microsoft Azure PowerShell - Monitor service cmdlets for Azure Resource Manager in Windows...
Az.OperationalInsights 3.3.0              MAR        Microsoft Azure PowerShell - Operational Insights service cmdlets for Azure Resource Manager ...
Az.Accounts            5.4.1   preview    MAR        Microsoft Azure PowerShell - Accounts credential management cmdlets for Azure Resource Manager ...
Az.PolicyInsights      1.7.4              MAR        Microsoft Azure PowerShell - Azure Policy Insights cmdlets for Windows PowerShell and PowerShell ...
Az.Compute             11.5.0             MAR        Microsoft Azure PowerShell - Compute service cmdlets for Azure Resource Manager in Windows ...
Az.Automation          1.11.2             MAR        Microsoft Azure PowerShell - Automation service cmdlets for Azure Resource Manager in Windows ...
aztools                1.1.0              PSGallery  Azure PowerShell Tools, as if you really needed them. This module contains a collection of Azure ...
Az.Storage             9.6.2   preview    MAR        Microsoft Azure PowerShell - Storage service data plane and management cmdlets for ...
```

## Specification Proposals
In PSResourceGet’s current design, install proceeds by iterating through repositories in priority order until the requested package is found in one. Dependencies are then resolved by searching only that same repository.

### ExternalModuleDependencies-based approach: 
When installing `aztools` module from PSGallery, `Az.OperationalInsights` would be listed as an external module dependency, so PSResourceGet would not install `Az.OperationalInsights` as part of the current flow. Instead, it would inform the user that the dependency was skipped and must be installed separately.

**Pros:** minimal change required in PSResourceGet (primarily emitting objects and clear messaging for skipped external dependencies). 

**Cons:** module owners must update their module manifest (.psd1) to move `Az.OperationalInsights` from `RequiredModules` to `ExternalModuleDependencies` section, and users must take additional steps (and potentially extra commands) to discover and install the skipped dependencies.

### Repository search pattern modification approach:
When installing `aztools`, PSResourceGet would find the parent package in PSGallery as usual. During dependency resolution, however, it would first search trusted repositories (starting with MAR) for each dependency rather than defaulting to the parent package’s source repository. If a dependency is found in a trusted repository, it would be installed from there; otherwise, PSResourceGet would fall back to the parent package's source repository.

**Pros**: no module manifest changes for module owners and minimal extra work for users; gives the team more control over MAR adoption; can be rolled back with a corrective release if needed.

**Cons**: requires more PSResourceGet changes and may impact performance because dependency resolution could search through additional repositories. The impact will depend on how many trusted repositories a user has configured. We would first collect performance metrics with only MAR registered as a trusted repository, then extrapolate for 1-2 additional trusted repositories a user may have to assess realistic performance impact.


