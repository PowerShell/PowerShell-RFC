---
RFC: RFCXXX-Native-Json-Adapters.md
Author: Jim Truher
Status: Draft
SupercededBy: 
Version: 0.9
Area: engine
Comments Due: May 1, 2023
Plan to implement: Yes
---

# Title

Improving the User Experience of Native Applications

## Motivation

As a user I can interact with native applications and have a more PowerShell-like experience.

### Native Applications which Emit Structured Data

Native command execution usually results in textual output which is a fairly poor experience for our users.
Some of our users use tools like `Crescendo` to enable those tools to "play nice" in the PowerShell arena,
but the problem of output handling is still troublesome.

Some applications (docker, kubectl, others) can emit `JSON` which is much easier for our users to convert to useful results in PowerShell,
but the user still has to know the incantation to have the tool emit `JSON` and then they must pipe that output to `Convert-FromJson`.
We can improve the experience for the user if we can help them discover these tools and additional parameters.
If the user types:

```powershell
PS> docker image ls
```

we can suggest:

```powershell
PS> docker image ls --format='{{json .}}' | ConvertFrom-Json | Format-Table
```

There are two aspects of making this work well:

- Knowing that the the parameter `--format='{{json .}}` is available for the `docker` command.
- Knowing that the invocation will emit json and then suggest how to convert that to objects.

### Providing feedback to the user that such usage is available

We can improve the experience by using the `FeedbackProvider` infrastructure and present feedback to the user which provides an improved pipeline.

For example, if the user types:

```powershell
PS> docker image ls
```

We can provide the following feedback to the user:

```powershell
docker image ls --format='{{json .}}' | ConvertFrom-Json
```

or if the user types the following:

```powershell
PS> uname -a
```

and the `jc` utility is available, we can provide feedback with an improved pipeline:

```powershell
PS> uname -a | jc --uname | ConvertFrom-Json
```

If the user types:

```powershell
PS> ipconfig
```

and if a script which converts the output of `ipconfig` into PowerShell objects named `ipconfig-json` exists, we could suggest the following:

```powershell
PS> ipconfig | ipconfig-json
```

This last suggested pipeline could be generalized to include a suggestion for any `<utility>-json` command if found.
This may encourage the community to create conversion functions/scripts.

#### Needed enhancement to current feedback system

Today, this suggestion is not possible because the feedback provider will only act if there was an error in execution.
To support this behavior, the feedback subsystem would need to be extended so it may be called when no error occurred.

### Open Questions

- How should errors during conversion be handled?
  - In the case where ConvertFrom-Json is used, I would suggest a new parameter for `ConvertFrom-Json` where it simply emits the input string if the conversion fails.

### Follow-on Work

A number of follow-on activities should be considered to bring continued improvement of the native command experience.

#### Conversion of Tabular Text

If the data is _known_ to be tabular it may be possible to provide an additional utility to convert that tabular data to
structured data via means of a generalized converter.
This would require some sort of registration/configuration to designate the native application and call the
converter to automatically construct objects from tabular data could be called.
This would enable the user to set it once and later sessions would be able to use it.
This can be done in a stand-alone module or incorporated in the the TextUtility module in the gallery.
It is reasonable to do this manually, or perhaps added to the feedback provider suggestion above.

#### PSDefaultNativeParameterValues

We can extend the _concept_ of `$PSDefaultParameterValue` to improve the user experience by automatically
adding parameters and their values to the invocation of the native command.
`$PSDefaultNativeParameterValue` could do for native applications what `$PSDefaultParameterValue` does for cmdlets.
This would enable our customers to type:

```powershell
PS> docker image ls
```

rather than

```powershell
PS> docker image ls --format='{{json .}}'
```

It would be difficult to extend the current `$PSDefaultParameterValue` behavior because parameter binding is so different between
native applications and cmdlets.
For example, some parameters must be in a specific location in the command line.

```powershell
PS> docker --debug run --rm -it mcr.microsoft.com/powershell:latest 
```

The use of `--debug` has a required position, it _must_ provided before the `run` subcommand.

An addition complication is that depending on the application, parameters should _not_ be added based on the earlier parts of the command.
For example, you may want to provide different formatting for `git log` vs `git branch`, so you must have more information to determine the proper behavior.
This means that rather than just capturing the parameter name and default value, we will need to be able to run arbitrary code (ala scriptblock)
which includes the command line AST so the proper parameter values may be created.

However, with an approach which provides for this complexity, we can continue to simplify the experience for our users from:

```powershell
PS> docker --debug run --rm -it mcr.microsoft.com/powershell
```

to

```powershell
PS> docker run mcr.microsoft.com/powershell
```

#### Improved TabCompletion

Current tab-completion (on Unix) is done via a somewhat fragile call to the native shell.
Tab-completion on Windows must be implemented specifically.
We should take the opportunity to improve our tooling around tab-completion to make it easier to create
and provide generalized help parsers to generate tab-completion (where possible).
`bash` provides a number of more generalized tab-completion, we should attempt to do the same for PowerShell users.

For example, bash has a single completer all known hosts which is used by nearly 20 executables.
We already will tab-complete for file system items,
a small effort should be able to increase our coverage for native executables.

#### Improved Formatting Tooling

Once text output is converted to objects,
we should improve the process for creating formatting directives.
When creating a formatting file,
the user needs to manually author the `XML` and then call `update-formatdata` to add that configuration to the current session.
Usually, formatting files are associated with a module and distributed in this way.
With stand-alone native applications, it means that providing for reasonable formatting will require an additional file.
A more dynamic approach for formatting would improve the experience for the user.

### History

It's not clear whether history should be altered to show the added pipeline elements.
We don't currently do that for `out-default` which can also add to the pipeline.
It should be relatively easy to determine whether the pipeline as the `app-json`, as `get-command app` will include the name of the json adapter.

## Alternatives

### Extending the pipeline to automatically add a JSON adapter

We could attempt to improve the experience for these users where we can by providing a simple way to implicitly support
extending the engine pipeline with a JSON to object converter.

Custom output filters can be written to take regular or irregular output and convert it to json or objects.
The output filter would also need to have the parameters in use for the original native application.
This would enable a single filter to be used with any combination of parameters.
If these custom filters could be added automatically to the invocation of the native app,
the experience for the user would again be greatly improved.

for example:

```powershell
PS> MyCustomApp.exe -p1 MyString -p2 42 -p3 resource=blue
```

_and_ if `MyCustomApp-json` is available as a conversion tool for `MyCustomApp.exe`,
we can automatically ensure that all the parameters which were used for `MyCustomApp.exe` are passed to the converter.
In this way, the converter has the exact way `MyCustomApp.exe` was invoked,
and can be composed to have different behaviors based on the parameters used by `MyCustomApp.exe`.

> **_Implementation note:_** My current prototype searches for an "<app>-json" when a native `ApplicationInfo`
> is being created during Command Discovery and if found, captures this information in the `ApplicationInfo` object.
> When the Pipeline processor is being constructed, the extension is checked and if found the "<app>-json" is inserted
> into the pipeline immediately following the original native application.

The proposed architecture makes it fairly trivial to add an adapter which only calls `ConvertFrom-Json`.
The combination of this feature, the follow-on work to automatically adding parameters to native applications,
and our formatting system could change this:

```powershell
PS> Docker image ls --format='{{json .}}' | ConvertFrom-Json | Where-Object {$_.Repository -match "microsoft"}| Format-Table Id,Repository,Tag,Size,CreatedAt

ID           Repository                             Tag                     Size   CreatedAt
--           ----------                             ---                     ----   ---------
4e9617ada198 mcr.microsoft.com/powershell           preview-7.3-mariner-2.0 235MB  2022-09-01 11:48:17 -0700 PDT
4bcdf53ee67b mcr.microsoft.com/powershell           latest                  339MB  2022-09-01 11:34:40 -0700 PDT
```

to this:

```powershell
PS> docker image ls | Where-Object Repository -match "microsoft" | Format-Table Id,Repository,Tag,Size,CreatedAt

ID           Repository                             Tag                     Size   CreatedAt
--           ----------                             ---                     ----   ---------
4e9617ada198 mcr.microsoft.com/powershell           preview-7.3-mariner-2.0 235MB  2022-09-01 11:48:17 -0700 PDT
4bcdf53ee67b mcr.microsoft.com/powershell           latest                  339MB  2022-09-01 11:34:40 -0700 PDT
```

#### Additional behaviors

It should be possible, in the case that an adapter is available,
to automatically annotate the emitted objects with a synthetic type so it could be used by our formatting system.
For example, if `Docker image ls` is invoked,
the pipeline could be extended to add `Add-Member -PassThru -TypeName Dockerimagels.JsonAdapted`.
Then a format file could be created to produce an improved display of the data.
This would further reduce the above example to:

```powershell
PS> docker image ls | Where-Object Repository -match "microsoft"

ID           Repository                             Tag                     Size   CreatedAt
--           ----------                             ---                     ----   ---------
4e9617ada198 mcr.microsoft.com/powershell           preview-7.3-mariner-2.0 235MB  2022-09-01 11:48:17 -0700 PDT
4bcdf53ee67b mcr.microsoft.com/powershell           latest                  339MB  2022-09-01 11:34:40 -0700 PDT
```

### Modify CommandDiscovery

While it may be possible to alter CommandDiscovery to prefer a custom alternative program rather than the named program,
this approach has some issues:

- One issue with this approach is that the alternative wrapper is extremely likely to reference the native application,
which means that CommandDiscovery would again hunt for the native wrapper when parsing is done on the wrapper.
CommandDiscovery would need to manage state to hunt only for the native executable (not the wrapper).
CommandDiscovery is largely state free, and this would represent a fairly substantial architectural change.
Further, a possible scenario would be that the wrapper doesn't call the named native app directly, but via proxy;
thus requiring even more state.

### Parser Modification

The parser does not make any disambiguation of the type of command to be executed.
It simply identifies the token which _will_ be executed at run time.
I believe it is not appropriate for the parser to know too much about the type of command to be executed,
its role is to analyze and tokenize the command line.
Moreover, adding this capability to the parser would inextricably link the parser to the runtime.
Architecturally, it seems wrong as well. The parser should _parse_ without too much of an opinion on _how_ it should execute.
