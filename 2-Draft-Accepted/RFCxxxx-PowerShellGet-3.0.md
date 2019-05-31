---
RFC: RFCxxxx
Author: Steve Lee
Status: Draft
SupercededBy: N/A
Version: 0.1
Area: PowerShellGet
Comments Due: 7/30
Plan to implement: Yes
---

# PowerShellGet 3.0 Module

PowerShellGet has enabled an active community to publish and install PowerShell
Modules.
However, it has some architectural and design limitations that prevent it from
moving forward more quickly.
This RFC proposes some significant changes to address this.

> [!NOTE]
> There are no intentions to break existing PSGet v2 users so existing
> users can stay on PSGet v2 and that will continue to work.

## Motivation

    As a PowerShell user,
    I can discover how to install missing cmdlets,
    so that I can be more productive.

    As a PowerShellGet contributor,
    I can more easily contribute code,
    so that PowerShellGet evolves more quickly.

## User Experience

```powershell
PS> Get-Satisfaction
Get-Satisfaction : The term 'Get-Satisfaction' is not recognized as the name of a cmdlet, function, script file, or operable program.
Check the spelling of the name, or if a path was included, verify that the path is correct and try again.
At line:1 char:1
+ Get-Satisfaction
+ ~~~
+ CategoryInfo          : ObjectNotFound: (Get-Satisfaction:String) [], CommandNotFoundException
+ FullyQualifiedErrorId : CommandNotFoundException

Suggestion: You can obtain `Get-Satisfaction` by running `Install-PSResource HappyDays`.

PS> Install-PSResource HappyDays
```

## Specification

### Rewrite of PowerShellGet

PowerShellGet is currently written as PowerShell script with a dependency on PackageManagement.
Proposal is to write PSResource in C# to reduce complexity and make easier to maintain.
In addition, remove dependency on PackageManagement completely as well as dependency on
nuget.exe.
This module will only use REST APIs to get and publish nupkgs.
This module would be shipped in PowerShell 7.

### Side-by-side with PowerShellGet

To ease transition due to the enormity of the breaking change, this module will
have a different names for the cmdlets to allow for side by side use
with the existing PowerShellGet module.
Since the current PowerShellGet 2.x version is a script module and this new one
is a C# based module, they can coexist side-by-side.

### Cmdlet compatibility

Function wrappers will be provided to provide compatibility with the two most often
used cmdlets: `Install-Module` and `Find-Module`.
These cmdlets will output a warning to use the new cmdlets.

### Local cache

Rather than connecting to PSGallery and other repositories for finding modules,
the user must explicitly request the current metadata of
modules from the repositories and that is cached locally.
`Find-PSResource` (see below) works against this local cache.
This will also enable changes in PowerShell to use `Find-PSResource -Type Command` to look
in this cache when it can't find a command and suggest to the user the module to install to
get that command.
This will be a local json file containing only the latest version of each module.
The cache will be stored in the user path.
There is no system cache that is shared.

Example cache entry:

```json
{
  "name": "This is my module",
  "version": "1.0.0.0",
  "type": "module",
  "tags": [
    "Linux",
    "PSEdition_Core",
    "PSEdition_Desktop",
    "AzureAutomation"
  ]
}
```

An example cache with 5000 resources is ~700KB in compressed json form.

### Automatic updating of the cache

On any operation that reaches out to a repository, a REST API will be called to
see if the hash of the cache matches the current cache and if not, a new one
is downloaded.
If the repository doesn't support this new API, it falls back to current behavior
in PSGet v2.

### Repository management

`Register-PSResourceRepository` will allow for registering additional repositories.
A `-Default` switch enables registering PSGallery should it be accidentally removed.
The `-URL` will accept the HTTP address without the need to specify `/api/v2` as
that will be assumed and discovered at runtime.
Support for local filesystem repositories must be maintained.
A `-InstallationPolicy` switch accepts `Trusted` or `Untrusted` (default) indicating
whether to prompt the user when installing resources from that repository.

`Get-PSResourceRepository` will list out the registered repositories.

`Set-PSResourceRepository` can be used to update a repository URL or installation policy.

`Unregister-PSResourceRepository` can be used to un-register a repository.

### Finding resources

Current design of PowerShellGet is to have different cmdlets for each of the
different resources types that it supports:

`Find-Command`
`Find-DscResource`
`Find-Module`
`Find-RoleCapability`
`Find-Script`

Instead, these will be abstracted by a `Find-PSResource` cmdlet.
A `-Type` parameter will accept an array of types to allow filtering.

With support of the generic `PSResource`, this means we can also find and
install arbitrary nupkgs that may only contain an assembly the user wants to
use in their script.

If the local cache does not exist, the cmdlet will call `Update-PSResourceCache`
first and then attempt a search.
However, this cmdlet does not update the cache if one exists.

### Installing resources

`Install-PSResource` will only work for modules.
A `-AllowPrerelease` switch allows installing prerelease versions.
Other types will use `Save-PSResource` (see below).
A `-Repository` parameter accepts a specific repository name.
If `-Repository` is not specified and the same resource is found in multiple
repositories, then the resource is automatically installed quietly from the
trusted repository with the highest version matching the `-Version` parameter
(if specified, otherwise newest non-prerelease version unless `-AllowPrerelease`
is used).
If there are no trusted repositories matching the query, then the newest version
fulfilling the query will be prompted to be installed.
If there are multiple repositories with the same trust level containing the same
version, the first one is used.

`-AllowUntrusted` can be used to suppress being prompted for untrusted sources.
`-AllowDifferentPublisher` can be used to suppress being prompted if the publisher
of the module is different from the currently installed version.
`-AllowReinstall` can be used to re-install a module.

A `-Quiet` switch will suppress progress information.

### Dependencies and version management

`Install-PSResource` can accept a path to a psd1 file (using `-RequiredModulesFile`)
or a hashtable (using `-RequiredModules`) where the key is the module name and the
value is either the required version specified using Nuget version range syntax or
a hash table where `repository` is set to the URL of the repository and
`version` contains the [Nuget version range syntax](https://docs.microsoft.com/en-us/nuget/reference/package-versioning#version-ranges-and-wildcards).

```powershell
Install-PSResource -RequiredModules @{
  "Configuration" = "[1.3.1,2.0)"
  "Pester"        = @{
    version = "[4.4.2,4.7.0]"
    repository = "https://www.powershellgallery.com"
  }
}
```

### Saving resources

With the removal of PackageManagement, there is still a need to support saving
arbitrary nupkgs (assemblies) used for scripts.

`Save-PSResource -Type Library` will download nupkgs that have a `lib` folder.
A `-AllowPrerelease` switch allows saving prerelease versions.
The dependent native library in `runtimes` matching the current system runtime
will be copied to the same destination.

This cmdlet has the same parameters as `Install-PSResource` with the addition
of mandatory `-Path` or `-LiteralPath` parameters.

### Updating resources

`Update-PSResource` will update all resources to most recent minor version by
default.
A `-OnlyMinorUpdates` switch will allow only updating to newer minor version.
If the installed resource is a pre-release, it will automatically update to
latest prerelease or stable version (if available).

### Publishing resources

`Publish-PSResource` will supporting publishing modules and scripts.
Modules will be given as a path to the module folder.
Publishing by name and version is no longer supported.

### Listing installed resources

`Get-PSResource` will list all installed resources with new `Type` column.

```output
Version  Name       Type    Repository  Description
-------  ----       ----    ----------  -----------
5.0.0    VSTeam     Module  PSGallery   Adds functionality for working with Visual …
0.14.94  PSGitHub   Module  PSGallery   This PowerShell module enables integration …
0.7.3    posh-git   Module  PSGallery   Provides prompt with Git status summary …
1.0.1    PSAutoMute Script  PSGallery   Powershell script to Auto Mute you sound devices …
```

## Alternate Proposals and Considerations

This RFC does not cover the module authoring experience on publishing a cross-platform
module with multiple dependencies and supporting multiple runtimes.

If there is a desire to explicitly update the local cache (like `apt`), we can introduce a
`Update-PSResourceCache` cmdlet with property on a PSRepository indicating whether
it auto-updates or not.
This would not be a breaking change.
