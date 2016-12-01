---
RFC: 0005
Author: Joey Fish
Status: Draft
Version: 1.0
Area: Implementing typed IPC.
---

# Typed Interprocess communication

Currently PowerShell does not have an efficient means of IPC, as the architecture [of unstructured/semi-structured text-based systems] itself forces extraneous computation, making long-chains highly inneffient and often error-prone.

## Motivation

As we know from prior experience languages like PHP the usage of text as an interface is inherently error-prone (which is often a key in security-vulnerabilities), even moreso when the implementation of the producer are not available to the consumer thus forcing an *ad hock* reverse-engineered solution.

A solution to this problem could be achieved through the use of a “typed stream” rather than untyped text. Another consideration is if the data passes through several processes/transforms and has to be serialized/deserialized [from text] a lot of unnecessary processing is being forced into the process — while this may be negligible in many cases it definitely adds up when dealing with large amounts of data and/or long chains of processes where the deserialize/serialize must be used.

Note: A more complete motivation is outlined in the blog-post “[Architecture Design Facilitating a Command-line Interface](http://edward.fish/index.php/2016/04/23/architecture-design-facilitating-a-command-line-interface/)”, this is merely the IPC portion.

## Specification

This RFC proposes the following:

0. The 'Text' of a command-line input be separated out from processing/data.
	* This will likely require some modification to how parameters and switches are handled.
	* This ensures that, from the commandlet-side, the actual data is not intermixed w/ switches.
		* The actual separation of data and switches/parameters will be in a forthcoming RFC. (RFC-0006)
0. The underlying type-system be extensible, so that types in one program can be used in another w/o having a common/shared source-file.
	* The system could use ASN.1 — Microsoft already has an OID: [1.2.840.113556](http://oid-info.com/get/1.2.840.113556)
		* I would recommend adding a 5TH subtree: Types (5)
		* Under which the following subtrees would reside:
			0. CLR — Tree containing the base-types of the CLR, could have subtrees for various .NET languages.
			0. Operating System — Tree containing OS-types. (E.G. HWnd.)
			0. Interface — Tree containing types for various external systems.
				* e.g. a type for TCP-connections or other items which are commonly implemented as Integers but not technically integers; or things like “Little-endian, 16-bit value, unsigned” and “Little-endian, 16-bit value, signed”.
			0. User — Defined for user-added types.
				* This could also be [or contain] an “undefined” tree where anything below is system-dependant. While that would defeat the much of the usage of the OID-system, it would allow a sort of “type-registry” for the user’s system. (Not recommended, but a possibility nonetheless.)
0. The system should provide for efficient transmission and be reliable (complete serialize/deserialize roundtrip).
	* ASN.1 has the advantage that proper, unambiguous message-passing (in our case typed information) is efficient.
0. The system should provide for error-checked values.
	* Error-checked values can have intermediate error-checking optimized away.
		* E.G. Given functions A, B, C, and D that take a single Positive parameter and return a Positive result, A(B(C(D( x )))) only needs to check that x is in Positive. (Assuming that none of them raise an exception.)
	* ASN.1 type-definitions can be easily error-checked via constraints.
	* If a general type-registry (e.g. using the OID arcs as outlined above) is used, constraints could be included either explicitly or implicitly (e.g. Interface.Big-endian.32-bit.signed).
	* Microsoft's R & D have had very good results from formal-methods (see “[Safe to the Last Instruction: Automated Verification of a Type-Safe Operating System](https://www.microsoft.com/en-us/research/publication/safe-to-the-last-instruction-automated-verification-of-a-type-safe-operating-system/)”), which use a more generalized view [that of considering properties] to ensure correctness.

## Alternate Proposals and Considerations

Invariably someone will suggest something like JSON as a solution to this problem; there are, however, serious issues with using JSON:

0. JSON does not provide a means to check/enforce constraints, meaning that all clients will have to manually implement the check.
0. JSON does not provide a means to check/enforce a structure, meaning that all clients will have to manually implement the check.
0. Because of #2 JSON is unsuitable for transmitting records ("structs"), because of #1 JSON is unsuitable for transmitting objects (essentially stateful records).
0. Because of #3 JSON is unsuitable for seralizing/deserializing complex/compound types such as are used in .NET.

Viable alternitives would include:

* The Wulf, Lamb, Nestor [Interface Description Language](http://repository.cmu.edu/compsci/2412/); see also [Snodgrass’s book](https://www.amazon.com/Interface-Description-Language-Definition-Principles/dp/0716781980) ISBN 0716781980.
	* Possibly w/ updated syntax to be more in-line with Ada-2012/SPARK-2014; see example #1 below.
* IBM's [SOM](https://en.wikipedia.org/wiki/IBM_System_Object_Model)
* COM/DCOM — This is not recommended, but there already exists an OID tree for [UUIDs](http://www.oid-info.com/get/2.25).

Altering the underlying architecture for IPC is a big change, and likely to break compatibility; however the benefits of the change – efficiency of transmitting values, ensuring that values are correct, and increased reliability – are quite desirable in a system.

--------

Example 1:

    Type Id_String is new String;
    
    -- SSN format: ###-##-####
    Subtype Social_Security_Number is ID_String(1..11)
      with Dynamic_Predicate =>
        (for all Index in Social_Security_Number'Range =>
          (case Index is
           when 4|7 => Social_Security_Number(Index) = '-',
           when others => Social_Security_Number(Index) in '0'..'9'
          )
         );
    
    -- EIN format: ##-#######
    Subtype EIN is ID_String(1..10)
      with Dynamic_Predicate =>
        (for all Index in EIN'Range =>
          (case Index is
           when 3 => EIN(Index) = '-',
           when others => EIN(Index) in '0'..'9'
          )
         );
    
    -- Tax_ID: A string guarenteed to be an SSN or EIN.
    -- SSN (###-##-####)
    -- EIN (##-#######)
    Subtype Tax_ID is ID_String
      with Dynamic_Predicate =>
          (Tax_ID in Social_Security_Number) or
          (Tax_ID in EIN);

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
