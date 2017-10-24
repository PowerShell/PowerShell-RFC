---
RFC: RFC
Author: Rhys Evans
Status: Draft
SupercededBy: N/A
Version: 0.1
Area: Module & Script Block Logging
Comments Due: <Date for submitting comments to current draft (minimum 1 month)>
---

# HTTP POSTing ScriptBlock Logging Provider
This RFC is to extend the current module / script block logging functionality in PowerShell to enable sending data directly to a webhook or an HTTP endpoint.

The PowerShell Transcript output could also be included and captured and optionally sent along with the ScriptBlock log.
## Motivation
From a security professional viewpoint having the ability to monitor PowerShell activity centrally without additional agents and collecting log files is very appealing. Especially the ability to detect and analyse encoded / obfuscated script block executions.

From a DevOp professional viewpoint having the ability to create and view dashboards derived from logs for large scale deployments in real-time would also prove useful to track the status and progress of deployments / builds.

Other benefits include :
* Reduced OS / Container storage footprint by not installing logging agents etc.
* Removes complexity of centrally collecting log files e.g. pulling logs from a container instance
* HTTP POST data is key value pairs making log searching and correlating far more efficient and efficient.
* No log data is written to disk
* Removes delay in retrieving logs i.e. The writing to disk followed by an agent polling/pushing the data to the central log aggregation tool
* As a result reducing the CPU overheads

As an example, [Splunk](https://www.splunk.com) is used internally at the organisation I work at. [Splunk](https://www.splunk.com) offers a [HTTP Collector](http://dev.splunk.com/view/event-collector/SP-CAAAE6M) service which will ingest HTTP POST, raw TCP, raw UDP data making it easily available for searching and reporting centrally near real-time. This service is highly available and will accept 

The inherent benefit of utilising an HTTPS POST method will ensure log data is encrypted during communication and the endpoint URI's identity is validated providing the possibility of posting data over the public internet securely. Great for remote workers!


## Specification
The required data is already made available in the "Microsoft-Windows-PowerShell/Operational" event log.
### Key Design Considerations
- The plan would be to create a registry key or a JSON file at system level for configuration to future proof and enable cross platform with PowerShell Core. Potential solution found here - [Dotnet/Corefx file-system based implementation of Microsoft.Win32.Registry](https://github.com/dotnet/corefx/issues/14896).
- HTTP(S) log posting would require the flexibility of providing either an authentication header (Token based) or provide Basic Authentication mechanism
- There should be a cmdlet created to convert a user's creds to Basic and encrypted with the machines private key and stored as a secure string. Then at runtime this secure string is decrypted and used in the HTTP headers.
- Ideally there would be a local cache / queue if the remote endpoint was unavailable for whatever reason. E.g. Remote external workers disconnecting from the VPN.

##### Configuration
The plan would be to import the configuration data with the following example properties with example values;
1. StreamLogType
    * TCP, UDP, HTTP, HTTPS
2. StreamLogHost - used for TCP / UDP Streams
    * HAProxyService:9997
3. StreamLogURI - used for HTTP(S) Streams  
    * https://HAProxyService:9997/services/collector
4. StreamLogAuthToken - used for HTTP header auth token
    * c2cedcc6-4de1-469c-8c8b-5b75f1d374b3
5. StreamLogAuthBasicSecureString - used for Basic Authentication using the HTTP type
6. StreamLogCaptureTranscript
    * true, false
7. StreamLogCertRemoteEKU

Some Notes:  
- StreamLogHost and StreamLogURI would be mutually exclusive  
- StreamLogAuthToken could also be converted to a secure string for added security

##### HTTP Post Example
Below is an example of what a PowerShell HashTable could look like before it is converted to a JSON object. There is an assumption made that the event that is passed to this function would consist of similar data as found in the 'Microsoft-Windows-PowerShell/Operational' Event ID 4104. 

````
@{
    host =   "$env:COMPUTERNAME",
    time =   "$((Get-date).DateTime)",
    event =   {
                  version =   "$($EventObject.Version)",
                  OpcodeDisplayName =   "$($EventObject.OpcodeDisplayName)",
                  scriptblocktext =   "$ScriptBlockText",
                  Level =   "$($EventObject.Level)",
                  Task =   "$($EventObject.Task)",
                  ExecutionProcessID =   "$($EventObject.ProcessId)",
                  TaskDisplayName =   "$($EventObject.TaskDisplayName)",
                  ExecutionTimeStamp =   "$($EventObject.TimeCreated)",
                  ActivityID =   "$($EventObject.ActivityId)",
                  AccountDomainSID =   "$($EventObject.UserId.AccountDomainSid)",
                  UserSID =   "$($EventObject.UserId.Value)"
                  sAMAccountName" : "$($sAMAccountName)"
                  OpCode =   "$($EventObject.Opcode)",
                  ScriptBlockData =   "$($EventObject.Message)",
                  Transcript =  "$($TranscriptOutput)"
              }
}
````

## Alternate Proposals and Considerations
Combining the ScriptBlock and the Transcript in to a single HTTP POST would be the ideal solution. However, providing the transcript output is paired with the ActivityID combining these in the reporting tool would be trivial.

A drawback of implementing a HTTP(S) POST will be performance due to the TCP/TLS handshakes involved.
An approach to solving this could be to open a socket connection and keep it open for the duration of the session. i.e. HTTP Keep-alive.

UDP could also be implemented for the more demanding scripts. E.g. similar to syslog however using structured key/value or JSON structured data. For most cases HTTP would suffice.