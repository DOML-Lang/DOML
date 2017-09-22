## What is this?
This is the formal specification document for *DOML*, mainly aimed at developers who wish to implement a parser.

## Terminology
The following are key words and their definitions;
- **must**/**need** : For a parser to be clarified as 'mostly compliant' with this specification document, it must/needs to implement this 'feature'
- **should** : For a parser to be clarified as 'completely compliant' with this specification document, it should implement this 'feature'
- **could**/**may** : A parser could support these features but they are not relevant to be 'compliant' with this specification document, there are many additional options they could add and I won't even try to list them all.
- *text* : Refers to any Unicode readable printable character including language characters, and 'emojis' (though a parser **could** warn against the use of using 'emojis') and underscore.
- *IR* : Refers to the bytecode produced through the 'compilation' of the *DOML* source file (stands for Intermediate Representation).
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
- [Interfacing DOML](#interfacing-doml)
  - [Static vs Reflection Bindings](#static-vs-reflection-bindings)
  - [Constructors](#constructors)
  - [Setters](#setters)
  - [Getters](#getters)
  - [Sizeof](#sizeof)
- [Binary Format](#binary-format)

## File Format
The file ending **must** be `.doml` for *DOML* files and `.odoml` for the *IR* (similarity to `.o` files).  Furthermore, they **should** be encoded with standard UTF-8, though other parsers **could** support other formats.

## Other Details
- *DOML*
  - **Must** be case sensitive
  - **Must** be white space insensitive
- *IR*
  - **Must** be case sensitive
  - **Must** be whitespace sensitive

## Syntax
The following details relate to the syntax.

### Comments
The following comments **must** be supported for all *DOML* code;  Further more block comments can occur anywhere as shown below.
```C
// A line comment that terminates at the end of the line
/* A block comment that can be nested! */
/* 0 */ @ /* 1 */ Example /* 2 */ = /* 3 */ System.Color /* 4 */ ... /* 5 */
/* 6 */ ; /* 7 */ .RGB(Normalised) /* 8 */ = /* 9 */ 212 /* 10 */,/* 11 */ 255 /* 12 */, /* 13 */ 150 /* 14 */,
```
> However, when they are preserved they should be preserved in 'order'.  I.e. 0, 1, 2, and 3 will occur before the `new` instruction.  4, 5, 6, 7, 8, 9 will occur before any of the pushes or `set` call and after the `new` call.  And 10 and 11 will occur after the first push but before the second (i.e. after the push of 212 but before 255); 12 and 13 will occur after push of 255 but before push of 150 and 14 will occur after push of 150 but before call.

The following comment **must** be supported for all *IR* code;  Since no block comments exist there is no debate about where they can occur.
```Assembly
; A line comment that terminates at the end of the line
```

### Types
Both *DOML* and *IR* share the same type system, all the following types **must** be implemented.
- Integers
  - signed 64 bit (8 bytes) by default
  - 0x prefix to make it hexadecimal, 0b prefix to make it binary, and 0o to make it octal (case insensitive)
  - Can have `_` in numbers.  Though they can't however occur at the beginning or ending of a number and you can't have two next to each other.
- Floating Point
  - Double Precision 64 bits (8 bytes), by default
  - IEEE 754
  - Can have `_` in numbers.  Though they can't however occur at the beginning or ending of a number and you can't have two next to each other and can't occur next to the `.` (if there is one).
- Decimals
  - `$` prefix (note the `+` or `-` goes before the `$` prefix i.e. `-$40.95` or `+$59.54`/`$59.54`).
  - Double Precision 64 bit (8 bytes) by default.
  - Decimal typed, C# shows it well [here](https://docs.microsoft.com/en-us/dotnet/api/system.decimal?view=netframework-4.7)
  - Can have `_` in numbers.  Though they can't however occur at the beginning or ending of a number and you can't have two next to each other, also have to occur after the `$` and can't occur next to the `.` (if there is one).
- String
  - Begins with `"` ends with `"`
  - You can escape quotes only using `\"`
  - You can insert a unicode character like `\u459\` (insert unicode character 459)
- Character
  - Begins with `'` ends with `'`
  - You can escape quotes only using `\'`
  - You can insert a unicode character like `\u459\` (insert unicode character 459)
- Boolean
  - `true` and `false`
  - Can't have any objects with names that match `true` or `false`.
- Objects
  - Represents any creation object for *DOML* and for *IR* refers to a register ID.
  
### DOML Specifics
#### Creation Assignments
Creation assignments are one of the two assignments you can perform using *DOML*.  They **must** be supported by the parser.  The syntax is as follows;
```C
@ MyColor = System.Color
```
The `@` signifies that it is a creation assignment (you are creating a new variable), the identifier `MyColor` can be used below that line to refer to the object created, `System.Color` is the function to call which is expected to push an object onto the stack.

Every creation assignment should form the following *IR* output;
```assembly
new Library.Object ; i.e. System.Color
regobj index       ; This is the index to place it in the registers should start at 0 and for every object increase by one
```

#### Set Assignments
Set assignments are the other assignment you can perform using *DOML*.  They **must** be supported by the parser.
The syntax is as follows;
```C
; MyColor.Color(HexAndName) = 0xFF4080, "Name"
```
The `;` signifies that it is a setting assignment (you are setting a variable of a function), the identifier `MyColor` refers to the object you are invoking `Color(HexAndName)` on (which itself represents the function your invoking) and both `0xFF4080` and `"Name"` are the parameters you're passing to the function.  The `,` is to separate parameters.

> Furthermore I want to point out that normally you would separate `Hex` and `Name` to two different set assignments but in this case, it just highlighted multiple parameters.

`MyColor.Color(HexAndName)` becomes a `pusho` for the register that `MyColor` exists in and a set function on `Color.HexAndName`, note that the `( ... )` became `.` the reason why is that a list of aliases exist for `.` which are; `<`, `>`, `-`, `.`, `(`, `)`, `[`, `]`, `{`, `}` so `Color(HexAndName)` is really `Color.HexAndName.` which is trimmed.

Every set assignment should form the following *IR* output;
- First, every parameter is pushed
```assembly
pushint 16728192
pushstr "Name"
; And so on
```
The following commands are available for using;
- `pushint` pushes integer value of 64 bit (8 bytes)
- `pushobj` pushes the object at the register ID equal to the parameter
- `pushnum` pushes floating point value of 64 bit (8 bytes)
  - IEEE754 format
- `pushdec` pushes decimal value of 64 bit (8 bytes)
  - Decimal Precision
- `pushstr`/`pushchar` pushes a string or character
  - Unicode Support
- `pushbool` pushes a boolean value
- `push` pushes whatever it can 'see', this means that it'll push integer if passed a number without a '.', and push floating point if passed a number with '.', else if it is quoted it'll push character/string (depending if its single/double) if its 'true/false' it'll push bool.  So it'll never push object or decimal.

Regardless after all values are pushed then the object is pushed and the set function is called.
```assembly
pushobj index      ; i.e. pushobj 0
set functionToCall ; i.e. set Color.Hex
```
The index **should** refer to the order of colours produced but in actual fact it doesn't matter and this is why when you register an object you supply an index this just means that how you manage the indexes internally is less important since the code produced will work anywhere thanks to the requirement of supplying an index to `regobj`.  Furthermore you really **should** order your objects from 0 upwards, incrementing by one each time.

#### Embedding IR
Embedding *IR* **should** be supported by a parser to allow for more complex operations to occur that aren't supported by the grammar (and in most cases will eventually become supported).
> I'm not particularly happy with any suggestion I've seen so far/thought of so this will remain empty till one is approved.

### IR Specifics
#### Whitespace
- Whitespace is important in *IR*
- At least one space has to exist between the *IR* command and the parameter.
- Each line can only contain one *IR* command and a parameter (i.e. there must be a newline between each 'Instruction')
- Comments can only appear between 'Instructions' and at the top/bottom, i.e. they can't exist after the *IR* command and before the parameter.

#### Architecture
The Architecture of *IR* has been standardized just to maintain consistency.
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
- `nop` does explicitly nothing
  - *IR*: `nop <any>` i.e. `nop false` (Note: the `false` **should** exist in outputted IR but any value should be able to exist)
- `comment` same as nop but when emitted will emit the comment (aka it maintains user comments)
  - *IR*: `; <User Comment>` i.e. `; Create a new color`
- `panic` panics (produces an error) if top value matches the parameter supplied
  - *IR*: `panic <any>` i.e. `panic true` or `panic 100`
- `makespace` reserves space in stack
  - The parameter represents the new size not the difference
  - Objects aren't carried across so effectively a wipe
  - *IR*: `makespace <long>` i.e. `makespace 2`
- `makereg` reserves space in object registers
  - The parameter represents the new size not the difference
  - Objects aren't carried across so effectively a wipe
  - *IR*: `makereg <long>` i.e. `makereg 2`
- `set` runs the set function
  - **could** be maintained on a single 'map' with a prefix 'set' (with either a space or a '\_') and with another prefix representing the objects initial creation state (i.e. `System.Color`) 
    - the functionality of having the same name for set/get/new and for different types **needs** to exist.
  - *IR*: `set <Root.Creation::SetFunction>` i.e. `set System.Color::RGB.Hex`
- `copy` copies top value x times
  - *IR*: `copy <long>` i.e. `copy 4` (which copies top value 4 times)
- `regobj` registers top object to index given
  - performs a pop then registers that object to index given
  - *IR*: `regobj <long>` i.e. `regobj 2` (registers top object to register 2)
- `unregobj` unregisters object at index
  - set it to 'null' basically
  - *IR*: `unregobj <long>` i.e. `unregobj 2` (registers top object to register 2)
- `pushobj` pushes object from register onto stack
  - *IR*: `pushobj <long>` i.e. `pushobj 0` (pushes object from register 0)
- `pushint` pushes integer onto stack
  - *IR*: `pushint <long>` i.e. `pushint 59`
- `pushnum` pushes floating point onto stack
  - *IR*: `pushnum <double>` i.e. `pushnum 32.59`
- `pushdec` pushes decimal onto stack
  - *IR*: `pushdec <decimal>` i.e. `pushdec 59.56`
- `pushstr` pushes string onto stack
  - *IR*: `pushstr <string>` i.e. `pushstr "Bob"`
- `pushchar` pushes character onto stack
  - *IR*: `pushchar <character>` i.e. `pushchar 'b'`
- `pushbool` pushes boolean onto stack
  - *IR*: `pushbool <bool>` i.e. `pushbool true`
- `push` pushes the default of type
  - If no decimal point then integer, if decimal point then floating point, if true/false then bool, if single quote then character, if double quote then string.
    - Therefore won't push decimal/object
  - *IR*: `push <bool/long/double/string/character>` i.e. `push true`
- `call` performs a function call on the parameter
  - **could** be maintained on a single 'map' with a prefix 'get' (with either a space or a '\_') and with another prefix representing the objects initial creation state (i.e. `System.Color::`)
  - **should** also have a sizeof parameter that refers to how many parameters it has (easily to implement with reflection else just get assigner to state).
  - *IR*: `call <Root.Creation::GetFunction>` i.e. `call System.Color::RGB.Hex`
- `new` creates a new object
  - **could** be maintained on a single 'map' with a prefix 'new' (with either a space or a '\_')
  - **should** only ever push one value else it is breaking 'new' convention.
  - *IR*: `new <Root.Creation>` i.e. `new System.Color`
- `pop` pops x values off the stack
  - Useful for when making sure stack has space.
  - *IR*: `pop <long>` i.e. `pop 3` (pops the top 3 objects off the stack)
- `compmax` check if parameter matches the maximum stack size
  - Pushes true if max stack size is less than the parameter value else false
  - *IR*: `compmax <long>` i.e. `compmax 9`
- `compsize`
  - Pushes true if current stack size is less than the parameter value else false
  - *IR*: `compsize <long>` i.e. `compsize 3`
- `compreg`
  - Pushes true if register size is less than the parameter value else false
  - *IR*: `compreg <long>` i.e. `compreg 10`

## Interfacing with DOML
There are a few ways to interface with *DOML*, each one **must** exist in either a static or reflected binding (offering both could allow users to choose one that is more applicable to them). 

#### Static vs Reflection Bindings
Both are valid, and both have their advantages (static often requires code generation but is fast whereas reflection is dynamic, doesn't require code generation, requires little to no input from 'users' but is often 20x slower).

#### Constructors
A constructor call is expected to push a single object, the one the user wanted.  It could be a 'new' one or could refer to an already created instance.  This means that objects require an empty constructor to then later be initialised.  This is utilised in the `new` opcode.

#### Setters
A set call is expected to push *no* objects, and pull one object for every expected parameter and *one* object for the object to call the set on.  A type is passed through the `...::` (i.e. in `System.Color::` the type is `System.Color`).  This is utilised in the `set` opcode.

#### Getters
A getter is expected to push objects, and pull only *one* object (for the object to get from). A type is passed through the `...::` (i.e. in `System.Color::` the type is `System.Color`).  This is utilised in the `get` opcode.

#### Sizeof
Both getters and setters have a sizeof operation (could be a string to int map for example) that refers to the *maximum* amount of objects they push or pull respectively.  This should be handled at compile time making sure that setters get the correct amount of parameters and that there is enough space for getters.
- A value of > 0 refers to an exact amount of parameters
- 0 refers to no parameters
- < 0 refers to atleast x parameter/s but no limit (i.e. -1 refers to atleast 1 parameter, -10 refers to atleast 10 parameters)

## Binary Format
*DOML* is also very useful for sending data to systems like robotics or other smaller 'computer systems' that rely on a small stack to make them cheaper and more efficient.  This could also be used in applications like fitness watches or the like to send data.  No parser is required to support this since no parser is required to support parsing bytecode.

#### Main Variant
The stream looks something like this (*note: for simplicity I've not included a footer and a header which you may or may not want to include but it often is case by case*);
`| OP_CODE (1 Byte) | LENGTH_IN_BYTES (1 Byte) | DATA (n Byte/s) |`
Now while have a length in bytes if we can already figure out how long the data is?  Well simply put it means we can make data shorter and more compact in a lot of scenarios for example if your sending a signed integer < 128 then we can just send it as a single byte of data with the 1 byte opcode and 1 byte length, thus resulting in a 3 byte message instead of a 9 byte message if we sent it as a 64 bit integer without the length.  This does allow lengths up to 255 bytes or 2,040 bits for strings and comments which should suffice (though I don't see how one couldn't extend this length to two bytes if required), further more this even supports sending more complex messages over more packages.
