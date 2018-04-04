---
RFC: 
Author: Aditya Patwardhan
Status: Draft
SupercededBy:
Version: 0.2
Area: Testing
Comments Due: 10/15/2017
Plan to implement: Yes
---

# Performance testing for PowerShell

This RFC proposes the plan to add AppInsights instrumentation to collect performance telemetry from daily CI runs.
This RFC specifies the additional telemetry that will be added.

## Motivation

To validate the performance of PowerShell, we need to gather performance measurements at regular intervals.
This will enable us in visualizing performance trends and help us identify performance regressions.

## Specification

This RFC proposes to add AppInsights telemetry data points to gather performance metrics.
These metrics will be collected only during daily CI runs in AppVeyor and Travis CI.
Typically, we would need large number of iterations to get stable numbers.
A [metric aggregator](https://docs.microsoft.com/en-us/azure/application-insights/app-insights-api-custom-events-metrics#trackmetric) will be used to compose the data from iterations and then upload the telemetry data points.
This will avoid throttling and dropping of events.

The metric collection code will be within a `#ifdef` and will only be compiled in when PowerShell is built with `Performance` configuration.
There will be no performance metric collection code in official builds.

### Proposed performance test scenarios

* PowerShell startup time for cold and warm start.

* `Get-Command` for a particular module.

* `Get-Help` for non-existent command, about topic and cmdlet.

* `Import-Module` for script-based, binary modules.

* `Get-Module -ListAvailable` of a scoped PSModulePath constrained to test modules.

* Parsing of scripts, classes.

Telemetry events will be added to PowerShell code at appropriate locations to collect these metrics.
Before recording data points, it will be verified that we are running a daily run in AppVeyor or Travis CI.
PowerBI dashboard will be created to visualize the metrics.

### Environment

* PowerShell assemblies are CrossGen'ed.
* PSModulePath will be scoped to performance test modules path.
* Windows, Linux and MacOS have separate AppInsights streams, so they can be tracked separately.
* All measurements will be for test artifacts to reduce variance due to product module changes.

## Alternate Proposals and Considerations

AppInsights was chosen to achieve cross platform compatibility.
Another alternative would be to develop a framework to log metric as cross platform module.
It would be lot of work to create a precise and reliable framework.

Defining goals for these metrics is out of scope for this RFC.

DotNet / CoreFX uses [xUnit performance framework](https://github.com/Microsoft/xunit-performance).
It requires us the develop the tests in C# and hence we would have to use Powershell APIs to execute tests.
The lowest granularity for measurement we get would be around `PowerShell.AddScript().Invoke()`.
This would be too broad of a measurement and might hide the scenario under test.
