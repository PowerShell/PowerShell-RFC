---
RFC: RFC0022
Author: Jody Whitlock
Status: Rejected
SupercededBy: RFC0037
Area: Commands
Comments Due: 
---

# Get-TestConnection cmdlet for non-Windows Systems

Test-Connection, introduced in PowerShell 2.0 under the Microsoft.PowerShell.Management class provided a mechanism for sending ICMP echo requests within the PowerShell language and have the results returned as a PowerShell object that could be manipulated as such.

While it's true that Test-Connection has evolved very little over the years and versions of PowerShell, on only need to look at the legacy that this cmdlet builds on, the ping command.  This simple command has mostly remained unchanged for as long as the ICMP protocol has existed.  It's a very simplistic function that while simplistic fills a massive void in network troubleshooting and validation of communication capabilities.

## Motivation

The motivation for this RFC is as simple as the ping command itself; provide a Test-Connection cmdlet that can run on any platform that PowerShell can.  This means stripping out some of the WSMan specific remoting capabilities that have been added to allow for distributed execution and removing WMI based methods and classes in favor of pure C# code.

This could also be back-ported into the Windows implementation, which while immediately removing the remoting capability would increase the performance of the cmdlet by not requiring WMI accessors and allow for tighter integration between multiple platforms that this cmdlet could run on.

It would also serve as a framework for further enhancement, such as using SSHRemoting to enable the distributed approach to performing a distributed ping, while allowing for some built in governor mechanism to prevent abuse of the distributed nature.

Lastly, and most importantly, while the current Test-Connection cmdlet in the Windows invocation of PowerShell is limited, it is used extensively to check for the existence and responsiveness of remote systems.  By not providing a non-Windows specific implementation that is as close in parameter set and return types then existing code must be re-written to overcome this limitation, or adoption of PowerShell on non-Windows platforms may be barred by the lack of this simple but widely used functionality.

## Specification

## Alternate Proposals and Considerations

An alternate proposal is to not implement Test-Connection but rather build a new cmdlet from the ground up to replace Test-Connection entirely.  This would be similar in impact to not implementing Test-Connection cross-platform by requiring an unknown amount of code changes to occur before adoption on non-Windows platforms can occur.

This may also lend to those that require this functionality to create seperate code-bases and modules to fill the gap, leading to many custom solutions and suffering from the lack of standards and community collaboration.