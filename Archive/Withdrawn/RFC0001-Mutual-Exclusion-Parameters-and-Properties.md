---
RFC: 0001
Author: Steve Lee
Status: Withdrawn
Area: Parameters
Comments Due: 1/31/2016
---

# Mutually Exclusive Parameters and Properties

ParameterSets were introduced in PowerShell v2.  This allowed definition of
different sets of parameters for advanced functions.  For example, the
Invoke-Command cmdlet can take a PSSession object that encapsulates all
the necessary information to successfully create a remote connection,
however, it also allows the users to specify each connection parameter
seperately which may be useful for interactive one-time operations.  This
example is for illustrative purposes and doesn't represent the actual
cmdlet:

```PowerShell
	function Invoke-Command {
		param (
			[Parameter(ParameterSetName="ComputerName", Position=0)]
			[string] $ComputerName,
			
			[Parameter(ParameterSetName="ComputerName")]
			[PSCredential] $Credential,
			
			[Parameter(ParameterSetName="ComputerName")]
			[Int32] $Port,
			
			[Parameter(ParameterSetName="PSSession", Position=0)]
			[PSSession] $PSSession
		)
	}
```

Through the use of ParameterSets, the user can know through help or
intellisense which additional parameters are appropriate and needed.

In PowerShell v5, we added the ability to author classes in PowerShell
script.  In addition, classes provided a much simpler development experience
to author DSC (Desired State Configuration) resources.  In the DSC case, they
also have use cases where you would set certain property values in a 
configuration document, but the properties are mutually exclusive.

Although mutual exclusion can be handled with ParameterSets (which could
be extended for class properties), it can be quite complex to define and can 
generate many resulting ParameterSets that can be confusing to the end user.

## Motivation

As the cmdlet or DSC resource author, I can simply annotate my code to 
define mutually exclusive parameters for advanced functions and properties
for classes so that the end user can easily understand which parameters and
properties are mutually exclusive at parse time rather than run time.

## Specification

The proposal is to use an attribute to describe parameters and properties
that are mutually exclusive.  This makes it simple to author and also 
makes it simple to read and understand.

### [MutuallyExclusive()] attribute on the function/class definition

```PowerShell
	# Prop1 and Prop3 are mutually exclusive
	# Prop2 and Prop4 are mutually exclusive
	# Prop2 and Prop5 are mutually exclusive
	[MutuallyExclusive("Prop1", "Prop3")]
	[MutuallyExclusive("Prop2", "Prop4")]
	[MutuallyExclusive("Prop2", "Prop5")]
	class foo {
		[string] $Prop1
		
		[string] $Prop2
		
		[string] $Prop3
		
		[string] $Prop4
		
		[string] $Prop5
	}	
```

Pros: easier to understand the resulting mutually exclusive sets

Cons: introduces new semantics

## Alternate Proposals and Considerations

### [ExclusionSet()] attribute on each parameter/property

```PowerShell
	class foo {
		# Prop1 and Prop3 are mutually exclusive
		# Prop2 and Prop4 are mutually exclusive
		# Prop2 and Prop5 are mutually exclusive
		[ExclusionSet("A")]
		[string] $Prop1
		
		[ExclusionSet("B","C")]
		[string] $Prop2
		
		[ExclusionSet("A")]
		[string] $Prop3
		
		[ExclusionSet("B")]
		[string] $Prop4
		
		[ExclusionSet("C")]
		[string] $Prop5
	}
```

Pros: consistent with ParameterSet annotation being on the parameters

Cons: harder to determine the resulting mutually exclusive sets after authoring

