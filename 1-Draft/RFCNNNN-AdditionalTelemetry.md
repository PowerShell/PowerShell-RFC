---
RFC: RFC
Author: Sydney Smith
Status: Draft
SupercededBy: N/A
Version: 1.0
Area: Telemetry
Comments Due: 3/29/19
Plan to implement: <Yes | No>
---

# Additional Telemetry in PowerShell

Add usage telemetry across PSEdition, Platform, Application, session type and command types.
Improve the opt-out mechanisms for PowerShell telemetry.Provide community access to aggregated
statistics.

## Motivation

To make the best possible investments in PowerShell we need answers backed by data to the following questions:

- Is PowerShell Core usage growing (in terms of number of starts)?
- Is the PowerShell Core user-base growing?
- How is PowerShell being used? What is the usage distribution across command types and session type?
- What are impediments to PowerShell Core usage growth?
- What are issues that customers are hitting in PowerShell Core?
- What versions of PowerShell tools and services should Microsoft continue to support?
- What PowerShell integration scenarios are people using? How many people are using PowerShell
  Editor services?
- Which experimental features are being used and tested? Which experimental features should we invest in?
- How can we optimize the engine size and efficiency of PowerShell for cloud scenarios?

To ensure we are getting an accurate picture of how everyone uses PowerShell, not just those most
vocal/involved in the community, we need to make improvements in our telemetry.
PowerShell usage telemetry will allow us to better prioritize testing, support, and investments.
It should be straightforward and easy to opt-out of PowerShell telemetry (presently the process is
not intuitive nor well documented).
To better share our plans and successes with the community we want more complete telemetry on our
[Public PowerBi report](https://msit.powerbi.com/view?r=eyJrIjoiYTYyN2U3ODgtMjBlMi00MGM1LWI0ZjctMmQ3MzE2ZDNkMzIyIiwidCI6IjcyZjk4OGJmLTg2ZjEtNDFhZi05MWFiLTJkN2NkMDExZGI0NyIsImMiOjV9&pageName=ReportSection5).
Simply put, we want to make PowerShell better and believe this can be achieved by better
understanding how PowerShell is being used.

### Non-Goals

The goal of this is **not** to collect any personal data of PowerShell users or infringe on
users' privacy.
Respecting the privacy of our users has been the principal factor in determining which metrics we
plan to collect.
The goal of this is not to collect patterns of individual users but to look at aggregate usage trends.

## Specification

Add telemetry to track the following metrics:

- Count of unique users (PowerShell)
- Count of execution types (ex. cmdlets, native binaries/applications) (PowerShell)
- Count of enabled PSEngine experimental features (only those created by the PowerShell Team) (PowerShell)
- Count of session types (ex. hosted vs not hosted sessions) (PowerShell)
- Hosting application of the of the PowerShell session (ex. VSCode, ISE, AZDevOps) (PowerShell)
- Version of PowerShell, PowerShellGet, PackageManagement, Nuget and platform
  used to install from PSGallery (PowerShellGet)

### Other features

- Public PowerBi DashBoard to make the aggregated statistics available to the community.
  Find a mocked-up PowerBi DashBoard with randomly generated data at the end of the document.
  It includes the following reports:
  - Unique user trends
  - Types of user &mdash;based on types of executions taking place
  - VSCode usage trends
  - Application trends
  - Hosted Scenario Trends
  - PowerShellGet installation trends
- A cmdlet for providing feedback (Send-PSFeedback):
  - This cmdlet would take in a string array and send it back to the PowerShell team
  - The cmdlet will also send Platform data (OS/version) and Shell data ($PSVersionTable)
  - The cmdlet will have an -OpenGitHubIssue switch that will open a browser to a new issue template
    in the PowerShell/PowerShell repository
  - The user will have the option to include the the last error using an -IncludeError switch
  - If the command is not successful (ex. there is no Web Connection) the cmdlet will return a
    warning and alternative instructions for support through GitHub
  - If no location switch is used the cmdlet will default send the feedback to the PowerShell
    User Voice
  - Based on usage of this cmdlet there would potential for additional parameters
- Disabling, and re-enabling telemetry will be available through a new cmdlet (Set-PSTelemetry):
  - The cmdlet will stop all collection of telemetry for the remainder of that session and stop
    telemetry from being collected on subsequent starts
  - The user can request their telemetry is deleted by using a Remove-PSTelemetry cmdlet
  - In order to have their telemetry deleted the user will be required to provide their User GUID
  - If the cmdlet is unable to send the information the cmdlet will return a warning and alternative
    instructions for support through GitHub. It will be specified that GUIDs are unique identifiers
    and should not be shared publicly on GitHub.
- There will be two options for the telemetry environment variable which can be set manually
  or using the Set-PSTelemetry cmdlet
  - $True indicates all telemetry will be collected
  - $False indicates no telemetry will be collected
- The existing disabling [mechanisms for PowerShell telemetry](https://docs.microsoft.com/en-us/powershell/scripting/whats-new/what-s-new-in-powershell-core-61?view=powershell-6#telemetry-can-only-be-disabled-with-an-environment-variable) will remain including the ability to
  set the telemetry environment variable before ever needing to launch PowerShell.
  Setting the telemetry environment variable to $False will block telemetry from being sent on 
  every session launch.

### Future work

A public API for module authors and PowerShell hosted application owners to use the same
infrastructure to collect telemetry for their module or application.
Making the API public is a phase two goal of this project.
We will make the telemetry publicly available once it is internally validated.
The intent of this API is to provide a simple consistent way for PowerShell authors to collect their own telemetry.
If external PowerShell authors want to use the API they will need to provide their own storage for the data.
Validating compliance/privacy will be up to the user of the API and not something PowerShell team/Microsoft will cover.

## Design

### PowerShell Changes

- Count of unique users will be based on a User GUID generated at PowerShell start time and stored
  in the user's configuration directory
- On first start up notification that telemetry is enabled will display in the console header
- The Application ID will be used to disambiguate PowerShell Hosted applications. It will be a
  standardized method for applications to identify themselves in our telemetry. It will be up to
  Application owners to tag their usage for inclusion in our counts.
- The count execution type will be collected throughout the PowerShell session
- The type of PowerShell session (hosted vs non-hosted) will be collected at PowerShell
  engine start time
- The count of enabled PSEngine experiemental features will be collected at PowerShell
  engine start time. The count will be based on the experimental feature list in the 
  powershell.config.json file under $PSHOME, and will only count features created by the PowerShell Team
- An example of what would be collected on PowerShell Start up:
  - User GUID: c58030d3-1a91-4086-9cb1-5bddd342056d
  - The ApplicationID: VSCode
  - The Platform being used: Windows 10 Pro
  - The Version of PowerShell: 6.2.0-preview.4
  - The type of PS session: Hosted
  - Experimental Features enabled: "A-Experimental-Feature-name", "Another-Experimental-Feature-Name"
- An example of what might be collected during a PowerShell Session:
  - Count of cmdlets and functions: 10
  - Count of native binaries/applications: 5
- Currently telemetry is collected at the console startup but we would move collection as close to
  the PowerShell engine start time as possible in order to capture hosted PowerShell scenarios in
  the telemetry
- Telemetry will be collected through Azure Application Insights and will be stored using Azure
  Storage Tables

### PowerShellGet Changes

Create a header when packages are installed from the PowerShell Gallery,
to collect/send the following information:

- Version of PowerShellGet, PackageManagement, and NugetProvider
- Version and Edition of PowerShell
- Identifier for Operating System

- An example of what would be collected at Package Installation:
  - The Version of PowerShellGet: 2.0.3
  - The Version of PackageManagement: 1.2.4
  - The NugetProvider: 3.0.0.1
  - The version of PowerShell: Core 6.2
  - The platform being used: macOS 10.3

## Mock PowerBi Report

![PowerBi1](https://github.com/SydneyhSmith/Internal-PowerShellTeam-Docs/blob/master/Telemetry/Powerbi1.PNG)
![PowerBi2](https://github.com/SydneyhSmith/Internal-PowerShellTeam-Docs/blob/master/Telemetry/Powerbi2.PNG)
![PowerBi3](https://github.com/SydneyhSmith/Internal-PowerShellTeam-Docs/blob/master/Telemetry/Powerbi3.PNG)
![PowerBi4](https://github.com/SydneyhSmith/Internal-PowerShellTeam-Docs/blob/master/Telemetry/Powerbi4.PNG)
![PowerBi5](https://github.com/SydneyhSmith/Internal-PowerShellTeam-Docs/blob/master/Telemetry/Powerbi5.PNG)

## Alternate Proposals and Considerations

Privacy concerns including GDPR regulations are a major consideration of this RFC.
We underwent a privacy review before drafting this RFC to ensure that we are respecting
the privacy of all users. The User GUID will be stored in $PSHome so that the user
can provide it and have their data deleted at anytime.
Performance impact is also a major consideration of these changes, and the metrics chosen are
designed to have as nominal an impact on end user experience as possible.

We considered collecting the module/function/cmdlet usage from only modules that are downloaded by the PowerShell Gallery.
The reason we did not chose to propose this is because of the large cost (in terms of performance, data collection, and data management) 
compared with the relatively smaller benefit of actionable questions we could answer with the information. 

We also considered automatically collecting a count of fully qualified error ids. The reason we decided against this was because 
of the potential performance impact and high cost. 

[Microsoft Privacy Policy](https://privacy.microsoft.com/en-US/privacystatement)
