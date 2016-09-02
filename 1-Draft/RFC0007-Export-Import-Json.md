---
RFC: 0007
Author: Doug Finke
Status: Draft
Version: 1.0
Area: Export/Import JSON files 
---

# Implement Export-Json and Export-Json 

Provide cmdlets for importing and exporting Json files (same as with the CSV format). This simplifies the command line interaction, reduces the amount of code needed to be written by users and the user needs to know fewer commands to interact with Json and the file system.  

## Motivation

Adding these cmdlets makes it far simpler to interact with Json and files.

`gc .\test.json | ConvertFrom-Json`

becomes

`Import-Json .\test.json`

Plus, creating an alias and using the pipeline, you can process all the json files in a directory.

`dir *.json | ipjson`

For the `Export-Json` similar benefits can be seen by adding this convenience cmdlet. 

## Specification

These cmdlets will follow the Import/Export CSV approach, and `Invoke` the `ConvertFrom-Json`, `ConvertTo-Json` cmdlets to convert the Json. These wrappers will work with the pipeline and files. 

## Alternate Proposals and Considerations