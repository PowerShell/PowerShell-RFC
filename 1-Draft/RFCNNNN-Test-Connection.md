---
RFC: XXXX
Author: Joel Sallow
Status: Draft
Version: 1.0
Area: Cmdlets
Comments Due: 6/10/19
Plan to Implement: Yes
---

# Proposed Changes to `Test-Connection` Cmdlet

## Syntax

```
# All Sets
Test-Connection [-TargetName] <string[]> [-IPv4] [-IPv6] [-ResolveDestination] [-TimeoutSeconds <int>] [-Quiet] [<CommonParameters>]

# Set 1
[-Ping] [-Count <int>] [-Delay <int>] [-Source <string>] [-MaxHops <int>] [-BufferSize <int>] [-DontFragment]

# Set 2
[-Ping] [-Continues] [-Delay <int>] [-Source <string>] [-MaxHops <int>]  [-BufferSize <int>] [-DontFragment] 

# Set 3
Test-Connection [-TargetName] <string[]> -Traceroute [-Source <string>] [-MaxHops <int>] 

# Set 4
Test-Connection [-TargetName] <string[]> -MTUSizeDetect

# Set 5
Test-Connection [-TargetName] <string[]> -TCPPort <int> [-Source <string>]
```

## Changes

1. All current host/information output is relegated to the `-Verbose` stream.
2. No progress bar per committee decision in [PowerShell#6768](https://github.com/PowerShell/PowerShell/issues/6768).
3. All output occurs as soon as data is available, individual records per ping reply / trace hop.
4. All presented data should be readily available as properties to the user.

## Output Type Proposed Members

### PingStatus

```csharp
class PingStatus
{
    string Source { get; }
    IPAddress Address { get; }
    string Destination { get; }
    int RoundtripTime { get; }
    int BufferSize { get; }
    PingOptions Options { get; }
    // Hidden in the formatter when not set; only populated & shown when using the -MtuSizeDetect switch
    int MtuSize { get; } = -1
}
```

### TraceStatus

```csharp
class TraceStatus
{
    int Hop { get; }
    int[] RoundTripTimes { get; }
    string Destination { get; }
    IPAddress DestinationAddress { get; }
    IPAddress HopAddress { get; }
    int BufferSize { get; }
    PingStatus[] Replies { get; }
}
```

## Mockups

### `Test-Connection www.google.com`

- Information from the `Replies` property of the output object should be included in the main output object, and primary output mode should be _multiple_ objects, each representing a single ping attempt/reply object.
- Buffer _data_ is generally irrelevant, as it cannot be specified, and only a `BufferSize` property should be exposed. `Replies.Buffer` property should remain `private`.
- `Replies.Options` property should be hidden from default formatting.
- Results output as a table, grouped by Destination address (in the case that multiple destinations are specified).

**Resulting output:**
```code
   Destination: www.google.com
Source  Address       RoundtripTime (ms) BufferSize
------  -------       ------------------ ----------
WS-JOEL 172.217.2.132                 36         32
WS-JOEL 172.217.2.132                 21         32
WS-JOEL 172.217.2.132                 25         32
WS-JOEL 172.217.2.132                 25         32
```

### `Test-Connection www.google.com -TraceRoute`

- Each hop should be output as a separate object, each containing the `PingReply` objects as a property accessible by hidden from formatting.
- Main `TraceRouteResult` object should contain either ETS or class properties that calculate summary data from their four PingReplies.
- **Note:** This object type we're using is currently **bugged**, and all `PingReply` objects report `TtlExpired` as their status. Recommend investigating progress of fix for .NET Core 3, or designing custom solution for TraceRoute backing to resolve the issue.
- Output as a table, grouped by `DestinationHost` (Why is this property name different to that of the other object type used for standard pings?)

**Resulting output:**
```code
   Destination: www.google.com
Hop RoundtripTimes (ms) DestinationAddress HopAddress     BufferSize
--- ------------------- ------------------ ----------     ----------
  1 {0, 0, 0}           172.217.2.132      192.168.22.254
  2 {0, 0, 0}           172.217.2.132      75.144.219.238
  3 {0, 0, 0}           172.217.2.132      96.120.37.17
  4 {0, 0, 0}           172.217.2.132      96.110.136.65
  5 {0, 0, 0}           172.217.2.132      69.139.180.170
  6 {0, 0, 0}           172.217.2.132      68.85.127.121
  7 {0, 0, 0}           172.217.2.132      68.86.165.161
  8 {0, 0, 0}           172.217.2.132      68.86.90.205
  9 {0, 0, 0}           172.217.2.132      68.86.82.154
 10 {0, 0, 0}           172.217.2.132      66.208.233.242
 11 {0, 0, 0}           172.217.2.132      0.0.0.0
 12 {0, 0, 0}           172.217.2.132      216.239.59.124
 13 {0, 0, 0}           172.217.2.132      216.239.59.61
 14 {32, 28, 20}        172.217.2.132      172.217.2.132
```
