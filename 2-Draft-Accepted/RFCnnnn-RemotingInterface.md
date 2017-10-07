---
RFC: RFC
Author: Klaus Frank
Status: Draft
SupercededBy:
Version: 1.0
Area: Core/Remoting
Comments Due: <Date for submitting comments to current draft (minimum 1 month)>
Plan to implement: <Yes | No>
---

# RemotingInterface

Currently Powershell only supports supports WinRM based remoting using the build in commands like "Invoke-Command", "Start-PSSession", "Enter-PSSession". As PowerShell is not only moving to other platforms other kinds of remoting are required, like (but not limited to) SSH. Therefore there needs to be a generic interface for programmers to extend this without implementing new remote commands. This also would improve compatibility with existing scripts through new transport mechanics.
I'll call this interface a transporter interface. Also this would allow PowerShell to be used in deployment of new Windows VMs before the network connectivity is established, or allow PowerShell to interact with hosts in a different network where no routes of any kind exist (and are prohibited by certification), but invoking commands through the hyper visor and guest tools is acceptable.
Also this would allow for future means of connecting to other machines without breaking existing scripts.
Like the current PackageManagement platform. OneGet provides a generic interface for others to write a package provider against, without requiring changing scripts when the package provider changes.

## Motivation

    As a System Admin in a Hosting Company,
    I would be able to use different kinds of remoting without changing existing scripts,
    so that I can run, debug and step trace scripts executed through e.g. VMware Invoke-VMScript.

## Specification

## Alternate Proposals and Considerations
