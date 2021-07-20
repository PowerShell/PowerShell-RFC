---
RFC: RFC0020
Author: James Truher
Status: Final
Area: FileSystem
Comments Due: 4/16/2017
---

# Default file encoding which optionally includes Byte Order Mark (BOM) 

Ensuring file creation is proper for the platform, including whether the BOM should be written.

## Motivation

Current PowerShell behavior is that a BOM is created by default when a file is created for those encodings where the BOM is needed.
This is a problem for Linux systems where the default encoding is UTF8 but a BOM is not written when a file is created.
Creating files on Linux with a BOM makes it difficult to interact with the native tools, as the following example illustrates.

```powershell
PS> "ĝoo" > file.txt
PS> get-content file.txt
ĝoo
PS> exit
james@jimtru-ops2:~$ /bin/cat file.txt
▒▒oo
```

This is due to the BOM being written into the file:

```powershell
PS /home/james> format-hex file.txt

           Path: /home/james/file.txt
           00 01 02 03 04 05 06 07 08 09 0A 0B 0C 0D 0E 0F
00000000   FF FE 1D 01 6F 00 6F 00 0A 00                    .þ..o.o...
           ^^ ^^
```
The native tools on Linux try to render the BOM as actual content, which results in mistranslated characters.
If the BOM could be written when the platform expects it, interaction with native tools will be less problematic.

## Specification

A new global variable `$PSDefaultEncoding` shall be available which allows the user to define the encoding for their system.
The allowed values for this variable shall be defined by the `Microsoft.PowerShell.Encoding` enum, with the following additions:

* UTF8NoBOM
* WindowsLegacy

The following is the complete list of `FileSystemCmdletProviderEncoding` members:
* Ascii
* BigEndianUnicode
* BigEndianUTF32
* Byte
* String
* Unicode
* UTF32
* UTF7
* UTF8
* UTF8NoBOM

When `$PSDefaultEncoding` is set to `UTF8NoBOM`, the file shall be created with UTF8 encoding and no BOM shall be written.

When `$PSDefaultEncoding` is set to `WindowsLegacy`, the behavior shall be:

```
CmdletName          Encoding
----------          --------
Add-Content         ASCII
Export-Clixml       UTF16
Export-CSV          ASCII
Out-File            UTF16
New-ModuleManifest  UTF16
Set-Content         ASCII
Export-PSSession    UTF8 (with BOM)
Redirection         UTF16
```
This persists the irregular file encoding on non-Windows platforms, and allows Linux files to be used on Windows with the same encoding as exists in previous releases of PowerShell.

The default on all platforms shall be `UTF8NOBOM` (including when `$PSDefaultEncoding` is not set).
Naturally, specific use of the `-Encoding` parameter when invoking the cmdlet shall override `$PSDefaultEncoding`. 

### Exclusions

Cmdlets which do not create a file are excluded from this change, so the `*-WebRequest` and `*-RestMethod` cmdlets shall not be changed.
Only those cmdlets listed in the table above are to be changed, any other cmdlet which create files with a specific encoding are out of scope.
Remoting protocol cmdlets shall also be unaffected with this change. 

### Optional

We should take this opportunity to rationalize our use of the `Encoding` parameter, and change the cmdlets which use Encoding as `string` or `System.Text.Encoding` type to use `Microsoft.PowerShell.Encoding`.
The following cmdlets use various types for the parameter `Encoding`

```PowerShell
PS> Get-Command -type cmdlet | ?{$_.parameters} |?{$_.source -match "microsoft"}|ft name,{$_.parameters['encoding'].ParameterType}

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

It will make these cmdlets easier to maintain over time.

### Examples
---
Creating a file without a BOM on a Linux system (the default):
```powershell
PS> "ĝoo" > file.txt
PS> get-content file.txt
ĝoo
PS> exit
james@jimtru-ops2:~$ cat file.txt
ĝoo
```

Additional details:
```powershell
PS /home/james> "©opyright" > c.txt
PS /home/james> format-hex c.txt
           Path: /home/james/c.txt
           00 01 02 03 04 05 06 07 08 09 0A 0B 0C 0D 0E 0F
00000000   C2 A9 6F 70 79 72 69 67 68 74 0A                 Â©opyright.

PS /home/james> /bin/cat c.txt
©opyright
PS /home/james> get-content c.txt 
©opyright
PS /home/james> bash
james@jimtru-ops2:~$ date >> c.txt
james@jimtru-ops2:~$ cat c.txt
©opyright
Thu Feb 16 15:02:58 PST 2017
james@jimtru-ops2:~$ exit
exit
PS /home/james> get-content -Encoding utf8 c.txt
©opyright
Thu Feb 16 15:02:58 PST 2017
```

Creating a file with a BOM on a Linux System, this will specifically put the BOM in the file and will render the file problematic on Linux:
```powershell
$PSDefaultFileEncoding = "UTF8"
PS> "ĝoo" > file.txt
PS> get-content file.txt
ĝoo
PS> exit
james@jimtru-ops2:~$ cat file.txt
▒▒oo
```

This mimics our current behavior and is due to the BOM being written into the file.
This file _would_ be suitable for use on a Windows system.

Creating a file without a BOM on Windows:
```powershell
PS> "ĝoo" |out-file -encoding UTF8NoBOM file.txt
```

### Commentary

`UTF8NoBOM` and `WindowsLegacy` are, of course, not actual encodings but neither are a number of the other values for `Microsoft.PowerShell.Encoding`. 
However, it is somewhat descriptive of our behavior. 

### Alternate Approaches
The setting need not be a PowerShell variable, it could be an environment variable or part of the configuration proposed by [PowerShell-StartupConfig](https://github.com/PowerShell/PowerShell-RFC/blob/master/1-Draft/RFC0015-PowerShell-StartupConfig.md).
However, this is the simplest approach and these alternatives can be done at later time.

## PowerShell Committee Decision

### Voting Results

Jason Shirk: Absent 

Joey Aiello: Accept

Bruce Payette: Accept

Steve Lee: Accept

Hemant Mahawar: Accept

### Majority Decision

Commmittee agrees that this RFC satisfies the motivation for changes to default file encoding.

### Minority Decision

N/A
