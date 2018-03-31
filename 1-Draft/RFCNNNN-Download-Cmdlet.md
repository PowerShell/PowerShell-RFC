---
RFC: RFC
Author: Mark Kraus
Status: Draft
SupercededBy: 
Version: 0.1
Area: Utility
Comments Due: 2018-06-01
Plan to implement: Yes
---

# Download Cmdlet

This RFC is to propose the design of a new cmdlet specialized for remote file downloads to complement the existing Web Cmdlets.

## Motivation

    As a user of multiple protocols,
    I can easily download remote files,
    so that I can work with the file contents locally.

The current Web Cmdlets `Invoke-WebRequest` and `Invoke-RestMethod`
derive their value primarily from the processing they perform on the results of web requests over HTTP and HTTPS.
This focus brings great utility to the PowerShell language by easing the methods of dealing with remote content.
However, this focus makes it difficult to implement features that are not related to result processing.

Additionally, when PowerShell Core was in development,
there was a need to switch from the `WebRequest` API to the `HttpClient` API.
This change greatly enhanced the flexibility and ease of coding new features for HTTP and HTTPS requests and responses.
However, this came at the cost of support for FTP and FILE URIs.

Further, as the Web Cmdlets are centered around HTTP and HTTPS,
it would be difficult and inadvisable to implement additional protocols.

There is a gap in functionality between PowerShell and other tools and shells.
This gap was made apparent by the inclusion of `curl.exe` in Windows.
While PowerShell and the Web Cmdlets should not seek 100% parity with similar technologies,
The ability to download files over multiple protocols is an essential feature.
A PowerShell user will want a PowerShell native way to perform this task.

As there is a feature gap that is essential to fill
and the current Web Cmdlets are not an ideal candidate for closing that gap,
a new cmdlet should be added whose primary objective is to support file downloads over multiple protocols.

## Specification

`Invoke-Download` shall be the name of the cmdlet with the alias of `idl`.

### Syntax

The syntax shall be as follows:

```none
Invoke-Download -Uri <Uri> [-Credential <PSCredential>]
    [-Path <String> | -LiteralPath <String>]
    [-Proxy <Uri>] [-ProxyCredential <PSCredential>]
    [-Resume] [-PassThru] [-AsJob]
```

```none
idl -Uri <Uri> [-Credential <PSCredential>]
    [-Path <String> | -LiteralPath <String>]
    [-Proxy <Uri>] [-ProxyCredential <PSCredential>]
    [-Resume] [-PassThru] [-AsJob]
```

Examples:

```powershell
idl https://i.imgur.com/gXnRf4J.jpg
Invoke-Download -Uri 'https://i.imgur.com/gXnRf4J.jpg'
Invoke-Download -Uri 'https://i.imgur.com/gXnRf4J.jpg' -Path 'c:\images\rin.jpg'
Invoke-Download -Uri 'https://i.imgur.com/gXnRf4J.jpg' -PassThru | Get-content
```

### Relationship with Web Cmdlets

The cmdlet shall not share base code with the existing Web Cmdlets.
The cmdlet shall have a subset of the Web Cmdlets features that are specific or essential to downloading files.
For example, the cmdlet will not support body content nor HTTP methods other than GET.

As the cmdlet is not centered around HTTP and HTTPS,
the available features need to be generalized for multiple protocols.
To this end, advanced authentication features in the Web Cmdlets will not exist in the download cmdlet.
Most protocols expect a username and password.
At minimum, a `PSCredential` will be accepted.

### Automatic Filename Resolution Feature

A key feature of the cmdlet is to require nothing more than a URI.
Commands such as `curl` and `wget` have long supported automatic file name resolution.
This allows the user to supply just a URI and the file will be saved in the current working directory.
The file name is automatically chosen based on the URI and the presence of existing files.

The cmdlet will automatically chose a file name when a path is not supplied.
The filename will be chosen in this manner:

1. The last segment of `Uri.Segments`
1. In the case that the last segment is `/`, the file name `index.html` will be used
1. Illegal file name characters will be replaced with their URL encoded equivalent.
1. If a file with that name is present, the basename will be appended with a period followed by `DateTime.Now.ToString("yyyMMddHHmmssfff")`
1. The above rule will repeat until it locates a name that does not exist or 100 attempts
1. If in the unlikely event all the above fails, an error will be generated and the user will be suggested to supply a path or change directories.

The URI used for name generation will be the final URI the cmdlet arrives at in the case of redirection.

Example 1:

* Uri: `https://i.imgur.com/gXnRf4J.jpg`
* Current Directory: `c:\images\`
* Directory files: none
* Current Time: 2018-03-31 08:11:29.632
* File name: `c:\images\gXnRf4J.jpg`

Example 2:

* Uri: `https://i.imgur.com/gXnRf4J.jpg`
* Current Directory: `c:\images\`
* Directory files: `gXnRf4J.jpg`
* Current Time: 2018-03-31 08:11:29.632
* File name: `c:\images\gXnRf4J.20180331081129632.jpg`

Example 3:

* Uri: `https://i.imgur.com/gXnRf4J.jpg`
* Current Directory: `c:\images\`
* Directory files: `gXnRf4J.jpg`, `gXnRf4J.20180331081129632.jpg`
* Current Time: 2018-03-31 08:11:29.632
* File name: `c:\images\gXnRf4J.20180331081129912.jpg`

Example 4:

* Uri: `https://i.imgur.com/`
* Current Directory: `c:\images\`
* Directory files: none
* Current Time: 2018-03-31 08:11:29.632
* File name: `c:\images\index.html`

Example 5:

* Uri: `https://i.imgur.com/`
* Current Directory: `c:\images\`
* Directory files: `index.html`
* Current Time: 2018-03-31 08:11:29.632
* File name: `c:\images\index.20180331081129632.html`

Example 6:

* Uri: `https://i.imgur.com/gXnRf4J.jpg?these=query&elements=will&be=ignored`
* Current Directory: `c:\images\`
* Directory files: none
* Current Time: 2018-03-31 08:11:29.632
* File name: `c:\images\gXnRf4J.jpg`

Example 6:

* Uri: `https://i.imgur.com/gXnRf4J>.jpg`
* Current Directory: `c:\images\`
* Directory files: none
* Current Time: 2018-03-31 08:11:29.632
* File name: `c:\images\gXnRf4J%3e.jpg`

### PassThru Feature

By default the cmdlet will return no output.
The cmdlet is specialized for downloading.
The cmdlet will perform no content encoding or decoding.
The cmdlet only works with raw bytes.

As the cmdlets primary function is to create a local file from a remote file,
The only output it shall provide is the file it creates.

By issuing the `-PassThru` switch,
the cmdlet will return the equivalent of `Get-Item`.
This will allow the cmdlet to pass the item to commands such as `Get-Content`
where the user can specify what encoding features, if any, they require.

This allows the cmdlet code to focus only on download features and optimizations
without being coupled to the output ecosystem.
For example, should the language adopt UTF-32 as the standard encoding,
the cmdlet will not be coupled to that change.

### AsJob Feature

By default the cmdlet will run synchronously and block the current thread until completion.
By supplying `-AsJob`, the user can run the cmdlet as a background job.
As file downloads may take significant time,
the ability to begin the download and perform other tasks should be native to the cmdlet.

When supplied, the cmdlet will output a Job object.
This feature will work in a manner similar to other cmdlets which implement it.

### Multiple Protocol Support

The cmdlet shall support multiple protocols.
The cmdlet shall be extensible to support additional protocols.
The protocol handlers shall be extensible to handle additional features.

The extensibility is accomplished by interfaces.

```csharp
internal interface IInvokeDownloadProtocol
{
    Stream GetStream(PSCmdlet Cmdlet);
    Uri GetUri();
    void SetUri(Uri RequestUri);
}

internal interface IInvokeDownloadResumable
{
    void SetResume(bool Resume);
    bool GetResume();
    bool IsResumeSuccessful();
    bool IsResumeComplete();
}

internal interface IInvokeDownloadAuthenticator
{
    void SetCredential(PSCredential Credential);
    PSCredential GetCredential();
}

internal interface IInvokeDownloadProxyable
{
    void SetProxy(Uri ProxyUri);
    Uri GetProxy();
    void SetProxyCredential(PSCredential Credential);
    PSCredential GetProxyCredential();
}

internal interface IInvokeDownloadRedirectable
{
    IList<Uri> GetRedirectionChain(PSCmdlet Cmdlet, Uri RequestUri);
}
```

Class implementing these interfaces will be registered to a static internal dictionary

```csharp
public class InvokeDownloadCmdlet : PSCmdlet
{
    internal static Dictionary<String, IInvokeDownloadProtocol> ProtocolHandlers;
}
```

These APIs will begin as internal for several versions.
Once they are mature, they will be made public.
This will allow users to extend and replace protocols with their own handlers.
A set of default handlers will be initialized.
The initial default protocols will be `HTTP`, `HTTPS`, `FTP`, and `FILE`.

The protocol handler chosen will be based on the `Uri.Scheme`.
When a URI does not include a scheme, `HTTPS` will be assumed.
In the event that the supplied scheme has not been registered, an error will be thrown.

The protocol handlers are primarily responsible for returning a `Stream`.
The `Stream` will be used by the cmdlet.
The provided `Stream` will be read from and the results will be written to the file.

`IInvokeDownloadRedirectable` handlers indicate that the protocol supports redirection.
They are responsible for following the redirection chain and returning the URIs in that chain.
The cmdlet will use the final URI to perform the download and resolve the file name if required.

### Resume Feature

For protocols that support resuming failed or interrupted file transfer,
the protocol handler can implement `IInvokeDownloadResumable`.

The protocol handler will be responsible for its own resume implementation.
The protocol handler will indicate if resume was successful.
The protocol handler will indicate if the resume is already complete,
i.e. the file is already completely downloaded and there is nothing to resume.

If the resume was successful, the cmdlet will append to existing file.
If resume was not successful, the cmdlet will attempt a full download load of the same file.
If the resume is already complete, no action will be taken.

## Alternate Proposals and Considerations

Instead of being extensible, the cmdlet could be a closed system.
There have been many lessons learned from the Web Cmdlets with regard to extensibility.
Adding features is made difficult when all the code is too tightly coupled.

Additional Features currently present in the Web Cmdlet might be considered.
`-Headers` for example may be useful.
However, the goal should be to keep the cmdlet focused on simple downloads.
Complex download scenarios can still be handled by the Web Cmdlets.
Also, the cmdlet should be considered as protocol agnostic as possible.
`-Headers` may make sense for `HTTP` and `HTTPS`,
but they will make no sense for `SSH`.

Alternate automated file name resolution strategies could be considered.
`wget` uses `.1` at the end of a file when a file exists and increments until if fined s file name that doesn't exist.
`index.html` is also questionable as default when `/` is the only `Uri.Segment`.
