---
RFC: Update $ErrorView with Single String Error Messages
Author: Jason Helmick
Status: Draft
Version: 0.0.0
Area: PowerShell
Comments Due: 
---

# Update $ErrorView with Single String Error Messages

When an error occurs in PowerShell, the customers on-screen error message experience currently
provides a level of detail that obscures the inner exception message from being read by
new or occasional PowerShell users.
To improve both comprehension and the troubleshooting experience, a single string of text
containing the error will be displayed.  A cmdlet will provide the full error details when desired.

## Motivation

The on-screen experience, when receiving an error message,
is controlled through the views NormalView (the default) and CategoryView. These are user selectable though the preference variable $ErrorView.
The addition of a “ConciseView” as default that contains only specific error
information producing a single line on screen and in logs will improve customer success.

Comprehensive error information is available to the customer through
accessing the error object. For new and occasional customers,
a cmdlet “Resolve-ErrorRecord” will simplify the process of getting detailed error information, improving customers success.

## Specification

The proposal is to add a new single-line default error message view called ConciseView
and add a new cmdlet (Resolve-ErrorRecord) to provide detailed error information.

__Key Design Considerations__

1. To reduce confusion and improve debugging success,
error messages by default should produce a single line of text, including the word “ERROR:” to make consistent with Verbose and Warning messages. This is added to $errorView as view named "ConciseView".

- $ErrorView should contain these views

    + NormalView
    + CategoryView
    + ConciseView

- Error Message syntax for ConciseView

```powershell
ERROR: <Exception Message>: <Positional Info>
ERROR: Cannot find path ‘C:\blah’ because it does not exist: At line:1 char:1
```

2. A new cmdlet “Resolve-ErrorRecord” will produce detailed error information similar to Format-List * -Force.

- Resolve-ErrorRecord will provide the following:

    + Display Last Error ($Error[0]) – default behavior
    + Accept Pipeline input – support $error[1] |
    + Option for the first X errors in the session
    + Option for the Last X errors in the session

- Resolve-ErrorRecord syntax

```powershell
Resolve-ErrorRecord  [-InputObject <psobject>] [-Newest <Int32>] [-First <Int32>] [-All] [<CommonParameters>]
```

First parameter set

- Newest

    + Datatype: int32
    + specifies one or more of last errors to display
    + Not required

- First

    + Datatype: Int32
    + Specifies one or more of the first errors that occurred
    + Not required

- All

    + Datatype: Switch
    + Specifies display all errors in session
    + Not required

Examples

```powershell
# Displays details of the last error displayed
Resolve-ErrorRecord

# Displays error details from pipeline error object
$error[1] | Resolve-ErrorRecord

# Display detailed error information for the last 5 errors
Resolve-ErrorRecord -Last 5

# Display detailed error information for the first 3 errors
Resolve-ErrorRecord -First 3

# Display detailed information for all errors that occured in the session
Resolve-ErrorRecord -All
```
__Example 1__

```powershell
PS C:\test> Get-item c:\blah
Get-item : Cannot find path 'C:\blah' because it does not exist. At line:1 char:1

PS C:\test> Resolve-ErrorRecord

PSMessageDetails      :
Exception             : System.Management.Automation.ItemNotFoundException: Cannot find path 'C:\blah' because it does
                        not exist.
                           at System.Management.Automation.LocationGlobber.ExpandMshGlobPath(String path, Boolean
                        allowNonexistingPaths, PSDriveInfo drive, ContainerCmdletProvider 

```

__Example 2__

```powershell
PS C:\test> slkjdfh
ERROR: The term 'slkjdfh' is not recognized as the name of a cmdlet, function, script file, or operable program. At line:1 char:1
PS C:\test> $error[0] | Resolve-ErrorRecord

PSMessageDetails      :
Exception             : System.Management.Automation.CommandNotFoundException: The term 'slkjdfh' is not recognized as
```

## Alternate Proposals and Considerations

__Alternative single-line display__

```powershell
<CommandName>:<Exception Message>
get-item: Cannot find path ‘C:\blah’ because it does not exist:

<CommandName>:<Exception Message>: <Position>
get-item: Cannot find path ‘C:\blah’ because it does not exist: At line:1 char:1

<CommandName>:<Alias>:<Exception Message>: <Position>
get-item: gi: Cannot find path ‘C:\blah’ because it does not exist: At line:1 char:1

ERROR:<Exception Message>
ERROR: Cannot find path ‘C:\blah’ because it does not exist

ERROR:<Exception Message>: <Position>
ERROR: Cannot find path ‘C:\blah’ because it does not exist: At line:1 char:1

ERROR:<CommandName>:<Alias>:<Exception Message>: <Position>
ERROR: get-item: gi: Cannot find path ‘C:\blah’ because it does not exist: At line:1 char:1
```

__Alternative color for all errors__

1. We could change the default error message color from the current RED foreground and BLACK background to...
2. Differentiating errors based on termination condition: terminating versus non-terminating is currently not intuitive. We are examining differentiating these conditions on the console. Example, adding a new property $host.PrivateData.NonTerminatingErrorForegroundColor ='Red'.
For occasional customers, all error messages remain as color Red. For advanced customers, they can change non-terminating errors to another color to separate the error termination type in the console.

__Alternative For Error, Warning, Verbose__

1. We could be more terse in our messages and increase consistency with verbose and warning messages by using a single letter to identify the message.

Legend: V = Verbose, W = Warning, E = Error(non-terminating future), F = Fatal

```powershell
V: You are running your code - what could possibly go wrong.

W: You are about to try something that probably will not work.

E: Your something broke, but Im still running. At line:1 char:1

E: Your something broke, but Im still running. At line:1 char:1

E: Your something broke, but Im still running. At line:1 char:1

F: Now you really broke me. At line:1 char:1

```
