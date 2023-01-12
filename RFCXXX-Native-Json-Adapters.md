---
RFC: RFCXXX-Native-Json-Adapters.md
Author: Jim Truher
Status: Draft
SupercededBy: 
Version: 0.8
Area: engine
Comments Due: March 1, 2023
Plan to implement: Yes
---

# Title

Improving the User Experience of Native Applications

## Motivation

As a user I can interact with native applications which can produce a more object like experience.

### Native Applications which Emit Structured Data

Native command execution usually results in textual output which is a fairly poor experience for our users.
Some of our users use tools like `Crescendo` to allow those tools to "play nice" in the PowerShell arena,
but the problem of output handling is still troublesome.

Some applications (docker, kubectl, others) can emit in `JSON` which is much easier for our users to convert to useful results in PowerShell,
but the user still has to know the incantation to have the tool emit `JSON` and then they must pipe that output to `Convert-FromJson`.
It would be a much better experience if the user could simply type:

```powershell
PS> docker image ls
```

rather than

```powershell
PS> docker image ls --format='{{json .}}' | ConvertFrom-Json | Format-Table
```

There are two aspects of making this work well:

- Automatically adding `ConvertFrom-Json` to the pipeline following the native command to automatically convert `Json` to object
- Automatically adding a parameter to a native command similarly to what `$PSDefaultParameterValue` does for cmdlets

### Automatically converting structured text data to PowerShell objects

We should attempt to improve the experience for these users where we can by providing simple way to implicitly support
extending the pipeline with a JSON to object converter.

Custom output filters can be written to take regular or irregular output and convert it to json or objects.
The output filter would also need to have the parameters in use for the original native application.
This would enable a single filter to be used with any combination of parameters.
If these custom filters could be added automatically to the invocation of the native app,
the experience for the user would again be greatly improved.

for example:

```powershell
PS> MyCustomApp.exe -p1 MyString -p2 42 -p3 resource=blue
```

_and_ if `MyCustomApp-json` is available as a conversion too for `MyCustomApp.exe`,
we could automatically construct the following:

```powershell
PS> MyCustomApp.exe -p1 MyString -p2 42 -p3 resource=blue | MyCustomApp-json -ParameterBlock @{ p1 = "MyString"; p2 = 42; p3 = "resource=blue" } | ConvertFrom-Json
```

The additional `-ParameterBlock` above ensures that the parameters used by `MyCustomApp.exe` would be available to the filter
so it could change its behavior based on the parameters in use.

> **_Implementation note:_** My current prototype searches for an "<app>-json" when a native `ApplicationInfo`
> is being created during Command Discovery and if found, captures this information in the `ApplicationInfo` object.
> When the Pipeline processor is being constructed, the extension is checked and if found the "<app>-json" is inserted
> into the pipeline immediately following the original native application.

A reasonable approach may be to specify the name of the application, then provide a scriptblock to determine the value of a given parameter.
This would be somewhat more complicated than our current `PSDefaultParameterValues` but the complexity is needed to evaluate the proper values to provide.

The proposed architecture makes it fairly trivial to add an adapter which only calls `ConvertFrom-Json`.
The combination of this feature, and the follow-on work to automatically adding parameters to native applications, and our formatting system could change this:

```powershell
PS> Docker image ls --format='{{json .}}' | ConvertFrom-Json | Where-Object {$_.Repository -match "microsoft"}| Format-Table Id,Repository,Tag,Size,CreatedAt

ID           Repository                             Tag                     Size   CreatedAt
--           ----------                             ---                     ----   ---------
4e9617ada198 mcr.microsoft.com/powershell           preview-7.3-mariner-2.0 235MB  2022-09-01 11:48:17 -0700 PDT
4bcdf53ee67b mcr.microsoft.com/powershell           latest                  339MB  2022-09-01 11:34:40 -0700 PDT
97a21961f76d mcr.microsoft.com/mssql/server         2022-latest             1.6GB  2022-08-18 16:59:49 -0700 PDT
02214dbfb621 mcr.microsoft.com/powershell/test-deps ubi-8.4                 640MB  2022-08-11 11:35:47 -0700 PDT
e3afdc6d8e5c mcr.microsoft.com/mssql/server         2019-latest             1.47GB 2022-07-25 01:45:18 -0700 PDT
```

to this:

```powershell
PS> docker image ls --format='{{json .}}'| Where-Object Repository -match "microsoft"

ID           Repository                             Tag                     Size   CreatedAt
--           ----------                             ---                     ----   ---------
4e9617ada198 mcr.microsoft.com/powershell           preview-7.3-mariner-2.0 235MB  2022-09-01 11:48:17 -0700 PDT
4bcdf53ee67b mcr.microsoft.com/powershell           latest                  339MB  2022-09-01 11:34:40 -0700 PDT
97a21961f76d mcr.microsoft.com/mssql/server         2022-latest             1.6GB  2022-08-18 16:59:49 -0700 PDT
02214dbfb621 mcr.microsoft.com/powershell/test-deps ubi-8.4                 640MB  2022-08-11 11:35:47 -0700 PDT
e3afdc6d8e5c mcr.microsoft.com/mssql/server         2019-latest             1.47GB 2022-07-25 01:45:18 -0700 PDT
```

#### Automatic Conversion of Tabular Text

If the data is _known_ to be tabular it may be possible to _automatically_ convert that tabular data to
structured data via means of a generalized converter.
This would require some sort of registration/configuration to designate the native application and call the
converter to automatically construct objects from tabular data could be called.
This would enable the user to set it once and later sessions would be able to use it.

#### PSDefaultNativeParameterValues

Additionally, we can extend the _concept_ of the `$PSDefaultParameterValue` variable to improve the user experience by automatically
adding parameters and their values to the invocation of the native command.
This would enable our customers to type:

```powershell
PS> docker image ls
```

rather than

```powershell
PS> docker image ls --format='{{json .}}'
```

Further, it would be better to extend the `$PSDefaultParameterValue` behavior because parameter binding is done differently;
some parameters must be in a specific location in the command line.
For example, there are certain parameters which apply to the executable and not the subcommand:

```powershell
PS> docker --debug run --rm -it mcr.microsoft.com/powershell:latest 
```

The use of `--debug` has a required position, it _must_ provided before the `run` subcommand.

An addition complication is that depending on the application, parameters should _not_ be added based on the earlier parts of the command.
For example, you may want to provide different formatting for `git log` vs `git branch`, so you must have more information to determine the proper behavior.
This means that rather than just capturing the parameter name and default value, we will need to be able to run arbitrary code (ala scriptblock)
which includes the command line AST so the proper parameter values may be created.

However, with an approach which provides for this complexity, we can continue to simplify the experience for our users to from:

```powershell
PS> docker --debug run --rm -it mcr.microsoft.com/powershell
```

to

```powershell
PS> docker run mcr.microsoft.com/powershell
```

or

```powershell
PS> Docker image ls --format='{{json .}}' | ConvertFrom-Json | Where-Object {$_.Repository -match "mcr"}| Format-Table Id,Repository,Tag,Size,CreatedAt

ID           Repository                             Tag                     Size   CreatedAt
--           ----------                             ---                     ----   ---------
4e9617ada198 mcr.microsoft.com/powershell           preview-7.3-mariner-2.0 235MB  2022-09-01 11:48:17 -0700 PDT
. . .
```

to this:

```powershell
PS> docker image ls | ? Repository -match "mcr"

ID           Repository                             Tag                     Size   CreatedAt
--           ----------                             ---                     ----   ---------
4e9617ada198 mcr.microsoft.com/powershell           preview-7.3-mariner-2.0 235MB  2022-09-01 11:48:17 -0700 PDT
. . .
```

### Follow-on Work

A number of follow-on activities should be considered to bring continued improvement of the native command experience.

#### Improved TabCompletion

Current tab-completion (on Unix) is done via a somewhat fragile call to the native shell.
Tab-completion on Windows must be implemented specifically.
we should take the opportunity to improve our tooling around tab-completion to make it easier to create,
and provide generalized help parsers (where possible) to generate tab-completion.
`bash` provides a number of more generalized tab-completion, we should attempt to do the same for PowerShell users.

For example, bash has a single completer all known hosts which is used by nearly 20 executables.
We already will tab-complete for file system items,
a small effort should be able to increase our coverage for other native executables.

#### Improved Formatting Tooling

Once text output is converted to objects,
we should improve the process for creating formatting directives.
When creating a formatting file,
the user needs to manually author the `XML` and then call `update-formatdata` to add that configuration to the current session.
Usually, formatting files are associated with a module and distributed in this way.
With stand-alone native applications, it means that providing for reasonable formatting will require an additional file.
A more dynamic approach for formatting would improve the experience for the user.

## Open Questions

### History

It's not clear whether history should be altered to show the added pipeline elements. We don't currently do that for `out-default` which can also add to the pipeline.
It should be relatively easy to determine whether the pipeline as the `app-json`, as `get-command app` will include the name of the json adapter.

### Adaptation of Native Apps by Fully Qualified Path

Should the automatic json adaptation be suppressed when a fully qualified path (`/bin/ls` vs `ls`) is provided?
This may be problematic as most scripted solutions may construct a full path to the application,
and relying on `$env:PATH` could be used as a code injection attack vector.
The expectation is that the `<app>-json` converter will be found as either a function, script, or other native app.
If a script, or other native app we would use normal command discovery to find the converter,
which would not be specified with fully qualified path.
Regardless of mechanism, it should be a feature to temporarily disable automatic adaptation.

### Selective non-adaptation

Should there be a way to suppress adaptation on a single invocation.
Shells such `bash` have a way to set a variable for a single invocation:

```bash
$ P=. ls -ld $P
drwxr-xr-x  10 james  staff  320 Dec  6 12:05 .
```

Can or should something like this be done for PowerShell?

## Alternatives

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
It simple identifies the token which _will_ be executed at run time.
I believe it is not appropriate for the parser to know too much about the type of command to be executed,
its role is to analyze and tokenize the command line.
Moreover, adding this capability to the parser would inextricably link the parser to the runtime.
Architecturally, it seems wrong as well. The parser should _parse_ without too much of an opinion on _how_ it should execute.
