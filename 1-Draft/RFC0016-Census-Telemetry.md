---
RFC: RFC0016
Author: Steve Lee
Status: Draft
SupercededBy: n/a
Version: 0.1
Area: Host
Comments Due: 1/6/2017
Feedback: https://github.com/PowerShell/PowerShell-Language-RFC/issues/#
---

# PowerShell Census Telemetry

Add basic census telemetry to PowerShell.

## Motivation

Downloads numbers do not tell us if we are successful in growing usage of PowerShell.
The platform (Windows, Linux, Mac) usage data helps to prioritize new feature work and investments.

## Specification

On every startup of the PowerShell Console host, telemetry will be sent via [ApplicationInsights](https://azure.microsoft.com/en-us/services/application-insights/) to collect the following information:
- A SHA256 hash of System.Management.Automation.dll
- Environment.OSVersion.VersionString
- GitCommitId (from $psversiontable)

## Design

ApplicationInsights provides a mechanism for sending a [`CustomEvent`](https://docs.microsoft.com/en-us/azure/application-insights/app-insights-api-custom-events-metrics) which is essentially three elements:
- The name of the event
- A dictionary of properties
- A dictionary of metrics

For PowerShell Core purposes we will send only the name of the event and a dictionary of properties consisting of the list shown in the the specification.
Initially, we will not provide any metrics, although we can easily do so at a later time.

## Alternate Proposals and Considerations

None
 
