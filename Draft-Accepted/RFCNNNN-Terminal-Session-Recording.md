---
RFC: RFCNNNN
Author: soroshsabz
Status: Draft
SupercededBy: N/A
Version: 1.0
Area: Console Host
Comments Due: 12/31/2026
Plan to implement: Yes
---

# Terminal Session Recording in Start-Transcript

This RFC proposes extending `Start-Transcript` with the ability to record the complete
terminal session — including full-screen, interactive TUI applications — as a
time-stamped terminal stream, and to save that recording either in the open
[asciicast v2](https://github.com/asciinema/asciinema/blob/develop/doc/asciicast-v2.md)
format or rendered as a video file (animated GIF, WebM, or MP4).

This brings to PowerShell a capability equivalent to session recording on
RHEL (`tlog` with SSSD/PAM) and to community tools such as `asciinema` and `script(1)`,
but integrated into the familiar `Start-Transcript` / `Stop-Transcript` experience.

## Motivation

The documentation for [Start-Transcript](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.host/start-transcript)
states: *"The transcript includes all command that the user types and all output that
appears on the console."* In practice, the transcript captures what flows through the
PowerShell host: the user's command line echo and pipeline output. Full-screen
interactive (TUI) applications do not flow through the pipeline:

- On Windows, applications such as `sconfig` (Windows Server) open the console directly
  by device name (`CONIN$` / `CONOUT$`), switch the console to raw mode, and paint the
  screen with VT sequences.
- On Linux, applications such as `nmtui` (RHEL), `vim`, `htop`, or any `curses`/`ncurses`
  program open `/dev/tty`, use the alternate screen buffer, and take over all input and
  output.

The result, as discussed in
[PowerShell/PowerShell#28296](https://github.com/PowerShell/PowerShell/discussions/28296),
is that a transcript of such a session is effectively empty for the duration of the TUI
application. `Start-Transcript` today is, at best, a best-effort text log for TUI programs.

Several real-world scenarios need full-fidelity capture:

- **Audit and compliance.** Organizations that administer servers through interactive
  tools (`sconfig`, `nmtui`, third-party consoles) need a defensible record of what an
  administrator actually did, not only what they typed at the PowerShell prompt. This is
  exactly the problem RHEL solves system-wide with [tlog](https://github.com/Scribery/tlog)
  and SSSD/PAM session recording.
- **Support and incident response.** Reproducing "what did you see on screen when the
  failure happened" is impossible from a plain-text transcript when the failure occurred
  inside a TUI application.
- **Training, documentation, and bug reports.** A short video/GIF of a terminal session,
  including TUI interactions, is the most efficient way to demonstrate a procedure or
  report a rendering bug.

```
As a server administrator,
I can record everything that appears in my terminal — including full-screen TUI
applications like sconfig and nmtui — with Start-Transcript,
so that I have an auditable, replayable, shareable record of the entire session,
not just the PowerShell pipeline output.
```

## Goals / Non-Goals

Goals:

- Capture **everything rendered in the terminal** during a transcript session: PowerShell
  output, native command output, colors, cursor movement, progress rendering, alternate
  screen buffer usage by TUI applications, and terminal resizes — with accurate timing.
- Keep the existing text transcript behavior completely unchanged; recording is additive.
- Use a **portable, open format** (asciicast v2) as the source of truth, so recordings
  interoperate with the existing ecosystem (`asciinema play`, `agg`, `ttyplot`-style
  tooling) instead of inventing a new container.
- Provide **video output** (animated GIF built-in; WebM/MP4 when `ffmpeg` is available)
  for sharing and documentation scenarios.
- Work cross-platform: Windows (pseudo-console/ConPTY), Linux and macOS (PTY).
- Be composable with existing transcription features (path resolution, headers, group
  policy enablement).

Non-Goals:

- Changing or "fixing" the existing text transcript content for TUI applications.
- Audio or graphical (non-terminal) screen recording. (Windows Graphics Capture is
  discussed under Alternate Proposals.)
- Recording remote sessions that are not hosted by the local PowerShell process
  (e.g., sessions inside `ssh` from PowerShell) — the recording covers the local host.
- Shipping a full terminal emulator/player UI in PowerShell. Playback is delegated to
  the asciinema ecosystem; only rendering to video is in scope.

## User Experience

### Recording a TUI session

```powershell
Start-Transcript -Path .\audit\session01 -RecordTerminal
sconfig        # full-screen TUI on Windows Server
# ... interact with sconfig ...
exit
Stop-Transcript
```

```output
Transcript started, output file is .\audit\session01.txt
Terminal recording started, recording file is .\audit\session01.cast
... (sconfig runs, full screen) ...
Windows PowerShell transcript end
End of terminal recording: .\audit\session01.cast (duration 00:02:41)
```

Three artifacts are produced:

| File | Contents |
| ---- | -------- |
| `session01.txt` | The existing text transcript (unchanged behavior) |
| `session01.cast` | Time-stamped terminal stream (asciicast v2) |
| `session01.gif` | Rendered video, only when a video format was requested |

### Recording directly to video

```powershell
Start-Transcript -Path .\demo\intro -RecordTerminal -RecordingFormat WebM
nmtui          # TUI on RHEL
# ... interact ...
Stop-Transcript
```

```output
Transcript started, output file is .\demo\intro.txt
Terminal recording started, recording file is .\demo\intro.cast
... (nmtui runs, full screen) ...
Rendering terminal recording to .\demo\intro.webm ...
Windows PowerShell transcript end
End of terminal recording: .\demo\intro.cast (duration 00:01:12), video: .\demo\intro.webm
```

### Replaying a recording

Recordings use the standard asciicast v2 format and play back with existing tooling:

```powershell
asciinema play .\audit\session01.cast   # full-fidelity replay, original timing
agg .\audit\session01.cast out.gif      # third-party render to GIF
```

### Optional: recording user input

```powershell
Start-Transcript -Path .\audit\session02 -RecordTerminal -RecordInput
```

Input events are stored in the `.cast` file as `i` events (hidden by most players unless
requested), enabling exact reconstruction of what the user typed, including into TUI
applications.

## Specification

### New parameters on `Start-Transcript`

```
Start-Transcript
   [[-Path] <string>]
   [-Append] [-Force] [-NoClobber]
   [-IncludeInvocationHeader] [-UseMinimalHeader]
   [-RecordTerminal]
   [-RecordingFormat <RecordingOutputFormat>]   # Asciicast | Gif | WebM | Mp4
   [-RecordInput]
   [-RecordIdleTimeLimit <int>]                 # seconds; 0 = unlimited (default 0)
   [<CommonParameters>]
```

- **`-RecordTerminal`** (switch, default off): starts a terminal recording for the
  session. The recording spans from `Start-Transcript` until `Stop-Transcript` (or the
  end of the host process, which finalizes the recording, mirroring transcript behavior).
- **`-RecordingFormat`** (default `Asciicast`): the deliverable format.
  - `Asciicast`: write only `*.cast` (the time-stamped terminal stream). No rendering
    cost; playable with the asciinema ecosystem.
  - `Gif`: write `*.cast` **and** render an animated GIF at stop time using a built-in
    managed renderer (no external dependencies).
  - `WebM` / `Mp4`: write `*.cast` **and** render the video via `ffmpeg` if `ffmpeg` is
    discoverable on `PATH`. If `ffmpeg` is not available, `Stop-Transcript` emits a
    warning and the `.cast` file is still complete and valid.
  - The `.cast` file is always written and is treated as the source of truth; video files
    are render products of it. This guarantees no data loss and allows re-rendering later
    with different settings or tools.
- **`-RecordInput`** (switch, default off): additionally records user keystrokes as `i`
  events. Off by default because it can capture secrets that are not echoed to the
  screen (see Security).
- **`-RecordIdleTimeLimit`** (default `0`): when greater than zero, idle periods longer
  than this limit are time-compressed in playback timing metadata, keeping video renders
  of long, idle-heavy sessions watchable.
- `-Append` is **incompatible** with `-RecordTerminal` (a recording cannot be appended
  to) and results in an error explaining the conflict.
- When `-Path` is omitted, the transcript file name is generated as today, and the
  recording file names are derived by replacing the extension with `.cast` and the video
  extension.

### Capture pipeline

PowerShell must capture the terminal byte stream *below* the application, because TUI
applications bypass stdin/stdout and write directly to the console device:

- **Windows**: the host allocates a pseudo-console (`ConPTY`) pair. Native child
  processes (including `sconfig` and any process opening `CONIN$`/`CONOUT$`) are attached
  to the pseudo-console. The host proxies the VT stream bidirectionally between the
  pseudo-console and the real terminal, teeing all output (and optionally input) bytes
  with timestamps into the recorder. Windows 10 1809+ / Windows Server 2019+ provide the
  required `CreatePseudoConsole` API; on older versions `-RecordTerminal` fails with a
  clear platform-not-supported error.
- **Linux/macOS**: the equivalent PTY master/slave arrangement is used, in the same
  manner as `script(1)`.
- PowerShell's own output (engine, cmdlets, PSReadLine rendering) is recorded directly by
  the host, since the host already produces the final VT stream it writes to the terminal.
- Terminal resize events are recorded as `r` events so full-screen TUI applications that
  re-query the terminal size replay correctly.

The recorder writes events incrementally (streaming, line-buffered), so a crashed or
killed session still leaves a playable `.cast` file up to the last flushed event.

### Recording file format (asciicast v2)

The `.cast` file conforms to [asciicast v2](https://github.com/asciinema/asciinema/blob/develop/doc/asciicast-v2.md):
a JSON header line followed by JSON event lines:

```json
{"version": 2, "width": 120, "height": 30, "timestamp": 1762300800, "env": {"SHELL": "pwsh", "TERM": "xterm-256color"}, "title": "PowerShell terminal recording"}
[0.031234, "o", "\u001b]0;PowerShell\u0007"]
[0.104512, "o", "PS C:\\> sconfig\r\n"]
[0.418003, "o", "\u001b[?1049h\u001b[H\u001b[2J"]
```

Event types used: `o` (output), `i` (input, only with `-RecordInput`), `r` (resize).
Choosing an established open format means PowerShell recordings immediately work with
`asciinema play`, `agg`, and other tooling, and gives auditors a text-diffable, greppable
artifact.

### Video rendering

At `Stop-Transcript` (or process exit), if a video format was requested:

1. The `.cast` stream is rendered frame-by-frame using a minimal internal VT interpreter
   at the recorded terminal size, with the default console font (configurable via the
   `PSTerminalRecordingFont` preference variable).
2. `Gif`: frames are encoded by the built-in managed GIF encoder.
3. `WebM`/`Mp4`: frames are piped to `ffmpeg` (located on `PATH` or via
   `PSTerminalRecordingFFmpegPath`). Absence of `ffmpeg` downgrades to a warning; the
   `.cast` remains authoritative.
4. `Stop-Transcript` reports the produced file paths and total duration.

Future work may expose rendering as a standalone cmdlet (for example
`Convert-Asciicast`) so existing `.cast` files can be re-rendered; that cmdlet is not
required by this RFC.

### Group policy and configuration

Consistent with [RFC0041 (Policy)](../Final/RFC0041-Policy.md), the `Transcription`
policy key gains optional values to enable terminal recording organization-wide:

| Policy | Value | Type |
| ------ | ----- | ---- |
| Transcription | `EnableTerminalRecording` | DWORD |
| Transcription | `RecordingFormat` | String (`Asciicast`, `Gif`, `WebM`, `Mp4`) |
| Transcription | `RecordingOutputDirectory` | String |

Policy-enabled recording overrides an explicit `Start-Transcript` invocation only by
*adding* the recording (never by disabling a user-requested one).

### Platform support and experimental rollout

- Implemented as an experimental feature per
  [RFC0029 (Experimental Features)](../Final/RFC0029-Support-Experimental-Features.md),
  with the proposed name `PSTerminalRecording`, enabled via
  `Enable-ExperimentalFeature PSTerminalRecording`. The final name is subject to
  Committee review.
- Windows 10 1809+ / Server 2019+ (ConPTY), Linux, macOS (PTY). Hosts other than the
  console host (for example, the VS Code integrated terminal or third-party hosts) may
  decline recording support; `-RecordTerminal` then fails with
  `PlatformNotSupported`.

### Security considerations

- A terminal recording captures **everything displayed**, including data that plain-text
  transcripts never see (TUI screens, colored output, host names). Where auditing is the
  goal this is the point; where recordings are shared, authors must review them first.
- `-RecordInput` can capture secrets that were typed but never echoed (for example,
  passwords entered into a TUI prompt that disables echo). It is therefore off by
  default, documented with a warning, and recorded in the header of the `.cast` file.
- Recording files are created with the same permissions as the transcript file and
  inherit the ACLs of the destination directory; the Group Policy story directs them to
  restricted audit directories.
- The transcript text log gains a header/footer line noting that a terminal recording
  was produced and where, so audits remain self-describing.

## Alternate Proposals and Considerations

1. **Do nothing; `Start-Transcript` stays best-effort for TUI.** Rejected: it leaves the
   documented promise ("all output that appears on the console") unmet for a large class
   of real admin tools, and the audit scenario unaddressed.
2. **A separate cmdlet, e.g. `Start-InteractiveTranscript`** (suggested in
   [the discussion](https://github.com/PowerShell/PowerShell/discussions/28296#discussioncomment-19362767)).
   Considered seriously. Arguments for: zero risk of muddying existing transcript
   semantics. Arguments against: discoverability and the audit/compliance configuration
   story are much stronger when recording is a first-class mode of the one cmdlet
   everyone already knows, and group policy already targets. This RFC keeps the change
   strictly additive (new switches only), which captures most of the benefit of a
   separate cmdlet. If the Committee prefers, the same specification could be shipped
   under a new cmdlet name with identical semantics.
3. **Ship it as a PSGallery module instead of an engine feature.** A pure-PowerShell or
   .NET module cannot transparently intercept `CONIN$`/`CONOUT$`/`/dev/tty` traffic of
   arbitrary child processes without the host's cooperation — it would have to wrap each
   invocation in a pseudo-console, changing TUI application behavior for every call site.
   Engine-level capture in the host is the only way to make this transparent and complete.
4. **True screen video via Windows Graphics Capture.** Windows-only, records the whole
   window including non-terminal overlays, produces large binary files with no text
   fidelity for audit, and cannot work over SSH sessions. The stream-recording approach
   is cross-platform, tiny, and text-addressable; video is produced by rendering when
   needed.
5. **Invent a new recording format.** Rejected in favor of asciicast v2 for ecosystem
   interoperability (`asciinema`, `agg`) and prior art (`script` timing files, tlog JSON
   events use the same timestamped-event model).
6. **Delegate entirely to the terminal emulator** (Windows Terminal, VS Code). Not all
   PowerShell hosts support it, it would not follow the session across hosts, and it
   cannot be driven by group policy for audit scenarios.

## References

- Discussion: [What should the transcript ideally capture when a TUI application starts in PowerShell?](https://github.com/PowerShell/PowerShell/discussions/28296)
- [console-logger](https://github.com/rhubarb-geek-nz/console-logger) — proof on Windows that a pseudo-console wrapper can display *and* capture interactive programs
- [tlog](https://github.com/Scribery/tlog) / [SSSD session recording](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_and_managing_session_recording/index) — the RHEL/PAM system-wide precedent
- [asciinema](https://asciinema.org/) and [asciicast v2 format](https://github.com/asciinema/asciinema/blob/develop/doc/asciicast-v2.md)
- [RFC0041 - Policy](../Final/RFC0041-Policy.md)
