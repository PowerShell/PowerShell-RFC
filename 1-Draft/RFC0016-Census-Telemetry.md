---
RFC: RFC0016
Author: Steve Lee
Status: Draft
SupercededBy: n/a
Version: 0.1
Area: Host
Comments Due: 1/6/2016
Feedback: https://github.com/PowerShell/PowerShell-Language-RFC/issues/#
---

# PowerShell Census Telemetry

Add basic census telemetry to PowerShell.

## Motivation

Downloads numbers do not tell us if we are successful in growing usage of PowerShell.
The platform (Windows, Linux, Mac) usage data helps to prioritize new feature work and investments.
Version checks in applications are common and also provide a way to inform the user if a newer version is available.
Send basic PowerShell and OS version information as part of version check request.

## Specification

On startup of PowerShell, the host does a check if a version check has happened that day, if so, we skip.
With interactive session, user is informed there is a newer version and how to update (e.g. apt-get, update-package)
A HTTP GET is performed to a well known https address (TBD) to obtain latest version information.
SSL cert is validated to avoid spoofing server.

### HTTP Request

```
GET /powershell/core/latestversion HTTP/1.1
Accept: application/json
User-Agent: PowerShell/<<$psversiontable.psedition>>/<<$psversiontable.gitcommitid>>; <<$psversiontable.osversion>>
```

Note that `$psversiontable.osversion` is dependent on [#1635](https://github.com/PowerShell/PowerShell/issues/1635) getting addressed.

### HTTP Response

```
HTTP/1.1 200 Ok
Content-Type: application/json
Content-Length: nn
Connection: close

{"psedition":"Core","latestversion":"6.0.0-alpha.12","severity":"critical"}
```

Severity:
- `Critical` indicates a security fix
- `Optional` indicates general bug fixes/features

### Configuration

Telemetry is on by default with a means to opt-out.
Since this is not the only configuration we would expose for PowerShell, we should have a [general configuration RFC] (https://github.com/PowerShell/PowerShell-RFC/blob/master/1-Draft/RFC0015-PowerShell-StartupConfig.md) to solve that, but this RFC assumes standard .config file format.

The census telemetry (sending `User-Agent` information) would be a specific configuration setting: `SendUserAgent=$false`.
Disabling the version check request would be a different configuration setting: `PerformVersionCheck=$false`.

## Alternate Proposals and Considerations

None
 
