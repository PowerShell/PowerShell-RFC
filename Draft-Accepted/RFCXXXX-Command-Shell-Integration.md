---
RFC: RFCNNNN
Author: Steve Lee
Status: Draft
SupercededBy: NA
Version: 1.0
Area: Interactive Shell
Comments Due: June 1, 2023
Plan to implement: Yes
---

# Command Shell Integration

Executables can provide a more integrated experience with a particular shell by indicating features it supports which the shell can leverage.

Shell integration feature areas:

- Structured output and formatting
- Tab completion
- Help
- Predictors (PowerShell)
- Feedback Providers (PowerShell)

Traditionally, POSIX shells have pre-defined filesystem locations where tools would put their tab completion scripts or help content.
However, this typically works for the system and not for a particular user who doesn't have administrative rights to the system.
This can also make it more complicated to uninstall tools that place files across the filesystem.

Although the examples here are focused on PowerShell, the design is intended to be generic and can be used by other shells.
However, PowerShell specific implementation details are included.

## Motivation

As a tool developer,
I can easily integrate with a shell,
so that users are more productive having a consistent experience within that shell.

## Current Experience

Some tools that are executables and not modules/cmdlets have already reached out on how they can have better integration with PowerShell.
The current experience requires the user to know about the integration scripts and add them to their `$profile` so that it gets loaded.
This greatly decreases usage and thus motivation for tooling to produce integration scripts due to lack of discovery and also
manual effort by the user.

## Enhanced User Experience

In these examples, we'll use a hypothetical version of `az` (Azure CLI) that implements all of the shell integration features.
The user would simply install the tool and it would automatically be integrated with the shell.

### Structured Output with Formatting

```powershell
az group list
```

Current output:

```output
[
  {
    "id": "/subscriptions/11111111-1111-1111-1111-111111111111/resourceGroups/RG1",
    "location": "westcentralus",
    "managedBy": null,
    "name": "RG1",
    "properties": {
      "provisioningState": "Succeeded"
    },
    "tags": {
      "RequestID": "1111"
    },
    "type": "Microsoft.Resources/resourceGroups"
  },
  {
    "id": "/subscriptions/11111111-1111-1111-1111-111111111111/resourceGroups/RG2",
    "location": "westcentralus",
    "managedBy": null,
    "name": "RG2",
    "properties": {
      "provisioningState": "Succeeded"
    },
    "tags": {
      "RequestID": "1111"
    },
    "type": "Microsoft.Resources/resourceGroups"
  }
]
```

Although in PowerShell, this output can be piped to `ConvertFrom-Json`, it's an additional step for every `az` command execution.
It would be preferrable to have the tool output structured data in a format that PowerShell can automatically convert to an object
and used similarly to a cmdlet.

Proposed output with custom formatting showing the most relevant information as well as a nested property:

```output
id                                                                                       location      name                  provisioningState
--                                                                                       --------      ----                  ----------
/subscriptions/11111111-1111-1111-1111-111111111111/resourceGroups/ERNetwork-PvtApp      westcentralus RG1                   Succeeded
/subscriptions/11111111-1111-1111-1111-111111111111/resourceGroups/ERNetwork-SVC         westcentralus RG2                   Succeeded
```

### Tab Completion

```powershell
az g<tab>
```

This should result in:

```powershell
az group
```

As the tab completer would automtically be discovered by PowerShell and used.  In this example, there is only one subcommand starting with `g`.

### Help

```powershell
get-help az
```

The help content would preferrably be in markdown with advanced rendering available in the shell.
Normal help searches would also look within the help content defined by the tool.

### Predictors and Feedback Providers

PowerShell has an extensibility model for Predictors and Feedback Providers.
Typically, these are implemented as modules with a cmdlet to register them with the PowerShell engine.
In this case, the extension would still need to be managed code, but would be automatically discovered and loaded by the shell.
Since the user installed the tool, the expectation is they want to use it.

Based on user feedback, there can be a configuration file to disable specific extensions.

## Specification

### Discovery

A JSON manifest file would be placed alongside the executable (or technically anywhere within `$env:PATH`) and discovered by the shell.
All paths are relative to the location of the manifest file and must use the forward slash path separator.

On Unix systems tools are sometimes synlinked to `/usr/bin`, for example, and on Windows, MSIX installed CLI tools are reparse points
to a different location.
Shells should look for the manifest file in the target location of a symlink or reparse point in addition to within `$env:PATH`.

In the case that multiple JSON manifests are for the same executable, the last one wins.
So if a user wants to override a manifest shipped with a tool, they can place their own in a location that is later in `$env:PATH`.

### Command shell integration manifest

The JSON manifest file name would be `<name>.<author>.command.json`.

- `<name>` should match the executable name without the extension.
- `<author>` is the name of the author or organization that created the tool to avoid collisions.

```json
{
    "$schema": "https://schemas.microsoft.com/commandshell/2023/05/01/command.json",
    "executable": "az",
    "version": "1.0.0",
    "author": "Microsoft",
    "description": "Azure CLI",
    "copyright": "Copyright (c) Microsoft",
    "license": "https://github.com/Azure/azure-cli/blob/dev/LICENSE",
}
```

The mandatory fields are:

- `$schema` includes the version of the manifest with the URL pointing to a published JSON schema.
- `executable` is the name of the executable that the manifest is for and expected to be found in `$env:PATH`
- `version` is the version of the manifest for the tool and only used for informational purposes.

The optional fields are:

- `author` is the name of the author or organization that created the tool.
- `description` is a short description of the tool.
- `license` is the URL to the license for the tool.

### Structured Output and Custom Formatting Registration

Tools can indicate they can output [JSONLines](https://jsonlines.org/) by default and also provide custom formatting.

```json
{
    "$schema": "https://schemas.microsoft.com/commandshell/2023/05/01/command.shellintegration.json",
    "executable": "az",
    "version": "1.0.0",
    "structuredOutput": {
        "default": "jsonlines"
    }
}
```

If a tool indicates it can output structured data in a format supported by the shell,
the shell will set the env var `COMMAND_SHELL_STRUCTURED_OUTPUT` with the value `jsonlines` in this case
in the environment block of the process that is launched.

Tools should check for this environment variable and output structured data in the format indicated if supported.

If the tool calls other executables, it is responsible for removing or propagating the environment variable which
can affect the behavior of child processes.

JSON output can optionally include a `$typeNames` member which is an array of strings that indicate the type hierarchy of the object.
PowerShell would use this as the same as the `TypeNames` member of a PSObject for formatting.
Future enhancement would support a JSON defined formatting schema that could be used by other shells.

Users may want to selectively disable structured output for a tool.
How this is done is shell specific.
For PowerShell, a user could set the environment variable `COMMAND_SHELL_STRUCTURED_OUTPUT` to `none` to disable structured output for all tools
and remove that environment variable to re-enable it.

### Tab Completion Registration

Within the same manifest, a tool can indicate it supports tab completion.

This can either be an executable with arguments.

Arguments can be built-in variables to provide context for the tab completion.

Currently supported variables:

- `{commandLine}` - The full command line that the user has typed so far.
- `{cursorPosition}` - The current cursor position within the command line, starting at 0 index.

```json
{
    "$schema": "https://schemas.microsoft.com/commandshell/2023/05/01/command.shellintegration.json",
    "executable": "az",
    "version": "1.0.0",
    "tabCompletion": {
        "command": {
            "executable": "az",
            "arguments": [
                "completion",
                "powershell",
                "--commandLine",
                "{commandLine}",
                "--cursorPosition",
                "{cursorPosition}"
            ]
        }
    }
}
```

or a shell specific script:

```json
{
    "$schema": "https://schemas.microsoft.com/commandshell/2023/05/01/command.shellintegration.json",
    "executable": "az",
    "version": "1.0.0",
    "tabCompletion": {
        "script": {
            "powershell": "az-completion.ps1",
            "bash": "az-completion.sh",
            "zsh": "az-completion.zsh"
        }
    }
}
```

Multiple shells can be specified and up to the shell to determine which one to use.
Both a `command` and `script` can be specified, but only one of each and also up to the shell to determine which one to use.

In this PowerShell example, the `az-completion.ps1` would be called one time to register the argument completer.
A different shell may decide to call the script for each tab completion request.

### Help Registration

The same manifest may define a path to the help content:

```json
{
    "$schema": "https://schemas.microsoft.com/commandshell/2023/05/01/command.shellintegration.json",
    "executable": "az",
    "version": "1.0.0",
    "help": {
        "tags": [
            "azure",
            "az"
        ],
        "path": "./help/az-help.md"
    }
}
```

Optional `tags` can be specified to help with searching for help content.

### Predictors and Feedback Providers Registration

Specifically for PowerShell, a command may also want to optionally register a Predictor or Feedback Provider.

```json
{
    "$schema": "https://schemas.microsoft.com/commandshell/2023/05/01/command.shellintegration.json",
    "executable": "az",
    "version": "1.0.0",
    "predictor": {
        "powershell": "az-predictor.dll"
    },
    "feedbackProvider": {
        "powershell": "az-feedbackprovider.dll"
    }
}
```

Similar to `tabCompletion`, alternate shells can choose to adopt their own mechanism for registering these extensions
under their shell specific key.

In the case of PowerShell, predictors and feedback providers are assemblies so the value is the path to the assembly
instead of a `.ps1` script.
To enable support for MSIX installed tools and not load the dll directly from their location, PowerShell will
copy the dll to the user's folder if it is not already there (SHA256 hash comparison) and load it from there:

- On Unix, the dll will be copied to `$HOME/.local/share/powershell/Predictors` and `$HOME/.local/share/powershell/FeedbackProviders`
- On Windows, the dll will be copied to `$HOME\AppData\Local\Microsoft\PowerShell\Predictors` and `$HOME\AppData\Local\Microsoft\PowerShell\FeedbackProviders`

## Alternate Proposals and Considerations

Users may decide they don't want to use a tools shell integration feature by selectively disabling it.
This is shell specific and not part of this proposal.

Given that JSON is typically deeply nested, it may be useful to have PowerShell render _list view_ as YAML instead of
the current flat format.
Additionally, have PowerShell detect and render YAML for a single object and a table for multiple objects, by default.
This would apply to all objects and not just those from a shell integrated tool.
