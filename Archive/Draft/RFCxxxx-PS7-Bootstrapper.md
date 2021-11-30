---
RFC: RFCnnnn
Author: Steve Lee
Status: Draft
SupercededBy: n/a
Version: 0.1
Area: Install
Comments Due: Dec 31st, 2021
Plan to implement: Yes
---

# Bootstrapper for PS7 in Windows

## Motivation

    As an admin,
    I can use PS7 to provision a new Windows machine,
    so that I can leverage new capabilities in PS7.

    As a user,
    I can use PS7 on a new Windows machine,
    so that I can leverage new capabilities in PS7.

Explicitly installing PS7 is a blocker for some customers not only due to discoverability, but also some customers have policies that
don't allow for installing software onto Windows after initial provisioning.

## Shipping in Windows

There are several issues that prevent simply shipping PS7 in Windows:

- Size
- Support lifecycle differences

### Size constraints

A complete installation of PowerShell 7 currently is around 200 MB (unpacked).
This is due to including everything (modules and .NET assemblies) to provide parity with what Windows PowerShell 5.1 with .NET Framework provides.
There is currently no plan by Windows or the .NET team to ship newer .NET in Windows, so PS7 will need to continue to ship .NET with PS7.

### Support lifecycle

Windows has a 5+5 year support lifecycle.
.NET supports up to 3 years for Long Term Support (LTS) versions.
So even if size was not an issue, we cannot check in a version of PS7 with a .NET that is not supported while Windows itself is supported.

### Solution

To solve both of these issues, the proposal is to ship a bootstrapper in Windows that would be supported within the Windows support lifecycle.
The boostrapper will update PS7 as needed to ensure a supported version of .NET is being used.
Details of this bootstrapper follow.

## Specification

### pwsh.exe bootstrapper

We would check-in a `pwsh.exe` into Windows 11+ (under `System32`) written in [native code](#native-code) that will:

- Detect if PS7 is already installed
  - If installed, pass-through all arguments to the new detached `pwsh` child process
    - This means that if PS7 is already in the `PATH`, then that version (whether installed via MSI, MSIX, or other means)
      needs to be updated outside of the bootstrapper
  - Otherwise, [install](#installer) PS7 and then pass-through all arguments to the new detached `pwsh` child process

The boostrapper should explicitly create a detached child `pwsh` process and then exit.
As a detached child process, it will continue execution even after the parent has exited.

Expectation is that the bootstrapper will always be first in the `$env:PATH` and executed first, so it can also serve
to [auto-update](#auto-updating) PS7.

This will allow `pwsh -c customer_script.ps1` to work seamlessly as though PS7 was available inbox.

The bootstrapper would need to start the real `pwsh.exe` and bind stdin/stdout so that inputs and outputs come from `pwsh.exe` seamlessly.
Unlike Unix, there isn't a way to `exec` a new process that replaces itself so for the first execution, there would be two `pwsh.exe` processes.

However, if the bootstrapper detects that stdin, stdout, or stderr is redirected, then it would need to not exit and instead
continue to redirect until the child `pwsh.exe` process has exited.

Additional Windows requirements:

- registered with Windows so that `Start->Run->pwsh` will start the bootstrapper
- a separate Windows feature that can be uninstalled (so that using `pwsh` would not automatically work, or is being installed by other means)
- different icon for the bootstrapper to indicate it is a bootstrapper

### .ps1 registration

`pwsh.exe` will be registered to execute .ps1 scripts, by default.
However, it will have code to detect if started from the `explorer.exe` and instead
give an error message that scripts need to be run from the console and not from Explorer.
This means that from cmd.exe, you can execute .ps1 scripts without explicitly calling `pwsh.exe` and that applications
can start depending on .ps1 scripts instead of wrapping them in batch files.
The default execution policy on both Client and Server SKUs should be RemoteSigned.

### Native code

We can't rely on .NET Framework being available on some Windows skus we want to support and keeping the bootstrapper as
small as possible, we don't want to rely on .NET (Core) and ship that runtime.
Also, we want the bootstrapper to have minimal dependencies as future Windows SKUs may not ship with certain dependencies.

#### Rust

Rust has the benefit over C/C++ of having well defined contracts for resource management thus not having issues that persist in C/C++ code
with regards to resource management which results in not only leaks but security issues.
Resulting Rust binary code is very similar in size and performance as C/C++.
Rust built code is supported in Windows.

Rust uses the MS C runtime at execution, but we should statically link so that there aren't external dependencies that may be missing on the system.

Need to evaluate what APIs are available on the smallest Windows SKU for downloading.

This is the current preferred option.

### Installer

MSI has dependencies that are not available on Windows SKUs so would not be appropriate as the installer for the bootstrapper.
Instead, we should look at using [Squirrel](https://github.com/Squirrel/Squirrel.Windows) which supports installing even if the product (existing version)
is already in use.

The bootstrapper would download the installer using Windows Update as the store to avoid issues with enterprises not allowing
network access to other sites.
The installer and binaries may need to be Windows signed instead of Microsoft signed.
Packages of PS7 outside of this installer used by the boostrapper would continue to be Microsoft signed.

### User Experience

Since the initial bootstrapping will take time to download and install, the bootstrapper should have progress information and
indicator during download and installation via stderr.
Stderr is used instead of stdout so that user redirection of stdout only contains output from successful execution of their commandline.
Use of stderr for informational messages is a convention used by native commands already.

Should the install or download fail, the bootstrapper needs to output appropriate information and direct user to documentation on alternate
install method and return a non-zero exit code.

### Auto-updating

Upon startup of the bootstrapper (following rules of [admin vs non-admin](#admin-vs-non-admins)) will detect if the installed
version is installed by the bootstrapper and if so, if it is the latest stable version.

If the version is older, then it will asynchronously download updated PS7 and continue with execution.
A message will be emitted that on next startup PS7 will be updated.
Upon next startup, if a newer version is downloaded, then the signature will be validated and installed providing
progress information to the user.

In general, auto-updates will be to a servicing release like 7.x.y to 7.x.y+1.
However, when 7.x is out of support, then it should auto-update to 7.x+1.
This is consistent with how our Microsoft Update support works to keep users patched and on a supported release.

The auto-update applies regardless if PS7 is started interactively or via automation (explicitly using the `-noninteractive`, `-file`, or `-command` switch).

### Configuration

Some registry configuration (settable via Group Policy) would be needed to disable auto-updating.
However, auto-updating will be the default to ensure users have the latest version with security updates.
Configuration would allow for auto-updating, update via prompt, or don't update.

### Admin vs. Non-Admins

If the process is elevated, then the install will be for the system under `$env:ProgramFiles`.

However, the bootstrapper must work for non-admins.
Otherwise, PS7 is installed only for the current user if the process is non-elevated.
In this case, PS7 would be installed to `$env:LOCALAPPDATA\Microsoft\PowerShell7` and the user `$env:PATH` would be appended with this path.
`PowerShell7` as a folder is used instead of just `PowerShell` as that folder is already being used by PS7 for persisting state.

If PS7 was installed by the bootstrapper elevated so that it's available system wide (under `$env:PROGRAMFILES`), and the user
is non-admin, but the bootstrapper detects a newer version is available, then it should emit a message indicating that a newer
version is available, but requires elevation or an administrator to install.

>[Note] Need to verify that Squirrel supports admin installs for system-wide access

### Windows Terminal

Windows Terminal is now a default installed app and has code to detect PS7.
Currently, it will not detect the prescence of `pwsh.exe` bootstrapper if it exists in system32.
We may submit a PR to update their detection logic so that the bootstrapper is available as the default shell.

### Offline scenarios

Need to look into integration with [Windows ADK (Assessment and Deployment Kit)](https://docs.microsoft.com/en-us/windows-hardware/get-started/)
to support pre-provisioning Windows images that already include PS7 rather than bootstrapping at runtime where the machine may be on a factory
floor (WinPE) without internet access.

### Remoting endpoint

By default, neither WinRM nor SSHD will be setup for PS7 remoting.
So when remoting from one Windows machine into another, it would not connect to PS7 unless the user has manually configured it
(and thus also installed PS7).

We may consider adding capability to make it easy to setup PS7 remoting for large deployments in the future.
For example, maybe `pwsh.exe --enable-ssh-endpoint` and `pwsh.exe --enable-winrm-endpoint` as special switches only for the bootstrapper.

## Alternate Proposals and Considerations

### Reducing size

The bootstrapper itself is small, but installation would still carry a full PS7 package.
Details of reducing the size of PS7 post-install is outside the scope of this specific RFC.

### Auto-update when used non-interactively

Current design is based on auto-update (unless configuration is set not to) even if the usage is via automation.
For example, remotely or using task scheduler to execute a script.
A design decision could be made to not auto-update if we know the usage is not interactive,
but current design is to keep PS7 updated even in those cases unless configured otherwise.

### Feature-on-Demand

A Windows Feature-on-Demand (FOD) is similar to what is being proposed in that you would install the feature from the internet.
However, a FOD needs to be explicitly installed to be used and is also locked to a specific version built for that version of Windows.
In this case, it's critical to allow use of `pwsh.exe` right out of the box and have it execute a customer script and also
because PS7 aligns with .NET lifecycle, install the latest stable version.

### Windows Store MSIX pkg

Some applications have a shortcut that is shipped inbox and when started installs from the Windows Store.
This presents two problems:

- This works fine for interactive use, but for automation, they now have to bootstrap PS7 before they can use PS7 to bootstrap other configuration
- Windows Store is not supported on Windows Server which is required

### `Install-PS7` cmdlet in Windows PowerShell

Simplest solution may be to have a script cmdlet in Windows PowerShell that simply installs PS7.
This is an additional step required to provision PS7.
Some skus of Windows do not include Windows PowerShell so they would not be able to use this.

### PS7 zip pkg support

Instead of supporting bootstrapping using the zip package, in cases where the Windows SKU doesn't support MSI/MU, we may only
support offline installation at image creation time.

### [.NET NativeAOT](https://github.com/dotnet/runtimelab/tree/feature/NativeAOT)

We should look into this as an option, but this is still experimental and we may not be supported for production use.

### `pwshup.exe` instead of bootstrapper

There has been a community suggestion to have a [`pwshup`](https://github.com/PowerShell/PowerShell/issues/14217) for explicit but easy
installation of PS7.

This could be shipped in Windows instead of a bootstrapper (although the two are not mutually exclusive).
However, `pwshup` is more dev-centric and requires the user to initially use it to install PS7.
This means that an enterprise cannot simply start executing PS7 scripts and would need to initially use `pwshup` to bootstrap
the system to install PS7.

`pwshup` would also be useful in side-loading scenarios where an application has dependency on a specific version
of PS7 and doesn't want to ship and patch it themselves.

Also, for non-developers/ITPros that simply want to use the latest PowerShell as a shell, they would need to know
to run `pwshup` first instead of just running `pwsh` (or clicking on the pre-installed icon) and then simply getting an interactive shell.

### Out-of-scope features

- choice of installing preview, stable, or LTS releases of PS7
- ability to specify a specific version of PS7 or specify as argument like `pwsh -version 7.1`
- ability to use `#requires` to specify execution using WinPS vs PS7
- Nanoserver is out of scope
