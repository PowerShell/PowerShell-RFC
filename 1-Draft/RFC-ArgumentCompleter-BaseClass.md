---
RFC: RFC<four digit unique incrementing number assigned by Committee, this shall be left blank by the author>
Author: Staffan Gustafsson
Status: Draft
SupercededBy: 
Version: <Major>.<Minor>
Area: Command Completion
Comments Due: 1 month
Plan to implement: Yes
---

Provide a base class, ArgumentCompleterBase, to simplify writing custom argument completers

Some common tasks needs to be done almost every time a completer is written.

Things like quoting arguments if they contain spaces, and matching against `WordToComplete`.
Ensuring that the results are sorted is another such task.

## Motivation

    As a Developer,
    I can use the base class when writing custom argument completers,
    so that results are achieved faster, with high quality, and 
    so that users get a consistent completion experience.

## User Experience

```csharp
using namespace System.Management.Automation;

public class GitCommitCompleter : ArgumentCompleter
{
    protected override CompletionResultSortKind AddCompletionsFor(string commandName, string parameterName, IDictionary fakeBoundParameters)
    {
        switch (parameterName)
        {
            case nameof(GitRebaseCommand.CommitHash):
                CompleteGitCommitHash();
                break;
        }

        return CompletionResultSortKind.Sorted;
    }

    private void CompleteGitCommitHash()
    {
        var user = GetBoundParameterOrDefault<string>("User", defaultValue: null);
        foreach (var commit in GitExe.Log())
        {
            if (user is { } u && commit.User != u)
            {
                continue;
            }
            CompleteMatching(text: commit.Hash, toolTip: $"{commit.User}\r\n{commit.Description}", completionMatch: CompletionMatch.AnyContainsWordToComplete);
        }
    }
}
```


## Specification

```csharp
    /// <summary>
    ///     Base class for writing custom Argument Completers
    /// </summary>
    /// <para>Derived classes should override <see cref="AddCompletionsFor" /> </para>
    public abstract class ArgumentCompleter : IArgumentCompleter
    {
        private readonly StringComparison _stringComparison;
        private List<CompletionResult>? _results;
        private IDictionary? _fakeBoundParameters;
        private string? _wordToComplete;
        private CommandAst? _commandAst;
        private string _quote;

        protected ArgumentCompleter(StringComparison stringComparison = StringComparison.CurrentCultureIgnoreCase) : this(01, stringComparison)
        {
        }

        protected ArgumentCompleter(int capacity, StringComparison stringComparison = StringComparison.CurrentCultureIgnoreCase)
        {
            _stringComparison = stringComparison;
            if (capacity > 0)
            {
                _results = new List<CompletionResult>(capacity: capacity);
            }

            _quote = string.Empty;
        }
        private List<CompletionResult> Results => _results ??= new List<CompletionResult>();

#pragma warning disable CA1033 // Interface methods should be callable by child types
        IEnumerable<CompletionResult>? IArgumentCompleter.CompleteArgument(string commandName, string parameterName,
#pragma warning restore CA1033 // Interface methods should be callable by child types
            string wordToComplete, CommandAst commandAst, IDictionary fakeBoundParameters)
        {
            _fakeBoundParameters = fakeBoundParameters;
            _quote = CompletionCompleters.HandleDoubleAndSingleQuote(ref wordToComplete);
            WordToComplete = wordToComplete;
            CommandAst = commandAst;
            var sortKind = AddCompletionsFor(commandName: commandName, parameterName: parameterName, fakeBoundParameters: fakeBoundParameters);
            SortResults(sortKind);
            return _results;
        }

        [return: NotNullIfNotNull("defaultValue")]
        public T? GetBoundParameterOrDefault<T>(string parameterName, T? defaultValue) where T : class => _fakeBoundParameters?[key: parameterName] is T value ? value : defaultValue;

        /// <summary>
        ///     Override in child class to add completions by calling <see ref="CompleteWith" />
        /// </summary>
        /// <param name="commandName">the command to complete parameters for</param>
        /// <param name="parameterName">the parameter to complete</param>
        /// <param name="fakeBoundParameters">previously specified command parameters</param>
        protected abstract CompletionResultSortKind AddCompletionsFor(string commandName, string parameterName, IDictionary fakeBoundParameters);

        /// <summary>
        ///     Adds a completion result to the result set
        /// </summary>
        /// <param name="completionResult">the completion result to add</param>
        public void Complete(CompletionResult completionResult)
        {
            Results.Add(item: completionResult);
        }

        /// <summary>
        ///     Adds a completion result to the result set with the specified parameters
        /// </summary>
        /// <param name="text">the text to be used as the auto completion result</param>
        /// <param name="listItemText">the text to be displayed in a list</param>
        /// <param name="toolTip">the text for the tooltip with details to be displayed about the object</param>
        /// <param name="resultType">the type of completion result</param>
        /// <param name="isGlobbingPath"><see langword="true"/> if the parameter to complete is a globbing path. This escapes '[' and ']'.</param>
        public void Complete(string text, string? listItemText = null, string? toolTip = null, CompletionResultType resultType = CompletionResultType.ParameterValue, bool isGlobbingPath = false)
        {
            if (text == null) throw new ArgumentNullException(nameof(text));

            var quotedText = QuoteCompletionText(completionText: text, isGlobbingPath);
            var completionResult = new CompletionResult(completionText: quotedText, listItemText ?? text,
                resultType: resultType, toolTip ?? text);
            Results.Add(item: completionResult);
        }

        /// <summary>
        ///     Adds a completion result to the result set if the text starts with <see cref="WordToComplete" />
        /// </summary>
        /// <para>The comparison is case insensitive</para>
        /// <param name="text">the text to be used as the auto completion result</param>
        /// <param name="listItemText">the text to be displayed in a list</param>
        /// <param name="toolTip">the text for the tooltip with details to be displayed about the object</param>
        /// <param name="resultType">the type of completion result</param>
        /// <param name="completionMatch">specifies how item(s) are match against <see cref="WordToComplete"/>.</param>
        public void CompleteMatching(string text, string? listItemText = null, string? toolTip = null, CompletionResultType resultType = CompletionResultType.ParameterValue, CompletionMatch completionMatch = CompletionMatch.TextStartsWithWordToComplete)
        {
            switch (completionMatch)
            {
                case CompletionMatch.TextStartsWithWordToComplete:
                    CompleteIfTextStartsWithWordToComplete(text, listItemText, toolTip, resultType);
                    break;
                case CompletionMatch.TextContainsWordToComplete:
                    CompleteIfTextContainsWordToComplete(text, listItemText, toolTip, resultType);
                    break;
                case CompletionMatch.AnyStartsWithWordToComplete:
                    CompleteIfAnyStartsWithWordToComplete(text, listItemText, toolTip, resultType);
                    break;
                case CompletionMatch.AnyContainsWordToComplete:
                    CompleteIfAnyContainsWordToComplete(text, listItemText, toolTip, resultType);
                    break;
                default:
                    throw new ArgumentOutOfRangeException(nameof(completionMatch), completionMatch, null);
            }
        }

        /// <summary>
        ///     Adds a completion result to the result set if the text starts with <see cref="WordToComplete" />
        /// </summary>
        /// <para>The comparison is case insensitive</para>
        /// <param name="text">the text to be used as the auto completion result</param>
        /// <param name="listItemText">the text to be displayed in a list</param>
        /// <param name="toolTip">the text for the tooltip with details to be displayed about the object</param>
        /// <param name="resultType">the type of completion result</param>
        private void CompleteIfTextStartsWithWordToComplete(string text, string? listItemText = null, string? toolTip = null, CompletionResultType resultType = CompletionResultType.ParameterValue)
        {
            if (StartWithWordToComplete(text: text))
                Complete(text: text, listItemText ?? text, toolTip ?? text,
                    resultType: resultType);
        }

        /// <summary>
        ///     Adds a completion result to the result set if the any string argument starts with <see cref="WordToComplete" />
        /// </summary>
        /// <para>The comparison is case insensitive</para>
        /// <param name="text">the text to be used as the auto completion result</param>
        /// <param name="listItemText">the text to be displayed in a list</param>
        /// <param name="toolTip">the text for the tooltip with details to be displayed about the object</param>
        /// <param name="resultType">the type of completion result</param>
        private void CompleteIfAnyStartsWithWordToComplete(string text, string? listItemText = null,
            string? toolTip = null, CompletionResultType resultType = CompletionResultType.ParameterValue)
        {
            if (StartWithWordToComplete(text: text) || StartWithWordToComplete(text: listItemText) || StartWithWordToComplete(text: toolTip))
                Complete(text: text, listItemText ?? text, toolTip ?? text,
                    resultType: resultType);
        }

        /// <summary>
        ///     Adds a completion result to the result set if the any string argument starts with <see cref="WordToComplete" />
        /// </summary>
        /// <para>The comparison is case insensitive</para>
        /// <param name="text">the text to be used as the auto completion result</param>
        /// <param name="listItemText">the text to be displayed in a list</param>
        /// <param name="toolTip">the text for the tooltip with details to be displayed about the object</param>
        /// <param name="resultType">the type of completion result</param>
        private void CompleteIfAnyContainsWordToComplete(string text, string? listItemText = null,
            string? toolTip = null, CompletionResultType resultType = CompletionResultType.ParameterValue)
        {
            if (ContainsWordToComplete(text: text) || ContainsWordToComplete(text: listItemText) || ContainsWordToComplete(text: toolTip))
                Complete(text: text, listItemText ?? text, toolTip ?? text,
                    resultType: resultType);
        }

        /// <summary>
        ///     Adds a completion result to the result set if the text contains <see cref="WordToComplete" />
        /// </summary>
        /// <para>The comparison is case insensitive</para>
        /// <param name="text">the text to be used as the auto completion result</param>
        /// <param name="listItemText">the text to be displayed in a list</param>
        /// <param name="toolTip">the text for the tooltip with details to be displayed about the object</param>
        /// <param name="resultType">the type of completion result</param>
        private void CompleteIfTextContainsWordToComplete(string text, string? listItemText = null,
            string? toolTip = null, CompletionResultType resultType = CompletionResultType.ParameterValue)
        {
            if (ContainsWordToComplete(text: text))
                Complete(text: text, listItemText ?? text, toolTip ?? text,
                    resultType: resultType);
        }

        /// <summary>
        /// If necessary, puts quotation marks around the completion text
        /// </summary>
        /// <param name="completionText">The text to complete</param>
        /// <param name="isGlobbingPath"><see langword="true"/> if the characters [ and ] should be escaped.</param>
        /// <returns>A quoted string, if quoting was necessary. Otherwise <see param="completionText"/>.</returns>
        protected virtual string QuoteCompletionText(string completionText, bool isGlobbingPath = false)
        {
            if (CompletionCompleters.CompletionRequiresQuotes(completionText, isGlobbingPath))
            {
                var quoteInUse = _quote == string.Empty ? "'" : _quote;
                if (quoteInUse == "'")
                {
                    completionText = completionText.Replace("'", "''");
                }
                else
                {
                    // When double quote is in use, we have to escape the backtip and '$' even when using literal path
                    //   Get-Content -LiteralPath ".\a``g.txt"
                    completionText = completionText.Replace("`", "``");
                    completionText = completionText.Replace("$", "`$");
                }

                if (isGlobbingPath)
                {
                    if (quoteInUse == "'")
                    {
                        completionText = completionText.Replace("[", "`[");
                        completionText = completionText.Replace("]", "`]");
                    }
                    else
                    {
                        completionText = completionText.Replace("[", "``[");
                        completionText = completionText.Replace("]", "``]");
                    }
                }

                completionText = quoteInUse + completionText + quoteInUse;
            }
            else if (_quote != string.Empty)
            {
                completionText = _quote + completionText + _quote;
            }

            return completionText;
        }

        /// <summary>
        ///     Predicate to test if a string starts with <see cref="WordToComplete" />
        /// </summary>
        /// <param name="text">text to compare to  <see cref="WordToComplete" />.</param>
        /// <returns><see langword="true"/> if the text contains <see cref="WordToComplete" />, otherwise <see langword="false"/></returns>
        public bool StartWithWordToComplete(string? text) => StartWithWordToComplete(text, _stringComparison);

        /// <summary>
        ///     Predicate to test if a string starts with <see cref="WordToComplete" />
        /// </summary>
        /// <param name="text">text to compare to <see cref="WordToComplete" />.</param>
        /// <param name="stringComparison">Determines whether the beginning of this string instance matches the specified string when compared using the specified comparison option.</param>
        /// <returns><see langword="true"/> if the text contains <see cref="WordToComplete" />, otherwise <see langword="false"/></returns>
        public bool StartWithWordToComplete(string? text, StringComparison stringComparison) =>
            !string.IsNullOrEmpty(value: text) && text.StartsWith(value: WordToComplete, comparisonType: stringComparison);

        /// <summary>
        ///     Predicate to test if a string starts with <see cref="WordToComplete" />
        /// </summary>
        /// <param name="text">the text to test</param>
        /// <returns><see langword="true"/> if the text contains <see cref="WordToComplete" />, otherwise <see langword="false"/></returns>
        public bool ContainsWordToComplete(string? text) => ContainsWordToComplete(text, _stringComparison);

        /// <summary>
        ///     Predicate to test if a string starts with <see cref="WordToComplete" />
        /// </summary>
        /// <param name="text">the text to test</param>
        /// <param name="stringComparison">Determines whether the beginning of this string instance matches the specified string when compared using the specified comparison option.</param>
        /// <returns><see langword="true"/> if the text contains <see cref="WordToComplete" />, otherwise <see langword="false"/></returns>
        public bool ContainsWordToComplete(string? text, StringComparison stringComparison) =>
            !string.IsNullOrEmpty(value: text) && text.IndexOf(value: WordToComplete, comparisonType: stringComparison) != -1;

        /// <summary>
        /// Sort the completion results as specified by the <paramref name="completionResultSortKind"/> parameter.
        /// </summary>
        /// <param name="completionResultSortKind">Specifies of the completion results should be returned as is or sorted in some way.</param>
        protected virtual void SortResults(CompletionResultSortKind completionResultSortKind)
        {
            int CompareStartWithWordToCompleteCompletionResult(CompletionResult x, CompletionResult y)
            {
                var startsWith = x.CompletionText.StartsWith(WordToComplete, _stringComparison).CompareTo(y.CompletionText.StartsWith(WordToComplete, _stringComparison));
                return startsWith != 0 ? startsWith : string.Compare(strA: x.CompletionText, strB: y.CompletionText, comparisonType: StringComparison.Ordinal);
            }

            int CompareCompletionResult(CompletionResult x, CompletionResult y) => string.Compare(strA: x.CompletionText, strB: y.CompletionText, _stringComparison);

            switch (completionResultSortKind)
            {
                case CompletionResultSortKind.None:
                    break;
                case CompletionResultSortKind.PreferStartsWithWordToComplete:
                    Results.Sort(comparison: CompareStartWithWordToCompleteCompletionResult);
                    break;
                case CompletionResultSortKind.Sorted:
                    Results.Sort(comparison: CompareCompletionResult);
                    break;
                default:
                    throw new ArgumentOutOfRangeException(nameof(completionResultSortKind), completionResultSortKind, null);
            }
        }

        public bool Empty => _results?.Count == 0;

        /// <summary>
        ///     The word to complete
        /// </summary>
        public string WordToComplete
        {
            get => _wordToComplete ?? throw new InvalidOperationException();
            private set => _wordToComplete = value;
        }

        /// <summary>
        ///     The Command Ast for the command to complete
        /// </summary>
        public CommandAst CommandAst
        {
            get => _commandAst ?? throw new InvalidOperationException();
            private set => _commandAst = value;
        }
    }

    internal class CompletionCompleters
    {
        /// <summary>
        ///     Determines what the <see param="wordToComplete"/> is without quotes, and 
        ///     what quote character, if any, is uses
        /// </summary>
        /// <returns>The quote character, ' or ", or the empty string.</returns>
        internal static string HandleDoubleAndSingleQuote(ref string wordToComplete)
        {
            string quote = string.Empty;

            if (!string.IsNullOrEmpty(wordToComplete) && (wordToComplete[0].IsSingleQuote() || wordToComplete[0].IsDoubleQuote()))
            {
                char frontQuote = wordToComplete[0];
                int length = wordToComplete.Length;

                if (length == 1)
                {
                    wordToComplete = string.Empty;
                    quote = frontQuote.IsSingleQuote() ? "'" : "\"";
                }
                else if (length > 1)
                {
                    if ((wordToComplete[length - 1].IsDoubleQuote() && frontQuote.IsDoubleQuote()) || (wordToComplete[length - 1].IsSingleQuote() && frontQuote.IsSingleQuote()))
                    {
                        wordToComplete = wordToComplete.Substring(1, length - 2);
                        quote = frontQuote.IsSingleQuote() ? "'" : "\"";
                    }
                    else if (!wordToComplete[length - 1].IsDoubleQuote() && !wordToComplete[length - 1].IsSingleQuote())
                    {
                        wordToComplete = wordToComplete.Substring(1);
                        quote = frontQuote.IsSingleQuote() ? "'" : "\"";
                    }
                }
            }

            return quote;
        }


        /// <summary>
        ///     Determines if the item to complete requires quotes
        /// </summary>
        internal static bool CompletionRequiresQuotes(string completion, bool escape)
        {
            // If the tokenizer sees the completion as more than two tokens, or if there is some error, then
            // some form of quoting is necessary (if it's a variable, we'd need ${}, filenames would need [], etc.)

            Parser.ParseInput(completion, out Token[] tokens, out ParseError[] errors);

            ReadOnlySpan<char> charToCheck = escape ? stackalloc char[] { '$', '[', ']', '`' } : stackalloc char[] { '$', '`' };

            // Expect no errors and 2 tokens (1 is for our completion, the other is eof)
            // Or if the completion is a keyword, we ignore the errors
            bool requireQuote = !(errors.Length == 0 && tokens.Length == 2);
            if ((!requireQuote && tokens[0] is StringToken) ||
                (tokens.Length == 2 && (tokens[0].TokenFlags & TokenFlags.Keyword) != 0))
            {
                requireQuote = false;
                var value = tokens[0].Text.AsSpan();
                if (value.IndexOfAny(charToCheck) != -1)
                    requireQuote = true;
            }

            return requireQuote;
        }
    }

    internal static class CharExtensions {
        public static bool IsSingleQuote(this char c) => c == '\'';
        public static bool IsDoubleQuote(this char c) => c == '\"';
    }

```

## Alternate Proposals and Considerations
