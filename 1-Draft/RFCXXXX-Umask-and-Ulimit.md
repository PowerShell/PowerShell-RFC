---
RFC: XXXX
Author: Jim Truher
Status: Draft
Area: Strict Mode
Comments Due: 7/1/2019
---

# Umask and Ulimit

Unix shells have a number of built in commands which are used to configure the shell's behavior.
This RFC proposes two new areas of control for PowerShell, namely, `umask` and `ulimit`

## UMASK

On Linux and MacOS systems when a file is created, it is created with a specific set of permissions.
This is known as _file mode creation mask_.
These permissions are set via the system api `umask` and are usually altered by the user with the `umask` command.
In Unix shells, the `umask` command is built in to the shell, because the new value exists only for as long as the process that started it and any child process.
Most systems also include `/usr/bin/umask` which is a stand-alone command which may be used to _retrieve_ the value of the mask.
Using `/usr/bin/umask` can generate confusion as some may believe that setting the mask with `/usr/bin/umask` will persist this setting, but this is not the case as when the invocation of `/usr/bin/umask` concludes, the previous mask will be back in effect.
For PowerShell customers, this becomes problematic as there is no easy way to invoke a persisted `umask` setting as there is no built-in command to change the `umask` of the PowerShell process.

### Two new cmdlets

This section of the RFC proposes two new cmdlets:

* Get-Umask
* Set-Umask

These cmdlets (and associated engine support) will be available only on MacOS and Linux.
`Get-Umask` retrieves the current process umask setting and `Set-Umask` sets the current process umask setting.
For a good discussion of umask and its use, please visit: https://www.cyberciti.biz/tips/understanding-linux-unix-umask-value-usage.html.

#### Get-Umask Syntax

```powershell
NAME
    Get-Umask

SYNTAX
    Get-Umask [<CommonParameters>]

ALIASES
    None
```

#### Get-Umask Examples

When running `Get-Umask` an object will be returned which contains the traditional umask string, the Symbolic representation of the mask, and an integer representation as follows:

```powershell
PS> get-umask

Umask Symbolic        AsDecimal
----- --------        ---------
0022  u=rwx,g=rx,o=rx        18
```

When used in a string context, the traditional view of umask shall be presented:

```powershell
PS> "$(get-umask)"
0022
```

#### Set-Umask Syntax

```powershell
NAME
    Set-Umask

SYNTAX
    Set-Umask [-Umask] <string> [<CommonParameters>]
    Set-Umask -Symbolic <string[]> [<CommonParameters>]

ALIASES
    None
```

Two parameter sets for `Set-Umask` will be present:

* Umask
  * A string which represents the umask as an octal number
* Symbolic
  * a string which represents the umask as a comma separated group indicating the permissions of the file owner, group, and other.

#### Set-Umask Examples

```powershell
PS> Set-Umask -Umask 0111
PS> "" > /tmp/umask.txt
PS> /bin/ls -l /tmp/umask.txt
-rw-rw-rw-  1 jimtru  wheel  1 May 22 13:59 /tmp/umask.txt

PS> Set-Umask -Umask 0777
PS> rm /tmp/umask.txt
PS> "" >/tmp/umask.txt
PS> /bin/ls -l /tmp/umask.txt
----------  1 jimtru  wheel  1 May 22 14:00 /tmp/umask.txt

PS> Set-Umask 22
PS> Get-Umask

Umask Symbolic        AsDecimal
----- --------        ---------
0022  u=rwx,g=rx,o=rx        18
```

The `Set-Umask` command will also be able to take the Symbolic representation of the permissions and set them.

```powershell
PS> Set-Umask -Symbolic u=rwx,g=rx,o=rx
PS> Get-Umask

Umask Symbolic        AsDecimal
----- --------        ---------
0022  u=rwx,g=rx,o=rx        18
```

As with the traditional `/usr/bin/umask`, `Set-Umask` only requires so much of the mask as you wish to set.
If you wish to set your file permissions to 664, then `Set-Umask 2` shall be sufficient, or `Set-Umask -Symbolic o=rx`

## ULIMIT

`ulimit` sets or gets limitations on the system resources available to the current shell and its descendents.

### Two New Cmdlets

The PowerShell equivalent shall break these into two separate cmdlets:

* Get-Ulimit
* Set-Ulimit

### Get-Ulimit

Get

#### Get-Ulimit Syntax

```powershell
NAME
    Get-Ulimit

SYNTAX
    Get-Ulimit [[-Resource] {CpuTime | FileSize | DataSegmentSize | StackSize | CoreFileSize | AddresSpaceSize | MemoryAddressSize | 
    NumberOfProcesses | NumberOfOpenFiles}] [<CommonParameters>]

    Get-Ulimit [-All] [<CommonParameters>]

```

#### Get-Ulimit Examples

```powershell

PS> get-ulimit

Resource             Current             Maximum
--------             -------             -------
FileSize 9223372036854775807 9223372036854775807

PS> get-ulimit -Resource FileSize

Resource             Current             Maximum
--------             -------             -------
FileSize 9223372036854775807 9223372036854775807

PS> get-ulimit -resource FileSize,NumberOfOpenFiles,NumberOfProcesses

         Resource             Current             Maximum
         --------             -------             -------
         FileSize 9223372036854775807 9223372036854775807
NumberOfOpenFiles               10240 9223372036854775807
NumberOfProcesses                2837                4256

PS> get-ulimit -all

         Resource             Current             Maximum
         --------             -------             -------
          CpuTime 9223372036854775807 9223372036854775807
         FileSize 9223372036854775807 9223372036854775807
  DataSegmentSize 9223372036854775807 9223372036854775807
        StackSize             8388608            67104768
     CoreFileSize                   0 9223372036854775807
  AddresSpaceSize 9223372036854775807 9223372036854775807
MemoryAddressSize 9223372036854775807 9223372036854775807
NumberOfProcesses                2837                4256
NumberOfOpenFiles               10240 9223372036854775807


PS> get-ulimit -all |get-member

   TypeName: System.Management.Automation.Platform+ResourceLimitInfo
Name        MemberType Definition
----        ---------- ----------
Equals      Method     bool Equals(System.Object obj)
GetHashCode Method     int GetHashCode()
GetType     Method     type GetType()
ToString    Method     string ToString()
Current     Property   ulong Current {get;set;}
Maximum     Property   ulong Maximum {get;set;}
Resource    Property   System.Management.Automation.Platform+ResourceForUlimit Resource {get;set;}
```

#### Implementation Notes

The `ulimit` cmdlets do not totally replicate the Unix behavior.
Specifically, resources which are not alterable via the `setrlimit` API will not be supported.
The pipe size resource is not retrievable or alterable with the ulimit cmdlets.
The resources available will vary based on the platform, the following table shows what resources shall be returned based on their platform:

| Resource | MacOS | Linux |
| -------- | :---: | :---: |
| Maximum CPU Time | :heavy_check_mark: | :heavy_check_mark: |
| Maximum File Size | :heavy_check_mark: | :heavy_check_mark: |
| Maximum Data Segment Size | :heavy_check_mark: | :heavy_check_mark: |
| Maximum Stack Size | :heavy_check_mark: | :heavy_check_mark: |
| Maximum Core File Size | :heavy_check_mark: | :heavy_check_mark: |
| Maximum Resident Set Size | :heavy_check_mark: | :heavy_check_mark: |
| Memory Address Space | :heavy_check_mark: | :heavy_check_mark: |
| Maximum number of processes | :heavy_check_mark: | :heavy_check_mark: |
| Maximum number of open Files | :heavy_check_mark: | :heavy_check_mark: |
| Maximum locked-in-memory address space | :heavy_check_mark: | :heavy_check_mark: |
| Address space limit | | :heavy_check_mark: |
| Maximum number of file locks | | :heavy_check_mark: |
| Maximum number of pending signals | | :heavy_check_mark: |
| Maximum bytes in message queue | | :heavy_check_mark: |
| Maximum nice priority allowed | | :heavy_check_mark: |
| Maximum real-time priority | | :heavy_check_mark: |
| Timeout for real-time tasks (microseconds) | | :heavy_check_mark: |

#### Set-Ulimit Syntax

```powershell
NAME
    Set-Ulimit

SYNTAX
    Set-Ulimit [-ResourceInfo] <Platform+ResourceLimitInfo> [<CommonParameters>]

    Set-Ulimit -Resource {CpuTime | FileSize | DataSegmentSize | StackSize | CoreFileSize | AddresSpaceSize | MemoryAddressSize | 
    NumberOfProcesses | NumberOfOpenFiles} [-Current <ulong>] [-Maximum <ulong>] [<CommonParameters>]

ALIASES
    None
```

#### Set-Ulimit Examples

```powershell
PS> get-ulimit -Resource NumberOfOpenFiles

         Resource Current             Maximum
         -------- -------             -------
NumberOfOpenFiles   10240 9223372036854775807

PS> set-ulimit -Resource NumberOfOpenFiles -Current 20kb 
PS> get-ulimit -Resource NumberOfOpenFiles 

         Resource Current             Maximum
         -------- -------             -------
NumberOfOpenFiles   20480 9223372036854775807
```

# Alternative and Additional Proposals

## Umask behavior

A function `Umask` may be written to mimic the behavior of the Unix umask utility.
This would provide a smooth transition from Unix to PowerShell.

## Get-Ulimit should return a single object

`Get-Ulimit` returns a stream of objects, which includes the resource, the current setting, and the maximum setting.
This cmdlet could always return all the available values in a single object.