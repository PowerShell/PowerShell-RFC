---
RFC: 0006
Author: Joey Fish
Status: Draft
Version: 1.0
Area: Command-line parsing and Piping.
---

# Universal parsing and Piping of the Command-Line

The blog-post “[Architecture Design Facilitating a Command-line Interface](http://edward.fish/index.php/2016/04/23/architecture-design-facilitating-a-command-line-interface/)” sets forth an architectural guideline for making a robust and efficient command-line interpreter/system. RFC-0005 dealt with the IPC portion – the ‘piping’ proper, if you will – and this RFC deals with issues not strictly (though tangentially) related to the IPC — much of what was suggested for the interface proper is already present in PowerShell’s [Parsing](https://technet.microsoft.com/en-us/library/hh847892.aspx) and [Parameters](https://technet.microsoft.com/en-us/library/hh847824.aspx) functionalities.

IIUC, what is not is the separation of pass-through data and [usually user-provided] parameters.

## Motivation

Separating data from parameters could lead to a cleaner overall system — imagine, if you will, a function for adding parity:

    Type Integer_Vector is Array(Positive range <>) of Integer;
    
    -- This function appends a 0 or 1 to the given vector as appropriate.
    Function Add_Parity(Input : Integer_Vector; Even_Parity : Boolean) return Integer_Vector is
        Add_Even : Boolean := Even_Parity;
    Begin
        For Item of Input loop
            declare
                Even : Constant Boolean := Item mod 2 = 0;
            begin
                Add_Even:= Add_Even = Even_Parity and Even;
            end;
        end loop;
        
        Return Input & (if Add_Even then 0 else 1);
    End Add_Parity;

Conceptually this is the same as a parselet, where Even_Parity is an actual formal parameter (switch) supplied by the user and Input is the data to be operated on — separating these into an “options-stream” and a “data-stream” would present a uniform API for internal tools; this is conceptually making the ‘parameters’ of the process the tuple (options, data) where both are typed-streams.

## Specification

Internally, a process should have two input streams (data, options) where the switches/options are placed into the options-stream by the parser; both the data-stream and options-stream should be name-associative so that internal processes are able to access the proper data by name. (Just as the handling of Parameters currently is.)

Output should be at least one stream (data-stream), a secondary options-stream could be present – but such would essentially force the data-stream to be a name-associative value-list of name-associative values, which are passed through the function/process (this is not recommended because it would expose other functions [and their parameters] in the call-chain to the function/process).

It is recommended that the command-line parser parses all options along the call-chain and hands them to the executor via a list of (commandlet/process, options) tuples which are fed into the process’s options-stream upon process-creation. — This ensures that every process is unaware of other processes in the call-chain and prevents them from altering either the options or their own behavior based on options to other commandlet/processes.

## Alternate Proposals and Considerations

This proposal is essentially ‘polish’ on the current system which, honestly speaking, contains 90% of what “Architecture Design Facilitating a Command-line Interface” suggests for the process-interface. Most programmers will find the current system to be ‘good enough’ and therefore will provide little impetus towards changing the system as most of the benefits of the change will be enjoyed by RFC-0005 and its increased throughput-efficiency.

---------------
## PowerShell Committee Decision

### Voting Results

Jason Shirk: Reject 

Joey Aiello: Reject

Bruce Payette: Reject

Steve Lee: Reject

Hemant Mahawar: Reject

### Majority Decision

Commmittee agrees that this RFC does not provide sufficient details to move forward with an implementation.  PowerShell already supports IPC within PSRP (PowerShell Remoting Protocol).

### Minority Decision

N/A
