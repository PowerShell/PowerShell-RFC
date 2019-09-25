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
provides a level of detail that obscures the exception message from being recognized and read by
new or occasional PowerShell users.
To improve both comprehension and the troubleshooting experience,
a simplified conditional error message will be provided determined by
Interactive or Script use. A cmdlet will provide the full error details when desired.

## Motivation

The on-screen experience, when receiving an error message,
is controlled through the views NormalView (the default) and CategoryView. These are user selectable
through the preference variable $ErrorView.
This RFC describes changing the NormalView to a simplified conditional error message. The
CategoryView should remain unchanged.
A comprehensive detailed view of the fully qualified error, including inner exceptions,
will be provided by the `Resolve-ErrorRecord` cmdlet.

## Specification

The proposal is to modify NormalView to a simplified conditional error message
based on interactive versus script use.
For in-depth troubleshooting, a new cmdlet `Resolve-ErrorRecord` to provide detailed error information.

__Key Design Considerations__

1. To reduce confusion and improve debugging success,
error messages should call WriteErrorLine to produce a simplified message, including the word “ERROR:”
to make consistent with Verbose and Warning messages. This is a change to the existing NormalView.

- $ErrorView can refer to these views

    + NormalView
    + CategoryView

- Simplified error message syntax from Interactive and script.

![test](.\RFC00XX-Update-Error-View.png)

2. A new cmdlet `Resolve-ErrorRecord` will produce comprehensive detailed
view of the fully qualified error, including inner exceptions.

- Resolve-ErrorRecord will provide the following:

    + Display Last Error ($Error[0]) – default behavior
    + Accept Pipeline input – support $error[1] |
    + Option for the Newest X errors in the session

- Resolve-ErrorRecord syntax

```powershell
Resolve-ErrorRecord  [-InputObject <psobject>] [-Newest <Int32>] [-All] [<CommonParameters>]
```

First parameter set

- Newest

    + Datatype: int32
    + specifies one or more of last errors to display
    + Not required

__Example 1__
Error occurs in Interactive mode. Cmdlet displays details of the last error displayed

```powershell
PS C:\> Get-Childitem -Path c:\notreal
ERROR: Cannot find path ‘C:\notreal’ because it does not exist

PS C:\test> Resolve-ErrorRecord

**** Detailed message here ****
```

__Example 2__
Error occurs in Script. Cmdlet displays error details from pipeline error object

```powershell
PS C:\> .\MyScript.ps1
ERROR: Cannot find path 'C:\notreal' because it does not exist.
  ---> C:\GitHub\pri-errorview\RustTest\test.ps1
    |
15  | Get-ChildItem -Path c:\notreal
    |                     ^^^ Cannot find path 'C:\notreal' because it does not exist.
    |
    * Help: this is for additional help information

PS C:\> $error[0] | Resolve-ErrorRecord

**** Detailed message here ****
```

__Example 3__
Display detailed error information for the most recent 3 errors.

```powershell
PS C:\> Resolve-ErrorRecord -Newest 3

**** Detailed message here ****
**** Detailed message here ****
**** Detailed message here ****
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

1. We could change the default error message color from the current RED foreground and BLACK background to ?.
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
