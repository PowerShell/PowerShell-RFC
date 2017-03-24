---
RFC: RFC0000
Author: Steve Lee
Status: Draft
Area: Process
Version: 1.3.1
Feedback: https://github.com/PowerShell/PowerShell-Language-RFC/issues/5
---

# PowerShell RFC Process and Guidelines

A PowerShell RFC (Request for Comments) is a publication to propose design changes and improvements to PowerShell.
This provides the community an opportunity to provide feedback before code is written where it becomes harder to change at the risk of 
compatibility.
The complete list of RFCs are available at https://github.com/powershell/powershell-rfc

This process was adapted from the Chef RFC process as well as from DMTF.org process.

## Roles

* **Author**: All members of the community are allowed to author new RFCs and can provide feedback to any RFC.
* **PowerShell Committee**: The design committe that votes to accept or reject an RFC.
(Learn more about the PowerShell Committee [here](https://github.com/PowerShell/PowerShell/blob/master/docs/community/governance.md#powershell-committee).)
* **Committee Member**: An individual member of the PowerShell Committee.

## RFC Naming Convention

RFC documents shall follow the naming convention of "RFCNNNN-Title.md".
Authors of RFCs do not assign an RFC number.
When the Pull Request is submitted, ensure `Allow edits from maintainers` is checked so that the Committee can add the RFC number to the draft and also update the title accordingly.

## RFC Template

RFC documents shall follow the following template:

```markdown
---
RFC: RFC<four digit unique incrementing number - assigned by Committee>
Author: <First Last>
Status: <Draft | Experimental | Accepted | Rejected | Withdrawn | Final>
SupercededBy: <link to another RFC>
Version: <Major>.<Minor>
Area: <Area within the PowerShell language>
Comments Due: <Date for submitting comments to current draft (minimum 1 month)>
---

# Title

Description and rationale.

## Motivation

    As a <<user_profile>>,
    I can <<functionality>>,
    so that <<benefit>>.

## Specification

## Alternate Proposals and Considerations

```

## RFC Workflow

RFCs go through applicable stages:

* Draft

This is the initial draft of a RFC posted for comments and considered a work-in-progress.

New proposed drafts should be submitted as a Pull Request from the Author's fork.
The Author must ensure that 'Allow edits from maintainers' is checked so that a Committee Member can assign the next RFC number.

Committee Members also ensure that the RFC adheres to the template and the RFC process before merging the Pull Request.

After the Pull Request is merged, Maintainers will ask the Author to create an issue for this RFC to collect and respond to feedback on the RFC.
(The Author should create issue so that they get notified of new comments instead of the Committee Member.)
Typically, one or two months is allowed for comments (see `Comments Due` above).

At this point, the community discusses the RFC.

After the `Comments Due` date, the Author summarizes the comments, submits a new Pull Request to make the final RFC Draft, and asks the PowerShell Committee to vote.

Just before voting, the PowerShell Comittee merges the Pull Request.
The PowerShell Committee then votes to accept or reject the RFC Draft.

* Draft-Accepted

The PowerShell Committee has reviewed the RFC Draft and comments, and has voted to accept the RFC Draft.

At this point, new comments are not being sought.

No one has begun implementing the RFC, and there are no current plans to implement the RFC.

The community is invited to implement the RFC.

* Experimental

Comments have been reviewed and code is being written to provide an working example of the proposed design change to get further feedback.

* Experimental-Accepted

Feedback from the experimental implementation and RFC have been reviewed.
Engineering team will work towards final implementation in code to match the RFC.

* Rejected

Based on community feedback, this RFC was decided to not proceed any further.

* Withdrawn

Author has decided not to pursue this RFC any further

* Final

Design and implementation is considered complete.
Any proposed changes would be through a new RFC.

## History
v1.1 - 5-20-2016 - Updated to enable RFCs for design changes that don't require code changes.
Added Draft-Accepted state and Version header property.

v1.2 - 8-18-2016 - Open submitting RFCs to the community and update formatting.

v1.3 - 9-26-2016 - Added Withdrawn stage.  Comments Due field to template.  Updated guidance on RFC numbering.

v1.3.1 - 3-22-2017 - Cleaned up language and made explicit clarifications to process
