---
RFC:
Author: Robert Holt
Status: Draft
Area: Pipeline Chain Operators
Comments Due: 2019-07-12 0000
Plan to implement: Yes
---

# Pipeline Chain Operators

## Background

POSIX shells have what may be referred to as *AND-OR lists*,
what the [Open Group's Shell Command Language specification](https://pubs.opengroup.org/onlinepubs/007904875/utilities/xcu_chap02.html#tag_02_09_03)
describes as:

> a sequence of one or more pipelines separated by the operators `&&` and `||`

Semantically, these operators look at the exit code of the left-hand side pipeline
and conditionally execute the right-hand side pipeline.
`&&` executes the right-hand side if the left-hand side "succeeded",
and `||` executes the right-hand side if the left-hand side did not "succeed".

Some examples in bash:

```bash
echo 'Success' && echo 'Second success' # Executes both left-hand and right-hand command
echo 'Success' || echo 'Second success' # Executes only the left-hand command
false && echo 'Second success' # Executes only the left-hand command (false always "fails")
false || echo 'Second success' # Executes both left-hand and right-hand commands
false && echo foo || echo bar # Writes only "bar" to stdout (operators are left-associative)
true || echo foo && echo bar # Writes only "bar" to stdout
```

Similarly, `cmd.exe` also supports `&&` and `||`, which it terms *conditional processing symbols*.
From the [Command shell overview page](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-xp/bb490954(v=technet.10)#using-multiple-commands-and-conditional-processing-symbols):

> When you run multiple commands with conditional processing symbols,
> the commands to the right of the conditional processing symbol act
> based upon the results of the command to the left of the conditional processing symbol.
>
> ...
>
> `&&`: Use to run the command following `&&` only if the command preceding the symbol is successful.
> Cmd.exe runs the first command,
> and then runs the second command only if the first command completed successfully.
>
> ...
>
> `||`: Use to run the command following `||` only if the command preceding `||` fails.
> Cmd.exe runs the first command,
> and then runs the second command only if the first command did not complete successfully
> (receives an error code greater than zero).

Despite the wording here, `&&` has a lower precedence than `|` in `cmd.exe`
and can be used to sequence pipelines:

```cmd
dir C:\ | sort && echo 'done'
```

Historically, these operators have been reserved for implementation in PowerShell for some time.
Since PowerShell v2, including a `&&` token in a PowerShell script results in the following parse error:

> The token '&&' is not a valid statement separator in this version.

Despite the error message implying the intent to separate statements,
[the code to parse these operators](https://github.com/PowerShell/PowerShell/blob/6f0dacddc1b6ddc47a886f3943b56725d3d2e2f4/src/System.Management.Automation/engine/parser/Parser.cs#L5859-L5871)
occurs *within* the pipeline parsing logic,
possibly implying the intent to implement them as *command separators* (within a pipeline).

## Proposal outline

This RFC proposes:

- The addition of `&&` and `||` as *pipeline chain operators* to PowerShell.
- That `&&` and `||` may be used between PowerShell pipelines to conditionally sequence them.
- That `$?` (PowerShell's execution success indicator) be used to determine the sequence from pipeline to pipeline.
- That such sequences of pipelines using `&&` and `||` be called **pipeline chains**.
- To also allow control flow statements (`throw`, `break`, `continue`, `return` and `exit`) at the end of such chains

## Motivation

> As a PowerShell user, I can chain commands and pipelines with `&&` and `||`
> so that I have an ergonomic syntax to conditionally invoke side-effectful commands.

The chief motivation of pipeline chain operators is to make native commands
(i.e. commands run by invoking an executable as a subprocess)
simpler to use and sequence, as they are in other shells.

Often these commands perform some action,
emit some informational output (and/or error output) and return an exit code.

The aim of chain operators is to make the action success as easy to process as the output,
providing a convenient way to manipulate control flow around command outcome rather than output.
This is also the motivation behind allowing control flow statements at the end of pipelines.

## User Experience

Also see: [test cases for implementation](https://github.com/PowerShell/PowerShell/blob/0b700828f22824c29eac70f8db1d2bf504b212d1/test/powershell/Language/Operators/PipelineChainOperator.Tests.ps1).


Pipeline chain operators are intended to behave
as if pipelines were written as a sequence of statements conditioned on `$?`:

```powershell
cmd1 && cmd2 || cmd3
```

should be the same as

```powershell
cmd1
if ($?) { cmd2 }
if (-not $?) { cmd3 }
```

### Native commands

In these examples:

- `echo` is a native command that writes its argument as output and returns an exit code of 0
- `error` is a native command that writes its argument as output and returns an exit code of 1

```powershell
echo 'Hello' && echo 'Again'
```

```output
Hello
Again
```

---

```powershell
echo 'Hello' && error 'Bad'
```

```output
Hello
Bad
```

---

```powershell
error 'Bad' && echo 'Hello'
```

```output
Bad
```

---

```powershell
error 'Bad' || echo 'Hello'
```

```output
Bad
Hello
```

---

```powershell
echo 'Hello' || echo 'Again'
```

```output
Hello
```

---

```powershell
error 'Bad' || error 'Very bad'
```

```output
Bad
Very bad
```

---

```powershell
echo 'Hi' || echo 'Message' && echo '2nd message'
```

```output
Hi
```

---

```powershell
error 'Bad' || echo 'Message' && echo '2nd message'
```

```output
Bad
Message
2nd message
```

---

```powershell
echo 'Hi' && error 'Bad' || echo 'Message'
```

```
Hi
Bad
Message
```

---

###  Cmdlets and Functions

Cmdlets and functions work just like native commands,
except they don't set `$LASTEXITCODE`
and have other ways of expressing error conditions.

Here the same principle applies as with native commands;
the statements proceed as if the next is
wrapped in `if ($?) { ... }`.

```powershell
Write-Output "Hello" && Write-Output "Hello again"
```

```output
Hello
Hello again
```

---

```powershell
Write-Output "Hello" || Write-Output "Hello again"
```

```output
Hello
```

---

```powershell
Write-Error "Bad" && Write-Output "Hello again"
```

```output
Write-Error "Bad" : Bad
+ CategoryInfo          : NotSpecified: (:) [Write-Error], WriteErrorException
+ FullyQualifiedErrorId : Microsoft.PowerShell.Commands.WriteErrorException
```

`Write-Error` here emits a non-terminating error
and so we proceed to evaluate `$?`.

---

```powershell
Write-Error "Bad" || Write-Output "Hello again"
```

```output
Write-Error "Bad" : Bad
+ CategoryInfo          : NotSpecified: (:) [Write-Error], WriteErrorException
+ FullyQualifiedErrorId : Microsoft.PowerShell.Commands.WriteErrorException

Hello again
```

```powershell
echo 'Hi' && Write-Output 'Hello'
```

```output
Hi
Hello
```

---

```powershell
error 'Bad' && Write-Output 'Hello'
```

```
Bad
```

---

```powershell
Write-Error 'Bad' || echo 'Message'
```

```output
Bad
Message
```

---

### Pipelines

Pipeline chains allow whole pipelines between chain operators.
As above, when a pipeline ends other than with a terminating error,
`$?` determines the chain logic.

The whole pipeline on the left-hand side of an operator
will be evaluated before evaluating chain condition
and then right-hand side.

```powershell
1,2,3 | ForEach-Object { $_ + 1 } && Write-Output 'Hello'
```

```output
2
3
4
Hello
```

---

```powershell
1,2,3 | ForEach-Object { if ($_ -eq 2) { Write-Error 'Bad' } else { $_ } } && Write-Output 'Hello'
```

```output
1
1,2,3 | ForEach-Object { if ($_ -eq 2) { Write-Error 'Bad' } else { $_ } } && Write-Output 'Hello' : Bad
+ CategoryInfo          : NotSpecified: (:) [Write-Error], WriteErrorException
+ FullyQualifiedErrorId : Microsoft.PowerShell.Commands.WriteErrorException

3
Hello
```

---

```powershell
function FailInProcess
{
    [CmdletBinding()]
    param(
        [Parameter(ValueFromPipeline)]
        $Value
    )

    process
    {
        if ($_ -eq 3)
        {
            $err = Write-Error 'Bad' 2>&1
            $PSCmdlet.WriteError($err)
            return
        }

        $PSCmdlet.WriteObject($_)
    }
}

1,2,3,4 | FailInProcess && Write-Output 'Succeeded'
```

```output
1
2
FailInProcess : Bad
At line:22 char:11
+ 1,2,3,4 | FailInProcess && Write-Output 'Succeeded'
+           ~~~~~~~~~~~~~
+ CategoryInfo          : NotSpecified: (:) [Write-Error], WriteErrorException
+ FullyQualifiedErrorId : Microsoft.PowerShell.Commands.WriteErrorException,FailInProcess

4
```

(Even though the process block keeps going, `$?` is false)

---

### Terminating errors and error handling

Terminating errors supercede chain sequencing,
just as they would in a semicolon-separated sequence of statements.

Uncaught errors will terminate the script.

```powershell
function ThrowBad
{
    throw 'Bad'
}

ThrowBad && Write-Output 'Success'
```

```output
Bad
At line:3 char:5
+     throw 'Bad'                                                                                   +     ~~~~~~~~~~~
+ CategoryInfo          : OperationStopped: (Bad:String) [], RuntimeException
+ FullyQualifiedErrorId : Bad
```

---

This is the same with cmdlet terminating errors.

```powershell
function ThrowTerminating
{
    [CmdletBinding()]
    param()

    $err = Write-Error 'Bad' 2>&1
    $PSCmdlet.ThrowTerminatingError($err)
}

ThrowTerminating && Write-Output 'Success'
```

```output
ThrowTerminating : Bad
At line:1 char:1
+ ThrowTerminating && Write-Output 'Success'
+ ~~~~~~~~~~~~~~~~
+ CategoryInfo          : NotSpecified: (:) [Write-Error], WriteErrorException
+ FullyQualifiedErrorId : Microsoft.PowerShell.Commands.WriteErrorException,ThrowTerminating 
```

---

When the error is caught,
the flow of control abandons the chain for the catch as expected.

```powershell
function ThrowBad
{
    throw 'Bad'
}

try
{
    ThrowBad && Write-Output 'Success'
}
catch
{
    Write-Output $_.FullyQualifiedErrorId
}
```

```output
Bad
```

---

When the error is suppressed with an error action preference,
the result is again based on `$?`.

```powershell
function ThrowTerminating
{
    [CmdletBinding()]
    param()

    Write-Output 'In ThrowTerminating'
    $ex = [System.Exception]::new('Bad')
    $errId = 'Bad'
    $errCat = 'NotSpecified'
    $err = [System.Management.Automation.ErrorRecord]::new($ex, $errId, $errCat, $null)
    $PSCmdlet.ThrowTerminatingError($err)
}

ThrowTerminating -ErrorAction Ignore && Write-Output 'Success'
```

```output
In ThrowTerminating
```

---

```powershell
function ThrowTerminating
{
    [CmdletBinding()]
    param()

    Write-Output 'In ThrowTerminating'
    $ex = [System.Exception]::new('Bad')
    $errId = 'Bad'
    $errCat = 'NotSpecified'
    $err = [System.Management.Automation.ErrorRecord]::new($ex, $errId, $errCat, $null)
    $PSCmdlet.ThrowTerminatingError($err)
}

ThrowTerminating -ErrorAction Ignore || Write-Output 'Success'
```

```output
In ThrowTerminating
Success
```

---

If traps are set, they will continue or break the pipeline chain as configured.

```powershell
trap
{
    Write-Output 'TRAP'
    break
}

function ThrowTerminating
{
    [CmdletBinding()]
    param()

    Write-Output 'In ThrowTerminating'
    $ex = [System.Exception]::new('Bad')
    $errId = 'Bad'
    $errCat = 'NotSpecified'
    $err = [System.Management.Automation.ErrorRecord]::new($ex, $errId, $errCat, $null)
    $PSCmdlet.ThrowTerminatingError($err)
}

ThrowTerminating && Write-Output 'Success'
```

```output
In ThrowTerminating
TRAP
ThrowTerminating : Bad
At line:20 char:1
+ ThrowTerminating && Write-Output 'Success'
+ ~~~~~~~~~~~~~~~~
+ CategoryInfo          : NotSpecified: (:) [ThrowTerminating], Exception
+ FullyQualifiedErrorId : Bad,ThrowTerminating

```

---

```powershell
trap
{
    Write-Output 'TRAP'
    continue
}

function ThrowTerminating
{
    [CmdletBinding()]
    param()

    Write-Output 'In ThrowTerminating'
    $ex = [System.Exception]::new('Bad')
    $errId = 'Bad'
    $errCat = 'NotSpecified'
    $err = [System.Management.Automation.ErrorRecord]::new($ex, $errId, $errCat, $null)
    $PSCmdlet.ThrowTerminatingError($err)
}

ThrowTerminating && Write-Output 'Success'
```

```output
In ThrowTerminating
TRAP
```

---

```powershell
trap
{
    Write-Output 'TRAP'
    continue
}

function ThrowTerminating
{
    [CmdletBinding()]
    param()

    Write-Output 'In ThrowTerminating'
    $ex = [System.Exception]::new('Bad')
    $errId = 'Bad'
    $errCat = 'NotSpecified'
    $err = [System.Management.Automation.ErrorRecord]::new($ex, $errId, $errCat, $null)
    $PSCmdlet.ThrowTerminatingError($err)
}

ThrowTerminating || Write-Output 'Success'
```

```output
In ThrowTerminating
TRAP
Success
```

---

### Assignment

Because pipeline chains are statements, they can be assigned from.

```powershell
$x = Write-Output 'Hi' && Write-Output 'Hello'
$x
```

```output
Hi
Hello
```

---

With pipeline chains, if it would be written to the pipeline,
it's captured by assignment.
This is true even if something in the chain later throws.

```powershell
try
{
    $x = Write-Output 'Hi' && throw 'Bad'
}
catch
{
    $_.FullyQualifiedErrorId
}
$x
```

```output
Bad
Hi
```

(Compare this to `$x = $(1;2; throw 'Bad')`.)

---

### Flow control statements

Finally, pipeline chains can use flow control statements.

```powershell
```

```output
```

---

---

## Specification

### Grammar

Pipeline chains will be implemented using the already reserved `&&` and `||` operators.
These operators will have the following grammar:

```none
statement:
    | ... <existing statements except pipeline>   # Other statements
    | variable_expression "=" statement           # Assignment
    | pipeline_chain ["&"]                        # Pipeline chains

pipeline_chain:
    | pipeline
    | pipeline_chain "&&" [newlines] pipeline
    | pipeline_chain "||" [newlines] pipeline
```

#### Optional pipeline chaining

A pipeline chain is a chain of one or more pipelines
and takes the place of a pipeline in the current PowerShell syntax.

In the degenerate case, a pipeline chain of a single pipeline
will work exactly as pipelines currently do in PowerShell,
so there is no breakage to the existing grammar.

#### Line continuation

After a pipeline chain operator,
any newlines will be skipped in anticipation of the following pipeline.

For example, the following would be a single pipeline chain:

```powershell
cmd1 &&
    cmd2 ||
    cmd3
```

If the end of file is reached after a pipeline chain operator,
incomplete input will be reported so that integrating tools
will know to keep prompting for input.

#### Left-associativity

Like in POSIX shells, pipeline chain operators will be left associative,
meaning chain operators will group from left to right.

For example, given the following pipeline chain:

```powershell
cmd1 && cmd2 || cmd3
```

This will be grouped as:

```none
[[cmd1 && cmd2] || cmd3]
```

As a syntax tree, this would look like:

```none
            ||
          /    \
        &&      cmd3
      /     \
   cmd1    cmd2
```

With the syntax tree deepening on the left as more operators are chained.

Semantically, this means that `cmd1 && cmd2` would be evaluated first,
and its result used to govern the evaluation of `|| cmd3`.

#### Higher precedence than `&` and `;`

Pipeline chain operators will have higher precedence than
pipeline background operators (`&`)
or statement separators (`;`).

For example:

```none
cmd1 && cmd2 &
```

Will bind as:

```none
[[cmd1 && cmd2] &]
```

Having the syntax tree:

```none
           <statement>
          /           \
         &&            &
       /    \
     cmd1  cmd2
```

The consequence of this will be that an entire pipeline chain
can be sent to a background job for evaluation,
rather than individual pipelines within it.

### Semantics

#### Pipeline "success"

The `&&` and `||` operators proceed based on the "success"
of the previous pipeline.

The marker of command success proposed is the `$?` automatic variable.
This is proposed because:

- It builds off an existing PowerShell concept
- It applies to both native commands and cmdlets/functions
- Diverging from this would create inconsistency
- Changes to the behaviour of `$?` would be reasonably expected to change here

That is, there are the following direct equivalents:

```none
cmd1 && cmd2

    |
    v

$(cmd1; if ($?) { cmd2 })
```

```none
cmd1 || cmd2

    |
    v

$(cmd1; if (-not $?) { cmd2 })
```

```none
cmd1 && cmd2 || cmd3

    |
    v

$(cmd1; if ($?) { cmd2 }; if (-not $?) { cmd3 })
# Note that cmd1 failing runs cmd3
```

Also see the [alternate proposals section](#alternate-proposals-and-considerations).

#### Pipeline output

The output of a pipeline chain is the concatenation of all its pipelines,
in the sequence of their output:

```powershell
$x = 'a','b','c' | Write-Output && Write-Output 'd'
$x # 'a','b','c','d'
```

```powershell
$x = 'a','b','c' | Write-Output && Write-Error 'Bad' || Write-Output 'd'
# Writes the error record 'Bad'
$x # 'a','b','c','d'
```

Commands that fail will have any pipeline output
emitted before evaluation of the chain operator.

For example:

```powershell
$x = cmd1 && cmd2
```

If `cmd1` fails but emits output, `$x` will hold that value.

#### Error semantics

Errors will have the same semantics as the equivalent
`cmd1; if ($?) { cmd2 }` syntax.

Terminating errors will terminate the entire pipeline chain.
But output already emitted by earlier pipelines in the chain
will be output by the chain before terminating:

```powershell
try
{
    $x = cmd1 && $(throw "Bad")
}
catch
{
}

$x # Has values from cmd1
```

Non-terminating errors will cause the immediate pipeline to continue,
and the pipeline chain will evaluate as normal based on the value of `$?`.

### Other notes

#### New Abstract Syntax Tree (AST) types

The new AST type, `PipelineChainAst`,
will be created to represent a pipeline chain.
This will inherit from `PipelineBaseAst`
and may occur anywhere a `PipelineBaseAst` does in a PowerShell AST.

`ICustomAstVisitor2` and `AstVisitor2` would be extended to deal with this AST,
and .NET Core 3's new default interface implementation feature would be
leveraged to ensure this does not break things
as previous syntactic introductions have been forced to.

#### Statements may not be chained

PowerShell has a notable divergence from POSIX shells:

- In POSIX shells, a pipeline is composed of statements
- In PowerShell, a pipeline is a kind of statement

This means that bash allows the following:

```bash
if [ -n $VAR ]; then echo 'IF'; fi || echo 'AFTER-IF'
```

In fact, this can be evaluated in the background:

```bash
if [ -n $VAR ]; then echo 'IF'; fi || echo 'AFTER-IF' &
```

However, in PowerShell, pipelines are subordinate to statements.
The proposed implementation of chain operators precludes use of
`&&` and `||` between statements like `if` or `while`,
or constructions like `$x = cmd1 && $y = cmd2 && $x + $y`.

Also see the [alternate proposals section](#alternate-proposals-and-considerations).

## Related Material

- Original issue: [PowerShell/PowerShell #3241](https://github.com/PowerShell/PowerShell/issues/3241).

- Implementation: [PowerShell/PowerShell #9849](https://github.com/PowerShell/PowerShell/pull/9849).

- [Current handling of `&&` and `||` in the parser](https://github.com/PowerShell/PowerShell/blob/af1de9e88d28014438ff3414e82298e5b14f6e81/src/System.Management.Automation/engine/parser/Parser.cs#L5846-L5860).

## Alternate Proposals and Considerations

### Command vs pipeline vs statement separation

An important syntactic and semantic question is
what level `&&` and `||` operate at
(using brackets to denote syntactic groupings in `$x = cmd1 | cmd2 && cmd3`):

- Between commands, where they occur within pipelines (`$x = [[cmd1] | [cmd2 && cmd3]]`)
- Between pipelines, where they occur within statements (`$x = [[cmd1 | cmd2] && cmd3]]`)
- Between statements (`[$x = [cmd1 | cmd2]] && [cmd3]`)

In the POSIX shell, `&&` and `||` separate pipelines,
but pipelines encompass all statements.
Statelines like `if`, `for` and `case` are *compound commands*
where between keywords like `if` and `fi`,
everything is considered a single command.

So for example the following is possible:

```sh
if [ -e ./file.txt ]; then echo 'File exists'; fi && echo 'File does not exist' | cat -
```

(This always prints `File does not exist`, since the `if` is always considered to succeed.)

In `cmd.exe`, the more template-driven approach means that
`if` and `for` being commands treat `&&` as part of the argument:

```cmd
>for %i in (1 2 3) do echo %i && echo 'done'
```

```output
>echo 1   && echo 'done'
1
'done'

>echo 2   && echo 'done'
2
'done'

>echo 3   && echo 'done'
3
'done'
```

In PowerShell, unlike in the POSIX shell,
all pipelines are statements but not all statements are pipelines.
This means we must choose between separating statements and separating pipelines.

Allowing `&&` and `||` between statements in PowerShell might look like:

```powershell
$x = cmd1 && $y = cmd2 && $x + $y
```

Other possibilities would include:

```powershell
if ($condition) { cmd1 } && while ($anotherCondition) { cmd2 }
```

```powershell
foreach ($v in 1..100)
{
    cmd1 $v && break
}
```

#### Background operator changes

In bash, background operators are lower precedence than chain separators
(`&` has the same precedence as `;`).
In PowerShell, background operators are higher precedence.

Making chain operators apply to any statement would lead to the question
of the precedence of the background operator.

The example:

```powershell
cmd1 && cmd2 &
```

Could either be `[[cmd1 && cmd2] &]` or `[cmd1 && [cmd2 &]]`.

Having chain operators apply to statements would mean the second case
is the correct one, unless changes are made to the background operator precedence (which would also allow `if ($condition) { cmd1 } &)` for example).

The use of this is possibly less than the use of being able to background
an entire pipeline chain; `cmd1 & && cmd2` would be equivalent to
`cmd1 &; cmd2` since a backgrounded pipeline
will never fail in the invoking context.

Such a change to the background operator precedence
would cause ambiguity with uses like `return cmd1 &` and `throw cmd1 &`,
where the background operator would currently apply to the pipeline under the keyword.
To not break PowerShell's existing semantics,
special behaviour would need to be defined for `return $expr &`.

#### Reasons against

- Pipelines have an established concept of "success",
  whereas other statements generally do not.
- Background operators become less useful with respect to chains unless their
  syntax is changed in a significant way.

### Allowing control flow statements at the end of chains

A compromise to the above is to only allow "control flow statements"
at the end of chains.

For example:

```powershell
cmd1 || throw "cmd1 failed"
```

```powershell
function Invoke-Command
{
    cmd1 && return

    cmd2
}
```

```powershell
foreach ($v in 1..100)
{
    cmd1 $v && break
}
```

These would only be allowed *at the end* of chains because:

- Control flow makes any invocation after the statement unreachable
- `return` and `throw` allow pipelines as subordinate expressions,
   meaning it would be grammatically impossible to use those statements mid-chain.
   See below for more details.

#### Reasons against

- `throw` and `return` currently allow a subordinate pipeline:

   ```powershell
   throw "Bad"
   ```

   ```powershell
   throw 1,2,3 | Write-Output
   ```

   ```powershell
   throw "Useless" & # Throws a background job...
   ```

   If we keep the principle that anywhere a pipeline is allowed,
   a pipeline chain is now allowed, then constructions like this are possible:

   ```powershell
   cmd1 || throw "Bad" && cmd3
   ```

   However the grouping is this:

   ```none
   [cmd1 || throw ["Bad" && cmd3]]
   ```

   This could lead to confusing semantic corner cases

- The compromise nature of this approach means the PowerShell Language
  becomes more complicated and arguably less consistent,
  both conceptually and in terms of maintenance.

### Different evaluations of command "success"

The current proposal is to simply use `$?` to determine chain continuation.

Alternatives include:

- `$LASTEXITCODE`
- Whether errors have been written
- A new custom semantics

A problem is that exit code semantics assume
that a non-zero exit code means command failure.
POSIX shells also make this assumption,
but the convention may not be as widespread on Windows platforms.

#### Reasons against

- `$LASTEXITCODE` is specific to native commands
  and use with cmdlets and functions may lead to unexpected results.
  For example:

  ```powershell
  cmd_that_fails
  Write-Output "SUCCESS" && Write-Output "ALSO SUCCESS"
  ```

  Will never write `"ALSO SUCCESS"` if `$LASTEXITCODE` is used.

- Commands can fail without writing an error
  and some utilities will write to standard error without having failed
  (`time` being a good example on \*nix).

- Having chain operators use their own failure semantics
  would create more conceptual complexity
  and be inconsistent with the established `$?` concept.

- Allowing configurability of the success determination would
  likely only make sense with a per-command configuration.
  This would already be achieved simply with the proposed `$?`
  semantics using a wrapper function.

### Terminology

We should have a unified terminology
to describe these operators in PowerShell
for use with:

- The experimental feature name
- The about_ topic
- Web searchability

This RFC proposes **Pipeline chain operators**.
Other possibilities are:

- AND-OR lists
- Command chain operators
- Command control operators
- Bash control operators
- Control operators
- Short circuit operators