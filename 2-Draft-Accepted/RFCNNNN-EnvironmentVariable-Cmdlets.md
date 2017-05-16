---
RFC:
Author: Charu Bassi
Status: Draft
SupercededBy: n/a
Version: 0.1
Area: Cmdlet
Comments Due: 5/10/2017
---

# *-EnvironmentVariable Cmdlets

Add the new cmdlets to PowerShell to get\set\create the environment variables for machine\process\user.

Currently users are forced to use the .Net api's to retreive\set the value of environment variables :
```powershell
[System.Environment]::GetEnvironmentVariables()
[System.Environment]::SetEnvironmentVariable("foo","bar")
```

## Motivation

There are many scenarios in which user wants to read\modify\create environment variables. As of now, the only way to achieve this via PowerShell is to use dotnetcore api's. The goal is to provide a set of cmdlets to make user's task easier.
`Unix only supports 'Process' scope.`

## Specification

New cmdlets Get-EnvironmentVariable, Set-EnvironmentVariable, New-EnvironmentVariable, Add-EnvironmentVariable, Remove-EnvironmentVariable.
 
### Get-EnvironmentVariable

```powershell
Get-EnvironmentVariable [[-Name] <string[]>] [-ValueOnly] [-Scope {Process | Machine | User}]  [<CommonParameters>]
```

*	This cmdlet returns an array of EnvironmentVariable objects.
*	By default, the cmdlet returns the environment variables in �Machine� scope.

#### Parameters

1.	Name

	Specify the name or pattern for environment variable.
	
	Type = `array of strings`

    Position = 0

    ##### Attributes

    ValueFromPipeline = true

    ValueFromPipelineByPropertyName = true
	
	ValidateNotNull

2.	ValueOnly

	Type = `switch`
	
	Returns only the value of the environment variable. 

3.	Scope
	
	The default scope is 'Machine'. If specified, the cmdlet tries to set the value of the variable in the specified scope.

	Type = `string`

	##### Attributes

	ValidateSet = `Process`, `Machine`, `User`

#### Output

```powershell 
internal sealed class EnvironmentVariable
    {
        public string Name { get; } = String.Empty;
        public string Value { get; }
        public EnvironmentVariableTarget Scope;
        public EnvironmentVariable() { }
        public EnvironmentVariable(string name, string value) {
            this.Name = name;
            this.Value = value;
            this.Scope = EnvironmentVariableTarget.Process;
        }
        public EnvironmentVariable(string name, string value, EnvironmentVariableTarget scope)
        {
            this.Name = name;
            this.Value = value;
            this.Scope = scope;
        }
    }

`Name` is an environment variable name.

`Value` is value of the environment variable.

`Scope` is the scope in which the environment variable is generated.

```

### Set-EnvironmentVariable

```powershell
Set-EnvironmentVariable [-Name] <string> [[-Value] <string>] [-Force] [-PassThru] [-Scope {Process | Machine | User}] [-WhatIf] [-Confirm]  [<CommonParameters>]
```
*	Sets new value for an existing environment variable. If the variable does not exist, then '-force' will create the variable with the specified value.
*	The default scope for the cmdlet is `Machine`.

#### Parameter 

1.	Name

	Specify the name of the environment variable.

	Type = `string`

    Position = 0

    ##### Attributes

    ValueFromPipeline = true

    ValueFromPipelineByPropertyName = true
	
	ValidateNotNullOrEmpty

2.	Value

	New value for the environment variable.
	
	Type = `string`

    Position = 1

    ##### Attributes

    ValueFromPipeline = true

    ValueFromPipelineByPropertyName = true
	
	ValidateNotNullOrEmpty

3.	Scope

	The default scope is 'Machine'. If specified, the cmdlet tries to set the value of the variable in the specified scope.

	Type = `string`

	##### Attributes

	ValidateSet = `Process`, `Machine`, `User`

4.	Force
	
	If variable does not exist and force is set to true, the variable will be generated.

	Type = `switch`

5.	Passthru

	If set, returns an object of type EnvironmentVariable.

	Type = `switch`

6.	Whatif

	Enabling that switch turns everything previously typed into a test, with the results of what would have happened if the commands were actually run appearing on the screen

	Type = `switch`

7.	Confirm

	Confirm the action with user before making a permanent change.

	Type = `switch`

### New-EnvironmentVariable

```powershell
New-EnvironmentVariable [-Name] <string> [[-Value] <string>] [-Force] [-PassThru] [-Scope {Process | Machine | User}] [-WhatIf] [-Confirm] [<CommonParameters>]
```

*	Creates an environment variable in the specified\default scope with the specified name and value. 
*	The default scope for the cmdlet is 'Machine'.

#### Parameter

1.	Name

	Specify the name of the environment variable. The variable name should not contain wildcard character.

	Type = `string`

    Position = 0

    ##### Attributes

    ValueFromPipeline = true

    ValueFromPipelineByPropertyName = true
	
	ValidateNotNullOrEmpty

2.	Value

	Value for the environment variable.

	Type = `string`

    Position = 1

    ##### Attributes

    ValueFromPipeline = true

    ValueFromPipelineByPropertyName = true
	
	ValidateNotNullOrEmpty

3.	Scope

	The default scope is �Machine�. If specified, the cmdlet tries to set the value of the variable in the specified scope.

	Type = `string`

	##### Attributes

	ValidateSet = `Process`, `Machine`, `User`

4.	Force
	
	If variable does not exist and force is set to true, the variable will be generated.

	Type = `switch`

5.	Passthru

	If set, returns an object of type EnvironmentVariable.

	Type = `switch`

6.	Whatif

	Enabling that switch turns everything previously typed into a test, with the results of what would have happened if the commands were actually run appearing on the screen

	Type = `switch`

7.	Confirm

	Confirm the action with user before making a permanent change.

	Type = `switch`
 
### Add-EnvironmentVariable

```powershell
Add-EnvironmentVariable [-Name] <string> [-Value] <string> [-Force] [-Prepend] [-PassThru] [-Scope {Process | Machine | User}] [-WhatIf] [-Confirm] [<CommonParameters>]
```

*	The cmdlet appends\prepends the new value to the existing value of the environment variable.
*	The default scope for the cmdlet is 'Machine'.

#### Parameter

1.	Name

	Specify the name of the environment variable. The variable name should not contain wildcard character.

	Type = `string`

    Position = 0

    ##### Attributes

    ValueFromPipeline = true

    ValueFromPipelineByPropertyName = true
	
	ValidateNotNullOrEmpty

2.	Value

	Value for the environment variable.

	Type = `string`

    Position = 1

    ##### Attributes

    ValueFromPipeline = true

    ValueFromPipelineByPropertyName = true
	
	ValidateNotNullOrEmpty

3.	Scope

	The default scope is �Machine�. If specified, the cmdlet tries to set the value of the variable in the specified scope.

	Type = `string`

	##### Attributes

	ValidateSet = `Process`, `Machine`, `User`

4.	Force
	
	If variable does not exist and force is set to true, the variable will be generated.

	Type = `switch`

5. 	Prepend

	By default, new value will be appended to the existing value. If Prepend is set, the value will be prepended.

	Type = `switch`

6.	Passthru

	If set, returns an object of type EnvironmentVariable.

	Type = `switch`

7.	Whatif

	Enabling that switch turns everything previously typed into a test, with the results of what would have happened if the commands were actually run appearing on the screen

	Type = `switch`

8.	Confirm

	Confirm the action with user before making a permanent change.

	Type = `switch`

### Remove-EnvironmentVariable

```powershell
Remove-EnvironmentVariable [-Name] <string[]> [-Scope {Process | Machine | User}] [-WhatIf] [-Confirm]  [<CommonParameters>]
```

*	The cmdlet removes the environment variable with the specified name from specified\default scope.
*	The default scope for the cmdlet is 'Machine'.

#### Parameter

1.	Name

	Specify the name of the environment variable. The parameter value can contain wildcard character.

	Type = `string`

    Position = 0

    ##### Attributes

    ValueFromPipeline = true

    ValueFromPipelineByPropertyName = true
	
	ValidateNotNullOrEmpty

2.	Scope

	The default scope is �Machine�. If specified, removes the environment variable from the given scope.

	Type = `string`

	##### Attributes

	ValidateSet = `Process`, `Machine`, `User`

3.	Whatif

	Enabling that switch turns everything previously typed into a test, with the results of what would have happened if the commands were actually run appearing on the screen

	Type = `switch`

4.	Confirm

	Confirm the action with user before making a permanent change.

	Type = `switch`
