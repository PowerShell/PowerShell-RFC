

RFC: RFC<four digit unique incrementing number - assigned by Committee>
Author: Darwin Sanoy
Status: Draft
SupercededBy:
Version: 0.1
Area: Standard Modules
Comments Due: 5/1/2017
---

# Interop Module Which Emulates Windows On PowerShell Core On Non-Windows

I have written one script for configuring security for AWS which runs, without modification, on Linux or Windows.

I have written serveral debugging scripts which also run, unmodified, on Linux or Windows.  [Code is Here] (https://github.com/DarwinJS/DebugDeepPSHExecution)

I have learned serveral fixups that go a long way to making things work without changing the code.  For example, creating $env:temp and pointing it to /tmp on Linux.

I know that some will think that this support should be bi-directional - but I personally feel that the adoption of PowerShell on Linux will most likely be 
driven by those who know PowerShell well on Windows and want to use it to manage Linux.  I think it will be quite a while before there is much traction to
go the other direction (start learning PowerShell on Linux, get proficient, start using it to manage Windows).  It is also a little more challenging to go the other
was as Linux does not rely on as many environment variables to point to platform-wide standard paths.  (e.g. /tmp folder)  However, if it should be addressed up front, it 
would seem to make sense if this module had a neutral name and did fix ups in both directions.

I also want to full acknowledge that this type of support is not appropriate to compile right into the PowerShell code itself - as it would seem to then be attempting 
to "windowize" the Linux platform and would be confusing for Linux professionals and organizations who might adopt PowerShell to manage pure Linux environments.

## Motivation

    As an administrator who wishes to start managing Linux with PowerShell,
    I can port Windows scripts, snippets and modules that contain popular Windows references (usually environment variables) to Linux without changing every reference
    so that I can save work by reuse as much of my Windows PowerShell code as possible, and where desirable, create scripts that run on both Windows and Linux unmodified.

## Specification

- Standard shipped-with PowerShell Core module
- Written as a script
- That "fixes up" a Linux environment
- with common Windows standard pointers
- or code samples for common things that need to be done
- including code samples of methods that simply work on both platforms unchanged
- that requires some action from an administrator to enable (e.g. run a CMDLet in their script or a machine based profile)

###example
Only tested on CentOS - so may need improvement for other distros

`
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
`
## Alternate Proposals and Considerations

Alternatively any individual who has the need for such functionality will generally pursue it on their own, resulting in many custom solutions and lacking the value of community collaboration.