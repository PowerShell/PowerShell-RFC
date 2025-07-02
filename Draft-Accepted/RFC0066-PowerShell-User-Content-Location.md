---
RFC: RFC0066
Author: Justin Chung
Status: Draft
SupercededBy: N/A
Version: 1.0
Area: Core
Comments Due: 07/31/2025
Plan to implement: Yes
---

# PowerShell User Content Location

This RFC proposes moving the current PowerShell user content location out of OneDrive to the
`LocalAppData` directory on Windows machines.

## Motivation

```
As a user,
I can customize the location where PowerShell user content is installed,
so that I can avoid problems created by file sync solutions like OneDrive.
```

- PowerShell currently places profile, modules, and configuration files in the user's Documents
  folder, which is against established conventions for shell configurations and tools.
- PowerShell content files in OneDrive can lead to unwanted syncing of module files, leading to
  various issues.
- There is strong community demand for changing this behavior as the current setup is problematic
  for many users.
- Changing the default location would align PowerShell with other developer tools and improve
  usability.

## Specification

- This will be an experimental feature.
- The content folder location change will only apply to PowerShell on Windows.
- Configurability of the content folder will apply to all platforms.
- A configuration file in the PowerShell user content folder will determine the location of the user
  scoped **PSModulePath**.
  - By default, the PowerShell user content folder will be located in the
    `$env:LOCALAPPDATA\Microsoft\PowerShell`.
  - The new location becomes the location used as the `CurrentUser` scope for PSResourceGet.
- The proposed directory structure:

  ```
  C:\Users\UserName\AppData\Local\PowerShell\
  ├── powershell.config.json   (Not Configurable)
  └── <PSContent>              (Configurable)
      ├── Scripts                   (Not Configurable)
      ├── Modules                   (Not Configurable)
      ├── Help                      (Not Configurable)
      └── <*profile>.ps1            (Not Configurable)
  ```

- The following setting is added to the `powershell.config.json` file:

  **UserPSContentPath** specifies the full path of the content folder. The default value is
  `$env:LOCALAPPDATA\PowerShell\PSContent`. The user can change this value to a different path.

  ```json
  {
      "UserPSContentPath" : "$env:LOCALAPPDATA\\PowerShell\\PSContent",
  }
  ```

## User Experience

- On startup PowerShell will create a directory in AppData and a configuration file.
- The user scoped **PSModulePath** will point to `Modules` folder under the location specified by
  **UserPSContentPath**.
- Users can configure a custom location for PowerShell user content by changing the value of
  **UserPSContentPath**.
- Users will need to manually move/copy their existing PowerShell user content from the Documents
  folder to the new location after enabling the feature.

## Other considerations

- The following functionalities will be affected:
  - SecretManagement
    - SecretManagement extension vaults are registered for the current user context in:
      `$env:LOCALAPPDATA\Microsoft\PowerShell\secretmanagement\secretvaultregistry\`

      When an extension vault is registered, SecretManagement stores the full path to the extension
      module in the registry. Moving the PowerShell content to a new location will break the vault
      registrations.
  - Document instructions on how to re-register vaults after moving the content folder.
  - Document the need to keep Modules in the Documents folder to so that SecretManagement
    continues to work for multiple installs of PowerShell 7 (stable and preview).

- Use the following script to copy the PowerShell contents folder:

  ```pwsh
  $newPath = "C:\Custom\PowerShell\Modules"
  $currentUserModulePath = [System.Environment]::GetFolderPath('MyDocuments') + "\PowerShell"
  Copy-Item -Path $currentUserModulePath -Destination $newPath -Recurse -Force
  ```

- PowerShellGet is hardcoded to install scripts and modules in the user's `Documents` folder. It
  will not support this feature.

## Implementation questions

- Will the experimental feature be enabled by default?
  - Recommendation: No, the user should explicitly enable the feature and copy their existing
    PowerShell user content to the new location.

- How does `$PROFILE` get populated?
  - Can profile scripts be moved to `PSContent`?
  - The feature needs to update `$PROFILE` to point to profile scripts in the new location.

- What happens if **UserPSContentPath** is added to the machine-level configuration file in
  `$PSHOME/powershell.config.json`?
  - Recommendation: Ignore the setting in the machine-level configuration file since this is a user
    setting. No error - just ignore it.

- Will **UserPSContentPath** support environment variables (like `$env:USERNAME` or `%USERNAME%`)?
  - This could enable a global configuration scenario if we allowed configuration in `$PSHOME`.
