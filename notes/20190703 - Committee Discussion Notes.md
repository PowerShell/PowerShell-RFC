# PowerShell Committee Notes - July 3, 2019

## Attendees

* Steve Lee
* Jim Truher
* Dongbo Wang
* Joey Aiello
* Kenneth Hansen

## Notes

* Fix New-Item -Path/-Target with wildcard char #9259
    * No attached issue for discussion
    * Issue is that tab completion uses single quotes and escaped chars in what amounts to a literal path
    * Agreed not to fix it in the `New-Item` behavior, but we should fix it in the tab completion
* OneDrive mode for offline vs. online files #9895
    * OneDrive uses "surrogate reparse points" for its files
    * Disagreement around whether PS should report the filesystem as-is reported by the API vs. what the shell tells users in the File Explorer
    * PS already matches `cmd` by reporting `Length` with parens for non-cached files
    * Agreed to leave files as reporting `l` in the mode
    * Jim was concerned .NET Core not making their Win32 wrapper API public
    * Might make sense to add a property for surrogate reparse points to make the info scriptable
