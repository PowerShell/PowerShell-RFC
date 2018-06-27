---
RFC: RFC
Author: Bruce Payette
Status: Draft
SupercededBy: 
Version: 6.1
Area: Modules
Comments Due: July 30, 2018
Plan to implement: Yes
---

# Version-Specific $ENV:PSModulePath Variables

The proposal is to have a version-specific module path environment variable tied to the major version of PowerShell. For PowerShell 6, this variable would be `$ENV:PSModulePath6`.

## Motivation

Today `$ENV:PSModulePath` is loaded from a configuration file everytime PowerShell 6 starts up. This means that it's impossible for a PowerShell 6 instance to modify the environment variable such that child processes will inherit the modified value (see issue #6850).

A second problem (and the proximate cause for the first problem) is that if there is only one `$ENV:PSModulePath` variable that is inherited across processes, different PowerShell versions (e.g. 6 and 5.1) would have to share the same module path even though they have incompatible modules. By separating the module path variable by versions, it makes it safe to simply inherit the variable across processes. It also allows different versions of PowerShell to invoke each other with out causing problems from trying to load incompatible modules..

## Specification

The module path variable for PowerShell 5 will continue to be `$ENV:PSModulePath`. For PowerShell 6.x it will be `$ENV:PSModulePath6`. When a child instance of PowerShell 6 starts, if the module path is not set, it will be read from the configuration file. If it is set, then the new shell instance will inherit the `$ENV:PSModulePath6` setting from the parent process. PowerShell 5 will continue to behave as it currently does, initializing from the registry if `$ENV:PSModulePath` not set otherwise inheriting by the child process.


## Alternate Proposals and Considerations

Leave things the way they are.
