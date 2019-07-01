# RFC Meeting - July 1, 2019

## Attendees

* Steve Lee
* Dongbo Wang
* Jim Truher
* Joey Aiello

## Notes

* Spoke to Charu, and she believes she has an implementation of Get/Set-EnvironmentVariable to share
* Review `Out-GridView` RFC
    * Meta-question: do we need to review this module if it's only shipping in the Gallery?
        * RFC acceptance is not blocking publishing to the Gallery
        * Jim: "given our existing backlog, we should not spend too much time on RFCs that don't ship production code directly into PowerShell"
    * On its face, we don't see a problem publishing the module as written to the Gallery, but we don't
      believe Avalonia can ever be shipped as part of PowerShell.
* Reviewed and merged `Test-Connection` RFC0037 (PR #172)
* Reviewed #167 regarding `-Culture` and `-Comparison` parameters on `Select-String`
    * Shouldn't be too complicated: the discussion is difficult enough for us, much less for an IT admin
        * If you really want full functionality of comparers, it's fine to use C# APIs
    * Ultimately accepted Ilya's updated proposal 2 for its conformance to PowerShell's default case-insensitivity