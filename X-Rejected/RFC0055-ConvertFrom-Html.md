---
RFC: '0055'
Author: Josh Rickard
Status: Rejected
SupercededBy: 
Version: 1.0
Area: PowerShell Core Web CmdLets
Comments Due: August 31st, 2018
Plan to implement: 
---

# ConvertFrom-Html

The proposal is to create a new 'ConvertFrom-Html' cmdlet that will convert Html strings using PowerShell core.

Currently the PowerShell Core Web CmdLets do not have access to the `HtmlWebResponseObject` and currently only contains the `BasicHtmlWebResponseObject` type.  Because of this, the capability to parse HTML using the `ParsedHtml` property of the `HtmlWebResponseObject` type does not exist within PowerShell Core.

Windows PowerShell does contain the `HtmlWebResponseObject`, but PowerShell Core currently only contains the `BasicHtmlWebResponseObject` type.

Additionally, Windows PowerShell Web CmdLets utilize Internet Explorer to parse HTML content. Since non-Windows systems do not have Internet Explorer, PowerShell Core utilizes the `BasicHtmlWebResponseObject` which does not contain this property.

This RFC proposes that the creation of a new CmdLet named `ConvertFrom-Html`.  This CmdLet is to be implemented into PowerShell Core and should utilize the [AngelSharp](https://github.com/AngleSharp/AngleSharp) framework for converting HTML strings into a PSCustomObject.

## Motivation

As a PowerShell Core user, I can convert HTML content to objects so that I can easily work with downloaded or local HTML content.

As a IT Administrator, I can call `Invoke-WebRequest` and then use `ConvertFrom-Html` to convert the `Content` of my Web Request to a PSCustomObject so that I can easily work with HTML strings/content.

As a IT Administrator, I can call `Invoke-WebRequest` and then use `ConvertFrom-Html` to convert the `Content` of my Web Request to a PSCustomObject so that I can easily convert it to another type (json, csv, xml, etc.).

As a IT Administrator, I can pipe a string into `ConvertFrom-Html` to convert it to a PSCustomObject so that I can easily convert it to another type, modify the object, and use the `ConvertTo-Html` CmdLet to convert it back to Html.

## Specification

- InputObject parameter
  - Specifies the HTML strings to convert to PSCustomObject objects. Enter a variable that contains the string, or type a command or expression that gets the string. You can also pipe a string to ConvertFrom-Html.
  - The InputObject parameter is required, but its value can be an empty string. When the input object is an empty string, ConvertFrom-Html does not generate any output. The InputObject value cannot be $Null.

### Syntax

```text
ConvertFrom-Html [-InputObject] <String> [<CommonParameters>]
```

### PARAMETERS

#### -InputObject

Specifies the HTML strings to convert to HTML objects.
Enter a variable that contains the string, or type a command or expression that gets the string.
You can also pipe a string to **ConvertFrom-Html**.

The *InputObject* parameter is required, but its value can be an empty string.
When the input object is an empty string, **ConvertFrom-Html** does not generate any output.
The *InputObject* value cannot be $Null.

```yaml
Type: String
Parameter Sets: (All)
Aliases:

Required: True
Position: 0
Default value: None
Accept pipeline input: True (ByValue)
Accept wildcard characters: False
```

### INPUTS

#### System.String

You can pipe a HTML string to **ConvertFrom-Html**.

### OUTPUTS

#### PSCustomObject

### Examples

You can provide a string to convert to a Html object using the `InputObject` Parameter

```powershell
ConvertFrom-Html -InputObject $InvokeWebRequestObject
```

You can provide a string to convert to a Html object using Position 0 (`InputObject`) parameterization:

```powershell
ConvertFrom-Html $InvokeWebRequestObject
```

You can pipe a string into `ConvertFrom-Html`:

```powershell
$htmlString = @"
<HTML>

<HEAD>
    <TITLE>Your Title Here</TITLE>
</HEAD>

<BODY BGCOLOR="FFFFFF">
    <CENTER>
        <IMG SRC="clouds.jpg" ALIGN="BOTTOM"> </CENTER>
    <HR>
    <a href="http://somegreatsite.com">Link Name</a>
    is a link to another nifty site
    <H1>This is a Header</H1>
    <H2>This is a Medium Header</H2>
    Send me mail at
    <a href="mailto:support@yourcompany.com">
        support@yourcompany.com</a>.
    <P> This is a new paragraph!
        <P>
            <B>This is a new paragraph!</B>
            <BR>
            <B>
                <I>This is a new sentence without a paragraph break, in bold italics.</I>
            </B>
            <HR>
</BODY>

</HTML>
"@

$htmlObject = $htmlString | ConvertFrom-Html
```

Advanced example using Invoke-WebRequest and converting the returned content to a PSCustomObject.

```powershell
$dnsDumpsterURL = 'https://dnsdumpster.com/'
$dumpsterRequest = Invoke-WebRequest -Uri $dnsDumpsterURL -SessionVariable session

$props = @{
    Uri        = $dnsDumpsterURL
    Headers = @{Referer = $dnsDumpsterURL; 'Content-Type' = 'application / x-www-form-urlencoded'}
    WebSession = $session
    Body       = @{
      'csrfmiddlewaretoken' = $dumpsterRequest.InputFields.value;
      'targetip' = 'microsoft.com'
    }
    Method     = 'Post'
}

$dnsDumpsterObject = Invoke-WebRequest @props | ConvertFrom-Html
```

## Alternate Proposals and Considerations

Some considerations to keep in mind:

- Converted Html may be piped to any number of CmdLets.  For example, ConverTo-Json, ConverTo-Csv, ConvertTo-Xml, and ConvertTo-Html
- Based on conversations in #3267 and #2867, this CmdLet should use AngleSharp to parse Html strings and output a PSCustomObject
- We should support the same platforms that PowerShell is supported on: Win32, Ubuntu 14/16, CentOS7, MacOS10.
