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

We would not provide a separate entry for exported types in manifest, like we did for function, aliases.

## Design

Classes attributed with `hidden` keyword would not be exported from the module.
All other classes would be exported from the module.
