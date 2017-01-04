---
RFC: 
Author: Sergey Babkin
Status: Draft
Area: Error handling
Comments Due: 
---

# Nested Exceptions

.NET provides for the concept of the nested exceptions, however using the nested
errors in PowerShell is not easy: handling of the PowerShell Script Stack is 
non-obvious, and the default error formatter doesn't print the nested errors
thus requiring the manual support for them.

This proposal describes how the nested exceptions can be implemented in
PowerShell and supported transparently in the syntax of the language (finally and throw
statements).

## Motivation

The nested errors are highly useful for two reasons:

1. Nested error reporting as such, where the low-level inner error gets wrapped
into the more high-level and user-friendly explanations, allowing to trace the
whole sequence of dependencies that led to the error.

2. Reporting of the errors from the try/finally blocks. If an exception gets
thrown from the try block, and another one from the finally block, it's highly
desirable to see both errors (currently the error in the finally block overwrites
the error from the try block thus making it undiagnosable). The nested try/fianlly
blocks may cause the longish error chains to be created.

This proposal is based on my experience described in https://blogs.msdn.microsoft.com/sergey_babkins_blog/2016/12/30/reporting-the-nested-errors-in-powershell/ . This link describes an implementation with the scripting functions but the same
can be implemented in a more convenient and transparent way in the language itself.

## Specification

The proposed changes to the language are:

1. The "throw" statement with an array argument should create a nested exception.
The direction of nesting can be chosen at will but the specific proposed direction is
to make the first element of the array into the innermost exception, the last element
into the outermost exception that gets re-thrown. If any of the exceptions in the array
are nested themselves, they will become concatenated as a part of the resulting
full list of exceptions.

2. If a "finally" block was called due to an exception in the "try" block and
an exception happens in this "finally" block, the result will be a nested exception,
where the original exception from the "try" block will be the inner exception and the
new one will be the outer exception.

3. If a "catch" block throws an exception, it should probably also create a nested
exception in the same way as the "finally" block. This would be a change of behavior
for the programs that already re-throw an exception in some processed way, so possibly this
behavior should be explicitly enabled by setting some special variable, such as
$PSCatchNest = $true, or possibly by provising some parameter to "catch".

The proposed implementation the nested error chains from an array is as follows:

* The elements of the array may be of the types System.Management.Automation.ErrorRecord,
Exception or String.

* The String elements get converted to System.Management.Automation.RuntimeException,
just as it's currently done by the "throw" statement.

* The outermost System.Management.Automation.ErrorRecord gets created with the category
"OperationStopped".

* The field FullyQualifiedErrorId of this outermost error gets
built by concatenation (with separators "`r`n") of these fields from all the nested errors
from the argument list.  If an entry in the list is an ErrorRecord, the value for
concatenation from it is FullyQualifiedErrorId. If an entry in the list is a bare
Exception with possible further nesting then the Message fields from all the
Excpetions in this chain get concatenated. This allows to both handle the bare exceptions
and avoid the duplication of the messages in the concatenation.

* The field ScriptStackTrace of this outermost error is set equal to this field
of the innermost error (since it's the place of the original first error that is most
deeply nested, and all thr callers will likely already be in that stack). If
the innermost error is supplied not as an
System.Management.Automation.ErrorRecord object but as a bare Exception object,
the ScriptStackTrace of the deepest available ErrorRecord object is used.

* The Exception objects from the whole argument list become concatenated into
one chain.

## Alternative implementations

The implementation described in https://blogs.msdn.microsoft.com/sergey_babkins_blog/2016/12/30/reporting-the-nested-errors-in-powershell/
had no way to replace the field ScriptStackTrace in the new error, so it had to
carry the original stack as an error object. The implementation built into the
language would have no such limitation.

