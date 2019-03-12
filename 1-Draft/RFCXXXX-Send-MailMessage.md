---
RFC: RFCXXXX
Author: Travis Plunk
Status: Draft
SupercededBy: N/A
Version: 1.0
Area: cmdlets
Comments Due: 4/30/2019
Plan to implement: No
---

# Deprecating `Send-MailMessage`

`Send-MailMessage` does not support many modern protocols leading to the inability to use this with modern secure mail services.
See [DotNet DE0005](https://github.com/dotnet/platform-compat/blob/master/docs/DE0005.md).
Since the protocols it uses are no longer considered secure, the cmdlet as is should not be used.
I recommend that we implement a plan to remove the cmdlet from powershell core and, if needed,
re-implement as a separate module.

## Motivation

    Most major mail platforms now have REST methods to send mail, which allows `Invoke-RestMethod' to allow sending mail messages.

    * gmail
      * https://developers.google.com/gmail/api/v1/reference/users/messages/send
      * https://stackoverflow.com/questions/24460422/how-to-send-a-message-successfully-using-the-new-gmail-rest-api
    * Office365
      * https://docs.microsoft.com/en-us/previous-versions/office/office-365-api/api/version-2.0/mail-rest-operations#SendMessages

## Removal Plan

### Add a warning to 6.2 after the release of 6.2

In a 6.2 servicing release, we will add a warning to `Send-MailMessage` that an alternative method should be found.

### Remove the cmdlet in 6.3

During the development of 6.3, remove the `Send-MailMessage` cmdlet.

### If needed, develop an alternative

If needed, develop a new cmdlet based on the DotNet recommended solution of [MailKit](https://github.com/jstedfast/MailKit).

## Alternate Proposals and Considerations

The community could develop a replacement that can could be incorporated back into PowerShell Core.
This has the disadvantage of increasing the size of PowerShell Core and delaying the solution.

### GMail REST API

To use the GMail REST API the following additional items are needed

### WebSafeBase64

For GMail, a MIME message is required to be encoded in this format.
https://code.google.com/p/stringencoders/wiki/WebSafeBase64

### Something to compose the MIME message

For GMail, you have to pass the MIME message.

---------------

## PowerShell Committee Decision

### Voting Results

### Majority Decision

### Minority Decision

N/A
