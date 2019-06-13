---
RFC: RFCxxxx
Author: Steve Lee
Status: Draft
SupercededBy: N/A
Version: 0.2
Area: PowerShellGet
Comments Due: 6/30
Plan to implement: Yes, PS7
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
This module will depend on https://www.nuget.org/packages/NuGet.Client.
This module would be shipped in PowerShell 7 and on PSGallery supporting older
versions of PowerShell.

### Side-by-side with PowerShellGet

Since the current PowerShellGet 2.x version is a script module and this new one
is a C# based module, they can coexist side-by-side.

Script and module metadata will retain the same format as it exists with v2.

### Local cache

Instead of always connecting to PSGallery to perform an online search,
`Find-PSResource` (see below) works against a local cache.
This will also enable changes in PowerShell to use `Find-PSResource -Type Command` to look
in this cache when it can't find a command and suggest to the user the module to install to
get that command.
This will be a local json file containing only the latest version of each module.
The cache will be stored in the user path.
There is no system cache that is shared.
A system wide cache would require elevation or sudo to create and update preventing
it from being useful.

Example cache entry:

```json
{
  "name": "This is my module",
  "exportedFunctions": [
    "Get-One",
    "Set-Two"
  ],
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

An example cache with 5000 resources (approximately the number of unique packages
currently published to PowerShellGallery.com) is ~700KB in compressed json form.

The cache would have both latest stable and latest prerelease versions of resources.

>[!NOTE]
>Need to experiment if it makes sense to have a single cache file for all repositories
>or a different cache per repository for size and perf reasons.
>Perf tests will determine if the cache needs to be in binary form.

### Automatic updating of the cache

On any operation that reaches out to a repository, a REST API will be called to
see if the hash of the cache matches the current cache and if not, a new one
is downloaded.
If the repository doesn't support this new API, it falls back to current behavior
in PSGet v2.
This means that there is no local cache of that repository and operations will
always connect to that repository.

### Repository management

`Register-PSResourceRepository` will allow for registering additional repositories.
A `-Default` switch enables registering PSGallery should it be accidentally removed.
The `-URL` will accept the HTTP address without the need to specify `/api/v3` as
that will be assumed and discovered at runtime (trying v3 first, then falling
back to v2, then the literal URL).
Support for local filesystem repositories must be maintained.
A `-Trusted` switch indicates whether to prompt the user when installing resources
from that repository.
By default, if this switch is not specified, the repository is untrusted.
A `-Repositories` parameter will accept an array of hashtables equivalent to
the parameter names.
A `-Name` parameter allows for setting a friendly name.

A `-Default` switch will set one repository as the default (when not specified
with other cmdlets).
Each time it is specified, it sets the new one as default and the previous default
is no longer default.

A `-Scope` parameter will support `AllUsers` and `CurrentUsers`.

`Get-PSResourceRepository` will list out the registered repositories.

`Set-PSResourceRepository` can be used to update a repository URL, trust level,
or if that repository is the default.

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

For each repository, if the local cache does not exist, the cmdlet will call
`Update-PSResourceCache` first and then attempt a search.
However, this cmdlet does not update the cache if one exists.

`Find-Module` will be retained to provide compatibility with v2.

### Installing resources

`Install-PSResource` will only work for modules unless `-DestinationPath` is
specified which works for all resource types.
A `-Prerelease` switch allows installing prerelease versions.
Other types will use `Save-PSResource` (see below).
A `-Repository` parameter accepts a specific repository name or URL to the repository:

```powershell
Install-Module myModule -Repository 'https://mygallery.com'
```

If `-Repository` is not specified and the same resource is found in multiple
repositories, then the resource is automatically installed quietly from the
trusted repository with the highest version matching the `-Version` parameter
(if specified, otherwise newest non-prerelease version unless `-Prerelease`
is used).
If there are no trusted repositories matching the query, then the newest version
fulfilling the query will be prompted to be installed.
If there are multiple repositories with the same trust level containing the same
version, the first one is used.

`-TrustRepository` can be used to suppress being prompted for untrusted sources.
`-IgnoreDifferentPublisher` can be used to suppress being prompted if the publisher
of the module is different from the currently installed version.
`-Reinstall` can be used to re-install a module.

A `-DestinationPath` parameter allows specifying the target directory instead
of the default one.
This will be in a different parameter set than `-Scope`.

A `-NoClobber` switch will prevent installing modules that have the same cmdlets
as a differently named module already on the system.

A `-Quiet` switch will suppress progress information.

`-Scope` with `AllUsers` and `CurrentUser` will work the same as it does in v2.

`Install-Module` cmdlet will be retained for compatibility with v2.

A change from v2 is that installed modules use the full semver (if used) of
the module, so PSReadLine would be in a folder called `2.0.0-beta4` instead of
just `2.0.0` which resolves issues of attempting to overwrite assemblies currently
loaded on Windows.

### Dependencies and version management

`Install-PSResource` can accept a path to a psd1 or json file (using `-RequiredResourcesFile`),
or a hashtable or json (using `-RequiredResources`) where the key is the module name and the
value is either the required version specified using Nuget version range syntax or
a hash table where `repository` is set to the URL of the repository and
`version` contains the [Nuget version range syntax](https://docs.microsoft.com/en-us/nuget/reference/package-versioning#version-ranges-and-wildcards).

```powershell
Install-PSResource -RequiredResources @{
  "Configuration" = "[1.3.1,2.0)"
  "Pester"        = @{
    version = "[4.4.2,4.7.0]"
    repository = "https://www.powershellgallery.com"
    credential = $cred
  }
}
```

The json format will be the same as if this hashtable is passed to `ConvertTo-Json`:

```json
{
  "Pester": {
    "version": "[4.4.2,4.7.0]",
    "credential": null,
    "repository": "https://www.powershellgallery.com"
  },
  "Configuration": "[1.3.1,2.0)"
}
```

>[!NOTE]
>Credentials management will be using an upcoming RFC for generalized
>credential management.  Not sure how to make that work in json other
>than requiring json to have credential as a json object but that means
>password/secret is in clear text.

The `repository` property can be the name of the registered repository or the URL
to a repository.
If no `repository` is specified, it will use the `default` repository.

The older `System.Version` four part version type will be supported to retain
compatibility with existing published modules using that format.

Due to the differences between semver and `System.Version`, we may have to keep
the two distinct.

Dependent declared modules not found on the system will prompt to be installed
including module name, version, and size unless `-IncludeDependencies` is
specified which will install without prompting.
Declared dependencies are only searched within the same repository as the original
module to be installed.

### Saving resources

This cmdlet uses `Install-PSResource -DestinationPath`.

With the removal of PackageManagement, there is still a need to support saving
arbitrary nupkgs (assemblies) used for scripts.

`Save-PSResource -Type Library` will download nupkgs that have a `lib` folder.
The dependent native library in `runtimes` matching the current system runtime
will be copied to the root of the destination specified.
A `-IncludeAllRuntimes` can be used to explicitly retain the `runtimes` directory
hierarchy within the nupkg to the root of the destination.

>[!NOTE]
>Library support may not be available in 3.0.

A `-Prerelease` switch allows saving prerelease versions.
If the `-Version` includes a prerelease label like `2.0.0-beta4`, then this
switch is not necessary and the prerelease version will be installed.

### Updating resources

`Update-PSResource` will update all resources to most recent minor version by default.
A `-AllowMajorVersionUpdate` switch will allow updating to newer major version.
A `-PatchUpdateOnly` switch will allow updating only to patched versions.

If the installed resource is a pre-release, it will automatically update to
latest prerelease or stable version (if available).

### Publishing resources

`Publish-PSResource` will supporting publishing modules and scripts.
It will follow the same model as `Publish-Module` in v2.

A `-DestinationPath` can be used to publish the resulting nupkg locally.
`Publish-PSResource` can accept a path to the nupkg to publish to a repository.

A `-Nuspec` parameter can be used to specify a nuspec file rather than relying
on this module to produce one.

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

### Uninstalling resources

`Uninstall-PSResource` will be available to remove installed resources.

>[!NOTE]
>Uninstalling dependencies automatically will be something to consider in the future.

### PowerShellGallery status

Upon failure to connect to PSGallery, the cmdlets should retrieve status from
a well known REST API and return a more descriptive error message to the user.

## Alternate Proposals and Considerations

This RFC does not cover the module authoring experience on publishing a cross-platform
module with multiple dependencies and supporting multiple runtimes.

If there is a desire to explicitly update the local cache (like `apt`), we can introduce a
`Update-PSResourceCache` cmdlet with property on a PSRepository indicating whether
it auto-updates or not.
This would not be a breaking change.

The ability to set policy for PSGet is outside the scope of this RFC.

Automatic cleanup of old resources is out of scope of this RFC.
Keeping with current behavior, installs are always side-by-side.
