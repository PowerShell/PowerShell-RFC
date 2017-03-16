---
RFC: 0018
Author: Ilya Sazonov
Status: Draft-Accepted
SupercededBy: n/a
Version: 0.1
Area: Cmdlet
Comments Due: 3/2/2017
---

# Get-Hash cmdlet

Add the new cmdlet to PowerShell to get cryptographically strong hashes for strings, files and file streams.

Currently Powershell only has Get-FileHash cmdlet to get a file hashes.
The cmdlet enhance and replace `Get-FileHash` cmdlet to support strings. `Get-FileHash` becomes an alias of `Get-Hash`.

Current .Net implementation of GetHashCode method calculates a hash for strings that is not a global (across application domains) safe and is not a permanent value.
According to the documentation, this method has many practical limitations.
https://msdn.microsoft.com/en-us/library/system.object.gethashcode%28v=vs.110%29.aspx
https://msdn.microsoft.com/en-us/library/system.string.gethashcode%28v=vs.110%29.aspx
>As a result, the hash codes should never be used outside of the application domain in which they were created, they should never be used as key fields in a collection, and they should never be persisted.

Also users are forced to use a workaround to get a string hashes:
```powershell
Get-FileHash -InputStream ([System.IO.MemoryStream]::new([System.Text.Encoding]::UTF8.GetBytes("test string")))
```

## Motivation

With the new cmdlet users can use native Powershell cmdlet syntax without a workaround.

With the new cmdlet users can get string hashes which:

* is a cryptographically strong hashes
* is across-platform
* may be sended across application domains
* may be saved in databases
* may be used as key in collections
* may be used to compare strings and files

## Specification

### Output

The cmdlet output objects of `StringHashInfo` type for `StringHash` parameter set:

```powershell
public class StringHashInfo
{
    public string Algorithm { get; set;}
    public string Hash { get; set;}
    public string Encoding { get; set;}
    public string HashedString { get; set;}
}
```

The cmdlet output objects of `FileHashInfo` type for `PathParameterSet`, `LiteralPathParameterSet`, `StreamParameterSet` parameter sets:

```powershell
public class FileHashInfo
{
    public string Algorithm { get; set;}
    public string Hash { get; set;}
    public string Path { get; set;}
}
```

`Algorithm` is a hash algorithm name.

`Hash` is a hash of the file or the `HashedString` string calculated with `Algorithm` algorithm.

`Encoding` is a encoding of the `HashedString` string. It is not used for file hashes because we read files as binary.

`Path` is from `Path` or `LiteralPath` parameter. It is `null` for file streams.

### Parameter Sets

1. `StringHash`

2. `PathParameterSet`

3. `LiteralPathParameterSet`

4. `StreamParameterSet`

### Parameters

1. InputString (`StringHash` parameter set)

    Type = `array of strings`

    Position = 0

    ##### Attributes

    ValueFromPipeline = true

    AllowNull()

    AllowEmptyString()

        We allow `null` and `empty` strings because we can get it from pipeline. See below `#InputString accept null and empty input strings`

2. Algorithm (All parameter sets)

    Type = `string`

    Position = 1

    ##### Attributes

    ValidateSet = `SHA1`, `SHA256`, `SHA384`, `SHA512`, `MD5`

3. Encoding (`StringHash` parameter set)

    Type = `[Microsoft.PowerShell.Commands.FileSystemCmdletProviderEncoding]`

    Position = 2

4. Path (`PathParameterSet` parameter set)

    Type = `array of strings`

    Position = 0

    ##### Attributes

    ValueFromPipelineByPropertyName = true

5. LiteralPath (`LiteralPathParameterSet` parameter set)

    Type = `array of strings`

    Position = 0

    ##### Attributes

    Alias("PSPath")

5. InputStream (`StreamParameterSet` parameter set)

    Type = `Stream`

    Position = 0

    ##### Attributes

    ValueFromPipelineByPropertyName = true

### Pipeline

All of `InputString`, `Path`, `LiteralPath` parameters has `string` type and we can use `ValueFromPipeline` only for one parameter - `InputString`.
`Path`, `LiteralPath` parameters is accepted from pipeline by property name.
(If we want `ValueFromPipeline` for `Path` we should split on `Get-StringHash` and `Get-FileHash`.)

### Encoding

.Net hash algorithm methods work on byte buffer.

We read files as binary and so no encodings needed.

For strings the cmdlet must convert a input string to byte array. This process is a encoding sensitive. To correctly process users must specify the encoding of incoming strings by `Encoding` parameter.

### InputString accept null and empty input strings

.Net hash algorithm methods can calculate a hash for empty string without exception.

.Net hash algorithm methods throw for null input buffer.
This behavior is not convenient when processing an array of strings by using a pipeline:
```powershell
    "test1", $null, "test2" | Get-Hash
```
Best behavior is to create null as result hash for null string and generate non-terminating error.

## Alternate Proposals and Considerations

We cosider `Base64` as special encoding and not a hash. It is recomended to implement `Base64` encoding in some `ConvertTo-Base64` and `ConvertFrom-Base64` cmdlets.

We could split the `Get-Hash` functionality on `Get-FileHash` and `Get-StringHash` cmdlets or even `Get-FileHash`, `Get-StreamHash` and `Get-StringHash`.

---------------
## PowerShell Committee Decision

### Voting Results

Jason Shirk: Accept 

Joey Aiello: Accept

Bruce Payette: Accept

Steve Lee: Accept

Hemant Mahawar: Accept

### Majority Decision

Commmittee agrees that this RFC satisfies the motivation for supporting hashing of objects other than files.  Request is to ensure default parameter set matches existing `Get-FileHash` to maintain compatibility.  This should be in Microsoft.PowerShell.Utility module.

### Minority Decision

N/A
