---
RFC: RFC0000
Author: Steve Lee, Joey Aiello
Status: Draft
Area: Process
Version: 1.4
Feedback: https://github.com/PowerShell/PowerShell-Language-RFC/issues/5
---

# PowerShell RFC Process and Guidelines

A PowerShell RFC (Request for Comments) is a publication to propose design changes and improvements to PowerShell.
This provides the community an opportunity to provide feedback before code is written where it becomes harder to change at the risk of
compatibility.
The complete list of RFCs are available at https://github.com/powershell/powershell-rfc

This process was adapted from the Chef RFC process as well as from the DMTF.org process,
and then heavily modified based on the experiences of maintaining PowerShell over the first years
of the project.

## Roles

* **Author**: All members of the community are allowed to author new RFCs and can provide feedback to any RFC.
* **PowerShell Committee**: The design committe that votes to accept or reject an RFC.
(Learn more about the PowerShell Committee [here](https://github.com/PowerShell/PowerShell/blob/master/docs/community/governance.md#powershell-committee).)
* **Committee Member**: An individual member of the PowerShell Committee.
* **Working Group (WG)**: A group responsible for deciding whether or not an issue proposal in
  PowerShell/PowerShell, as well as for providing feedback within an RFC proposal.
  (Learn more about Working Groups [here](https://github.com/PowerShell/PowerShell/blob/master/docs/community/working-group.md).)

## RFC Submission Convention

* When submitted, RFC documents shall follow the naming convention of `RFCNNNN-<Title>.md`.
* Authors of RFCs shall not assign the RFC number (leave the `NNNN` in the filename).
* When the Pull Request is submitted, the author shall check `Allow edits from maintainers` so that the Committee can add the RFC number to the draft, update the title, and fix the filename.

## RFC Template

All RFC documents shall follow the [RFC template](RFCNNNN-New-RFC-Template.md).

## RFC Workflow

RFCs go through the following stages:

### Draft PR

This is the initial draft of an RFC posted for comments and considered a work-in-progress.

* New drafts should not be submitted until the appropriate Working Group has decided in
  a PowerShell/PowerShell repository issue that an RFC is required in order for an experimental
  implementation to be accepted.
* Drafts should be submitted as a draft Pull Request from the Author's fork into the
  `Draft-Accepted` folder.
* The Author shall ensure that the `Allow edits from maintainers` box is checked so that a Committee
  Member can assign the next RFC number.
* As a draft PR, contributors are encouraged to comment on the content of the submission.

### Non-Draft PR

* After two months, and when the Author feels that they've addressed feedback on the RFC,
  the Author should move the RFC PR out of the draft state.
  This signals to the Committee that the RFC is ready for their formal review.
* At this point, the Committee will add a `Review - Committee` label.

### Experimental Feature

Experimental features are used to provide a working example of proposed designs in order for the
Committee and other users to understand real-world usage of the proposal.

* If the Committee feels that the RFC is sound, or that it needs more real-world usage to be finalized,
  they will add the `Experimental - Approved` label
  to indicate that an implementation is okay (from the Committee perspective) to merge as an
  experimental feature in PS/PS.
* The Author may be asked to continue to update the RFC as the usage of the experimental feature
  drives new insight into how the feature should be designed.
* The Committee shall vote to merge or reject the RFC.

Note: the Committee may be slower to respond to RFCs where the Author has indicated that they do not plan to implement the RFC.

### Accepted or Rejected

* Feedback from the experimental implementation and RFC has been reviewed.
* Because this working prototype already exists in preview builds available on GitHub, the community
  provides feedback on the implementation as issues in the [PowerShell/PowerShell repository](https://github.com/powershell/powershell).
* As the engineering team or contributor works towards a final implementation,
  the contributor should submit PRs to PowerShell-RFC in order to keep the RFC in sync with the implementation.
* The Committee may ask the Author to continue updating the RFC as real world usage of the experimental
  feature provides new information.
* When the Committee feels that they have enough information from their own usage of and user
  feedback on the experimental feature to vote on the RFC,
  they will vote to accept or reject the RFC.
  This will be done through a simple merge/rejection of the RFC PR.

### Rejected

If the Committee decides not to proceed with the RFC, its PR is simply closed before it is merged.
The Committee should also add the `Rejected` label to denote that the Committee
rejected the RFC.
In the future, this can be done automatically with GitHub Actions.

### Withdrawn

Similarly, if an Author decides to withdraw their RFC, it will be closed before it is merged.
The Committee should also add the `Withdrawn` label to denote that the author
withdrew the RFC.
In the future, this can be done automatically with GitHub Actions.

### Final

RFCs in the `Final` state are considered fully complete and implemented in PowerShell.
Any proposed changes should be made through a new RFC or via an Issue in the [PowerShell/PowerShell repository](https://github.com/powershell/powershell).
New RFCs should reference old RFCs where applicable.

## History

v1.1 - 5-20-2016 - Updated to enable RFCs for design changes that don't require code changes.
Added Draft-Accepted state and Version header property.

v1.2 - 8-18-2016 - Open submitting RFCs to the community and update formatting.

v1.3 - 9-26-2016 - Added Withdrawn stage.  Comments Due field to template.  Updated guidance on RFC numbering.

v1.3.1 - 3-22-2017 - Cleaned up language and made explicit clarifications to process

v1.4 - 3-31-2017 - Revised the RFC process to leverage pull requests for comments instead of issues.
Continued cleanup of language and formatting.

v2.0 - 1-12-2021 - Updated process to leverage draft PRs and Working Groups (WGs) instead of folders.
