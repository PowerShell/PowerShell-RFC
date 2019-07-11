---
RFC: RFCxxxx
Author: Steve Lee
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

This RFC proposes new cmdlets to make this simpler and secure.

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
# Install the Az.KeyVault module to be used in this example.
# This module doesn't conform to the extensions model so would need to be
# updated to actually work.
Install-Module Az.KeyVault

# In this example, we explicitly register this extension
Register-SecretsVault -Name Azure -Cmdlet Get-AzKeyVaultSecret `
  -Module Az.KeyVault

# When using remote vaults, expectation is that secrets are
# stored using their tooling, so this module is only for retrieving secrets
# from remote vaults
$azDevOpsToken = Get-Secret -Name AzDevOps
Invoke-RestMethod https://dev.azure.com/… -Credential $azDevOpsToken -Authentication Basic
```

Registering and using local secret:

```powershell
# For local vault, we can register custom secrets
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

`Default` on Linux is [Hashicorp Vault](https://www.vaultproject.io/).

>[!NOTE]
>Linux has many options for local credential management, additional support may be
>added in the future or via community provided extensions.

### Credential Vault Extensions

Vault extensions whether they are local or remote are provided as modules.

Extensions are modules that either:

- Call `Register-SecretsVault` upon loading the module, or
- Expose a command to the user which calls `Register-SecretsVault` using
  input from the user

Modules that provide a vault extension should have the tag `SecretsVaultExtension`.

Extensions can register themselves by calling `Register-SecretsVault`
passing the name of the relevant cmdlet to `-Cmdlet` and the module name to `-Module`.
A `-Name` parameter is used for the user to define a friendly name.
There is an optional `-Parameters` parameter that takes a hashtable that will
be splatted to the extension cmdlet to support additional metadata needed
by the extension (such as authentication).

When using this model, the extension cmdlet would have to match the parameters and
output of `Get-Secret` to be compatible.

Alternatively, a `-ScriptBlock` parameter can be used instead of `-Cmdlet` and `-Module`
to allow using existing cmdlets without the need to write a wrapper module:

```powershell
Register-SecretsVault -Name AzKeyVault {
  param($Name)
  Get-AzKeyVaultSecret -VaultName (Get-Secret AzureKeyVaultName) -Name $Name |
    Select -Expand SecretValue
}
```

`Register-SecretsVault` has a `-Default` switch to register the
default local vault (if applicable, see above) if it has been unregistered.

If neither are supplied, then the current user context is used and relies on
the extension cmdlet to handle authentication.

If registering a vault with a Name that already exists, an error will be returned.

`Unregister-SecretsVault` should be used to remove any existing
registration or when updating a registration to a new version.
A `-Name` parameter is mandatory to remove a specific extension.

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

### Retrieving Secrets

The `Get-Secret` cmdlet is used to retrieve secrets as the same type as they
were originally added.
The `-Name` parameter retrieves the secret associated with that name.
The `-Vault` parameter defaults to the local vault.
The returned object will be the original stored secret type.

`Get-Secret` without `-Name` will enumerate all stored secrets returning the
name and the vault.
`-Vault` can be used to filter to a specific vault for the enumeration.

### Removing Secrets

The `Remove-Secret` cmdlet is used to remove a stored secret.

### Authorization

Access to the credential vault is always using the current process security context.
In the case of remote vaults, the remote credential is stored within the local
default vault and retrieved as needed when accessing secrets from that remote
vault.

### Secrets Supported

Secrets supported will be:

- PSCredential
- SecureString
- String
- HashTable
- Byte[]

## Alternate Proposals and Considerations

In this release, the following are non-goals that can be addressed in the future:

- Provision to rotate certs/access tokens
- Sharing local vaults across different computer systems
- Sharing local vault across different user contexts
- A PSProvider for a Secrets: PSDrive
- C# interface for extensions
- Delegation support
- Local vault requiring to be unlocked automatically
