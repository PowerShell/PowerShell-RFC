---
RFC: '0062'
Author: Aditya Patwardhan
Status: Withdrawn
Version: 0.2
Area: Help System
Comments Due: 03/30/2018
Plan to implement: Yes
---

# Help System as a standalone PowerShell Module  

This RFC proposes to remove the Help System from System.Management.Automation.dll and make it a standalone PowerShell module.

## Motivation

Currently, help system is part of System.Management.Automation.dll.
Help system is used predominantly in interactive sessions and hence not required for automation scenarios.
Since, it is part of System.Management.Automation.dll it is always loaded with PowerShell.
Help system is sizeable piece of code, which increases the runtime footprint of the process.
This RFC proposes to make the help system a PowerShell module, so it is loaded only when used.

## Specification

All the help system code resides under [help folder](https://github.com/PowerShell/PowerShell/tree/master/src/System.Management.Automation/help).
This code will be moved to a separate assembly and be published as a module.

All the public APIs from the help system will be moved to the new namespace 'Microsoft.PowerShell.HelpSystem'.
This will be breaking change for code taking dependency of these public APIs.

| Public API | Breaking change impact assessment
| ------------ | --------
| HelpCatergory | Currently internal, will be changed to public.
| HelpCategoryInvalidException | Exception class, breaking change impact is low.
| GetHelpCommand | Sealed class, breaking change impact is low.
| GetHelpCodeMethods | Used by formatting and output, refactor to remove Help System dependency.
| HelpNotFoundException | Exception class, breaking change impact is low.
| SaveHelpCommand | Sealed class, breaking change impact is low.
| UpdatableHelpCommandBase | Used by UpdateHelpCommand, breaking change impact is medium.
| UpdateHelpCommand | Sealed class, breaking change impact is low.

[CLR Type Forwarding](https://docs.microsoft.com/en-us/dotnet/framework/app-domains/type-forwarding-in-the-common-language-runtime) will be considered to lessen the impact of the breaking change.

The interactive user experience would not change as command discovery will find the ```Get-Help```, ```Save-Help``` and ```Update-Help``` commands from the 'Microsoft.PowerShell.HelpSystem' module instead from 'Microsoft.PowerShell.Core' module (System.Management.Automation.dll).

## Help provider plugins

We do not have a need to make the help providers as plugins.
Implementing a plugin model for help providers is out of scope for this RFC.

## Updatable help

Updatable help will be updated to directly consume markdown help content.
It includes adding the markdown help content into the cab / zip files of help content that is downloaded.
CommandHelpProvider and HelpFileHelpProvider will be updated to consume markdown help content.

## Alternate Proposals and Considerations

Instead of making it a different module, do conditional compilation for non-interactive SKU of PowerShell.
This will increase complexity to the build script and creates two SKUs for PowerShell, interactive and non-interactive.
This might be desirable to achieve other goals, but is unnecessary for the HelpSystem.
