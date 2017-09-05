## What is this?
This is the formal specification document for DOML, mainly aimed at developers who wish to implement a parser.

## Terminology
The following are key words and their definitions;
- **must**/**need** : For a parser to be clarified as 'mostly compliant' with this specification document, it must/needs to implement this 'feature'
- **should** : For a parser to be clarified as 'completely compliant' with this specification document, it should implement this 'feature'
- **could**/**may** : A parser could support these features but they are not relevant to be 'compliant' with this specification document, there are many additional options they could add and I won't even try to list them all.
- *text* : Refers to any Unicode readable printable character including language characters, and 'emojis' (though a parser **could** warn against the use of using 'emojis') and underscore.
- *IR* : Refers to the bytecode produced through the 'compilation' of the DOML source file (stands for Intermediate Representation).
- *DOML* : Refers to Data Oriented Markup Language (this 'language')

## Table of Contents
- [File Format](#file-format)
- [Other Details](#other-details)
- [Syntax](#syntax)
- [Comments](#comments)
- [Types](#types)
- [DOML Specifics](#doml-specifics)
  - [Creation Assignments](#creation-assignments)
  - [Set Assignments](#set-assignments)
  - [Embed IR](#embed-ir)
- [IR Specifics](#ir-specifics)
  - [Whitespace](#whitespace)
  - [Architecture](#architecture)
  - [Required Commands](#required-commands)

## File Format
The file ending **must** be `.doml` for *DOML* files and `.odoml` for the IR (similarity to `.o` files).  Furthermore, they **should** be encoded with standard UTF-8, though other parsers **could** support other formats.

## Other Details
- *DOML*
  - **Must** be case sensitive
  - **Must** be white space insensitive
- *IR*
  - **Must** be case sensitive
  - **Must** be whitespace sensitive
    - More on this under syntax

## Syntax
The following details relate to the syntax.

### Comments
The following comments **must** be supported for all *DOML* code;
```C
// A line comment that terminates at the end of the line
/* A block comment that can be nested! */
```
The following comment **must** be supported for all *IR* code;
```Assembly
; A line comment that terminates at the end of the line
```

### Types
Both DOML and IR share the same type system, all the following types **must** be implemented.
- Integers
  - signed 32 bit (4 bytes) by default
  - 8 bit (1 byte), 16 bit (2 bytes) and 64 bit (8 bytes) variants through the B/S/L postfixes (case insensitive)
    - Standing for Byte, Short, and Long
  - By adding a U postfix (case insensitive) you can get unsigned integers (note you can add both a u postfix and the above size postfixes, in any order but they have to go after all digits)
  - 0x prefix to make it hexadecimal, 0b prefix to make it binary, and 0o to make it octal (case insensitive)
- Floating Point
  - Single Precision 32 bits (IEEE 754) by default
  - 16 bit (1 byte), 64 bit (2 bytes), 80 bit (8 bytes), and 128 bit (16 bytes) variants through the FH/FD/FE/FQ postfixes (case insensitive)
    - Standing for Half, Double, Extended, and Quad
- Decimals
  - Decimal typed, C# shows it well [here](https://docs.microsoft.com/en-us/dotnet/api/system.decimal?view=netframework-4.7)
  - Single Precision 32 bit by default
  - 64 bit (8 bytes), 128 bit (16 bytes) variants through the D/DQ postfixes (case insensitive)
    - Standing for Double, and Quad
- String
  - Begins with `"` ends with `"`
  - You can escape quotes only using `\"`
- Character
  - Begins with `'` ends with `'`
  - You can escape quotes only using `\'`
- Boolean
  - `true` and `false`
- Objects
  - Represents any creation object for DOML and for ODOML refers to a register ID.
  
### DOML Specifics
#### Creation Assignments
Creation assignments are one of the two assignments you can perform using DOML.  They **must** be supported by the parser.  The syntax is as follows;
```C
@ MyColor = System.Color
```
The `@` signifies that it is a creation assignment (you are creating a new variable), the identifier `MyColor` can be used below that line to refer to the object created, `System.Color` is the function to call which is expected to push an object onto the stack.

Every creation assignment should form the following IR output;
```assembly
new Library.Object ; i.e. System.Color
regobj index       ; This is the index to place it in the registers should start at 0 and for every object increase by one
```

#### Set Assignments
Set assignments are the other assignment you can perform using DOML.  They **must** be supported by the parser.
The syntax is as follows;
```C
; MyColor.Color(HexAndName) = 0xFF4080, "Name"
```
The `;` signifies that it is a setting assignment (you are setting a variable of a function), the identifier `MyColor` refers to the object you are invoking `Color(HexAndName)` on (which itself represents the function your invoking) and both `0xFF4080` and `"Name"` are the parameters you're passing to the function.  The `,` is to separate parameters.

> Furthermore I want to point out that normally you would separate `Hex` and `Name` to two different set assignments but in this case, it just highlighted multiple parameters.

`MyColor.Color(HexAndName)` becomes a `pusho` for the register that `MyColor` exists in and a set function on `Color.HexAndName`, note that the `( ... )` became `.` the reason why is that a list of aliases exist for `.` which are; `<`, `>`, `-`, `.`, `(`, `)`, `[`, `]`, `{`, `}` so `Color(HexAndName)` is really `Color.HexAndName.` which is trimmed.

Every set assignment should form the following IR output;
- First, every parameter is pushed
```assembly
push32i 16728192
pushstr "Name"
; And so on
```
The following commands are available for using;
- `push8i`/`push16i`/`push32i`/`push64i` all push signed integer values of bit length 8/16/32/64 (or byte length 1/2/4/8)
- `push8u`/`push16u`/`push32u`/`push64u` all push unsigned integer values of bit length 8/16/32/64 (or byte length 1/2/4/8)
- `pushobj` pushes the object at the register ID equal to the parameter
- `push16f`/`push32f`/`push64f`/`push80f`/`push128f` all push floating point values of bit length 16/32/64/80/128 (or byte length 2/4/8/10/16
  - IEEE754 format
- `push32d`/`push64d`/`push128d` all push decimal values of bit length 32/64/128 (or byte length 4/8/16)
  - Decimal Precision
- `pushstr`/`pushchar` pushes a string or character
  - Unicode Support
- `pushbool` pushes a boolean value
- `push` pushes whatever it can 'see' (since you can't define the difference between floating points/decimals and between both signed/unsigned integers it'll push the default; that being 32 bit for unsigned/signed integers, floating points and 64 bits for decimals).  For booleans, strings, and characters it'll work properly and as long as you aren't 'emitting' the bytecode then you **could** use it also since the actual DOML representation has prefixes and if you store the values in an `object` (like C# Object) then it'll retain that information.

The following commands from the above list **need** to be implemented (implemented defined as being native types such as `Int32` in C#, all types **need** to have some implementation but it can be through something lie up-casting i.e. treating a `push8i` as the same as `push16i` but just checking the ranges);
- `push16i`/`push32i`/`push64i`/`push16u`/`push32u`/`push64u`
- `push32f`/`push64f`
- `push64d`/`push128d`
- `pushstr`/`pushchar`
- `pushbool`
- `pushobj`
- Note: if you can't implement one of these solutions due to the lack of native support and not wanting to involve external libraries then a note in the readme will suffice for you to implement a solution akin to upcasting or similar.

The following **should** some kind of implementation but it doesn't have to be native;
- `push32d`/`push16f`/`push80f`/`push8i`/`push8u`/`push128f`
- Note: If for some reason you can't implement the solution without an external library then you should at-least provide DLL imports (or whatever) for a library that adds this functionality then either comment it out (or use `#if`) so that a user could easily enable the functionality.  Furthermore since most users will want the code to work as a DLL / similar it would be a nice solution to use ways that don't mean you have to recompile it (either use `#if` with something like C# or use managed C++ perhaps or whatever), again it's up to you and the above types *only* **should** be implemented to meet all the specs.
- More types **could** be introduced though you should refrain from adding extra IR commands since those reduce your compatibility

Regardless after all values are pushed then the object is pushed and the set function is called.
```assembly
pushobj index      ; i.e. pushobj 0
set functionToCall ; i.e. set Color.Hex
```
The index **should** refer to the order of colours produced but in actual fact it doesn't matter and this is why when you register an object you supply an index this just means that how you manage the indexes internally is less important since the code produced will work anywhere thanks to the requirement of supplying an index to `regobj`.  Furthermore you really **should** order your objects from 0 upwards, incrementing by one each time.

#### Embedding IR
Embedding IR **should** be supported by a parser to allow for more complex operations to occur that aren't supported by the grammar (and in most cases will eventually become supported).

An example for embedding IR is;
```c
@ MyColor = System.Color
@ IRBlock = System.IRBlock ...
;         .push32i  = 59
;         .pusho    = 0
;         .set      = "Color.Hex"
```
This will become the following, you **could** always simplify it and just translate the instructions one by one.
```assembly
new     System.Color
regobj  0
new     System.IRBlock
regobj  1
push32i 59
pushobj 1
set     push32i
push32i 0
pushobj 1
set     pusho
pushstr "Color.Hex"
pushobj 1
set     set
```
Note: System is a standardised library but its spec is located under [here](system_formal_spec.md).

### IR Specifics
#### Whitespace
- Whitespace is important in IR
- At least one space has to exist between the IR command and the parameter.
- Each line can only contain one IR command and a parameter (i.e. there must be a newline between each 'Instruction')
- Comments can only appear between 'Instructions' and at the top/bottom, i.e. they can't exist after the IR command and before the parameter.

#### Architecture
The Architecture of IR has been standardized just to maintain consistency.
- All pushing/popping operations should operate on a stack based model
  - It **shouldn't** be dynamically allocated since the `makespace` command should be akin to a malloc.
  - You **must** expose both the current size of the stack and the max size.
    - The current size represents how many elements have currently been pushed
    - The max size represents the total space available for elements (equal to `makespace` parameter)
  - It **should** *ONLY* allow pushing/popping/peeking operations (though note : hat a peek can be implemented through popping and then push that value as well as return it, though this is inefficient and a more native method should be implemented)
  - You **could** implement it using either a `stack` or a static array with pointer/index
- `pushoj` and ll register object operations operate on a register based model
  - It **shouldn't** be dynamically allocated since the `makereg` command should be akin to a malloc.
  - You **must** expose the current size of the registers which represents the total size available for objects (equal to `makereg` parameter)
  - It **should** *ONLY* allow indexing operations (both to set and to 'unset' - or set to null)
- All the commands **should** be implemented through a byte value (which is an unsigned 8-bit integer) and **could** be placed into an enum.

#### Required Commands
The following are all the required commands, as stated previously you **could** add more but should refrain from it since that lends itself to incompatibility.  All parsers **need** to support the following.
> Note: All the pushing commands are detailed in depth under the [set](#set-assignments) section so I won't list them here
- `nop` does explicitly nothing
  - IR: `nop <any>` i.e. `nop false` (Note: the `false` **should** exist in outputted IR but any value should be able to exist)
- `comment` same as nop but when emitted will emit the comment (aka it maintains user comments)
  - IR: `; <User Comment>` i.e. `; Create a new color`
- `panic` panics (produces an error) if top value matches the parameter supplied
  - IR: `panic <any>` i.e. `panic true` or `panic 100`
- `makespace` reserves space in stack
  - The parameter represents the new size not the difference
  - Objects aren't carried across so effectively a wipe
  - IR: `makespace <int32>` i.e. `makespace 2`
- `makereg` reserves space in object registers
  - The parameter represents the new size not the difference
  - Objects aren't carried across so effectively a wipe
  - IR: `makereg <int32>` i.e. `makereg 2`
- `checkspace` checks if space equals the sizeof a getter
  - The parameter represents a function
  - A parser **could** just do this during parsing and avoid a checkspace call
  - Though they should still provide the lines but just comment them out
  - IR: `checkspace <Root.Creation::SetFunction>` i.e. `checkspace System.Color::RGB`
- `set` runs the set function
  - **could** be maintained on a single 'map' with a prefix 'set' (with either a space or a '\_') and with another prefix representing the objects initial creation state (i.e. `System.Color`) 
    - the functionality of having the same name for set/get/new and for different types **needs** to exist.
  - IR: `set <Root.Creation::SetFunction>` i.e. `set System.Color::Color.Hex`
- `copy` copies top value x times
  - IR: `copy <int32>` i.e. `copy 4` (which copies top value 4 times)
- `clear` clears all values
  - Deprecated
- `clearreg` clears all registers
  - Deprecated
- `regobj` registers top object to index given
  - performs a pop then registers that object to index given
  - IR: `regobj <int32>` i.e. `regobj 2` (registers top object to register 2)
- `unregobj` unregisters object at index
  - set it to 'null' basically
  - IR: `unregobj <int32>` i.e. `unregobj 2` (registers top object to register 2)
- `call` performs a function call on the parameter
  - **could** be maintained on a single 'map' with a prefix 'get' (with either a space or a '\_') and with another prefix representing the objects initial creation state (i.e. `System.Color`) 
