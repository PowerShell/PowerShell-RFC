# RFC Meeting - July 8, 2019

## Attendees

* Steve Lee
* Jim Truher
* Joey Aiello

## Notes

* Review improvement of argument generation for native executables #90
    * AI next meeting: get quorum decision
    * okay implementing behind experimental feature
* Environment variable cmdlets #92
    * Charu has the code, Jim wants to give it a look
* Reviewed HTML cmdlets #137
    * Inspired meta conversation on when/how we take modules into PS Core
        * Jim: "we need a 'feeder' system that allows folks to incubate modules on the Gallery"
        * We need to publish a blog that gives our position on this
    * AngleSharp *seems* like the best option, but we should really see an implementation
* Reviewed policy RFC #180
    * Jim had a couple concerns
        * Window of opportunity as PowerShell starts where they could execute code before machine takes precedence
        * User can use reflection to change object model built after policy has been applied
