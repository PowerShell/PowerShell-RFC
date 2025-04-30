---
RFC: RFC<four digit unique incrementing number assigned by Committee, this shall be left blank by the author>
Author: Dongbo Wang
Status: Draft
SupercededBy:
Version: 1.0
Area: Core
Comments Due: 5/30/2025
Plan to implement: Yes
---

# Path update for package installation

On Windows, when a user installs a tool using package managers like `WinGet` and `Chocolatey` in PowerShell,
the tool is often added to the system or user PATH environment variable. However, changes to PATH made during
the installation are not automatically propagated to the running PowerShell session. As a result, users frequently
need to start a new PowerShell session to recognize and use the newly installed command-line tools.

This behavior can be confusing and disrupts the workflow, especially in automation scenarios or when users expect
tools to be immediately available post-installation. Addressing this limitation would improve usability, align with
user expectations, and reduce friction in both interactive and scripted environments.

## Motivation

As a PowerShell user, I would like tools installed by package managers like `WinGet` or `Chocolatey` to be useable
in my current session so that I can use the new tool without interrupting my existing work in the current session.

## Solution in CMD

CMD suffered with the same problem and the Windows Terminal team already [made a fix](https://microsoft.visualstudio.com/OS/_git/os.2020/pullrequest/10131240)
to mitigate the issue:

> With this change, CMD will detect the execution of `winget.exe` and `choco.exe`,
> as well as other package managers (the list is configurable in the registry given HKLM write permission).
> If we are launching a package manager, we will save the current user and system `PATH` and diff them
> after the package manager exits.
>
> To make this safe, there are a couple caveats:
>
> 1. We will only detect changes to the BEGINNING or END of either `PATH` (user/system).
> 2. We will only ever **append** to CMD's current `PATH`.
> 3. This feature can be disabled by removing the list of package managers at `HKLM\SOFTWARE\Microsoft\Command Processor\KnownPackageManagers`.
> 4. This feature can be disabled by setting `HKLM\SOFTWARE\Microsoft\Command Processor\InteractiveExtensionLevel` to 0.
> 5. This feature only activates if STDIN is a `Console`.
>
> This will only apply in the current shell and children.
> Batch files **run from an interactive shell** will detect package managers as well,
> and this is going to help things like `install_all_my_packages.bat`.

## Specification

### What should align with CMD

PowerShell should align with the CMD behavior as much as we can:
- `NativeCommandProcessor` should also detect the execution of package managers defined in the list in registry
- If we are launching a package manager, we will save the user and system `PATH`, find the diff after execution, and update the process `PATH` accordingly.

More specifically, we should retain the following CMD behaviors:

1. Use the same registry key for the list of package managers to detect: `HKLM\SOFTWARE\Microsoft\Command Processor\KnownPackageManagers`.
2. Only detect changes to the BEGINNING or END of either `PATH` (user and system).
3. Only ever append to PowerShell's current `PATH`.
4. Can be disabled by removing the list of package managers from registry.

### What are different

In CMD, the feature is enabled under the following conditions:

- CMD is running interactively
  - not running a single command or batch file via `cmd /c`
  - `stdin` is `Console`
- Registry key `HKLM\SOFTWARE\Microsoft\Command Processor\InteractiveExtensionLevel` is >=1

  ```c
  // code snippet for checking if it's considered interactive. 
  if (!SingleBatchInvocation && !SingleCommandInvocation) {
      fIsInteractiveForExtensionPurposes = FileIsConsole(STDIN) ? TRUE : FALSE;
  }

  // code snippet for checking if package manager detection is on
  if (fIsInteractiveForExtensionPurposes && dwInteractiveExtensionLevel >= 1) {
      // code for detecting package manager
  }
  ```

In PowerShell, the concept of interactive/non-interactive is different from CMD.

Non-interactive in PowerShell (`pwsh -noninteractive`) is more like a restriction to script -- the script should not expect to prompt user for input for things like `Read-Host`, `ShouldProcess`, `ShouldContinue`, mandatory parameter input, and etc.
An attempt to do so will result in an error and terminate the script, so that in an automation environment, a script won't hang waiting for input.

However, even in a non-interactive PowerShell session, user can still run commands interactively.

#### Discusson points

> **[NOTE]:** The draft spec has been discussed in an internal PowerShell design review meeting, and the "**[PS REVIEW COMMENT]**" tag indicates the review decision.

1. When PowerShell runs with `pwsh -c` or `pwsh -f` without the `-noexit` flag,
it runs a single command or file and then terminates, which is very similar to the first _"CMD running interactively"_ check above.
So, should we disable the feature in this case?

   - [dongbow] IMHO, the feature is useful to script execution too.
     For example, a bootstrap script installs a tool and call it right after the installation is done.
     Today, the script will have to hardcode the tool's location or start another PowerShell process to continue the rest work.

   - **_[Followed up with WT]_** Why do they disable the feature for `cmd /c`.
     - WT response:
       > - `/c` doesn't result in the sesison sticking around, so it didn't seem necessary.
       > - we did not consider whether users would want to install something and immediately use it within one sub-invocation of cmd with a parameter pack containing commands.
       > - '/c' feels more like "one-line batch file" than "user sitting at the session", so we don't consider it an interactive scenario.
     - To summarize:
       - WT team didn't consider the need to install something and use it immediately with `cmd /c`.
       - WT team support this feature only in interactive session, which `cmd /c` is not.

   - **[PS REVIEW COMMENT]** We prefer to having this feature enabled consistently in PS.

2. Unlike CMD, PowerShell can be hosted in other applications, such as a GUI application which doesn't have a `stdin`.
Will the feature still be useful in that case (`stdin` is not `Console`)?

   - **_[Followed up with WT]_** Why do they disable it when `stdin` is not `Console`
     - WT response:
       > - unlike PowerShell, CMD isn't designed to be driven so easily from another application.
       > - we chose to err on the side of caution with this design and used "stdin a console" to detect whether the session was truly single-user-direct-to-cmd-interactive.

   - **[PS REVIEW COMMENT]** We prefer to having this feature enabled consistently in PS.

3. The registry key `InteractiveExtensionLevel` is used in CMD for more than the package manager detection feature.
It's also used to guard a `"WingetCmdNotFound"` feature. Should we also disable the feature based on this key?

   - Would it be reasonable to disable the feature in PowerShell but not in CMD?
   - [dongbow] IMHO, it should be OK for us to honor this key too.
     It seems not reasonable to have a separate way to disable this feature in PowerShell while keeping the feature on in CMD.

   ```c
   if (DosErr == MSG_DIR_BAD_COMMAND_OR_FILE) {
       if (Feature_WingetCmdNotFound::IsEnabled()) {
           if (fIsInteractiveForExtensionPurposes && dwInteractiveExtensionLevel >= 1) {
                // handle winget not found error specially
           }
       }
   } else { ... }
   ```

   - **_[Followed up with WT]_** on `InteractiveExtensionLevel`
     - WT response:
       > - it was our "get out of jail" card in case we legitimately broke something someone cared about.
           If you have a risk of breakage due to this new behavior, users may want a way to turn it off.
       > - we haven't heard that we have broken anyone. But the new changes don't have so much market adoption yet.
       > - feel free to follow it. However, other things in PowerShell DON'T follow it today
     - To summarize:
       - It's for turning off all interactive improvements they made to CMD, not exclusive to the PATH refreshing feature
       - PS can choose to use it, but it's not used by PowerShell for any other things.

   - **[PS REVIEW COMMENT]** We will use the same registry key for the list of package manager but **Will NOT** hornor this key: `InteractiveExtensionLevel`,
     because that key is not exclusively for the package manager detection feature.

   - **[PS REVIEW COMMENT]** _"Do we need to have a PowerShell way to disable the feautre?"_ -- maybe NOT, just depend on clearing the list.
     - WT response:
       > clearing the list will change it for cmd as well, of course. A windows upgrade might restore part of the list (the three entries we know about)
       > - we migrate `InteractiveExtensionLevel` and the package manager info from build to build
       > - however, I don't know if migration manifests account for deletions of keys. that's where it gets weird for me, and that's where I don't actually know
     - [dongbow && jason]'s thoughts on this comment
       - It's unlikely that a user wants to use different settings for that list between PowerShell and CMD.
       - Windows upgrade shouldn't mess up with the changes a user made to those registry keys. It sounds like a bug if it does.
       - Even if deleted key will be restored with the default value by windows upgrade, clearing the list but keeping the key around should be fine with windows upgrade.

4. What about `Start-Process`, should we do the detection within that cmdlet too?

   - **[PS REVIEW COMMENT]** No, the feature is only for the native command processor.

   - **_[WT comments]_** CMD only covers EXE, but `scoop` is a `.ps1` or `.bat` file, so CMD doesn't see it. PowerShell may want to broaden your support to account for any command.
     - [dongbow && jason]'s thoughts on this comment
       - [Scoop](https://scoop.sh/) creates shims inside the `~\scoop\shims` folder for terminal applications, which is accessible in the PATH. So `Scoop` doesn't need PATH refreshing.
       - We should scope the work to native command only. Supporting non-native commands would need broader changes outside of the native command processor without a clear gain.

5. The fix to CMD checks command name only, so it will try refreshing `PATH` even if `winget` or `choco` are invoked for purpose other than installing a new package, e.g. `winget search`.
Should we optimize it in PowerShell to only refresh `PATH` when `winget install` or `choco install` actually runs?

   - **[PS REVIEW COMMENT]** No, make it simple.

6. Refreshing PATH could cause a problem in command resolution -- a command that could be found in an unloaded module may be resolved to a native command due to the appending of a new path to `PATH`. How much should we care about this situation? (it's an existing issue anyway)

   - [dongbow] IMHO, this should not be a concern for script execution/automation, as it should use module-name-qualified command name.
   I also don't think it's a real problem to interactive usage -- if the user run `foo` after `winget install foo`, they would expect the installed tool to be invoked instead of a function named `foo` from an unloaded module.

   - **[PS REVIEW COMMENT]** Not concerned about this except for the WDAC and AppLocker scenario (see below).

### WDAC and AppLocker scenario

- If `Env` provider is disabled or read-only, the feature should be disabled.
