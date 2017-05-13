---
RFC: 0008
Author: Sr Gom
Status: Draft
Version: 1.0
Area: Interactive Shell
Comments Due: 2016-10-26
---

# Save Last Result

Powershell should automatically save the output/result of last command(s) to facilitate reuse.

##Motivation

There are instances in using Powershell's `Get` commands where, because of Powershell's facility with passing objects around for piping, the command is written in a way such that the output is verbose with "all the information". Getting the desired output in a specific format in Powershell can take two steps, one the raw command and the other with appropriate filters and selctions of the raw command. It would be useful, specially in case of long running commands, if the user had to launch the command itself only once and could reuse the result from the first invocation the second time. 

##Specification

Powershell should allow command output reuse. Such a reuse could be limited to the interactive command line shell and be not available in the scripting environment to prevent breaking already existing scripts. The reuse would be affected by populating one or more variables. The following two variables should be used for that:

* Populate a single variable `$LastResult` that automatically holds the output of a single line of command upon the completion of that lne.

* Populate an array with `$Result[0]` holding the output of the first command, `$Result[$Result.length-1]` holding the output of the most recent command. We will call such an array LRU array. This is different from the behavior of another self-populated variable, `$Error` which has the most recent error in `$Error[0]`. We call such an array MRU array. 

Scala REPL does something similar except the output is stored in separate variables. 

Some other measures related to this proposal:

* For each rendering of Powershell prompt, it should display the index that will be the index of result corresponding to this line of command in the `$Result` array.

* User should be given an option via Powershell profile to use different name for `$Result` and `$LastResult` variables. 

* The reason `$Result` is an LRU array and not an MRU array is to facilitate quick reuse of history if using multiple previous values without having to rewrite the indices. While it may seem confusing to frequent users of `$Error`, this would be the right way of doing things in the context of output reuse and a deviation from trend would be justified. 

* The array `$Result` should have a limited size . The proposer admits his inexperience with Powershell and does not have an opinion on how accesses to indices no longer in memory should perform. This may seem like a downside of using LRU array but the upsides of ease of reuse outweight the downsides by far. Not having to decide beforehand what values we want to only save, only display, both save and display is something that would make the use of the command-line shell quicker. 

* The size of `$Result` array should be modifiable on the fly in an interactive session for scenarios where a user would like to save reuse very old outputs. Although the recommended way of doing this would be to save such a variable in a user-created variable by assiging from the array.

* In case of statements like `$X=SomeCommand` `$LastResult` (and the corresponding array variable) should not be equal to the output of this line of command (which is an empty string) but should be same as the output of `SomeCommand`

#History

v1.0 - 2016-09-26 - First version that discusses motivation and proposes use of one running variable and one array to save Result.
