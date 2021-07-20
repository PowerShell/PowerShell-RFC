---
RFC: RFC0045
Author: Dongbo Wang
Status: Final
SupercededBy: N/A
Version: 0.1
Area: Language
Comments Due: August 31st, 2019
Plan to implement: Yes, PS7
---

# Add Ternary Operator to PowerShell Language

The ternary operator is one of the highly demanded enhancement to PowerShell language.
It received 46 up-votes in the issue [Suggestion: implement ternary conditionals](https://github.com/PowerShell/PowerShell/issues/3239) as of the writing of this RFC.
The [prototype draft pull request](https://github.com/PowerShell/PowerShell/pull/10161) for this feature has also received a lot of attention and discussions.

The PowerShell Committee has approved to add the ternary operator to the language,
and this RFC is mainly to capture the design points and implementation details.

## Motivation

> As a PowerShell user, I can use ternary operator as a replacement of if-else statement for simple conditional cases.

## Specification

We will follow the C# ternary operator syntax:

```none
<condition> ? <if-true> : <if-false>
```

Two basic design points:

- Ternary operator produces an expression, just like the binary and unary operators
- Ternary operator has lower precedence order than the binary operator

The ternary operator behaves like the simplified if-else statement.
The `condition-expression` will always be evaluated,
and its result will be converted to boolean to determine which branch will be evaluated next:

- `if-true-expression` will execute if the condition's result is evaluated as `true`
- `if-false-expression` will execute if the condition's result is evaluated as `false`

Some usage examples (see more examples [in the tests](https://github.com/PowerShell/PowerShell/pull/10367/files#diff-b63d92046948a571e210b3abb5a7e685)):

```powershell
...
$psExec = $IsCoreCLR ? 'pwsh' : 'powershell'
...

...
$argument = $env:CI -eq 'true' ? '-Tag CI' : '-Tag Daily'
...

...
$message = (Test-Path $path) ? "Path exists" : "Path not found"
...

...
return $blackList -contains $target ? (Block-Target $target) : (Register-Target $target)
...
```

#### Parser Updates

A new AST type `TernaryExpressionAst` will be introduced to represent a ternary operator expression.
The public API surface of this type is as follows:

```c#
public class TernaryExpressionAst : ExpressionAst
{
    public TernaryExpressionAst(IScriptExtent extent, ExpressionAst condition, ExpressionAst ifTrue, ExpressionAst ifFalse);
    public ExpressionAst Condition { get; }
    public ExpressionAst IfTrue { get; }
    public ExpressionAst IfFalse { get; }

    public Ast Copy();
}
```

The parsing grammar is as follows:

```none
<expression>  '?'  new-lines:opt  <expression>  new-lines:opt  ':'  new-lines:opt  <expression>
```

The challenge in the parser update is around how to deal with number constant expressions more naturally with the ternary operators.
When digits are mixed with `?` or `:`, such as `?123`, `:456`, `123:` and `?123:456`, they are recognized as generic tokens today,
because it's totally legit to have a function or script with those names. However, when it comes to the ternary expression,
this means confusing errors would occur for scripts like `return ${succeed}?0:1` since `?0:1` would be treated as a generic token.
Therefore, spaces around `?` and `:` would be required to use number constant expressions with the ternary operator.
That would certainly be counter-intuitive and should be addressed in the parser update.

The approach I took in the prototype is to make `?` and `:` conditionally force to start a new token when scanning for numbers.
When it's known for sure that we are expecting an expression,
allowing a generic token like `123?` is not useful and bound to result in parsing errors.
In those cases, we will force to start a new token upon seeing characters `?` and `:` when scanning for a number,
so that that number constant expressions can work with ternary operators more intuitively.

Note that, the handling of `?` and `:` doesn't change how those two characters can be used in variable expressions,
and also doesn't affect function and commands with those two characters in names.

#### AST Visitors

Given the fact that we are adding a new AST type, necessary changes are required to `ICustomAstVisitor2`, `AstVisitor2`,
and all other visitor types that implemnent those two.
Obvious examples are the `VariableAnalysis` and `Compiler` as you have to update them properly to correctly enable the new language element.
The not-so-obvious ones include `ConstantValueVisitor`, `SafeValueVisitor`, `TypeInferenceVisitor` and more.
Take the `TypeInferenceVisitor` as an instance, failing to update it may cause tab completion issues that are hard to catch.

It's worth to discuss the `InternalVisit` implemention of the new AST type a bit.
Many `Find*Visitor` types depend on this method to search through an AST tree,
and almost all of them derive from `AstVisitor` instead of `AstVisitor2` because
currently the AST types covered in `AstVisitor2` all have their own special scopes,
such as the `TypeDefinitionAst` and `DynamicKeywordStatement`.
In order to avoid making changes to all those existing `Find*Visitor` types,
the `InternalVisit` method of `TernaryExpressionAst` is implemented as follows to accept an `AstVisitor` visitor for its AST members.
In this way, those existing `Find*Visitor` types can work with this new AST type to search through its members.

```c#
// This pattern appears in several other ASTs.
internal override AstVisitAction InternalVisit(AstVisitor visitor)
{
    var action = AstVisitAction.Continue;
    if (visitor is AstVisitor2 visitor2)
    {
        action = visitor2.VisitTernaryExpression(this);
        if (action == AstVisitAction.SkipChildren)
            return visitor.CheckForPostAction(this, AstVisitAction.Continue);
    }

    if (action == AstVisitAction.Continue)
        action = Condition.InternalVisit(visitor);

    if (action == AstVisitAction.Continue)
        action = IfTrue.InternalVisit(visitor);

    if (action == AstVisitAction.Continue)
        action = IfFalse.InternalVisit(visitor);

    return visitor.CheckForPostAction(this, action);
}
```

### Alternate Proposals and Considerations

Different syntax proposals were brought up in the discussions:

```none
## Use '-then -else' instead of '? :'
<condition-expression> -then <then-expression> -else <else-expression>
```
```none
## Use syntax similar to '-replace' and '-f'
<condition-expression> -ternary <then-expression>, <else-expression>
```

One of the concerns with using `? :` is the visual acuity impact of scanning code where `?` typically has only been used as the alias for `Where-Object`.
However, we expect that as we implement other language features like null-coalescing, null-conditional assignment, and null-conditional access,
we are likely to use `?` in some other forms.
Also, users can choose to opt out of using this new language feature,
and the expectation is that most usage of this new language element will be from C# developers.

After a discussion at length, the PowerShell committee agrees to align with the C# syntax given the above considerations.
