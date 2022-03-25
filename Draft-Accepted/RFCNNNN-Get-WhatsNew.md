---
 RFC: RFC00XX
 Author: Jason Helmick
 Status: Draft
 SupercededBy: N/A
 Version: 1.0
 Area: Core
 Comments Due: 4/30/2022
 Plan to implement: Yes
---
# Get-WhatsNew

[Get-WhatsNew](https://github.com/PowerShell/WhatsNew) is a cmdlet that provides information about
changes and new features for a version of PowerShell, delivered to the local terminal experience.

## Motivation

Customers are unaware of benefits, new features and changes, that could impact their automation,
performance and security. Today, this useful information is provided through
[Microsoft Docs](https://docs.microsoft.com/powershell/scripting/whats-new/what-s-new-in-powershell-70)
and the PowerShell GitHub repository. PowerShell customers would benefit from being able to get this
information in the terminal as they expect.

- Customers expect to learn about available features that may enable new solutions
- Customers expect version specific information available from within PowerShell to make upgrade
  decisions.

> As an admin,
> I can list the new features released in my current version of PowerShell, similar
> to [webview](https://docs.microsoft.com/powershell/scripting/whats-new/what-s-new-in-powershell-72)
> so that I can discover and implement new features in managing my products.

> As a developer,
> I can list the new features released in my current version of PowerShell, similar
> to [webview](https://docs.microsoft.com/powershell/scripting/whats-new/what-s-new-in-powershell-72)
> so that I can discover and implement new features to develop automation solutions.

## Goals/Non-Goals

Goals

- Be distributed as a PowerShell module on the Gallery
- Support disconnected scenarios (data ships with module)
- Supports Windows PowerShell 5.1
- Supports the ability to compare versions
- Supports a message-of-the-day (motd) option

Non-goals/Alternatives

- Data endpoint (Microservice) to synchronously deliver or update the data.
- Changelog information

Future feature considerations

- Render markdown output in the console with ANSI styling (enabled by switch parameter)
- Use ANSI strings to make hyperlinks clickable for terminals that support it
- Add `Get-Changelog` cmdlet to display Changelog information

## Solution

Ship a **Microsoft.PowerShell.WhatsNew** module on the PowerShell Gallery that contains the cmdlet
and would:

- Be discoverable using normal PowerShell conventions
- Be available down-level to Windows PowerShell 5.1

## Specification

This specification is based on the cmdlet [Get-WhatsNew](https://github.com/PowerShell/WhatsNew)

- Module Name: Microsoft.PowerShell.WhatsNew
- Cmdlet name: Get-WhatsNew
- Alias: gwn

### Syntax and parameter sets

- ByVersion (Default)

```
Get-WhatsNew [[-Version] <string>] [-Daily] [-Online] [<CommonParameters>]
```

- CompareVersion

```
Get-WhatsNew [[-Version] <string>] -CompareVersion <double> [<CommonParameters>]
```

- AllVersions

```
Get-WhatsNew -All [<CommonParameters>]
```

The **Version** parameter has the following validation attribute: `[ValidateSet('5.1','7.0','7.1','7.2','7.3')]`

### Data model

To minimize content maintenance and allow for disconnected scenarios, the data presented by the
cmdlet is stored with the module as version specific markdown files.

- Uses the same markdown files as the [webview](https://docs.microsoft.com/powershell/scripting/whats-new/what-s-new-in-powershell-72)
- Version files are updated in the module at the time of a new release
- This allows for disconnected scenarios
- Minimizes the maintenance of the content
- Update support
  - Updates to the data require publishing a new version of the module to the Gallery
  - To get the latest data, the user runs `Update-Module`
  - Eliminates the need for a Microservice to serve the data or updates

### Output

Output from the cmdlet is in the form of markdown strings that are displayed to the terminal.

### Demo.txt - Features

- To list the new features released in the current version:

  ```powershell
  Get-WhatsNew
  ```

  Example output:

  ```output
  (truncated for clarity)
  ## Installation updates

  Check the installation instructions for your preferred operating system:

  - [Windows][Windows]
  - [macOS][macOS]
  - [Linux][Linux]

  Additionally, PowerShell 7.2 supports ARM64 versions of Windows and macOS and ARM32 and ARM64
  versions of Debian and Ubuntu.

  For up-to-date information about supported operating systems and support lifecycle, see the
  [PowerShell Support Lifecycle][lifecycle].
  (truncated for clarity)
  ```

- To receive a **Message Of The Day** (motd) displaying a single new feature

  ```powershell
  Get-WhatsNew -Daily
  ```

  Example output

  ```output
  ## Engine updates

  - Add `LoadAssemblyFromNativeMemory` function to load assemblies in memory from a native PowerShell
    host by awakecoding · Pull Request #14652
  ```

- To list only the features of a specific version:

  ```powershell
  Get-WhatsNew -Version 7.3
  ```

  Example output

  ```output
  (truncated for clarity)
  ## Experimental Features

  > [!NOTE]
  > There is a known issue in 7.3.0-preview.1 - Alpine Linux packages are missing the
  > `powershell.config.json` file, causing experimental features to be disabled by default. For
  > details, see Issue #16636.

  PowerShell 7.3 introduces the following experimental features:
  (truncated for clarity)
  ```

- To list both the current version and a specified version for comparison:

  ```powershell
  Get-WhatsNew -CompareVersion 7.3
  ```

    Example output

  ```output
  (truncated for clarity)
  ## Installation updates

  Check the installation instructions for your preferred operating system:

  - [Windows][Windows]
  - [macOS][macOS]
  - [Linux][Linux]

  Additionally, PowerShell 7.2 supports ARM64 versions of Windows and macOS and ARM32 and ARM64
  versions of Debian and Ubuntu.
  (truncated for clarity)
  ```

- To list features from all versions:

  ```powershell
  Get-WhatsNew -all
  ```

  Example output:

  ```output
  (truncated for clarity)
  ## Running PowerShell 7

  PowerShell 7 installs to a directory separately from Windows PowerShell. This enables you to run
  PowerShell 7 side-by-side with Windows PowerShell 5.1. For PowerShell 6.x, PowerShell 7 is an
  in-place upgrade that removes PowerShell 6.x.

  - PowerShell 7 is installed to `%programfiles%\PowerShell\7`
  - The `%programfiles%\PowerShell\7` folder is added to `$env:PATH`
  (truncated for clarity)
  ```

- To view an online version of the `Whats New in PowerShell` documentation.

  ```powershell
  Get-WhatsNew -Version 7.3 -Online
  ```

  ```output
  A browser will open to the `Whats New in PowerShell 7.3` documentation.
  ```
