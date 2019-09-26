---
RFC: Update $ErrorView with simplified views for Error Messages
Author: Jason Helmick
Status: Draft
Version: 0.0.0
Area: PowerShell
Comments Due: 10/31/2019
---

# Update $ErrorView with simplified views for Error Messages

When an error occurs in PowerShell, the customers on-screen error message experience currently
provides a level of detail that obscures the exception message from being recognized and read by
new or occasional PowerShell users. The addition of a simplified error view will improve both
comprehension and troubleshooting experience. A new cmdlet 'Resolve-ErrorRecord' will provide
complete detailed view of the fully qualified error when desired.

## Motivation

The on-screen experience, when receiving an error message,
is controlled through the views NormalView (the default) and CategoryView. These are user selectable
through the preference variable $ErrorView.
<<<<<<< HEAD
<<<<<<< HEAD
The addition of a “ConciseView” as default that contains only specific error
=======
The addition of a “SimpleView” as default that contains only specific error
>>>>>>> Error-View
information producing a single line on screen and in logs will improve customer success.
=======
This RFC describes Changing '$ErrorView to an enumeration and adding three additional views
to improve readability,'Message', 'Analytic', and 'Dynamic'.
>>>>>>> Error-View

- Message - provides a concise error message suitable for new or occasional PowerShell users.
- Analytic - provides a refactored form of NormalView
- Dynamic - provides conditional switching between Message and Analytic.

A comprehensive detailed view of the fully qualified error, including inner exceptions,
will be provided by the `Resolve-ErrorRecord` cmdlet.

'$ErrorView' shall contain the original views for backward compatibility and renamed
to remove 'view' from the name. The view list is as follows:

- Normal
- Category
- Message
- Analytic
- Dynamic

## Specification

<<<<<<< HEAD
<<<<<<< HEAD
The proposal is to add a new single-line default error message view called ConciseView
=======
The proposal is to add a new single-line default error message view called SimpleView
>>>>>>> Error-View
and add a new cmdlet (Resolve-ErrorRecord) to provide detailed error information.

__Key Design Considerations__

1. To reduce confusion and improve debugging success,
<<<<<<< HEAD
error messages by default should produce a single line of text, including the word “ERROR:”
to make consistent with Verbose and Warning messages. This is added to $errorView as view named "ConciseView".
=======
error messages should call WriteErrorLine to produce a single line of text, including the word “ERROR:”
to make consistent with Verbose and Warning messages. This is added to $errorView as view named "SimpleView".
>>>>>>> Error-View
=======
The proposal is to add three new views to help improve error message comprehension based on user
needs. For in-depth troubleshooting, a new cmdlet `Resolve-ErrorRecord`
to provide detailed error information.

__Key Design Considerations__

1. To reduce confusion and improve debugging success for new and occasional users,
error messages should call WriteErrorLine to produce a simplified message using view 'Message'
and include the preface word “ERROR:”
>>>>>>> Error-View

- Simplified error message syntax from 'Message'. (See graphic below)

<<<<<<< HEAD
    + NormalView
    + CategoryView
<<<<<<< HEAD
    + ConciseView
    + DetailedView

- Error Message syntax for ConciseView
=======
    + SimpleView
    + DetailedView

- Error Message syntax for SimpleView
>>>>>>> Error-View
=======
```powershell
PS C:\> Get-Childitem -Path c:\notreal
ERROR: Cannot find path ‘C:\notreal’ because it does not exist
```

2. To improve script debugging for advanced PowerShell users and scripters,
a refactored error view 'Analytic' will be added displaying the error category,
exception, code line and position, and include additional help for the user.
>>>>>>> Error-View

```powershell
PS C:\> .\MyScript.ps1
ERROR: ItemNotFoundException
  ---> C:\GitHub\pri-errorview\RustTest\test.ps1
    |
15  | Get-ChildItem -Path c:\notreal
    |                     ^^^ Cannot find path 'C:\notreal' because it does not exist.
    |
    * Help: Additional help information provided here
```

3. The 'Dynamic' view is a combination of the views 'message' and 'Analytic' for users working
in the console that also run scripts. When the user is working Interactively in the CLI,
they will receive errors in the view of 'Message'.  When the user executes a script,
errors will use the view 'Analytic'.

- An example is in the image below:

![Message and Analytic](.\RFC00XX-Update-Error-View.png)

4. A new cmdlet `Resolve-ErrorRecord` will produce comprehensive detailed
view of the fully qualified error, including nested inner exceptions.

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
    + specifies one or more of the newest errors to display
    + Not required

<<<<<<< HEAD
<<<<<<< HEAD
Examples
=======
__Example__
>>>>>>> Error-View

```powershell
# Displays details of the last error displayed
Resolve-ErrorRecord

# Displays error details from pipeline error object
$error[1] | Resolve-ErrorRecord

# Display detailed error information for the most recent 3 errors
Resolve-ErrorRecord -Newest 3
<<<<<<< HEAD

__Example 1__

```powershell
PS C:\test> Get-item c:\blah
Get-item : Cannot find path 'C:\blah' because it does not exist. At line:1 char:1
=======
```

=======
>>>>>>> Error-View
__Example 1__
Error occurs in Interactive mode. Cmdlet displays details of the last error displayed

```powershell
<<<<<<< HEAD
PS C:\test> Get-item c:\blah
ERROR: Cannot find path 'C:\blah' because it does not exist.
>>>>>>> Error-View
=======
PS C:\> Get-Childitem -Path c:\notreal
ERROR: Cannot find path ‘C:\notreal’ because it does not exist
>>>>>>> Error-View

PS C:\test> Resolve-ErrorRecord

**** Detailed message here ****
```

__Example 2__
<<<<<<< HEAD
<<<<<<< HEAD

```powershell
PS C:\test> slkjdfh
ERROR: The term 'slkjdfh' is not recognized as the name of a cmdlet, function, script file, or operable program. At line:1 char:1
PS C:\test> $error[0] | Resolve-ErrorRecord

PSMessageDetails      :
Exception             : System.Management.Automation.CommandNotFoundException: The term 'slkjdfh' is not recognized as
=======
SimpleView of Error message. Piped error to Resolve-ErrorRecord shows sample DetailedView
=======
Error occurs in script, shows error from view 'Analytic', and then is piped
from $error array to 'Resolve-ErrorRecord' to display more details.
>>>>>>> Error-View

```powershell
PS C:\> .\MyScript.ps1
ERROR: ItemNotFoundException
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

<<<<<<< HEAD
PS>Resolve-ErrorRecord -Newest 1
Exception calling "GetHostByName" with "1" argument(s):
"No such host is known."
At line:1 char:1
>>>>>>> Error-View
=======
**** Detailed message here ****
**** Detailed message here ****
**** Detailed message here ****
>>>>>>> Error-View
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
