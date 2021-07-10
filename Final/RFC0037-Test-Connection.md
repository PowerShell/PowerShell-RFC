---
RFC: XXXX
Author: Joel Sallow
Status: Final
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
-Traceroute [-Source <string>] [-MaxHops <int>] 

# Set 4
-MTUSizeDetect

# Set 5
-TCPPort <int> [-Source <string>]
```

## Changes

1. All current host/information output is output only to the `-Verbose` stream.
2. No progress bar per committee decision in [PowerShell#6768](https://github.com/PowerShell/PowerShell/issues/6768).
3. All output occurs as soon as data is available, individual records per ping reply / trace hop.
4. All presented data should be readily available as properties to the user.

## Output Type Proposed Members

### PingStatus

```csharp
class PingStatus
{
    string Source { get; }
    string Destination { get; }
    IPAddress Address { get; }
    int Latency { get; }
    IPStatus Status { get; }
    int BufferSize { get; }
    
    // These properties are not displayed by default
    PingReply Reply { get; }
    PingOptions Options { get; }
}
```

#### PingMtuStatus

```csharp
class PingMtuStatus : PingStatus
{
    public int MtuSize { get; }
}
```

### TraceStatus

```csharp
class TraceStatus
{
    private PingStatus _status;
    
    int Hop { get; }
    string Hostname { get; }
    IPAddress HopAddress { get; }
    int Latency { get; }
    IPStatus Status { get; }
    string Source { get; }
    string Target { get; }
    IPAddress TargetAddress { get; }
    PingReply Reply { get; }
    PingOptions Options { get; }
}
```

## Mockups

### `Test-Connection www.google.com`

- Information from the `Replies` property of the output object should be included in the main output object, and primary output mode should be _multiple_ objects, each representing a single ping attempt/reply object.
- Buffer _data_ is generally irrelevant, as it cannot be specified, and only a `BufferSize` property should be exposed.
- `Replies.Options` property should be hidden from default formatting.
- Results output as a table, grouped by `Destination` IP / name (in the case that multiple destinations are specified).

**Resulting output:**
```code
   Destination: 8.8.8.8
Source           Address         Latency BufferSize      Status
                                    (ms)        (B)
------           -------         ------- ----------      ------
WS-JOEL          8.8.8.8              29         32     Success
WS-JOEL          8.8.8.8              22         32     Success
WS-JOEL          8.8.8.8              26         32     Success
WS-JOEL          8.8.8.8              35         32     Success
```

### `Test-Connection www.google.com -TraceRoute`

- Each ping response from a hop should be output as a separate object, each containing the `PingReply` object as a property accessible but hidden from formatting.
- **Note:** .NET Core's `PingReply` objects will always report bad data for `RoundtripTime`, `Status`, and `Options` properties when running a `-TraceRoute` as their status is reported (as it should be) as `TtlExpired` and the rest of the data is essentially discarded. For this reason, these properties are manually populated for `TraceStatus` and the `PingStatus` used in traceroutes.
    - A separate `StopWatch` can be used to obtain reasonably accurate `Latency` values.
    - The original `PingOptions` and `BufferSize` information can be manually inserted when the `PingStatus` object is created, giving the user the same data as would normally be returned on the `PingReply`
    - The `Status` property will be overridden in a traceroute if the status is `TtlExpired`, as that behaviour is completely expected.
- Output as a table, grouped by `Target`.

**Resulting output:**
```code
   Target: 8.8.8.8
Hop Hostname               Latency      Status      Source       TargetAddress
                              (ms)
--- --------               -------      ------      ------       -------------
  1 192.168.22.254               1     Success      WS-JOEL      8.8.8.8
  1 192.168.22.254               1     Success      WS-JOEL      8.8.8.8
  1 192.168.22.254               1     Success      WS-JOEL      8.8.8.8
  2 75.144.219.238               2     Success      WS-JOEL      8.8.8.8
  2 75.144.219.238               7     Success      WS-JOEL      8.8.8.8
  2 75.144.219.238               6     Success      WS-JOEL      8.8.8.8
  3 96.120.97.17                18     Success      WS-JOEL      8.8.8.8
  3 96.120.97.17                11     Success      WS-JOEL      8.8.8.8
  3 96.120.97.17                12     Success      WS-JOEL      8.8.8.8
  4 96.110.136.85               16     Success      WS-JOEL      8.8.8.8
  4 96.110.136.85               17     Success      WS-JOEL      8.8.8.8
  4 96.110.136.85               27     Success      WS-JOEL      8.8.8.8
  5 68.85.127.121               27     Success      WS-JOEL      8.8.8.8
  5 68.85.127.121               28     Success      WS-JOEL      8.8.8.8
  5 68.85.127.121               25     Success      WS-JOEL      8.8.8.8
  6 68.86.165.161               37     Success      WS-JOEL      8.8.8.8
  6 68.86.165.161               26     Success      WS-JOEL      8.8.8.8
  6 68.86.165.161               30     Success      WS-JOEL      8.8.8.8
  7 68.86.90.205                25     Success      WS-JOEL      8.8.8.8
  7 68.86.90.205                24     Success      WS-JOEL      8.8.8.8
  7 68.86.90.205                36     Success      WS-JOEL      8.8.8.8
  8 68.86.82.70                 26     Success      WS-JOEL      8.8.8.8
  8 68.86.82.70                 26     Success      WS-JOEL      8.8.8.8
  8 68.86.82.70                 27     Success      WS-JOEL      8.8.8.8
  9 23.30.207.242               28     Success      WS-JOEL      8.8.8.8
  9 23.30.207.242               32     Success      WS-JOEL      8.8.8.8
  9 23.30.207.242               28     Success      WS-JOEL      8.8.8.8
 10 108.170.253.17              26     Success      WS-JOEL      8.8.8.8
 10 108.170.253.17              26     Success      WS-JOEL      8.8.8.8
 10 108.170.253.17              29     Success      WS-JOEL      8.8.8.8
 11 216.239.59.125              25     Success      WS-JOEL      8.8.8.8
 11 216.239.59.125              39     Success      WS-JOEL      8.8.8.8
 11 216.239.59.125              25     Success      WS-JOEL      8.8.8.8
 12 8.8.8.8                     23     Success      WS-JOEL      8.8.8.8
 12 8.8.8.8                     29     Success      WS-JOEL      8.8.8.8
 12 8.8.8.8                     27     Success      WS-JOEL      8.8.8.8
```


## Alternate Proposals & Considerations

### Grouping `TraceStatus` output
The behaviour of `Test-Connection -TraceRoute` prior to this RFC is to output trace data only after all 3 connection attempts have been made.
This behaviour could be retained, and the `TraceStatus` class contain all three `PingStatus` reply objects, with the formatting for `Latency` and `Status` properties showing multiple entries each.

#### `Test-Connection www.google.com -TraceRoute`

- Each hop should be output as a separate object, each containing the `PingReply` objects as a property accessible but hidden from formatting.
- Main `TraceRouteResult` object should contain either ETS or class properties that calculate summary data from their four PingReplies.
- Output as a table, grouped by `Destination`

**Resulting output mockup:**
```code
   Destination: www.google.com
Hop Latency (ms) DestinationAddress HopAddress     BufferSize
--- ------------ ------------------ ----------     ----------
  1 {0, 0, 0}    172.217.2.132      192.168.22.254
  2 {0, 0, 0}    172.217.2.132      75.144.219.238
  3 {0, 0, 0}    172.217.2.132      96.120.37.17
  4 {0, 0, 0}    172.217.2.132      96.110.136.65
  5 {0, 0, 0}    172.217.2.132      69.139.180.170
  6 {0, 0, 0}    172.217.2.132      68.85.127.121
  7 {0, 0, 0}    172.217.2.132      68.86.165.161
  8 {0, 0, 0}    172.217.2.132      68.86.90.205
  9 {0, 0, 0}    172.217.2.132      68.86.82.154
 10 {0, 0, 0}    172.217.2.132      66.208.233.242
 11 {0, 0, 0}    172.217.2.132      0.0.0.0
 12 {0, 0, 0}    172.217.2.132      216.239.59.124
 13 {0, 0, 0}    172.217.2.132      216.239.59.61
 14 {32, 28, 20} 172.217.2.132      172.217.2.132
```

However, by outputting one `TraceStatus` object per ping reply, the user is able to follow the trace more closely, and the output is not halted whenever there is a connection timeout.
Additionally, the user can always choose to group the output later simply by doing `Group-Object -Property [Hop|HopAddress|HopName]` if they wish.

### Use the same type of object for `-MtuSizeDetect` switch

This is _plausible_ but requires a change in the table formatter to permit conditional display of properties.
Currently, this has only been shown to be feasible potentially with list-type formatting.
As such, it makes more sense at the present time to have `-MtuSizeDetect` output an object that inherits from the main `PingStatus` object with the added property for `MtuSize`.
