---
RFC: RFC0019
Author: Darwin Sanoy
Status: Draft
SupercededBy:
Version: 0.1
Area: Standard Modules
Comments Due: 5/1/2017
---

# Interop Module(s) Which Patch(es) a PowerShell Runspace to Match The "Developed On" Platform of the PowerShell Code

I have written one script for configuring security for AWS which runs, without modification, on Linux or Windows.

I have written serveral debugging scripts which also run, unmodified, on Linux or Windows.  [Code is Here] (https://github.com/DarwinJS/DebugDeepPSHExecution)

I have learned serveral fixups that go a long way to making things work without changing the code.  For example, creating $env:temp and pointing it to /tmp on Linux.

I personally feel that the adoption of PowerShell on Linux will most likely be 
driven by those who know PowerShell well on Windows and want to use it to manage Linux.  I think it will be quite a while before there is much traction to
go the other direction (start learning PowerShell on Linux, get proficient, start using it to manage Windows).  It is also a little more challenging to go the other
was as Linux does not rely on as many environment variables to point to platform-wide standard paths.  (e.g. /tmp folder)  


I have addressed the issue as bi-directional in this RFC, but would be quick to back off to Dev on Windows => Run on Linux if the other way seems not worth the effort.
If both directions were desirable upfront, it would seem to make sense if this module had a neutral name and did fix ups in both directions - 
unless that would make the code too complex - then perhaps a module per "interop direction".

I also want to full acknowledge that this type of support is not appropriate to compile right into the PowerShell code itself - as it would seem to then be attempting 
to "windowize" the Linux platform and would be confusing for Linux professionals and organizations who might adopt PowerShell to manage pure Linux environments.

## Motivation

    As an administrator who wishes to start managing Linux with PowerShell,
    (or vice versa) I can port scripts, snippets and modules that that contain 
    popular references on my initial development platform (eg environment variables) 
    to the other platform without changing every reference, so that I can save work 
    by reusing as much of my "Developed On" platform PowerShell code as possible, and 
    where desirable, create scripts that run on both Windows and Linux unmodified.

## Specification

- Standard shipped-with PowerShell Core module
- Written as a script
- That "fixes up" a target environment
- dynamically (does not make persistent changes)
- within the PowerShell session (does not dynamically change things beyond the current session)
- with common standard pointers for the "Developed on" platform
- or code samples for common things that need to be done from the "developed on" platform
- using existing reference technology common to both platforms and available in PowerShell (e.g. Environment Variables & PowerShell Variables)
- and includes code samples of methods that simply work on both platforms unchanged (e.g. A way to check bitness that works unmodified on both platforms - even though there is not a PowerShell standard for exposing this information)
- that can work under all Windows PowerShell editions (not just Core - even if only distributed with Core)
- that requires some action from an administrator to enable (e.g. run a CMDLet in their script or a machine based PowerShell profile)

###Example

~~~~PowerShell
#Code that adapts Windows PowerShell to Linux
#NOTE: Only tested on CentOS - so may need improvement for other distros
$RunningOnWindows = $true
If ((Test-Path variable:IsWindows) -AND !$IsWindows)
{
  write-output 'Running on PowerShell Core, exposing runtime information'
  If ((!(Test-Path env:temp)) -AND (Test-Path '/tmp'))
  { $env:temp = '/tmp' }
  If ($(hostname) -ilike '*.*')
    { $env:computername = ($env:computername).split('.')[0] }
  Else
    {  $env:computername = hostname }
  
  If ((uname -a) -ilike '*_64*')
  { $OSBitness = 64 }

  $RunningOnWindows = $false
}

#Demonstrates *specific method* that works on both Windows and Linux for Process Bitness
$PROCBitness = 32 #Default
If ([System.IntPtr]::Size -eq 8)
{
  $PROCBitness = 64
}
~~~~

## Alternate Proposals and Considerations

Alternatively any individual who has the need for such functionality will generally pursue it on their own, resulting in many custom solutions and lacking the value of community collaboration.
