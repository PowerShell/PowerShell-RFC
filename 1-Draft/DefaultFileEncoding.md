---
RFC: Default File Encoding
Author: James Truher
Status: Draft
Area: FileSystem
Comments Due: 4/16/2017
---

# Default file encoding which optionally includes Byte Order Mark (BOM) 

Ensuring file creation is proper for the platform, including whether the BOM should be written.

## Motivation

Current PowerShell behavior is that a BOM is created by default when a file is created for those encodings where the BOM is needed.
This is a problem for Linux systems where the default encoding is UTF8 but a BOM is not written when a file is created.
Creating files on Linux with a BOM makes it difficult to interact with the native tools, as the following example illustrates.

```PowerShell
PS> "ĝoo" > file.txt
PS> get-content file.txt
ĝoo
PS> exit
james@jimtru-ops2:~$ cat file.txt
▒▒oo

```
This is due to the BOM being written into the file:
```powershell
PS /home/james> format-hex file.txt

           Path: /home/james/file.txt
           00 01 02 03 04 05 06 07 08 09 0A 0B 0C 0D 0E 0F
00000000   FF FE 1D 01 6F 00 6F 00 0A 00                    .þ..o.o...
```
The native tools on Linux try to render the BOM as actual content, which harms the output.
If the BOM could be written when the platform expects it, interaction with native tools will be less problematic.

## Specification

A new global variable `$PSDefaultFileEncoding` shall be available which allows the user to define the encoding for their system.
The allowed values for this variable shall be defined by

We should take this opportunity to rationalize our use of the `Encoding` parameter, and change the cmdlets which use Encoding as `string` or `System.Text.Encoding` type to use `Microsoft.PowerShell.Commands.FileSystemCmdletProviderEncoding`.
The following cmdlets use various types for the parameter `Encoding`

```PowerShell
PS> Get-Command -type cmdlet | ?{$\_.parameters} |?{$\_.source -match "microsoft"}|ft name,{$\_.parameters['encoding'].ParameterType}

Name             $_.parameters['encoding'].ParameterType
----             ---------------------------------------
Add-Content      Microsoft.PowerShell.Commands.FileSystemCmdletProviderEncoding
Export-Clixml    System.String
Export-Csv       System.String
Export-PSSession System.String
Get-Content      Microsoft.PowerShell.Commands.FileSystemCmdletProviderEncoding
Import-Csv       System.String
Out-File         System.String
Select-String    System.String
Send-MailMessage System.Text.Encoding
Set-Content      Microsoft.PowerShell.Commands.FileSystemCmdletProviderEncoding
``` 

`Microsoft.PowerShell.Commands.FileSystemCmdletProviderEncoding` shall be extended to include:

* UTF8NoBOM

which result in the membernames of this enum being:
* Ascii
* BigEndianUnicode
* BigEndianUTF32
* Byte
* Default
* Oem
* String
* Unicode
* Unknown
* UTF32
* UTF7
* UTF8
* UTF8NoBOM

The default on Windows systems shall remain unchanged, non-Windows platforms shall be defaulted to `UTF8NoBOM` via the `$PSDefaultFileEncoding` variable.
If the `$PSDefaultFileEncoding` is not set, `UTF8NoBOM` shall be the default for non-Windows systems, and the current behavior () on Windows.

### Examples

### Commentary

UTF8NoBOM is, of course, not an encoding but neither are a number of the other values for `FileSystemCmdletProviderEncoding`. 
However, it _is_ descriptive of what we are doing. 
