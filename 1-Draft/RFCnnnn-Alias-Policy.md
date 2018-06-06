---
RFC: nnnn
Author: Ilya Sazonov
Status: Draft
Area:
Comments Due: 6/30/2018
---

# The document purpose

The purpose of this document is to establish a new policy for the creation and use of aliases to ensure:

* Reliable, predictable behavior of scripts and modules
* Increase readability and maintainability of scripts and modules
* UX enhancements in interactive sessions

# Problem description

1. Windows PowerShell has many aliases that conflict with the names of standard Unix utilities.

    These aliases was removed from PowerShell Core.
    This causes a backward compatibility issue when Windows users do not find the familiar aliases in the interactive session.
    It also disrupts the operation of custom scripts that use these aliases.

1. Now there are dozens of modules developed by MSFT and thousands of modules of third developers. This causes a name conflict.

    Developers try to create unique cmdlet names.
    They can use module prefixes to resolve name conflicts.

    On the other hand, developers of the best intentions can complement their modules cmdlet aliases.
    At the same time, users can create their own aliases.

    With a large number of existing modules and cmdlets and a wide variety of custom aliases, these short names inevitably conflict with each other.

    Moreover, the behavior becomes unpredictable because each subsequent download module or script can change any alias.
    Either the user can destroy a script behavior by creating a new alias or by deleting an existing one.

    It is even more unpredictable and dangerous to create scripts in the assumption that there are some predefined (standard) aliases because they can be changed by the user or script at any time.

    Modern versions of ScriptAnalyzer already recommend replacing aliases in scripts with their full names.

    >The definition of aliases in modules or scripts and their automatic import becomes an undesirable and dangerous practice.

1. UX in interactive sessions.

    Having many cmdlets and parameters with long names can make it difficult to enter them in interactive sessions.
    Users who typically use standard settings are typically satisfied with the ```IntellySense``` capabilities.
    On the other hand, some users actively using interactive sessions may still be annoyed by the need to type names.
    These users should be able to upload their familiar and useful aliases sets to their interactive sessions.
    Given the limited functionality of aliases, these users will also configure their environment by executing commands by importing their own functions and various modules using PowerShell profiles.
    It should be noted that we are planning to add a domain-specific language (DSL). This will provide users with much greater customization capabilities than aliases.

# PowerShell alias policy

1. PowerShell implements standard aliases in all sessions including interactive:

    * % -> ForEach-Object
    * foreach -> ForEach-Object
    * ? -> Where-Object
    * where -> Where-Object

    These aliases are actively used in interactive sessions and scripts.
    ScriptAnalyzer will continue to recommend not to use them in scripts and modules.

1. By default, PowerShell does not load aliases from modules and InitialSessionState.

    To ensure module backward compatibility:
    * ```Import-Module -Aliases``` - explicitly loads aliases from the module.
    * Global configuration option ```GlobalAliasImport``` in ```powershell.config``` turns on loading aliases from modules (and InitialSessionState). It is turned off by default.

1. Backward compatibility of standard aliases.

    To ensure backward compatibility, three sets of aliases are created:
    * ```Default``` - current alias set (from ```InitialSessionState```) in PowerShell Core. This is the minimum set of aliases that have no conflicts on UNIX systems.
    * ```Extended``` - the ```Default``` set of aliases and additional popular aliases for Windows users (which can have conflicts on Unix).
    * ```WindowsPowerShell``` - the full set of standard aliases from Windows PowerShell. This set provides full backward compatibility for Windows users.

    No alias set are loaded by default.

    Global configuration option ```GlobalAliasSet``` in ```powershell.config``` turns on loading the desired set. It is turned off by default.

1. PowerShell prevents the use of aliases in scripts and modules.

    Use of aliases is allowed only in interactive sessions.
    Global configuration option ```GlobalAliasEnable``` in ```powershell.config``` allows the use of aliases in scripts and modules. It is turned off by default.

1. Interactive session customization are delegated to users.

    It is recommended to use and develop ```IntellySense``` and other means of assistance to users including ML.
    Users can use aliases in interactive sessions but in the future it is better to use DSL.

# UX Improvements

To easily load aliases into interactive sessions, the ```Import-Alias``` cmdlet is extended by the ```GlobalAliasSet``` parameter:

* ```Import-Alias -GlobalAliasSet Default```
* ```Import-Alias -GlobalAliasSet Extended```
* ```Import-Alias -GlobalAliasSet WindowsPowerShell```

Values of the parameter is defined above.

Users can add this cmdlet to their profile to automatically import the necessary aliases.

# Transition

All elements of the current alias policy can be implemented independently and are in a state that does not change the current behavior.

Each element of the current alias policy is enabled by a decision of the PowerShell Committee.
