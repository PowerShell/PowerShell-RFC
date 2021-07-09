---
RFC: RFC0013
Author: Keith Bankston, Manikyam Bavanda
Status: Final
Version: 1.0.0
Area: PowerShellGet, PowerShell Gallery
Comments Due: 12/23/2016
---

# PowerShellGet and PowerShell Gallery Pre-Release Version Support

Based on feedback from module authors, there is no consistent approach to identifying modules in the PowerShell Gallery that are pre-releases. 
Module consumers have requested the ability to automate acquisition of the latest stable version of PowerShell Gallery items in a way that excludes pre-release versions, unless they specify that they want pre-releases.  

SemVer has defined a convention to solving this, which uses a string following the 3rd segment of the version, as in "2.7.0-alpha".
RFC0004 proposes that PowerShell to align as much as possible with SemVer, but at this time the core PowerShell module handling does not support strings in the version identifier, and adding that support would be a breaking change. 

The PowerShell Gallery and related PowerShellGet cmdlets need to provide solutions for PowerShell versions 3 and up. 
Solutions for the PowerShell Gallery cannot require a breaking change in the core PowerShell module versioning, or the module manifest keys. 

This proposal is specific to PowerShell Gallery, and the PowerShellGet cmdlets.
It will use a new "PreRelease" string that MAY be added to the PSData section of a module manifest which indicates the item is pre-release. 
This proposal shows how the PowerShellGet cmdlets would leverage this metadata to allow publishers and consumers to work with pre-release items from the PowerShell Gallery in a predictable way.  

## Motivation

As a module publisher, I can release a preview version of a major update to my PowerShell code via the PowerShell Gallery that is clearly identifiable as not yet final, and associated with a major version change.
This allows me to gather feedback on the update from users who are willing to work with a pre-release, without impacting the users who rely on stable versions of my PowerShell Gallery items.

As a consumer of PowerShell Gallery artifacts, I want to know that by default the items I find on the Gallery are stable versions.
I want to have the ability to request pre-release versions of items, particularly from trusted publishers who have requested my feedback.
Having control over when I do and do not see pre-release code will enable me to have greater faith that the items I acquire from the Gallery are believed by their publishers to be stable. 

## Specification

__Key Design Considerations__

The authors of this RFC considered the following elements as important to the design:

* Provide a mechanism for module authors to identify an item published to the PowerShell Gallery is a pre-release item. 
* Align with the SemVer convention. 
* Module authors MUST be able to mark any item they can publish to the gallery as a pre-release.
* Backwards compatibility with PowerShell versions 3, 4, and 5 is a high priority for the PowerShell Gallery and PowerShellGet. 
* Having a solution support repositories based on Nuget version 2 server is a high priority, as those are frequently used for on-premise deployments.
* The solution needs to include the impact on the PowerShellGet cmdlets for Find-, Save-, Install-, Update-, and Get-Installed (item type), where item type may be modules, scripts, or other item types supported by PowerShellGet cmdlets.  

__Details for Specifying PreRelease Items__

Modules to be published in the PowerShell Gallery MAY specify that it is a PreRelease version by adding a field to the PrivateData section of the item manifest as follows:

	PrivateData = @{
	    PSData = @{
	       PreRelease = 'SomeString'
	    }
	}

The PreRelease string in PSData MUST follow the [SemVer 1.0.0](http://semver.org/spec/v1.0.0.html) definition, and contain characters 0-9a-zA-Z only, with the exception that a hyphen ("-") MAY be added as the first character of the string. 
Multiple prerelease versions of an item will be sorted in lexicographical order using the PreRelease string. 
Module authors MAY, and are encouraged to, use the strings "alpha001", "beta001", and "rc001" to indicate less stable (alpha) to most stable (rc) pre-release versions, and to allow them to publish multiple iterations of a pre-release tagged with alpha, beta, or rc. 

An example of a module manifest declaration that includes this option would be similar to:

	#
	# Example module manifest based on the module 'PSScriptAnalyzer'
	#
	@{
	# Author of this module
	Author = 'Microsoft Corporation'
	...
	# Version number of this module.
	ModuleVersion = '2.0.0'
	...
	PrivateData = @{
	    PSData = @{
	       PreRelease = '-alpha001'
	    }
	}

Script authors MAY identify a script as a pre-release by modifying the 3rd element of the version in the PSScriptInfo comments to include a pre-release string.
For scripts, the pre-release string MUST follow the [SemVer 1.0.0](http://semver.org/spec/v1.0.0.html) definition, and be part of the 3rd element, begin with a hyphen, after the initial hyphen contain at least 1 other character and include only characters 0-9a-zA-Z. 
An example is shown below:

	<#PSScriptInfo
	
	.VERSION 1.1.0-alpha003
	
	.GUID ...

This pre-release string will be treated the same way as the PreRelease value in PSData for a module. 

PreRelease can only be specified if the version of the item has at least 3 elements (examples: version 1.0.0, version 1.0.0.14278). 
Module authors MUST NOT specify the PreRelease field if they are using a 2-element version string, as this will constitute an error when the module is published to the PowerShell Gallery. 
Script authors MUST NOT specify a PreRease string on anything other than the 3rd element of the version. 

When an item is published to the PowerShell Gallery, the manifest is parsed.
At that time, if PreRelease is properly specified, the string will be extracted and added to the metadata attached to the Nuget Package (.nupkg). 

Authors MAY, for any module or script, specify a version that begins with 0, which SemVer says is in initial development.
In SemVer, PreRelease items are identified separately from items in initial development. 
Authors MAY, and are encouraged to, add a PreRelease identifier for items with a version number that begins with 0. 

__Details for PowerShell Gallery Impact__

The impact of this change will only be for items that include a PreRelease string that is defined in the PSData section of the manifest or in the version string in the PSScriptInfo.
There will be no impact to existing items that do not include a PreRelease identifier. 

The PowerShell Gallery UI will clearly identify pre-release items on the item page with a highlighted UI element saying something like:

_This item is identified as a pre-release by the module author_
 
When displaying an item that specifies a PreRelease string, the PowerShell Gallery will include the string in the version number shown, as in:

__ContosoServer__ _3.02.0-alpha001_

The PowerShellGallery displays previous versions of a module in sorted order, and will include pre-release versions in the list.
The PreRelease string will be included in the sort criteria as a part of the version string, as in the following:

Version History

| Version Title | Downloads | Last Updated |
| ------------- | --------- | ------------ |
| ContosoServer 3.20.0-alpha001       _this version_ | 15 | Wednesday, November 30 2016 |
| ContosoServer 3.19.32       _stable version_ | 327 | Wednesday, November 02 2016 |
| ContosoServer 3.19.31       _stable version_ | 150 | SomeDay, October 13 2016 |
| ContosoServer 3.10.-rc002       _pre-release version_ | 15 | SomeDay, SomeMonth ## #### |
| ContosoServer 3.10.-rc001       _pre-release version_ | 15 | SomeDay, SomeMonth ## #### |
| ContosoServer 3.10.-beta001       _pre-release version_ | 15 | SomeDay, SomeMonth ## #### |
| ContosoServer 3.10.-alpha00       _pre-release version_ | 15 | SomeDay, SomeMonth ## #### |


__Details for PowerShellGet Impact__

Users of the PowerShellGet cmdlets who wish to see or interact with items identified by the author as PreRelease MUST specify -AllowPreRelease with the PowerShellGet Find-, Install-, Save-, and Update- cmdlets in order for the Gallery to return PreRelease items.
Users with older versions of the cmdlets will not be able to see PreRelease items, as the -AllowPreRelease flag is not available.

Using any of the version specifiers (-RequiredVersion, -MinimumVersion, -MaximumVersion), with a pre-release string will generate an error informing the user that specifying a PreRelease string without specifying -AllowPreRelease is not allowed. 

An example using Find-Module, based on the gallery having ContosoServer versions 0.1.0, 1.0.0 and 1.1.0-alpha would look _very similar to_ the following:


	> Find-Module -Name ContosoServer
	Version        Name                    Repository           Description
	-------        ----                    ----------           -----------
	1.0.0          ContosoServer           PSGallery            ContosoServer module
	
	
	> Find-Module -Name ContosoServer -AllowPrerelease
	Version        Name                    Repository           Description
	-------        ----                    ----------           -----------
	1.1.0-Alpha    ContosoServer           PSGallery            ContosoServer module
	
	
	> Find-Module -Name ContosoServer -RequiredVersion '1.1.0-Alpha'
	Error: Specifying a version with a pre-release string, without specyfing -AllowPreRelease, is not allowed.


	> Find-Module –Name ContosoServer -AllVersions
	Version        Name                    Repository           Description
	-------        ----                    ----------           -----------
	1.0.0          ContosoServer           PSGallery            ContosoServer module
	
	
	> Find-Module -Name ContosoServer -AllowPrerelease -AllVersions
	Version        Name                    Repository           Description
	-------        ----                    ----------           -----------
	1.1.0-alpha    ContosoServer           PSGallery            ContosoServer module
	1.0.0          ContosoServer           PSGallery            ContosoServer module
	0.1.0          ContosoServer           PSGallery            ContosoServer module

The metadata available to the PowerShellGet cmdlets Find-, Save-, Install-, Update-, and Get-Installed (item type) will include a PreRelease property that can be used as a filter.
This will allow someone to, for example, enumerate all modules in the PowerShell Gallery by doing something like:

	> Find-Module -AllowPrelrelease| Select-Object -Property PreRelease 

PowerShell version 5 and above support side-by-side installation of multiple versions of the same module by adding a folder with the version number to the folder hierarchy.  
The pre-release string will not be included in the version portion of the folder structure when a pre-release module is installed.  
As an example, installing the modules listed in the example immediately above would result in a folder structure like the following:

	C:\Program Files\WindowsPowerShell\Modules\ContosoServer\1.1.0
	C:\Program Files\WindowsPowerShell\Modules\ContosoServer\1.0.0
	C:\Program Files\WindowsPowerShell\Modules\ContosoServer\0.1.0

Note that this means it is not possible for multiple pre-release modules to be installed side-by-side, regardless of the PreRelease string identified.
If a user attempts to __install__ version 1.1.0-beta001 when they have version 1.1.0-alpha009, the installation will be equivalent to trying to do an in-place update and will fail because the versions are considered to be the same.
Users must add a -Force to accomplish this example, so the command would look like:

	Install-Module ContosoServer -AllowPrelrelease -Force

Update-Module will also support -AllowPreRelease. 
A user wishing to upgrade to version 1.1.0-beta001 from version 1.1.0-alpha009 can use the following command:
 
	Update-Module -AllowPreRelease

Output from Get-InstalledModule MUST include the PreRelease property as well, if it is specified by the module author.
This allows administrators to find any installed modules that are identified as PreRelease versions, and take subsequent actions as needed. 

	> Get-InstalledModule -Name ContosoServer | fl Name, Version, PreRelease 
	# Below metadata properties are also available on Find-Module metadata.
	Name    : ContosoServer
	Version : 1.1.0
	PreRelease : -alpha

Users MUST add -AllowPreRelease to see any pre-release items, even if a version they have supplied looks like a pre-release version.
Users MAY specify a version number that includes a prerelease identifier with -RequiredVersion, -MaximumVersion, and -MinimumVersion, but only if they specify -AllowPrelrelease.
 

## Alternate Proposals and Considerations

* Implementing SemVer across all PowerShell module handling:
	* Supported preview versioning in the core PowerShell module handling was considered, and might or might not be done in future releases of PowerShell Core. 
That work does not solve the problem for handling modules in the PowerShell Gallery, as is a breaking change that it cannot be applied to existing versions of PowerShell that are supported by the PowerShell Gallery and PowerShellGet. 
* Regarding SemVer 1.0.0 support (and not 2.0.0): 
	* At this time, we are only planning to support SemVer 1.0.0.
The implication of this is that we will not support periods in PreRelease strings, and we will not support build metadata. 
	* Neither of these are listed as required in SemVer 2.0.0. 
	* The reason for the PowerShell Gallery to take this approach is that users frequently deploy Nuget-based repositories as an on-premise PowerShell Gallery, and Nuget only supports SemVer 1.0.0. 
If we supported these new elements of SemVer 2.0.0, items could be posted to the PowerShell Gallery that would not be usable by on-premise Nuget repositories without requiring users to modify the items. 
* Regarding what is defined as "preview":
	* Only items that include a valid PreRelease string in PSData will be considered Preview for the PowerShellGet cmdlets and the PowerShell Gallery.
This is consistent with SemVer, which treats versions starting with 0 as initial development, and describes PreReleases separately.  
	* _An alternate design would be to treat versions 0.(anything) as PreRelease regardless of whether or not there is a PreRelease string defined. 
The reason this alternate approach was not taken is that it would change the behavior for large numbers of existing modules that have the highest version as 0.(something), and also because SemVer treats initial development separately from PreRelease._
* Regarding folder structure for module installations
	* This spec assumes the PreRelease string will not be added to the version number in the folder structure.
	The primary reason is that this would be incompatible with PowerShell version 5. 
	* The impact is that side-by-side installation of multiple pre-release versions is not supported. 
	This means that installing a newer pre-release version when an older pre-release exists will follow the same rules as any Update-Module or Install-Module command where the user specifies the same version that is already on the local system (which means -Force is required). 
	The newer version will over-write the previously-installed version in that instance. 
	In updated versions of the PowerShellGet cmdlets, a verbose message will be displayed to indicate this had occurred.
* Filtering out pre-releases happens on client
	* This RFC was initially written incorrectly stating that the filtering is in the REST layer.
	That has been corrected in version 0.3.0 of this RFC.
	* The reason for doing this on the client (in PowerShell Get) stems from the fact that PowerShell Gallery and PowerShellGet are based on Nuget, which provides the filtering for PreRelease items. 
	Nuget handles this in the client layer, not in the REST API.
	
