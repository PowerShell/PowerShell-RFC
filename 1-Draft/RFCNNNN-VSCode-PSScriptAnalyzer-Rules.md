---
RFC: RFC<four digit unique incrementing number assigned by Committee, this shall be left blank by the author>
Author: Tyler Leonhardt
Status: Draft
SupercededBy: N/A
Version: 0.1
Area: Tooling - VSCode extension & PSScriptAnalyzer
Comments Due: December 15th
Plan to implement: Yes
---

# New default rules in the VSCode Extension and a new "Aggressive Analysis mode" rules

## Context
The VSCode extension provides integration with PSScriptAnalyzer (referred to as PSSA going forward) to expose rule violations in a friendly way.
These rule violations show up as green or red squiggles under the associated code that violates the rules.
Here's an example:

![example hover dialog](https://i.imgur.com/ulR5Bhv.png)

Today,
the VSCode extension only enables a subset of the default PSSA rules.
This was done because some of the PSSA default rules can be opinionated and we don't want to intrude on user's that don't want too much chatter.
If a user wanted to opt-in for more rules,
they have a few options:

- Using the
"PowerShell: Select PSScriptAnalyzer rules"
command pallet action
- Create a
`PSScriptAnalyzerSettings.psd1`
file with the rules you want and update your
`powershell.scriptAnalysis.settingsPath`
VSCode setting to point to that

## Problem

Using these mechanisms require understanding of what PSSA is,
what rules are,
how to configure PSSA,
etc.

It's a lot of information to go digging for.

It should be easier to opt-in to more rules without needing to go through PSSA docs.

## Solution - "Aggressive Analysis mode"

I'm proposing an 
"Aggressive Analysis mode"
set of rules that you can flip on with one setting in VSCode. 
In addition,
adding a few other rules to the default rules.

`powershell.scriptAnalysis.mode`

This can be set to:

- None - disables script analysis
- Default - some crutial rules but nothing too crazy
- Aggressive - more rules, more fun

Here is the table of:

- the current Defaults
- the ones that will be a part of the "Aggressive analysis mode"
- the ones we are considering to add to the Defaults (marked with an x)

| Rule                                         | PSSA Type   | Mode       | Add to default |
|----------------------------------------------|-------------|------------|----------------|
| AlignAssignmentStatement                     | Warning     |            |                |
| AvoidAssignmentToAutomaticVariable           | Warning     | Aggressive |                |
| AvoidDefaultValueForMandatoryParameter       | Warning     | Default    |                |
| AvoidDefaultValueSwitchParameter             | Warning     | Default    |                |
| AvoidGlobalAliases*                          | Warning     | Aggressive | x              |
| AvoidGlobalFunctions                         | Warning     | Aggressive | x              |
| AvoidGlobalVars                              | Warning     | Aggressive | x              |
| AvoidInvokingEmptyMembers                    | Warning     | Aggressive | x              |
| AvoidNullOrEmptyHelpMessageAttribute         | Warning     |            |                |
| AvoidShouldContinueWithoutForce              | Warning     | Aggressive |                |
| AvoidUsingCmdletAliases                      | Warning     | Default    |                |
| AvoidUsingComputerNameHardcoded              | Error       | Default    |                |
| AvoidUsingConvertToSecureStringWithPlainText | Error       | Default    |                |
| AvoidUsingDeprecatedManifestFields           | Warning     | Aggressive | x              |
| AvoidUsingEmptyCatchBlock                    | Warning     |            |                |
| AvoidUsingInvokeExpression                   | Warning     |            |                |
| AvoidUsingPlainTextForPassword               | Warning     | Default    |                |
| AvoidUsingPositionalParameters               | Warning     | Aggressive |                |
| AvoidTrailingWhitespace                      | Warning     |            |                |
| AvoidUsingUsernameAndPasswordParams          | Error       | Aggressive |                |
| AvoidUsingWMICmdlet                          | Warning     | Aggressive | x              |
| AvoidUsingWriteHost                          | Warning     | Aggressive |                |
| DSCDscExamplesPresent                        | Information |            |                |
| DSCDscTestsPresent                           | Information |            |                |
| DSCReturnCorrectTypesForDSCFunctions         | Information |            |                |
| DSCStandardDSCFunctionsInResource            | Error       |            |                |
| DSCUseIdenticalMandatoryParametersForDSC     | Error       |            |                |
| DSCUseIdenticalParametersForDSC              | Error       |            |                |
| DSCUseVerboseMessageInDSCResource            | Error       |            |                |
| MisleadingBacktick                           | Warning     | Default    |                |
| MissingModuleManifestField                   | Warning     | Default    |                |
| PossibleIncorrectComparisonWithNull          | Warning     | Default    |                |
| PossibleIncorrectUsageOfAssignmentOperator   | Warning     | Aggressive |                |
| PossibleIncorrectUsageOfRedirectionOperator  | Warning     | Default    |                |
| ProvideCommentHelp                           | Information |            |                |
| ReservedCmdletChar                           | Error       | Default    |                |
| ReservedParams                               | Error       | Default    |                |
| ShouldProcess                                | Error       | Default    |                |
| UseApprovedVerbs                             | Warning     | Default    |                |
| UseBOMForUnicodeEncodedFile                  | Warning     |            |                |
| UseCmdletCorrectly                           | Warning     | Aggressive |                |
| UseDeclaredVarsMoreThanAssignments           | Warning     | Default    |                |
| UseLiteralInitializerForHashtable            | Warning     | Aggressive |                |
| UseOutputTypeCorrectly                       | Information | Aggressive |                |
| UsePSCredentialType                          | Warning     | Aggressive |                |
| UseShouldProcessForStateChangingFunctions    | Warning     | Aggressive |                |
| UseSingularNouns                             | Warning     | Aggressive | x              |
| UseSupportsShouldProcess                     | Warning     | Aggressive | x              |
| UseToExportFieldsInManifest                  | Warning     | Default    |                |
| UseCompatibleCmdlets                         | Warning     |            |                |
| PlaceOpenBrace                               | Warning     |            |                |
| PlaceCloseBrace                              | Warning     |            |                |
| UseConsistentIndentation                     | Warning     |            |                |
| UseConsistentWhitespace                      | Warning     |            |                |
| UseUTF8EncodingForHelpFile                   | Warning     | Aggressive |                |

I'd love feedback on which rules go into the `Default` and `Aggressive` modes.

Keep in mind that this is a very opinionated topic.

## Motivation

    As a powershell script writer using VSCode,
    I can easily opt in to a more aggressive analysis of my scripts,
    so that I can learn and write better scripts.

## Specification

Adding a new setting to the VSCode extension:

`powershell.scriptAnalysis.mode`

Adding a [new array to PowerShell Editor Services](https://github.com/PowerShell/PowerShellEditorServices/blob/8e29349c9c1f1edb60f878de1c37f8b903397273/src/PowerShellEditorServices/Analysis/AnalysisService.cs#L33-L48) to add the proposed
"Aggressive analysis".

Down the road we can add a "first-run" experience that will show a dropdown or switch to select the analysis mode but for now,
this will be a one click configuration in the VSCode settings.

## Alternate Proposals and Considerations

N/A
