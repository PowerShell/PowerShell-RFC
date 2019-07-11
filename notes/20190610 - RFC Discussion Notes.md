# RFC Meeting - June 10, 2019

## Attendees

* Steve Lee
* Dongbo Wang
* Joey Aiello

## Action Items

* Joey: ping @charub
* Joey: write committee member RFC
* Steve: re-rev PowerShellGet 3 RFC
* Steve: PR for experimental features

## Notes

* Experimental Features
    * Agreement to take the following out of experimental due to low risk and interactivity:
        * Temp drive
        * Abbreviation expansion
            * Will wait for feedback to see if this should be configurable to "off"
    * Will wait on Cloud Shell -> Exchange Online usage to take implicit remoting batching out of experimental
        * Check to see if they enable it already today
    * Will wait on "Telemetry 2.0" before we make general decision on how to take features out of experimental
    * Fuzzy matching suggestion on CommandNotFound should be blocked on suggestion framework fixes
        * suggestions are batched to the end of a script
        * not shown in VS Code
* Outside Committee members
    * AI Joey: Need to write an RFC on process for accepting outside committee members
        * Nomination will be fuzzy (based on very holistic criteria)
        * Needs formalized requirements on how they get accepted
        * Requirements to stay on should be defined
            * Apache +0/-0 system
        * Requirements should maybe be loose initially just to get moving
        * Definitely need to standard for removing people based on non-participation criteria (e.g. code of conduct)
    * Should start small (probably with one, maybe two people)
    * Geographic tradeoff: domestic is good for calls, international would push asynchronicity
    * Start by looking at Apache Foundation and Debian to see their process
* Reviewed and commented on Test-Connection (#172)
* Agreed to use label "Review - Committee" for RFCs that have been partially reviewed/discussed
  and may be waiting on additional review
* Need to close on PowerShell 6 specific module path (#133) once we have quorum
* @charub is back, we need to close with her on whether Environment Variable (#92) code exists
