---
RFC: 
Author: Sergey Babkin
Status: Draft
Area: Error handling
Comments Due: 
---

# Printing the script stack trace

The stack trace of an error is a very valuable development and debugging tool.
Currently the script stack trace is available in the error objects but are not
printed out. Providing an easy way to enable such printing would be very useful.

## Motivation

The decision to not print the stack trace was apparently driven by the drive for
the reduced output clutter for the casual users. However it is a major pain point
for the authors of the scripts.

The printing of the stack trace can be added by modifying the default formatter
for the class System.Management.Automation.ErrorRecord but it's painful and not
widely known. It would be much more convenient to enable by setting a global variable,
such as

$PSPrintStack = $true

And additional convenience of using a variable that modifies the action of the
pre-defined formatter ather than loading a custom formatter is that it becomes
easy to use in the remote sessions. The errors get converted to strings in the
remote sessions before they are sent back, so the formatter of the remote machine
is used. Loading a custom formatter there requires copying the format file
(since the Update-FormatData command reads only from a file, not from a pipeline)
to the remote machine and then loading it there. This is much more complicated
than setting a variable in the remote session.

## Specification

In the file $PSHOME\PowerShellCore.format.ps1xml in the entry for 
$PSHOME\PowerShellCore.format.ps1xml add one more ExpressionBinding block:

<ExpressionBinding>
    <ScriptBlock>
        if ($PSPrintStack) { $_.ScriptStackTrace }
    </ScriptBlock>
</ExpressionBinding>

Also, it happens that the current output of the formatter is not always properly
terminated by a "`n", so to avoid sticking the stack trace to the end of the
previous line, fix the preceding code to add "`n":

    elseif (! $_.ErrorDetails -or ! $_.ErrorDetails.Message) {
        $_.Exception.Message + $posmsg + "`n"  # SB-changed
    } else {
        $_.ErrorDetails.Message + $posmsg + "`n" # SB-changed
    }

