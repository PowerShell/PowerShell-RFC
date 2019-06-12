---
RFC:
Author: Robert Holt
Status: Draft
Area: Pipeline Chain Operators
Comments Due: 2019-07-12 0000
Plan to implement: Yes
---

# Pipeline Chain Operators

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

Also from the Shell Command Language specification:

> A ';' or &lt;newline&gt; terminator shall cause the preceding AND-OR list to be executed sequentially;
> an '&' shall cause asynchronous execution of the preceding AND-OR list.

Meaning that the entire AND-OR list is backgrounded:

```bash
true && echo 'Success' & # The list `true && echo 'Success'` is executed in the background
```

This RFC proposes adding AND-OR lists to PowerShell, using the same `&&` and `||`.
Because "AND-OR list" is neither intuitive nor common as a terminology,
instead the term **pipeline chains** is proposed.

## Motivation

> As a PowerShell user, I can chain commands with `&&` and `||`
> so that I have an ergonomic syntax to conditionally invoke side-effectful commands.

## User Experience

```powershell
Write-Output "Hello" && Write-Output "Hello again"
```

```output
Hello
Hello again
```

```powershell
Write-Output "Hello" || Write-Output "Hello again"
```

```output
Hello
```

```powershell
Write-Error "Bad" && Write-Output "Hello again"
```

```output
Write-Error "Bad" : Bad
+ CategoryInfo          : NotSpecified: (:) [Write-Error], WriteErrorException
+ FullyQualifiedErrorId : Microsoft.PowerShell.Commands.WriteErrorException
```

```powershell
Write-Error "Bad" || Write-Output "Hello again"
```

```output
Write-Error "Bad" : Bad
+ CategoryInfo          : NotSpecified: (:) [Write-Error], WriteErrorException
+ FullyQualifiedErrorId : Microsoft.PowerShell.Commands.WriteErrorException

Hello again
```

Using `echo` as a convenient example of a successful command
and `false` as an example of an unsuccesful command.

```powershell
echo "Hello" && Write-Output "Hi"
```

```output
Hello
Hi
```

```powershell
false && Write-Output "Reached"
```

```output
```

```powershell
false || Write-Output "Reached"
```

```output
Reached
```

```powershell
false || Write-Output "Command failed" && Write-Output "Backup"
```

```output
Command failed
Backup
```

Also see: [intended test cases for implementation](https://github.com/PowerShell/PowerShell/blob/f8b899e42c86957a1b58273be0029f40a67bf1b6/test/powershell/Language/Operators/BashControlOperator.Tests.ps1).

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
            "||"
          /      \
        "&&"     "cmd3"
      /      \
   "cmd1"    "cmd2"
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
         "&&"          "&"
        /    \
     "cmd1"  "cmd2"
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

- Work-in-progress implementation: [PowerShell/PowerShell #9849](https://github.com/PowerShell/PowerShell/pull/9849).

- [Current handling of `&&` and `||` in the parser](https://github.com/PowerShell/PowerShell/blob/af1de9e88d28014438ff3414e82298e5b14f6e81/src/System.Management.Automation/engine/parser/Parser.cs#L5846-L5860).

## Alternate Proposals and Considerations

### `&&` and `||` as a statement separator

To be more bash-like, an alternative implementation might treat
`&&` and `||` with the same precedence as `;`, as a way to separate statements.

This would allow assignment within chains, like:

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

#### Reasons against

- Pipelines have an established concept of "success" compared to statements
- Background operators become less useful with respect to chains unless their
  syntax is changed in a significant way

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