---
RFC: 0052
Author: Dongbo Wang
Status: Final
SupercededBy: N/A
Version: 1.0
Area: Console
Comments Due: 4/30/2019
Plan to implement: Yes
---

# Notification on PowerShell version updates

Today, to find out whether a new version of PowerShell is available,
one has to check the release page of the `PowerShell\PowerShell` repository,
or depend on communication channels like `twitter` or `GitHub Notifications`.
It would be convenient if `pwsh` itself can notify the user of a new update on startup.

## Motivation

    As a PowerShell user, I get notified when a new version of `pwsh` becomes available.

## Specification

This feature is in the PowerShell console host, not in the PowerShell engine.

### Target Goals

1. No notification or update check when the running `pwsh` is a self-built version.
No notification or update check for non-interactive sessions.
Also, no notification when the PowerShell banner message is suppressed.

2. When there is a new update, assuming you use `pwsh` every day
and at least one interactive `pwsh` session lasts long enough for the update check,
then you should be able to see an update notification during the `pwsh` startup on the same day of a release or the next day at the latest.

3. This feature must have very minimal impact on the startup time of `pwsh`.
This means the check for update must not happen during `pwsh` startup.
The only acceptable extra overhead to the `pwsh` startup should just be the work related to printing the notification.

4. Check for updates should not blindly run for every interactive `pwsh` session.
For a particular version of `pwsh`, only one check at most can run to complete per day
no matter how many interactive session of the `pwsh` are started/opened in that day.

5. After a new update is detected during a successful check,
all subsequent interactive sessions of that version of `pwsh` should show the notification at startup time.
And subsequent checks can be avoided for a reasonable period of time, such as a week.

6. `pwsh` of preview versions should check for the new preview version as well as the new GA version.
`pwsh` of GA versions should check for the new GA version only.

7. The notification and update check are not needed in some scenarios,
such as when `pwsh` is in a container image.
Hence, you should be able to suppress them altogether by setting an environment variable.

### Non-Goals

1. Notification shows up right after a new version of `pwsh` is released.

   _This is not a goal._
   Assuming you use `pwsh` interactively every day,
   then a notification about the new release may show up on the same day,
   but is guaranteed no later than the next day.

2. If an update check detects a new release, the notification should show up in the same session.

   _This is not a goal._
   An update check should happen way after the startup of an interactive session,
   and thus it has no impact on whether or not a notification will be shown at the startup of that session.
   If new release is detected,
   the subsequent interactive sessions will show a notification about that new release.

3. If a new release is available, `pwsh` is able to automatically upgrade.

   _This is not a goal._
   A notification message is printed, but `pwsh` will not auto-upgrade.

### Implementation

This section talks about

- how to control the update notification behavior
- when to do the update check
- how to persist the detected new release for subsequent `pwsh` sessions to use
- how to synchronize update checks from different processes of the same version `pwsh` so that at most only one can run to complete during a day
- how to do the update check
- how to display the notification

#### How to control the update notification behavior

The environment variable `POWERSHELL_UPDATECHECK` will be introduced to control the behavior of the update notification feature.
The environment variable supports 3 values:

- `Off`. This turns off the update notification feature.

- `Default`. This gives you the default behaviors:
  - `pwsh` of preview versions check for the new preview version as well as the new GA version.
  - `pwsh` of GA versions check for the new GA version only.

- `LTS`. `pwsh` of both preview and stable versions check for the new LTS GA version only.

The notification behavior is mapped to the following enum:

```c#
private enum NotificationType
{
    /// <summary>
    /// Turn off the udpate notification.
    /// </summary>
    Off = 0,

    /// <summary>
    /// Give you the default behaviors:
    ///  - the preview version 'pwsh' checks for the new preview version and the new GA version.
    ///  - the GA version 'pwsh' checks for the new GA version only.
    /// </summary>
    Default = 1,

    /// <summary>
    /// Both preview and GA version 'pwsh' checks for the new LTS version only.
    /// </summary>
    LTS = 2
}
```

#### When to do the update check

During the startup, `pwsh` creates a `Task` of the update check work,
but delays the task run for 3 seconds by using `Task.Delay(3000)`.
The typical startup time for `pwsh` with a moderate size profile should be less than 1 second.
Given that, I think it's reasonable to delay the update check work for 3 seconds,
so that it has close-to-zero impact on the startup performance.

#### How to persist information about a new version

The version of new release is persisted using a file,
not as the file content, but instead baked in the file name in the following template,
so that we can avoid extra file loading at the startup.

```none
update<notification-type>_<version>_<publish-date>
```

A separate file is used for each supported notification type,
indicated by the integer value of the corresponding `NotificationType` member.
For example,
- when the notification type is `NotificationType.Default`,
the file name would be like `update1_<version>_<publish-date>`.
- when the notification type is `NotificationType.LTS`,
the file name would be like `update2_<version>_<publish-date>`.

The file should be in a folder that is unique to the specific version of `pwsh`.
For example, for the `v6.2.0 pwsh`, the folder `6.2.0` will be created in the `pwsh` cache folder (shown below),
and the update check related files for that version of `pwsh` are put there exclusively.
In this way, the update information for different versions of `pwsh` doesn't interfere with each other.

- Windows: `$env:LOCALAPPDATA\Microsoft\PowerShell\6.2.0`
- Unix: `$env:HOME/.cache/powershell/6.2.0`

#### How to synchronize update checks

The most challenging part is to properly synchronize the update checks started from different `pwsh` processes,
**so that for a specific version of `pwsh` and a specific notification type,
only one update check task, at most, will run to complete per a day.**
Other tasks should be able to detect "a check is in progress" or "the check has been done for today" and bail out early,
to avoid any unnecessary network IO or CPU cycles.

For each notification type, we need two more files to achieve the synchronization,
`"_sentinel<notification-type>_"` and `"sentinel<notification-type>-{year}-{month}-{day}.done"`.
The `<notification-type>` part will be the integer value of the corresponding `NotificationType` member.
The `{year}-{month}-{day}` part will be filled with the date of current day when the update check task starts to run,
and they will be in the version folder too.

The file `"_sentinel<notification-type>_"` serves as a file lock among `pwsh` processes.
The file `"sentinel<notification-type>-{year}-{month}-{day}.done"` serves as a flag that indicates a successful update check as been done for the day.
Here are the sample code for doing this synchronization:

```c#
const string TestDir = @"C:\arena\tmp\updatetest";
const string SentinelFileName = "_sentinel1_";
const string DoneFileNameTemplate = "sentinel1-{0}-{1}-{2}.done";

static void CheckForUpdate()
{
    // Some pre-validation needs to happen to see if we need to do anything at all.
    // - If the current running `pwsh` is a self-built version, let's bail out early.
    // - Check if a file like `_update_<version>_<publish-date>` already exists.
    //   If so, check the `publish-date` to see if it's still relatively new, say within a week.
    //   If so, let's bail out early.

    DateTime today = DateTime.UtcNow;
    string todayDoneFileName = string.Format(
        CultureInfo.InvariantCulture,
        DoneFileNameTemplate,
        today.Year.ToString(),
        today.Month.ToString(),
        today.Day.ToString());

    string sentinelFilePath = Path.Combine(TestDir, SentinelFileName);
    string todayDoneFilePath = Path.Combine(TestDir, todayDoneFileName);

    if (File.Exists(todayDoneFilePath))
    {
        // A successful update check has been done today,
        // so we can bail out early.
        return;
    }

    try
    {
        // Use 'sentinelFilePath' as the file lock.
        // The update check tasks started by every 'pwsh' process will compete on holding this file.
        using (FileStream s = new FileStream(sentinelFilePath, FileMode.OpenOrCreate, FileAccess.Write, FileShare.None, bufferSize: 1, FileOptions.DeleteOnClose))
        {
            if (File.Exists(todayDoneFilePath))
            {
                // After grab the file lock, it turns out a successful check has finished.
                // Then let's bail out early.
                return;
            }

            // Now it's guaranteed that I'm the only process that reaches here.
            foreach (string oldFile in Directory.EnumerateFiles(TestDir, "sentinel-*-*-*.done"))
            {
                // Clean up the old '.done' file, there should be only one.
                File.Delete(oldFile);
            }

            // Do the real update check
            //  - Send HTTP request to query for the new release/pre-release;
            //  - If there is a valid new release that should be reported to the user, create the file `_update_<new-version>`,
            //    or rename the existing `_update_<old-version>` to `_update_<new-version>`.
            // ... more ...

            // Finally, create the `todayDoneFilePath` file as an indicator that a successful update check has finished today.
            new FileStream(todayDoneFilePath, FileMode.CreateNew, FileAccess.Write, FileShare.None).Close();
        }
    }
    catch (Exception e)
    {
        // An update check is in progress from another `pwsh` process. So it's OK to just return.
    }
}
```

> Note: `FileOptions.DeleteOnClose` is used when opening the sentinel file,
so the sentinel file will be removed after being used as a lock.

With the file lock, only one process can get in the guarded `using` block at a given time.
So only one process will be creating the file `_update_<version>_<publish-date>`, or renaming an old such file to reflect the new version.
Yes, other processes could be looking at the old file name (when a `pwsh` session tries to print a notification),
or working with an outdated file name (another update check tries to do a pre-validation).
But it's fine for that to happen:

- In the former case, that particular `pwsh` session will show a notification about an outdated version,
  but the next `pwsh` session will show the right notification.
- In the latter case, the other update check will continue and find the `.done` file already exists for today.

It's possible that a `pwsh` session terminates while the update check task is still running,
in the middle of the `using` block for example.
Creating the `.done` file is the very last step in the `using` block.
So if the session ends before the `.done` file is created,
another update check will happen when the next `pwsh` session starts and finish the work.

#### How to do the update check

This is comparatively the easy part.

- Determine the URL to use depending on the notification type.
  - For latest preview release-info: `https://<buildinfo-blob>/preview.json`
  - For latest stable release-info: `https://<buildinfo-blob>/stable.json`
  - For latest LTS release-info: `https://<buildinfo-blob>/lts.json`
- Send HTTP query request and parse the response.
- If there is a new update, create the file `update<notification-type>_<version>_<publish-date>` if one doesn't exists yet;
  or rename the existing file with the new version.

#### How to display the notification

`pwsh` checks to see if notification should be printed only if it's allowed to print the banner message and feature is not turned off.

- Run `Directory.EnumerateFiles` with the the version directory and the pattern `update<notification-type>_v*.*.*_????-??-??` to find such a file.
- If a file path is returned, then get the version information from the file name.
- Use that version to construct the notification message, including the URL to that GitHub release page.
- The notification message is printed with the foreground and background colors inverted.

## Alternate Proposals and Considerations

When thinking about how to reduce unnecessary update checks,
the first design I had was to depend on the `Day` of the month.
So for instance, we can check for updates every 3 days by checking `DateTime.UtcNow.Day % 3 == 0`.
But that means in the worst case, a user won't be notified of a new release until 3 days after the release.
That makes this feature somewhat broken from the UX perspective.

Another design is to let all `pwsh`, including different versions, share the same update file,
whose name contains both the latest stable release tag and latest preview release tag.
When `pwsh` starts, it parses the file name, compare the latest stable/preview release version with its current version,
and decides if a notification should be printed.
This would reduce the number of helper files in the cache folder,
however, with the cost of additional work at startup time for all versions of `pwsh`.
Especially, for the latest stable or preview `pwsh` in use, it also needs to spend those extra cycles when it should not.
Besides, I think having the helper files isolated in a version folder makes it flexible in case we need to make change to the design at a later time.

We may consider to add an API in PowerShell engine to check for updates in future,
so that other hosts can offer similar features using the API.
