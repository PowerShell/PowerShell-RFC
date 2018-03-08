---
RFC: RFC
Author: Paul Higinbotham, Travis Plunk, Dongbo Wang
Status: Draft
SupercededBy: N/A
Version: 1.0
Area: Engine
Comments Due: March 30, 2018
---

# Machine Learning based Command Line Completion Suggestions for PowerShell

The RFC proposes enhancing PowerShell's existing command line completion with machine learning based suggestions.  

## Motivation

PowerShell's existing command line completion looks for closest matches of known commands, cmdlets, and cmdlet parameters. The match is performed on the text fragment currently typed on the command line. Commands are shown alphabetically and parameters based on parameter set.  

It would be interesting if machine learning could enhance this by providing completion results based on common usage patterns. These patterns could be from the global community that would help new users of PowerShell. Or they could be based on the current user's history to help that user recall complex patterns without having to refer to notes. The machine learning model could provide a richer experience by encompassing context over multiple commands either piped together or typed on multiple lines.  

At this point it is not clear what the right machine learning model is to provide the best experience. It is expected to take many iterations of models and data processing before good and useful suggestions are provided. Customer privacy is critically important and so this feature will be opt-in since data collection is required to train the models. Collected data may be kept locally but more likely will be transferred to the cloud to create a larger data set for training. We must adhere to General Data Protection Regulation (GDPR) requirements to ensure that data is safe and in control of the customer.  

The architecture will allow running components locally or in the cloud.  The cloud has tremendous processing power for training and running models, but latency may make locally running components more attractive in some cases.  

Machine Learning is an inherently experimental process where different models and data features are examined, trained and evaluated. This RFC does not attempt to solve the machine learning and user experience problems, but instead provide an architecture to support experimentation of various Machine learning models and UI candidates.  

### Goals

* Support for both local and cloud based implementations of any component
* Integrates with existing PowerShell command completion
* Provides for privacy protection
* Supports feature opt-in

## Specification 

### Modules
#### Prediction Module  
* Collects command line text used in PowerShell for model training
* Returns prediction from submitted command line fragment
* Creates suggestion success statistics based on user choice

#### General Data Protection Regulation (GDPR) support Module
These regulations apply to any personal data that is collected, and allows the user to:
* View
* Edit
* Delete
* Restrict
* Export

The mechanism for feeding the model with data and retrieving results will closely follow the existing command completion implementation.  The Prediction Module will implement APIs that will:  
* Accept command line fragments with which to generate predictions
* Provide prediction information including confidence level
* Collect complete command lines processed by PowerShell (GDPR applied)
* Generate success statistics based on user choices

The user experience will be dependent on the PowerShell host and is outside the scope of this RFC. But hosts will need to be updated to take advantage of the new opt-in prediction feature. For example PowerShell shell PSReadline should integrate it into its tab completion and/or Ctrl+Space suggestion list display. VSCode should likewise integrate it into its tab completion and intellisense.  

Because of potential lag concerns (especially when the machine learning model is implemented in the cloud), querying prediction results will be an asynchronous operation.  

### Interfaces

```
     public sealed class CommandLinePredictionResult
    {
        /// <summary>
        /// Command line text suggested by the ML model
        /// </summary>
        public string SuggestionText { get; }

        /// <summary>
        /// Value indicating confidence level of prediction (0-100)
        /// </summary>
        public int SuggestionConfidence { get; }

        /// <summary>
        /// Null instance when no prediction is made
        /// </summary>
        public static readonly CommandLinePredictionResult NullInstance = new CommandLinePredictionResult();
    }

    public sealed class CommandLinePredictionStatistics
    {
        // TBD Prediction success statistics
    }

    interface CommandLinePredictor
    {
        /// <summary>
        /// Performs machine learning prediction on provided command line fragment
        /// </summary>
        /// <param name="input">Command line fragment</param>
        /// <param name="cursorIndex">Cursor index</param>
        /// <param name="OnPredictionCallback">Call back delegate to receive prediction results</param>
        void PredictFromInput(string input, int cursorIndex, Action<Collection<CommandLinePredictionResult>> OnPredictionCallback);

        /// <summary>
        /// Submit command line text processed by PowerShell
        /// </summary>
        /// <param name="processedLine">Processed command line</param>
        /// <param name="suggestedLine">Suggested command line</param>
        /// <param name="suggestionChosen">True if the command line was based on a model suggestion chosen by the user</param>
        void ProcessedCommandLine(string processedLine, string suggestedLine, bool suggestionChosen);

        /// <summary>
        /// Returns prediction success statistics
        /// </summary>
        CommandLinePredictionStatistics GetPredictionSuccessStatisticsForCurrentUser();
    }
```

### Feature Configuration

A user can opt in to this feature through the `powershell.config.json` configuration file. The feature is disabled by default.  

```json
{"Microsoft.PowerShell:CommandLineSuggestions":"Disabled"}
```

In addition a user can see the state of this feature and configure, enable or disable it via cmdlets.  

```powershell
Get-CommandLineSuggestions
Set-CommandLineSuggestions
Enable-CommandLineSuggestions
Disable-CommandLineSuggestions
```

### Open Issues

#### General Data Protection Regulation (GDPR) Conformance

Research into this needs to be done in order to ensure conformance. Data stored on the local machine and in the cloud have different protection requirements.  

### Alternate Proposals and Considerations

No alternate proposals.  