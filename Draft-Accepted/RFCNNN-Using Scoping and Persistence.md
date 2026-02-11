---
RFC: 
Author: Ryan Strope
Status: Draft
SupercededBy: 
Version: 1.0
Area: using statements
Comments Due: 4/1/2023
Plan to implement: Yes
---

# Add options for scoping and persistence in using statements

- Powershell currently does not persist namespace prefixes properly.

  - Subsequent calls to `using namespace <namespace>` blocks clobber previous `using namespace <namespace>` blocks.  

- `using assembly <assembly name>` has a very limited syntax.

## Motivation

    Assembly, and Namespace importing is an important tool in a programmer's toolbox. 

    Being able to predefine namespace prefixes, and when needed the assemblies helps to:

- Write Cleaner code.

- Minimize possible errors from misspelled types.

- Centralizes most assembly and namespace references to a single point in the file (the top).

- Clearly reference classes in ambiguous scenarios.

    - e.g. `[MessageBox]` (`[System.Windows.MessageBox]`, or `[System.Windows.Forms.MessageBox]`)    

## User Experience

- In a script file define something simple like

  - 
        using namespace System.Management.Automation.Language
        [ExpressionAst]

- In the shell execute

  - `using namespace System.Collections.Generic`

  - `[List[int]]`

  - `. .\Script.ps1`

  - `[List[int]]`


- The first call to `[List[int]]` will output

  - 
        IsPublic IsSerial Name BaseType                                         
        -------- -------- ----   --------
        True     True     List`1 System.Object

- The script will output

  - 
        IsPublic IsSerial Name          BaseType
        -------- -------- ----          --------
        True     False    ExpressionAst System.Managme...

- The second call to `[List[int]]` will output

  - 
        InvalidOperation: Unable to find type [List].

    - Windows PowerShell Error is more verbose than PowerShell

**NOTE:** The same happens if the script file's lines are entered manually in the shell

### __That one line from that Weezer song about Sweaters__  
  
    No one can possibly know every namespace prefixes one might need in a particular shell session it should be easier to append a new namespace prefix to the existing ones instead of having to literally redefine (and more importantly have to remember) all the previously defined namespace prefixes.  

This thread unwravels exponentionally as you get into more complicated scenarios

- Modules

- Modules utilize other modules

- Manual execution of code during debugging

- etc

## Specification

- First any ___new___ calls to `using namespace <Namespace Prefix>` within a scope should append to the existing scope and go away ONLY when that scope is gone.

- Second the syntax of the `using namespace` statement should be expanded for forced persistence.

  - Implemented the same hashtable option that `using module` allows for example

    - 
        ```
        using namespace @{
            Prefix = 'System.Collections.Generic'
            Scope = Module | Global | Script    
        }
        ```

- Third the syntax for the `using assembly` statement should also be expanded similarly

  - Implement a hashtable that can emulate the full or most of the options for `Add-Type`

    - 
        ```
        using assembly @{
            Name = 'PresentationFramework'
            Path = 'c:\...'
            Namespace = 'MyNamespace.WPF'
            UsingNamespaces = @(
                'System.Windows'
                'System.Windows.Controls'
            )
        }


## Alternate Proposals and Considerations

    Right now the only sure fire way to ensure a type is always found is to fully qualify each type in the code.

Even if you redefine ALL `using namespace` blocks in every file in a module they can be clobbered by calls to other modules, script files, or just manual shell use that define new `using namespace` blocks