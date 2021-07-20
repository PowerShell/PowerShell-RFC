---
RFC: RFC0016
Author: Steve Lee, Jim Truher
Status: Final
SupercededBy: n/a
Version: 0.3
Area: Host
Comments Due: Done
Feedback: https://github.com/PowerShell/PowerShell-Language-RFC/issues/#
---

# PowerShell Census Telemetry

Add basic census telemetry to PowerShell.

## Motivation

Downloads numbers do not tell us if we are successful in growing usage of PowerShell.
The platform (Windows, Linux, Mac) usage data helps to prioritize new feature work and investments.

## Specification

On every startup of the PowerShell Console host, telemetry will be sent via [ApplicationInsights](https://azure.microsoft.com/en-us/services/application-insights/) to collect the following information:
- `[System.Runtime.InteropServices.RuntimeInformation]::OSDescription` (equivalent to `uname -a` on Unix)
- GitCommitId (from $psversiontable)

Performance will be measured to ensure collecting and sending telemetry does not have an impact to PowerShell startup time.
Target is <5ms impact.

Telemetry is only collected if the file DELETE_ME_TO_DISABLE_CONSOLEHOST_TELEMETRY exists in $PSHome.
Eventually, we want to adopt [RFC0015 Startup Configuration](https://github.com/PowerShell/PowerShell-RFC/blob/master/1-Draft/RFC0015-PowerShell-StartupConfig.md) as the way to enable/disable telemetry.

## Design

ApplicationInsights provides a mechanism for sending a [`CustomEvent`](https://docs.microsoft.com/en-us/azure/application-insights/app-insights-api-custom-events-metrics) which is essentially three elements:
- The name of the event
- A dictionary of properties
- A dictionary of metrics

For PowerShell Core purposes we will send only the name of the event and a dictionary of properties consisting of the list shown in the the specification.
Initially, we will not provide any metrics, although we can easily do so at a later time.

## Alternate Proposals and Considerations

None
 
---------------
## PowerShell Committee Decision

### Voting Results

Jason Shirk: Accept 

Joey Aiello: Accept

Bruce Payette: Accept

Steve Lee: Accept

Hemant Mahawar: Accept

### Majority Decision

Commmittee agrees that this RFC satisfies the initial telemetry for census data.  Future telemetry changes will be new RFCs.

### Minority Decision

N/A
