---
RFC: 0018
Author: Ilya Sazonov
Status: Draft
SupercededBy: n/a
Version: 0.1
Area: Cmdlet
Comments Due: 3/2/2017
---

# Get-StringHash cmdlet

Add the new cmdlet to PowerShell to get cryptographically strong hashes for strings.

Current .Net implementation of GetHashCode method calculates a hash that is not a global (across application domains) safe and is not a permanent value.
According to the documentation, this method has many practical limitations.
https://msdn.microsoft.com/en-us/library/system.object.gethashcode%28v=vs.110%29.aspx
https://msdn.microsoft.com/en-us/library/system.string.gethashcode%28v=vs.110%29.aspx
>As a result, hash codes should never be used outside of the application domain in which they were created, they should never be used as key fields in a collection, and they should never be persisted.

Currently Powershell only has Get-FileHash cmdlet to get a file hashes.
Users are forced to use a workaround to get a string hashes:
```powershell
Get-FileHash -InputStream ([System.IO.MemoryStream]::new([System.Text.Encoding]::UTF8.GetBytes("test string")))
```

## Motivation

With the new cmdlet users can use native Powershell cmdlet syntax without a workaround.

With the new cmdlet users can get hashes which:

* is a cryptographically strong hashes
* is across-platform
* may be sended across application domains
* may be saved in databases
* may be used as key in collections
* may be used to compare strings and texts

## Specification

### Output

The cmdlet output objects of `StringHashInfo` type:

```powershell
public class StringHashInfo
{
    public string Algorithm { get; set;}
    public string Hash { get; set;}
    public string Encoding { get; set;}
    public string HashedString { get; set;}
}
```
`Hash` is a hash of the `HashedString` string calculated with `Algorithm` algorithm.

`Encoding` is a encoding of the `HashedString` string.

### Parameters

1. InputString

    Type = array of strings

    Position = 0

    ##### Attributes

    ValueFromPipeline = true

    AllowNull()

    AllowEmptyString()

2. Algorithm

    Type = string

    Position = 1

    ##### Attributes

    ValidateSet = SHA1, SHA256, SHA384, SHA512, MD5

3. Encoding

    Type = [Microsoft.PowerShell.Commands.FileSystemCmdletProviderEncoding]

    Position = 2

### Encoding

.Net hash algorithm methods work on byte buffer. So the cmdlet must convert a input string to byte array.
This process is a encoding sensitive.
To correctly process the user must specify the encoding of incoming strings.

### InputString accept null and empty input strings

.Net hash algorithm methods can calculate a hash for empty string without exception.

.Net hash algorithm methods throw for null input buffer.
This behavior is not convenient when processing an array of strings by using a pipeline:
```powershell
    "test1", $null, "test2" | Get-StringHash
```
Best behavior is to create null as result hash for null string and generate non-terminating error.

## Alternate Proposals and Considerations

None
