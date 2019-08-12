---
RFC: RFC0042
Author: Travis Plunk
Status: Final
SupercededBy: N/A
Version: 1.0
Area: cmdlets
Comments Due: 4/30/2019
Plan to implement: No
---

# Obsoleting `Send-MailMessage`

`Send-MailMessage` does not support many modern protocols leading to the inability to use this with modern secure mail services.
See [DotNet DE0005](https://github.com/dotnet/platform-compat/blob/master/docs/DE0005.md).
Since the protocols it uses are no longer considered secure, the cmdlet as is should not be used.
I recommend that we implement a plan to remove the cmdlet from powershell core and, if needed,
re-implement as a separate module.

## Motivation

### Primary

The underlying API, [`SmtpClient`](https://docs.microsoft.com/dotnet/api/system.net.mail.smtpclient), doesn't support many modern protocols.
It is compat-only.
It's great for one off emails from tools, but doesn't scale to modern requirements of the protocol.

### Secondary

    Most major mail platforms now have REST methods to send mail, which allows `Invoke-RestMethod` to allow sending mail messages.

    * gmail
      * https://developers.google.com/gmail/api/v1/reference/users/messages/send
      * https://stackoverflow.com/questions/24460422/how-to-send-a-message-successfully-using-the-new-gmail-rest-api
    * Office365
      * https://docs.microsoft.com/en-us/previous-versions/office/office-365-api/api/version-2.0/mail-rest-operations#SendMessages

## Obsoletion  Plan

### Add a warning to 6.2

In a 6.2, we have added an obsolete warning to `Send-MailMessage` that an alternative method should be found.

**Note:** Adding an obsolete warning should not break compatibility in any way.

### If needed, develop an alternative

If needed, develop a new cmdlet based on the DotNet recommended solution of [MailKit](https://github.com/jstedfast/MailKit).

## Alternate Proposals and Considerations

### Remove the cmdlet during 6.3

During the development of 6.3, remove the `Send-MailMessage` cmdlet.

Feedback on this option was negative and the RFC has been updated with then next most reasonable option.

### Community replacement

The community could develop a replacement that can could be incorporated back into PowerShell Core.
This has the disadvantage of increasing the size of PowerShell Core and delaying the solution.

### Positive confirmation of issue in 6.3

We could add a switch requiring the user to accept the risk of using older **possibly** less secure implementations.
This switch would be called `-AllowUnsecureConnection` and would be mandatory.

**Note:** The switch name is based on the [DSC Property](https://docs.microsoft.com/en-us/powershell/dsc/managing-nodes/metaconfig#configuration-server-blocks)

However, given the high usage of `Send-MailMessage` in automated scenarios (like scheduled tasks),
and the plan to work on a `MailKit` based alternative, we'd rather leave the cmdlet working as-is
for the time being.

### Additional considerations

#### GMail REST API

To use the GMail REST API the following additional items are needed

#### WebSafeBase64

For GMail, a MIME message is required to be encoded in this format.
https://code.google.com/p/stringencoders/wiki/WebSafeBase64

#### Something to compose the MIME message

For GMail, you have to pass the MIME message.
