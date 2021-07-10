---
RFC: 0004
Author: J. Keith Bankston
Status: Final
Area: PowerShell Gallery, Module Versioning
Comments Due: 5/28/2016
---

# Module Naming and Versioning for PowerShell Gallery

The original naming guidelines for PowerShell DSC resources stated that a module would start out named "x-Something" for experimental to indicate that it was a community-developed module, and that when a module reached a production-ready state, it needed to be renamed to just "Something" (without the x prefix). This was based on faulty assumptions, and the result was that code developed using the experimental module would have to be modified to accommodate the production version.

Aligning the guidelines with Semantic Versioning (SemVer) addresses these issues by using version numbers to describe the state of the code. 

No naming or versioning scheme is perfect. The advantages to aligning with Semantic Versioning are 

* It is well described at [semver.org](http://www.semver.org), and has been broadly adopted. 
* Semantic Versioning covers the states that are of greatest concern
* This approach leaves the names of modules, commands, and resources unmodified as the state changes

## Motivation

When PowerShell DSC was released, the original guidelines for module naming were released along with a set of example DSC resources. At that time, it was believed those modules would be built into Windows, so the names were protected for future use by identifying them with "x" for experimental, with the rest of the name reserved. This plan allowed the modules to be renamed without the "x" prefix when it shipped as part of the Windows OS. 

Unfortunately, the names of modules and DSC resources are built into the code developed by customers who consume those modules. As modules move from experimental to a production-ready state, the developers who follow the original guidelines are faced with the choice of breaking compatibility with previous users by renaming the modules, or staying with a naming convention that indicates the code to be experimental. 

PowerShell has moved to align with more common open source development practices, wherein code is developed in public forums such as [github.com/powershell](https://github.com/powershell), then published to a repository like the [PowerShell Gallery](https://powershellgallery.com). Using  the [PowerShell Gallery](https://powershellgallery.com) as a delivery vehicle allows Microsoft to avoid concerns about reserving a name for use in Windows, which allows more choices for a solution.

## Specification

This RFC proposes the following guideline for the naming and versioning of modules published to the [PowerShell Gallery](https://powershellgallery.com):

* Modules, scripts, commands, and DSC resources should be named as they are expected to be used, regardless of the state (experimental, preview, production, etc.)
* Modules, scripts, commands, and DSC resources should be versioned in a way that aligns with Semantic Versioning, meaning:
  * The version should be structured as 3 numeric blocks separated by periods, as in 0.1.1 or 4.11.192
  Versions starting with "0" indicate that the package is not yet production ready, and the first number should only begin
  with "0" if that is the only number used
  * Changes in the first number (1.9.9999 to 2.0.0) indicate major and breaking changes between the versions
  * Changes to the second number (1.1 to 1.2) indicate feature-level changes, such as adding new cmdlets to a module
  * Changes to the third number indicate non-breaking changes, such as new parameters, updated samples, or new tests
  * When listing versions, PowerShell will sort the versions as strings, so 1.01.0 will be treated as greater than 1.001.0
* PowerShell was created before SemVer was published, so it provides support for most but not all elements of SemVer, specifically:
  * It does not support prerelease strings in version numbers. This is useful when a publisher wishes to deliver a preview
    release of a new major version after providing a version 1.0.0. This will be supported in a future release of the PowerShell Gallery
    and PowerShellGet cmdlets.
* PowerShell and the PowerShell Gallery allow version strings with 1, 2, and 4 segments. Many early modules did not follow the
  guidelines, and product releases from Microsoft include build information as a 4th block of numbers (for example 5.1.14393.1066). From
  a versioning standpoint, these differences are ignored.
* This guideline affects publishing PowerShell modules, commands, scripts, and DSC resources via the [PowerShell Gallery](https://powershellgallery.com), and should not be implied as having broader applicability. 

### Alternate Proposals and Considerations
* Strict adherence to SemVer excludes support for build number. 
  * In the proposal, build number has been added due to unavoidable issues caused by internal Microsoft build systems. 
  * The assumption is that so long as the build number implies no meaning and no change to the item, it can be ignored.
* Dealing with preview versions of major updates
  * This proposal avoids the issue by not providing support in the guidelines for non-production versions released after a 1.0.0 version is delivered via the PowerShell Gallery. 
  * *Alternative 1:* Developers could release a new package to the gallery with a package name indicating the preview state, while the module and code names remained unchanged. Since the package name does not conflict, it could be published to the gallery with any version. The issue that results with this approach is that the preview module would remain in the Gallery long its value as an early view of new work is gone. 
  * *Alternative 2:* Support the SemVer approach of adding strings such as "-Alpha" as part of the Patch element of the version. In this scheme 2.0.0-Preview would indicate a preview release of a major update to some item. This is preferable, but cannot be supported in PowerShell today. Any module versioning scheme we choose must work in PowerShell, which relies on the .Net version checking APIs that only support numbers. That makes this not a viable option today. 
  * *Alternative 3:* Developers could publish the update with a major version of 0, so long as the numbers in Minor.Patch was greater than what existed previously. This option was discussed, but it is not being considered as it has a large number of problems. It does not align with any other versioning scheme, and it has the likelihood of confusing consumers. The PowerShell Gallery does not currently support this, as one can only publish using a version that is higher than what existed before - and 1.0.0 is greater than 0.(anything). The Gallery could be modified to support this, but it does not today. 

#### Open Questions
Is there a better alternative for dealing with preview releases of major versions? 
