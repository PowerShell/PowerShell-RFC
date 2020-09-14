---
RFC: RFCxxxx
Author: Sydney Smith
Status: Draft
SupercededBy: N/A
Version: 0.3
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
    I can easily install modules without specifying lots of switches,
    so that I can be more productive.

    As a PowerShellGet contributor,
    I can more easily contribute code,
    so that PowerShellGet evolves more quickly.

## User Experience

```powershell
PS> Install-PSResource HappyDays
```

## Specification

### Rewrite of PowerShellGet

PowerShellGet is currently written as PowerShell script with a dependency on PackageManagement.
Proposal is to write PSResource in C# to reduce complexity and make easier to maintain.
In addition, remove dependency on PackageManagement completely as well as dependency on
nuget.exe.
This module will depend on Nuget Server and Client packages including https://www.nuget.org/packages/NuGet.Client.
This module would be shipped in the PSGallery supporting PowerShell 5.1+.
This module will likely ship in PowerShell 7.2 alongside PowerShellGet compatibility module.

### Side-by-side with PowerShellGet

Since the current PowerShellGet 2.x version is a script module and this new one
is a C# based module, they can coexist side-by-side.

Script and module metadata will retain compatible formatting as it exists with v2.

We will introduce a compatibility module that will allow users to use known cmdlets like Install-Module
from the PowerShellGet 3.0 implementation.
This module will ship separately from PowerShellGet 3.0 but is planned as a part of the transition effort.

### Repository management

`Register-PSResourceRepository` will allow for registering additional repositories.
A `-PSGallery` switch enables registering PSGallery should it be accidentally removed.
This would be in a different parameter set from `-URL` and `-Repository`.

The `-URL` will accept the HTTP address with support for v2 and v3 protocols.
Support for local filesystem repositories must be maintained.
A `-Trusted` switch indicates whether to prompt the user when installing resources
from that repository.
Trusted repositories automatically have a priority of 0.
`-Trusted` is a different parameter set from `-Priority`.

By default, if this switch is not specified, the repository is untrusted.
A `-Repository` parameter will accept an array of hashtables equivalent to
the parameter names (like splatting).

```powershell
Register-PSResourceRepository -Repository @(
  @{ URL = "https://one.com"; Name = "One"; Trusted = $true; Credential = $cred }
  @{ URL = "https://powershellgallery.com"; Name = "PSGallery"; Trusted = $true; Default = $true }
  @{ URL = "\\server\share\myrepository"; Name="Enterprise"; Trusted = $true }
)
```

A `-Name` parameter allows for setting a friendly name.

A `-Priority` parameter allows setting the search order of repositories.
A lower value has higher priority.
If not specified, the default value is 50.
PSGallery will have a default value of 50.

`Get-PSResourceRepository` will list out the registered repositories.

`Set-PSResourceRepository` can be used to update a repository URL, trust level,
or priority.

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

A `-Version` paramter will accept a string of a [Nuget version range syntax](https://docs.microsoft.com/en-us/nuget/reference/package-versioning#version-ranges-and-wildcards).

A `-Prerelease` switch will allow include prerelease resources in the results.

A `-ModuleName` paramter will accept a string, and will return commands, DSC resources, and Role Capabilities found in modules with that name.

A `-Tags` parameter will accept a string array and will filter results based on PSResource tags.

A `-Proxy` parameter will accept a uri.

A `-ProxyCredential` parameter will accept a PSCredential

A `-Credential` paramter will accept a PSCredential.

An `-IncludeDependencies` switch will include dependent resources in the returned results.

`Find-Module` will be retained to provide compatibility with v2.

### Installing resources

`Install-PSResource` will mirror the parameter set for `Find-PSResource`.
The cmd will only work for modules unless `-DestinationPath` is
specified which works for all resource types.
Other types will use `Save-PSResource` (see below).
A `-Repository` parameter accepts a specific repository name or URL to the repository:

```powershell
Install-PSResource myModule -Repository 'https://mygallery.com'
# or
Install-Module myModule -Repository 'https://mygallery.com'
```

If `-Repository` is not specified and the same resource is found in multiple
repositories, then the resource is automatically installed quietly from the
trusted repository with the highest version matching the `-Version` parameter
(if specified, otherwise newest non-prerelease version unless `-Prerelease`
is used).
If there are no trusted repositories matching the query, then the newest version
fulfilling the query will be prompted to be installed from the highest priority
repositories.
If there are multiple repositories with the same priority level containing the same
version, the first one registered is used and will be prompted to install.

`-TrustRepository` can be used to suppress being prompted for untrusted sources.
`-IgnoreDifferentPublisher` can be used to suppress being prompted if the publisher
of the module is different from the currently installed version.
`-Reinstall` can be used to re-install a module.

A `-DestinationPath` parameter allows specifying the target directory instead
of the default one.
This will be in a different parameter set than `-Scope`.

A `-Type` parameter will accept an array of types to allow filtering.

A `-Version` parameter will accept a string of a [Nuget version range syntax](https://docs.microsoft.com/en-us/nuget/reference/package-versioning#version-ranges-and-wildcards).

A `-Tags` parameter will accept a string array and will filter based on PSResource tags.

A `-Proxy` parameter will accept a uri.

A `-ProxyCredential` parameter will accept a PSCredential

A `-Credential` parameter will accept a PSCredential.

A `-NoClobber` switch will prevent installing modules that have the same cmdlets
as a differently named module already on the system.

A `-Quiet` switch will suppress progress information.

`-Scope` with `AllUsers` and `CurrentUser` will work the same as it does in v2.
Default is `CurrentUser`.

If aliases are enabled `Install-Module` will call to `Install-PSResource -Type Module`.

v3 will continue to use the versioning folder format as v2 in that it will be
Major.Minor.Patch and not contain the semver prerelease label.

### Dependencies and version management

`Install-PSResource` can accept a path to a psd1 or json file (using `-RequiredResourceFile`),
or a hashtable or json (using `-RequiredResource`) where the key is the module name and the
value is either the required version specified using Nuget version range syntax or
a hash table where `repository` is set to the URL of the repository and
`version` contains the [Nuget version range syntax](https://docs.microsoft.com/en-us/nuget/reference/package-versioning#version-ranges-and-wildcards).

```powershell
Install-PSResource -RequiredResource @{
  'Configuration' = '[1.3.1,2.0)'
  'Pester'        = @{
    version = '[4.4.2,4.7.0]'
    repository = 'https://www.powershellgallery.com'
    credential = $cred
    allowPrerelease = $true
  }
}
```
In this case the modules named "Configuration", and "Pester" will be installed. 
The json format will be the same as if this hashtable is passed to `ConvertTo-Json`:

```json
{
  'Configuration': '[1.3.1,2.0)',
  'Pester': {
    'version': '[4.4.2,4.7.0]',
    'credential': null,
    'repository': 'https://www.powershellgallery.com',
    'prerelease': true
  }
}
```

The older `System.Version` four part version type will be supported to retain
compatibility with existing published modules using that format.

Declared dependencies are searched and installed using the same trust algorithm
as described for `Install-PSResource` above.

### Saving resources

This cmdlet uses `Install-PSResource -DestinationPath`.

With the removal of PackageManagement, there is still a need to support saving
arbitrary nupkgs (assemblies) used for scripts.

A `-AsNupkg` switch will save the resource as a nupkg (if it was originally a
nupkg) instead of expanding it into a folder.

### Updating resources

`Update-PSResource` will update all resources to most recent version by default.

If the installed resource is a pre-release, it will automatically update to
latest prerelease or stable version (if available).

### Publishing resources

`Publish-PSResource` will supporting publishing modules, scripts and nupkgs.
It will follow the same model as `Publish-Module` in v2.

A `-DestinationPath` can be used to publish the resulting nupkg locally.
`Publish-PSResource` can accept a path to the nupkg to publish to a repository.

A `-Nuspec` parameter can be used to specify a nuspec file rather than relying
on this module to produce one.

A `-SkipDependenciesCheck` switch can be used to bypass the default check that
all dependencies are present.

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

If a package is unable to be uninstalled because it is currently loaded, the cmdlet will emit a warning
which enumerates the packages that were unable to be installed.
We will explore a dependency clean parameter in a future version of PowerShellGet.

When uninstalling the dependency check will be performed not only on the RequiredModules.Name,
but also on RequiredModules.Version, which should be less-than-or-equal to the version of the module being
uninstalled in order to be considered a dependency.

### Proxy support

Each cmdlet will have `-Proxy` and `-ProxyCredential` parameters.

### Open Sourcing the module

In order to open source PowerShellGet 3.0 we will fork the current PowerShellGet repo, rename it as PowerShellGetv2,
and use the existing PowerShellGet repo as the primary support and development contact for PowerShellGet 3+.

## Alternate Proposals and Considerations

These are items are outside the scope of this RFC and version 3.0.
This list includes a number of features that were included in the inital versions 
of the RFC but did not meet the final scope of PowerShellGet 3.0.
Many of these items can be addressed in a future version of PowerShellGet 3.x without
introducing a breaking change:

- This RFC does not cover the module authoring experience on publishing a cross-platform
  module with multiple dependencies and supporting multiple runtimes.

- [Should be the opposite] If there is a desire to explicitly update the local cache (like `apt`), we can introduce a
  `Update-PSResourceCache` cmdlet in the future and a property on PSRepository registrations
  indicating whether it auto-updates or not.

- The ability to set policy for PSGet

- Automatic cleanup of old resources when upgrading or removing resources

- Signing incorporated into `Publish-PSResource`

- `-Trusted` switch to `Install-PSResource` and `Find-PSResource` that pre-validates
  the signing cert is trusted on the system

- Notify users that they have outdated versions of a module (just patch versions?)

- A way to filter resources to those allowed to be installed on the system

- Enterprise management of local cache/the ability to configure a PSRepository on a system-wide level

- Full semver support won't happen until there is a semver type in .NET

- If the prerelease switch is not used, and only a prerelease version of the module is found a warning will be emitted.

- Using a merkle tree to validate modules

- Support for automatic searches from Find-PSResource -Type Command when a command in not found in module discovery
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

- PowerShell module loading needs to be updated to [understand semver](https://github.com/PowerShell/PowerShell/issues/2983).

- Local cacheing,instead of always connecting to a repository to perform an online search,
`Find-PSResource` (see below) works against a local cache.
This will also enable changes in PowerShell to use `Find-PSResource -Type Command` to look
in this cache when it can't find a command and suggest to the user the module to install to
get that command.
This will be a local json file (one per repository) containing sufficient metadata
for searching for resources and their dependencies.
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

- Updating of the local cache

- Credentials management using [Secrets Management RFC](https://github.com/PowerShell/PowerShell-RFC/pull/208) for generalized credential management.
The `repository` property can be the name of the registered repository or the URL
to a repository.
If no `repository` is specified, it will use the `default` repository.

- `Save-PSResource -Type Library` will download nupkgs that have a `lib` folder.
The dependent native library in `runtimes` matching the current system runtime
will be copied to the root of the destination specified.
A `-IncludeAllRuntimes` can be used to explicitly retain the `runtimes` directory
hierarchy within the nupkg to the root of the destination.

- Upon failure to connect to PSGallery, the cmdlets should retrieve status from
a well known REST API and return a more descriptive error message to the user.
