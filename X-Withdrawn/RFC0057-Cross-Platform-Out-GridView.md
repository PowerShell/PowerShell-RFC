---
RFC: '0057'
Author: John Zeiders
Status: Withdrawn
SupercededBy:
Version: 0.1
Area: Commands
Comments Due: 6/25/2019
---

# Cross Platform Out-GridView

Out-Gridview was a popular command in Windows PowerShell.
Its dependence on Windows Presentation Foundation API's meant that it didn't make it into PowerShell Core.

> The Out-GridView cmdlet sends the output from a command to a grid view window where the output is displayed in an interactive table.

# Motivation

The feature is a commonly cited reason for not migrating to Powershell Core and its implementation will help further adoption.

# Specification

## cmdlet

The cmdlet will be a modern implementation of [Out-GridView](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/out-gridview?view=powershell-5.1) and also cross platform supported.

```powershell
Out-GridView
   [-InputObject <PSObject>]
   [-Title <String>]
   [-PassThru]
   [<CommonParameters>]

Out-GridView
   [-InputObject <PSObject>]
   [-Title <String>]
   [-Wait]
   [<CommonParameters>]

Out-GridView
   [-InputObject <PSObject>]
   [-Title <String>]
   [-OutputMode <OutputModeOption>]
   [<CommonParameters>]
```

## GUI

The GUI will be implemented on top of [Avalonia UI Framework](https://github.com/AvaloniaUI/Avalonia).
The project was chosen for its active development, implemented capabilities, modeling of WPF, and cross platform support.

A new feature `Show Script` will be added that will generate an equivalent PowerShell script that represents the filters manually applied to Out-GridView.
The goal being to enable better automation at scale while also allowing graphical editing of filters.

Wireframe of the new design.
The wireframe is intended for visual reference, it is **not** a pixel perfect representation.
The most important part to note is the new layout for filters as opposed to the original "Criteria Panel".

![Out-GridViewMockup](https://github.com/PowerShell/PowerShell-RFC/blob/9112feb522323ffbb55782ada8cdb6d618bbd83b/assets/Out-GridView/out_gridview_mockup.png)

## Module

Out-GridView will be added to the PowerShell Graphical Tools Module Microsoft.PowerShell.GraphicalTools.
Forming the basis for future cross-platform graphical commands.

## Removed Features

* Search in Column - Better solved by filters

# Alternative Proposals and Considerations

## GUI Frameworks

### Electron

Despite it's popularity as a cross-platform framework, it's slow boot times make it the wrong choice for a command that is frequently used to create confirmation and selection prompts.

### Web

An interesting concept, particularly interesting for the ability to integrate directly into VSCode.
However, the dependency on having a browser pre-installed makes it less than ideal.
