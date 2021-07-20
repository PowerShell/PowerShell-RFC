---
RFC: RFC0021
Author: Steve Lee
Status: Final
SupercededBy: N/A
Version: 0.1
Area: Cmdlets
Comments Due: 3/10/2017
---

# Support for Link header based pagination for WebCmdlets

Invoke-RestMethod makes it easy to automate against REST APIs.
The standard for pagination based on https://tools.ietf.org/html/rfc5988#page-6 relies on a response header that you can only get via Invoke-WebRequest and requires each script to parse and utilize.

## Motivation

REST APIs have a standard for pagination that Invoke-WebRequest and Invoke-RestMethod should adopt to simplify scripts so script authors don't have to write similar code for pagination.

## Specification

Invoke-WebRequest would expose a RelationLink property in the output object.
The RelationLink property would be a hash table of the relation links from the Link header.
The `rel` value would be the key and the URL would be the value in the hash table.

Example:
```
   Link: </TheBook/chapter2>;
         rel="previous"; title*=UTF-8'de'letztes%20Kapitel,
         </TheBook/chapter4>;
         rel="next"; title*=UTF-8'de'n%c3%a4chstes%20Kapitel
```

This would result in RelationLink property hash table consisting of:

|Key|Value|
|---|---|
|previous|/TheBook/chapter2|
|next|/TheBook/chapter4|

Since `title` is not a relation link, it is not exposed by this RFC.
Malformed Link headers that do not conform to RFC5988 will be quietly ignored making this loose validating.

Invoke-RestMethod would expose a `-FollowRelLink` switch to automatically follow the relation `next` links to the end if it exists.

For use cases that have to throttle REST API calls (such as querying GitHub API), recommendation would be to use Invoke-WebRequest for finer control.

## Alternate Proposals and Considerations

None

## PowerShell Committee Decision

### Voting Results

Jason Shirk: Accept 

Joey Aiello: Accept

Bruce Payette: Accept

Steve Lee: Accept

Hemant Mahawar: Accept

### Majority Decision

Commmittee agrees that this RFC satisfies the motivation for pagination in webcmdlets.  Requests:

- Resolve relative URLs to absolute URLs for Invoke-WebRequest which is needed for Invoke-RestMethod capability
- `-FollowRelLink` should have a way to limit the number of links to follow, default should be MaxInt.  When non-success code is received, it should output where it stopped.

### Minority Decision

N/A
