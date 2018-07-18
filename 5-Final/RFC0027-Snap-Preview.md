---
RFC: RFC
Author: Travis Plunk (@TravisEz13)
Status: Final
Version: 0.1
Area: Packaging
Comments Due: July 15, 2018
Plan to implement: Yes
---

# Snaps of Preview builds

We currently have not published any [Snap packages](https://snapcraft.io) of a preview build.  There are some significant questions before we do.
Primarily, do we follow Snap's model of channels or do we have a separate package called `powershell-preview`.
I propose we have a separate package as it will allow side-by-side installation with the stable Snap and will be consistent with our Linux package names.

## Motivation

Snaps are easy to use and having a Snap of the preview would allow people to adopt the preview across a wide variety of platforms.
We want to design the preview Snap in a way that allows for the preview Snap to be installed easily without losing access to the stable version of PowerShell.

## Specification

* There shall be two Snap packages available:
  * Stable: a production-ready, supported build, called `powershell`.
  * Preview: an unsupported build that may contain incomplete new features and high-impact bugs,
    and should therefore not be used in production, called `powershell-preview`.   Our hope is that the development process ensures that every build is stable and high quality.
* Aliases
  * Stable package: `pwsh`
  * Preview package: `pwsh-preview`
* Snap channels
  * Stable package: Will always use the `stable` channel, because it is stable.
  * Preview package: Will always use the `stable` channel, because the package name and version indicates the quality.
    * Alternative: Preview builds could use the `edge` channel, and release candidates could use the `candidate` channel.
      This would mean you would always have to use the one of the following commands to install `sudo snap install powershell-preview --edge --classic` or `sudo snap install powershell-preview --candidate --classic`

## Alternate Considerations and Proposals

### Use Snap's model of channels

The main dis-advantage here is that preview builds cannot be install side-by-side with a stable build (at least with Snap alone).  Snap does allow you to switch between things you have downloaded fairly quickly so this is not fatal.

* There shall be one Snap package available:
  * Called `powershell`.
* Alias
  * `pwsh`
* Snap channels
  * Stable builds would be published to the stable channel and installed using the following command:
    `sudo snap install powershell --classic`
  * Release Candidate builds would be published to the stable channel and installed using the following command:
    `sudo snap install powershell --classic --candidate`
  * Preview builds would be published to the stable channel and installed using the following command:
    `sudo snap install powershell --classic --edge`

## Open Questions

---------------
## PowerShell Committee Decision

### Voting Results

Joey Aiello: Accept

Bruce Payette: Accept

Steve Lee: Absent

Hemant Mahawar: Accept

Jim Truher: Accept

Dongbo Wang: Accept

Kenneth Hansen: Absent

### Majority Decision

We support side-by-side and want to keep it that way, hence the distinct packages.
The alias names also make sense given the pattern we set with the existing Linux packages.

### Minority Decision

N/A
