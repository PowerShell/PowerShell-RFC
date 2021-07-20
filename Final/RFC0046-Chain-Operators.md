---
RFC: RFC0046
Author: Robert Holt
Status: Final
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

## User Experience

Also see: [test cases for implementation](https://github.com/PowerShell/PowerShell/blob/7457d526258988a632e8dada08486ee9785c6c3c/test/powershell/Language/Operators/PipelineChainOperator.Tests.ps1).


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

#### Simple successful command chain

```powershell
echo 'Hello' && echo 'Again'
```

```output
Hello
Again
```

---

#### Simple error after successful command

```powershell
echo 'Hello' && error 'Bad'
```

```output
Hello
Bad
```

---

#### Error followed by command in success case

```powershell
error 'Bad' && echo 'Hello'
```

```output
Bad
```

---

#### Error followed by command in failure case

```powershell
error 'Bad' || echo 'Hello'
```

```output
Bad
Hello
```

---

#### Command followed by command in failure case

```powershell
echo 'Hello' || echo 'Again'
```

```output
Hello
```

---

#### Error followed by error in failure case

```powershell
error 'Bad' || error 'Very bad'
```

```output
Bad
Very bad
```

---

#### Composite chain: 1st succeeds, 2nd is skipped, 3rd is run

```powershell
echo 'Hi' || echo 'Message' && echo '2nd message'
```

```output
Hi
2nd message
```

---

#### Composite chain: 1st fails, 2nd is run, 3rd is run

```powershell
error 'Bad' || echo 'Message' && echo '2nd message'
```

```output
Bad
Message
2nd message
```

---

#### Composite chain: 1st succeeds, 2nd fails, 3rd is run

```powershell
echo 'Hi' && error 'Bad' || echo 'Message'
```

```
Hi
Bad
Message
```

---

### Cmdlets and Functions

Cmdlets and functions work just like native commands,
except they don't set `$LASTEXITCODE`
and have other ways of expressing error conditions.

Here the same principle applies as with native commands;
the statements proceed as if the next is
wrapped in `if ($?) { ... }`.

#### Simple cmdlet chain: success then success

```powershell
Write-Output "Hello" && Write-Output "Hello again"
```

```output
Hello
Hello again
```

---

#### Simple cmdlet chain: success otherwise success

```powershell
Write-Output "Hello" || Write-Output "Hello again"
```

```output
Hello
```

---

#### Simple cmdlet chain: error then success

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

#### Simple cmdlet chain: error otherwise success

```powershell
Write-Error "Bad" || Write-Output "Hello again"
```

```output
Write-Error "Bad" : Bad
+ CategoryInfo          : NotSpecified: (:) [Write-Error], WriteErrorException
+ FullyQualifiedErrorId : Microsoft.PowerShell.Commands.WriteErrorException

Hello again
```

---

#### Simple command chain: native success then cmdlet success

```powershell
echo 'Hi' && Write-Output 'Hello'
```

```output
Hi
Hello
```

---

#### Simple command chain: native error then cmdlet success

```powershell
error 'Bad' && Write-Output 'Hello'
```

```output
Bad
```

---

#### Simple command chain: cmdlet error otherwise native success

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

#### Succeeding pipeline on the left hand side of a chain

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

#### Non-terminating error in pipeline

When some input fails while processing a pipeline,
that sets `$?` for that command invocation
and the pipeline chain proceeds accordingly.

```powershell
function Test-NonTerminatingError
{
    [CmdletBinding()]
    param(
        [Parameter(ValueFromPipeline)]
        $Input
    )

    process
    {
        if ($Input -ne 2)
        {
            return $Input
        }

        # Write a non-terminating error when $Input is 2
        # Note that Write-Error will not set $? for the caller here

        $exception = [System.Exception]::new('Bad')
        $errorId = 'Bad'
        $errorCategory = 'InvalidData'
        $errorRecord = [System.Management.Automation.ErrorRecord]::new($exception, $errorId, $errorCategory, $null)

        $PSCmdlet.WriteError($errorRecord)
    }
}

1,2,3 | Test-NonTerminatingError && Write-Output 'Hello'
```

```output
1
1,2,3 | ForEach-Object { if ($_ -eq 2) { Write-Error 'Bad' } else { $_ } } && Write-Output 'Hello' : Bad
+ CategoryInfo          : NotSpecified: (:) [Write-Error], WriteErrorException
+ FullyQualifiedErrorId : Microsoft.PowerShell.Commands.WriteErrorException

3
```

#### Non-terminating error with `||`

```powershell
1,2,3 | Test-NonTerminatingError || Write-Output 'Problem!'
```

```output
1
1,2,3 | ForEach-Object { if ($_ -eq 2) { Write-Error 'Bad' } else { $_ } } && Write-Output 'Hello' : Bad
+ CategoryInfo          : NotSpecified: (:) [Write-Error], WriteErrorException
+ FullyQualifiedErrorId : Microsoft.PowerShell.Commands.WriteErrorException

3
Problem!
```

---

#### Pipeline-terminating error in a chain

```powershell
function Test-PipelineTerminatingError
{
    [CmdletBinding()]
    param(
        [Parameter(ValueFromPipeline)]
        $Input
    )

    process
    {
        if ($Input -ne 2)
        {
            return $Input
        }

        # Write a non-terminating error when $Input is 2

        $exception = [System.Exception]::new('Bad')
        $errorId = 'Bad'
        $errorCategory = 'InvalidData'
        $errorRecord = [System.Management.Automation.ErrorRecord]::new($exception, $errorId, $errorCategory, $null)

        # Note the use of ThrowTerminatingError() rather than WriteError()
        $PSCmdlet.ThrowTerminatingError($errorRecord)
    }
}

1,2,3 | Test-PipelineTerminatingError && Write-Output 'Succeeded'
```

```output
1
Test-PipelineTerminatingError : Bad
At line:1 char:9
+ 1,2,3 | Test-PipelineTerminatingError && Write-Output 'Succeeded'
+         ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
+ CategoryInfo          : InvalidData: (:) [Test-PipelineTerminatingError], Exception
+ FullyQualifiedErrorId : Bad,Test-PipelineTerminatingError

```

Note that unlike with the non-terminating error,
the pipeline does not proceed to process `3`.

#### Pipeline termination is not chain termination

```powershell
1,2,3 | Test-PipelineTerminatingError || Write-Output 'Failed'
```

```output
1
Test-PipelineTerminatingError : Bad
At line:1 char:9
+ 1,2,3 | Test-PipelineTerminatingError || Write-Output 'Failed'
+         ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
+ CategoryInfo          : InvalidData: (:) [Test-PipelineTerminatingError], Exception
+ FullyQualifiedErrorId : Bad,Test-PipelineTerminatingError

Failed
```

Here, the pipeline to the left of `||` is terminated,
but the chain continues since `||` is used and `$?` is false,
meaning `Write-Output 'Failed'` is executed.

#### Interaction with `try`/`catch`

If an error is caught from within a pipeline chain,
the chain will be abandoned for the catch block.

```powershell
try
{
    1,2,3 | Test-PipelineTerminatingError || Write-Output 'Failed'
}
catch
{
    Write-Output "Caught error"
}
```

```output
1
Caught error
```

---

### Script-terminating errors and error handling

Script-terminating errors supercede chain sequencing,
just as they would in a semicolon-separated sequence of statements.

Uncaught errors will terminate the script.

```powershell
function ThrowBad
{
    throw 'Bad'
}

ThrowBad || Write-Output 'Failed'
```

```output
Bad
At line:3 char:5
+     throw 'Bad'                                                                                   +     ~~~~~~~~~~~
+ CategoryInfo          : OperationStopped: (Bad:String) [], RuntimeException
+ FullyQualifiedErrorId : Bad
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
    ThrowBad || Write-Output 'Failed'
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

ThrowTerminating || Write-Output 'Continued'
```

```output
In ThrowTerminating
TRAP
Continued
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

With pipeline chains, as with any other PowerShell statements,
assignment either succeeds or does not;
there is no concept of partial success.

```powershell
try
{
    $x = Write-Output 'Hi' && throw 'Bad'
}
catch
{
    Write-Output "Error: $($_.FullyQualifiedErrorId)"
}
Write-Output "`$x: $x"
```

```output
Error: Bad
$x:
```

(Compare this to `$x = . { 'Hi'; throw 'Bad' }`.)

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

. { cmd1; if ($?) { cmd2 } }
```

```none
cmd1 || cmd2

      |
      v

. { cmd1; if (-not $?) { cmd2 } }
```

```none
cmd1 && cmd2 || cmd3

         |
         v

. { cmd1; if ($?) { cmd2 }; if (-not $?) { cmd3 } }
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

Non-terminating and pipeline-terminating errors will cause the immediate pipeline to continue,
and the pipeline chain will evaluate as normal based on the value of `$?`.

Script-terminating errors will terminate the entire pipeline chain,
unless a `trap { continue }` is used.

While output from a chain that later throws a script-terminating error
will be written to the pipeline,
it will not be assigned to a variable.
This is consistent with existing assignment semantics in PowerShell.

### Other notes

#### New Abstract Syntax Tree (AST) types

A single new AST leaf type is proposed, `PipelineChainAst`,
which refers to a pipeline chain
that can be used anywhere where a pipeline could be currently.
This inherits from `PipelineBaseAst`.

`ICustomAstVisitor2` and `AstVisitor2` would be extended to deal with this new AST type,
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

An original addendum to the pipeline chain proposal
was to allow adding control flow statements at the end of pipeline chains:

- `cmd1 && return 'Done'`
- `cmd2 || throw 'Error'`
- `cmd3 && break`
- `cmd4 || continue`
- `cmd5 || exit 1`

This introduces complications:

- A pipeline chain can be both over and under a `return`:

    ```powershell
    cmd1 && return cmd2 && cmd3
    ```

    groups as

    ```text
    cmd1 && [return [cmd2 && cmd3]]
    ```

- A `throw` can do the same:

    ```powershell
    cmd1 && throw 'a' && 'b'
    ```

    This is especially unhelpful since `throw` stringifies its given value,
    making a construction like the above much less useful than for `return`.

This also complicates the grammar and the AST, since:

- We have to complexify logic about stamtents vs pipelines and control flow logic
- More AST types may be needed to prevent bad AST constructions

By keeping control flow statements directly out of pipeline chains:

- The grammar is simplified
- We only need one AST type
- There's no confusing embedding of chains over and under a `return`/`throw`/`exit`

Control flow statements can still be used by embedding them into a subexpression:

- `cmd1 && $(return 'Done')`
- `cmd2 || $(throw 'Error')`
- `cmd3 && $(break)`
- `cmd4 || $(continue)`
- `cmd5 || $(exit 1)`

#### Reasons against

- Control flow manipulation with native commands is currently not ergonomic
- A construction like `cmd || throw 'Failed'` is very handy
- Bad or confusing cases are corner cases, unlikely to be hit under normal usage

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
