---
RFC: RFC0000
Author: Steve Lee
Status: Draft
Area: Process
Version: 1.3
Feedback: https://github.com/PowerShell/PowerShell-Language-RFC/issues/5
---

# PowerShell RFC Process and Guidelines

A PowerShell RFC (Request for Comments) is a publication to propose design changes and improvements to PowerShell.
This provides the community an opportunity to provide feedback before code is written where it becomes harder to change at the risk of 
compatibility.
The complete list of RFCs are available at https://github.com/powershell/powershell-rfc

This process was adapted from the Chef RFC process as well as from DMTF.org process.

## Roles

All members of the community are allowed to author new RFCs and can provide feedback to any RFC.

## RFC Naming Convention

RFC documents shall follow the naming convention of "RFC####-Title.md" where #### is the RFC number and Title is the title of the document.
Authors of RFCs do not assign an RFC number.  The Committee will assign an RFC number and update the title accordingly.

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
Comments Due: <Date for submitting comments to current draft>
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
New proposed drafts should be submitted as a Pull Request from your fork.
Ensure the 'modifiable by maintainers' is checked so that the maintainers can assign the next RFC number.
Maintainers only ensure that the RFC adheres to the template and process.
Maintainers will ask the author to create an issue for this RFC and used for collection and response to comments (author creates it so that they get notified of new comments instead of the maintainer).
Typically, one or two months is allowed for comments.
When the author is ready to submit to the committee for voting, they submit a Pull Request and indicate they are ready for voting.

* Draft-Accepted

Comments have been reviewed and new comments are not being sought.
Code work has not started/planned or not needed.

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
