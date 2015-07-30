---
RFC: 0002
Author: Sergei Vorobev
Status: Draft
Area: Types, modules
---

# Types to export

## Goal

PowerShell module author should be able to explicitly export types (classes, enums, interfaces) from the module.

## Manifest

We need to introduce a new optional entry **TypesToExport** in the module manifest.
Proposed syntax:

```powershell
# Aliases to export from this module 
TypesToExport = @('ClassName', 'EnumName')
```

We should also support `'*'` wildcard to export all types, defined in the root module file. 
This would be used by default.

```powershell
# Aliases to export from this module 
TypesToExport = '*'
```
### Default behavior

There are two reasonable options for default:

1. `'*'` - export everything.
1. `@()` - export nothing.

Author find 'export everything' reasonable and suitable for the most common scenarios.

