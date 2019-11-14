---
RFC: RFCxxxx
Author: Sydney
Status: Draft
SupercededBy: N/A
Version: 0.1
Area: Security
Comments Due: July 31st, 2019
Plan to implement: Yes, PS7
---

# Simplify and secure use of secrets in scripts

Advanced scripts that touch many systems require multiple secrets and
types of secrets particularly when orchestrating across different clouds.
Best practice is to not hard code any secrets in scripts, but currently
this requires custom code on different platforms to handle this securely.

The secrets management module provides commands for registering vault extensions, and accessing vault secrets. This greatly reduces the temptation to hard code secrets directly into production source code, and instead use the Secrets Management module to dynamically retrieve secrets at runtime.

## Motivation

> As a PowerShell script author,
> I can securely use multiple secrets,
> so that I can automate complex orchestration across multiple remote resources.

## High Level Design

This is a new independent module called `Microsoft.PowerShell.SecretsManagement`.
Secrets are stored securely in a local vault.
The local vault is expected to only allow access to the user who owns that
vault.
Secrets required to access remote vaults are stored in the local vault and used by the secrets management
cmdlets to retrieve remote secrets.

`User Context` --> `Local Vault` --> `SecretsVault` --> `Remote Vault`

## User Experience

Registering and using remote secrets:

```powershell
# Install the AzKeyVault Extension module to be used in this example.
Install-Module AKVaultExtension

# In this example, we explicitly register this extension
Register-SecretsVault -Name Azure -ModuleName AKVaultExtension -VaultParamters "_SPT_Parameters_AzKeyVault_"

# Once the vault is registered, and populated with secrets, the secret can be retrieved using Get-Secret
$azDevOpsToken = Get-Secret -Name AzDevOps
Invoke-RestMethod https://dev.azure.com/… -Credential $azDevOpsToken -Authentication Basic
```

Registering and using local secret:

```powershell
# For local vault, and vaults that implement the Add-Secret cmdlet we can register custom secrets
# In this example, we store a PSCredential object
Add-Secret -Secret $cred -Name MyCreds
New-PSSession -ComputerName myServer -Credential (Get-Secret -Name MyCreds)
```

## Specification

### Secrets Vault

There are two types of vaults: local and remote.
A local vault, by default, is already created and named `Default`.
`Default` on Windows is [CredMan](https://docs.microsoft.com/en-us/windows/desktop/SecAuthN/credentials-management).
`Default` on macOS is [KeyChain](https://developer.apple.com/documentation/security/keychain_services).

>[!NOTE]
>KeyChain support is unlikely to be available in the first release of this feature.

`Default` on Linux is [Gnome Keyring](https://wiki.gnome.org/Projects/GnomeKeyring/).

>[!NOTE]
>Linux has many options for local credential management.  The choice of Gnome Keyring
>is that it's a simple local vault we can easily test against and validate that our
>design works with a well-known Linux local vault.  Since the extension model is
>open, we expect additional vault support to come from owners of those vaults or
>the community.

### Credential Vault Extensions

Vault extensions are PowerShell modules that provide implementations of four required functions:

- SetSecret - Adds a secret to the vault

- GetSecret - Retrieves a secret from the vault

- RemoveSecret - Removes a secret from the vault

- GetSecretInfo - Returns information about one or more secrets (but not the secret itself)

The extension module can expose the above functions either through a C# class implementing an abstract type, or by publicly exporting script cmdlet functions.
Each function implementation takes a set of parameter arguments needed for secret manipulation and error reporting.
In addition, each function implementation takes an optional parameter argument that is a dictionary of additional parameters, specific to that implementation.
When registering the vault extension, the additional parameters are stored securely in the built-in local vault as a hashtable.
At runtime the additional parameters are read from the built-in local vault and passed to the vault implementing functions.

### C# class implementation

The PowerShell module must include a 'RequiredAssemblies' entry in the module manifest which provides the name of the binary that implements the abstract type.

```powershell
@{
    ModuleVersion = '1.0'
    RequiredAssemblies = @('AKVaultExtension')
}
```

```C#
// AKVaultExtension implements these abstract methods
public abstract bool SetSecret(
    string name,
    object secret,
    IReadOnlyDictionary<string, object> parameters,
    out Exception error);

public abstract object GetSecret(
    string name,
    IReadOnlyDictionary<string, object> parameters,
    out Exception error);

public abstract bool RemoveSecret(
    string name,
    IReadOnlyDictionary<string, object> parameters,
    out Exception error);

public abstract KeyValuePair<string, string>[] GetSecretInfo(
    string filter,
    IReadOnlyDictionary<string, object> parameters,
    out Exception error);
```

When PowerShell loads the module, the required assembly will be loaded and the implementing type becomes available.

### PowerShell script implementation

For a script implementation, the PowerShell module must include a subdirectory named 'ImplementingModule' in the same directory containing the module manifest file.
The ImplementingModule subdirectory must contain PowerShell script module files named 'ImplementingModule' that implements the required script functions.

ImplementingModule.psd1

```powershell
@{
    ModuleVersion = '1.0'
    RootModule = '.\ImplementingModule.psm1'
    FunctionsToExport = @('Set-Secret','Get-Secret','Remove-Secret','Get-SecretInfo')
}
```

ImplementingModule.psm1

```powershell
function Set-Secret
{
    param (
        [string] $Name,
        [object] $Secret,
        [hashtable] $AdditionalParameters
    )
}

function Get-Secret
{
    param (
        [string] $Name,
        [hashtable] $AdditionalParameters
    )
}

function Remove-Secret
{
    param (
        [string] $Name,
        [hashtable] $AdditionalParameters
    )
}

function Get-SecretInfo
{
    param (
        [string] $Filter,
        [hashtable] $AdditionalParameters
    )
}
```

## Vault registration cmdlets

The following cmdlets are provided for vault extension registration.

```powershell
Register-SecretsVault
Get-SecretsVault
Unregister-SecretsVault
```

`Register-SecretsVault` registers a PowerShell module as an extension vault for the current user context.
Validation is performed to ensure the module either provides the required binary with implementing type or the required script commands.
If a dictionary of additional parameters is specified then it will be stored securely in the built-in local vault, assuming that some parameter arguments likely contain secrets.

`Get-SecretsVault` returns a list of extension vaults currently registered in the user context.

`Unregister-SecretsVault` un-registers an extension vault.

`Get-SecretsVault` will enumerate registered extensions returning
the module name and cmdlet used (if appropriate) and the friendly name.

A `SecretsVaultInfo` object will contain properties for all supported
parameters by registration.

```output
Name       Module     Cmdlet               ScriptBlock
----       ------     ------               -----------
AzKeyVault AzKeyVault Get-AzKeyVaultSecret
myVault                                    param($Name)
```

### Storing Secrets

The `Add-Secret` cmdlet is used to store a secret.
The `-Name` must be unique within a vault.
The `-Vault` parameter defaults to the local vault.
A `-NoClobber` parameter will cause this cmdlet to fail if the secret already exists.
A `-Secret` parameter accepts one of the supported types outlined below.
When a string secret is added it will be stored as a secure string. 
For all other secret types the secret will be stored as object type of the secret specified. 

### Retrieving Secrets

The `Get-Secret` cmdlet is used to retrieve secrets as the same type as they
were originally added.
The `-Name` parameter retrieves the secret associated with that name.
The `-Vault` parameter defaults to the local vault.
The `-AsPlainText` switch will return a string secret as plain text. 
By default a string secret will be returned as a secure string.
For all other secret types the returned object will be the original stored secret type.

`Get-Secret` returns a single secret and will result in an error if passed an empty string.

### Removing Secrets

The `Remove-Secret` cmdlet iremoves a secret by name from a given vault.

### Authorization

Access to the credential vault is always using the current process security context.
In the case of remote vaults, the remote credential is stored within the local
default vault and retrieved as needed when accessing secrets from that remote
vault.

### Secrets Supported

Secret objects supported by this module are currently limited to:

- byte[] - Blob secret

- string - String secret

- SecureString - Secure string secret

- PSCredential - PowerShell credential secret

- Hashtable - Hash table of name value pairs, where values are restricted to the above secret types.

## Alternate Proposals and Considerations

In this release, the following are non-goals that can be addressed in the future:

- Provision to rotate certs/access tokens
- Sharing local vaults across different computer systems
- Sharing local vault across different user contexts
- A PSProvider for a Secrets: PSDrive
- C# interface for extensions
- C# API to retrieve secrets for C# based modules
- Delegation support
- Local vault requiring to be unlocked automatically
- Ubiquitous `-Secret` parameter taking a hashtable to automatically populate
  a cmdlet's parameter taking a secret from the vault:

```powershell
Invoke-WebRequest -Secret @{Credential="GitHubCred"}
# this retrieves the secret called GitHubCred and passes it to the `-Credential`
# parameter of Invoke-WebRequest
```
