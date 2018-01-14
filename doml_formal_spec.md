# DOML Formal Specification

## What is this

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
    - [Short Form Set Assignments](#short-form-set-assignments)
    - [Arrays / Dictionaries](#arraysdictionaries)
  - [Embed IR](#embed-ir)
- [IR Specifics](#ir-specifics)
  - [Whitespace](#whitespace)
  - [Architecture](#architecture)
  - [Required Commands](#required-commands)
- [Interfacing DOML](#interfacing-doml)
  - [Static vs Reflection Bindings](#static-vs-reflection-bindings)
  - [Constructors](#constructors)
    - [Parameters](#parameters)
  - [Setters](#setters)
  - [Getters](#getters)
  - [Sizeof](#sizeof)
- [Binary Format](#binary-format)
  - [Main Variant](#main-variant)
  - [Native Variant](#native-variant)
  - [Other Variants](#other-variants)

## File Format

The file ending **must** be `.doml` for *DOML* files and `.odoml` for the *IR* (similarity to `.o` files). Furthermore, they **should** be encoded with standard UTF-8, though parsers **could** support other formats.

## Other Details

- *DOML*
  - **Must** be case sensitive
  - **Must** be white space insensitive
- *IR*
  - **Must** be case sensitive
  - **Must** be whitespace sensitive
  - **Should** use 2 digit numbers for their opcodes (i.e. 09 instead of 9) since the count right now is 18 (which has an opcode value of 17) and it won't go over 99 in the foreseeable future. This just makes alignment look nice.

## Syntax

The following details relate to the syntax.

### Comments

The following comments **must** be supported for all *DOML* code;

```C
// A line comment that terminates at the end of the line
/* A block comment that can be nested! */
/* 0 */ @  Example = System.Color ... /* 1 */
/* 2 */ .RGB->Normalised = 212, 255, 150 /* 3 */
```

> As shown you can have block comments before and after any line but not in any line, the numbers represent order of preservation.

The following comment **must** be supported for all *IR* code;

```Assembly
; Line Comment with no content before
03 59 ; A line comment that terminates at the end of the line
```

### Types

Both *DOML* and *IR* share the same type system, all the following types **must** be implemented.

- Integers
  - signed 64 bit (8 bytes) by default
  - 0x prefix to make it hexadecimal, 0b prefix to make it binary, and 0o to make it octal (case insensitive)
  - Can have `_` in numbers. Though they can't however occur at the beginning or ending of a number and you can't have two next to each other.
- Floating Point
  - Double Precision 64 bits (8 bytes), by default
  - IEEE 754
  - Can have `_` in numbers. Though they can't however occur at the beginning or ending of a number and you can't have two next to each other and can't occur next to the `.` (if there is one).
- Decimals
  - `$` prefix (note the `+` or `-` goes before the `$` prefix i.e. `-$40.95` or `+$59.54`/`$59.54`).
  - Double Precision 64 bit (8 bytes) by default.
  - Decimal typed, C# shows it well [here](https://docs.microsoft.com/en-us/dotnet/api/system.decimal?view=netframework-4.7)
  - Can have `_` in numbers. Though they can't however occur at the beginning or ending of a number and you can't have two next to each other, also have to occur after the `$` and can't occur next to the `.` (if there is one).
- String
  - Begins with `"` ends with `"`
  - You can escape quotes only using `\"`
  - You can insert a unicode character like `\u459\` (insert unicode character 459)
- Boolean
  - `true` and `false`
  - Can't have any objects with names that match `true` or `false`.
- Objects
  - Represents any creation object for *DOML* and for *IR* refers to a register ID.
  - Should be represented as a pointer or similar (i.e. a reference in C#)

### DOML Specifics

#### Creation Assignments

Creation assignments are one of the two assignments you can perform using *DOML*. They **must** be supported by the parser. The syntax is as follows;

```C
@ MyColor = System.Color
```

The `@` signifies that it is a creation assignment (you are creating a new variable), the identifier `MyColor` can be used below that line to refer to the object created, `System.Color` is the function to call which is expected to push an object onto the stack.

Every creation assignment should form the following *IR* output;

```assembly
06 Library.Object  ; i.e. System.Color
07 index           ; This is the index to place it in the registers should start at 0 and for every object increase by one
```

#### Set Assignments

Set assignments are the other assignment you can perform using *DOML*. They **must** be supported by the parser.
The syntax is as follows;

```C
; MyColor.Color->HexAndName = 0xFF4080, "Name"
```

The `;` signifies that it is a setting assignment (you are setting a variable of a function), the identifier `MyColor` refers to the object you are invoking `Color->HexAndName` on (which itself represents the function your invoking) and both `0xFF4080` and `"Name"` are the parameters you're passing to the function. The `,` is to separate parameters.

> Furthermore I want to point out that normally you would separate `Hex` and `Name` to two different set assignments but in this case, it just highlighted multiple parameters.

`MyColor.Color->HexAndName` becomes a `pushobj` for the register that `MyColor` exists in and a set function on `Color.HexAndName`.

Every set assignment should form the following *IR* output;

- First, every parameter is pushed

```assembly
12 16728192 ; Converted to integer from hex value 0xFF4080
15 "Name"
; And so on
```

The following commands are available for using;

- `pushobj` (11) pushes the object at the register ID equal to the parameter
- `pushint` (12) pushes integer value of 64 bit (8 bytes)
- `pushnum` (13) pushes floating point value of 64 bit (8 bytes)
  - IEEE754 format
- `pushdec` (14) pushes decimal value of 64 bit (8 bytes)
  - Decimal Precision
- `pushstr` (15) pushes a string
  - Unicode Support
- `pushbool` (16) pushes a boolean value
- `push` (17) pushes whatever it can 'see', this means that it'll push integer if passed a number without a '.', and push floating point if passed a number with '.', else if it is quoted it'll push string (depending if its single/double) if its 'true/false' it'll push bool. So it'll never push object or decimal.
- `pushvec` (18) pushes an empty static array onto the stack (type is determinent of first type) the parameter corresponds to how large the array is.
  - This also changes all push commands to now rather index the array starting at 0 and working up to the length.
- `pushmap` (19) pushes an empty hash map/dictionary onto the stack, the parameter corresponds to the capacity of the map/dictionary.
  - This also changes all push commands to now rather first create a key, then secondly set it (effectively will pop twice in a row till reach capacity).

Regardless after all values are pushed then the object is pushed and the set function is called.

```assembly
11 index      ; i.e. pushobj 0
04 Library.Object::functionToCall ; i.e. set System.Color::RGB.Hex
```

The index **should** refer to the order of colours produced but in actual fact it doesn't matter and this is why when you register an object you supply an index this just means that how you manage the indexes internally is less important since the code produced will work anywhere thanks to the requirement of supplying an index to `regobj`. Furthermore you really **should** order your objects from 0 upwards, incrementing by one each time.

##### Short Form Set Assignments

This is mainly just in-place to effectively 'register' objects created.  The following;

```C
@ Register = System.System ...
           .X = Y
```

Can be shortened down to `@ System->X = Y`, `System.System` can be anything though it has to be in format `X.X`, the system will only ever create one register and just re-use it across all instances of this so don't worry about it being inefficient i.e. the following;

```C
@ Register = System.System ...
           .A = B
           .C = D
           .E = F
```

Produces the exact same IR code as;

```C
@ System->A = B
@ System->C = D
@ System->E = F
```

Due to the reusing of system, though the standard is to keep these statements short and not overuse it.

#### Arrays/Dictionaries

Since they are more complicated then other things I'll include in a bit more depth how they are done, they are done in such a way to balance speed/memory and simplicity. Only two more IR commands were added with the standard push commands becoming index commands when you have effectively 'activated' either a vector/map mode (by calling the command), the arrays/dictionaries themselves should not be created till the full type signature is known (since the static arrays are singular type and so are the dictionaries - this is to keep inline with the majority of coding languages), though of course they could be created before if the types aren't needed (i.e. python).

Effectively the following code; `Test.RGB = [255, 125, 245]` (noting that this variant requires an array rather then 3 separate values, and also note that an array =/= a list of parameters i.e. `[255, 125, 245]` =/= `255, 125, 245`).  The code would generate the following IR;

```Assembly
18    3     ; Create a static array with length 3

; Push the 3 integers (which acts as index operations rather than pushes)
12    255
12    125
12    245

11    0                 ; Push the first object onto the stack
04    System.Color::RGB ; Set call
```

Key note: that effectively the system would recognise that its now utilising arrays and would change the functionality, this is to keep it simple.   It also wouldn't require any stack space for the three integers (a big penalty to the other methods) and wouldn't require the pushing and popping to create the array at the end (another big penalty to other methods).

Dictionaries are similar but the following code; `Test.BigNumbers = [ 255 : true, 1 : false, 3939 : true ]` would produce the following IR (separated since it can be confusing at first);

```Assembly
19    3     ; Create a dictionary with length 3

12    255   ; Push the first keypair
16    true

12    1     ; Push the second keypair
16    false

12    3939  ; Push the third keypair
16    true

11    0                 ; Push the first object onto the stack
04    System.Color::RGB ; Set call
```

Decides types after first two pushes, then will continue the pattern (length * 2, due to requiring both key and value).

#### Embedding IR

Embedding *IR* **should** be supported by a parser to allow for more complex operations to occur that aren't supported by the grammar (and in most cases will eventually become supported).
> I'm not particularly happy with any suggestion I've seen so far/thought of so this will remain empty till one is approved.

Currently my favourite thought for embedding IR is;

```C
@ Example = System.Color ...
          .RGB = 4, 55, 255
// `.RGB = 4, 55, 255` is equivalent to:
{
  12 4, 12 55, 12 255
  11 0
  04 System.Color::RGB
}
```

### IR Specifics

#### Whitespace

- Whitespace is important in *IR*
- At least one space has to exist between the *IR* command and the parameter.
- Each line can only contain one *IR* command and a parameter (i.e. there must be a newline between each 'Instruction')
  - You can put multiple *IR* commands on the same line using `,`
- Comments can only appear between 'Instructions' and at the top/bottom, i.e. they can't exist after the *IR* command and before the parameter.

#### Architecture

The Architecture of *IR* has been standardized just to maintain consistency.

- All pushing/popping operations should operate on a stack based model
  - It **shouldn't** be dynamically allocated since the `makespace` command should be akin to a malloc (i.e. allocating an array).
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

The following are all the required commands, as stated previously you **could** add more but should refrain from it since that lends itself to incompatibility. All parsers **need** to support the following.
> For the sake of readability in the examples I'll include the name of the command rather than the opcode value, some parsers **could** support using the name vs the opcode value, but the official way is the opcode value (included next to the command name).

- `nop` (00) does explicitly nothing
  - *IR*: `nop <any>` i.e. `nop false` (Note: the `false` **should** exist in outputted IR but any value should be able to exist)
- `comment` (01) same as nop but when emitted will emit the comment (aka it maintains user comments)
  - *IR*: `; <User Comment>` i.e. `; Create a new color`
- `makespace` (02) reserves space in stack
  - The parameter represents the new size not the difference
  - Objects aren't carried across so effectively a wipe
  - *IR*: `makespace <long>` i.e. `makespace 2`
- `makereg` (03) reserves space in object registers
  - The parameter represents the new size not the difference
  - Objects aren't carried across so effectively a wipe
  - *IR*: `makereg <long>` i.e. `makereg 2`
- `set` (04) runs the set function
  - **could** be maintained on a single 'map' with a prefix 'set' (with either a space or a '\_') and with another prefix representing the objects initial creation state (i.e. `System.Color`)
    - the functionality of having the same name for set/get/new and for different types **needs** to exist.
  - **should** also have a sizeof parameter that refers to how many parameters it pops.
  - *IR*: `set <Root.Creation::SetFunction>` i.e. `set System.Color::RGB.Hex`
- `call` (05) performs a function call on the parameter
  - **could** be maintained on a single 'map' with a prefix 'get' (with either a space or a '\_') and with another prefix representing the objects initial creation state (i.e. `System.Color::`)
  - **should** also have a sizeof parameter that refers to how many parameters it pushes.
  - *IR*: `call <Root.Creation::GetFunction>` i.e. `call System.Color::RGB.Hex`
- `new` (06) creates a new object
  - **could** be maintained on a single 'map' with a prefix 'new' (with either a space or a '\_')
  - **should** only ever push one value else it is breaking 'new' convention.
  - *IR*: `new <Root.Creation>` i.e. `new System.Color`
- `regobj` (07) registers top object to index given
  - performs a pop then registers that object to index given
  - *IR*: `regobj <long>` i.e. `regobj 2` (registers top object to register 2)
- `unregobj` (08) unregisters object at index
  - set it to 'null' basically
  - *IR*: `unregobj <long>` i.e. `unregobj 2` (registers top object to register 2)
- `copy` (09) copies top object
  - The parameter represents how many times to copy top object
  - Effectively a peek + (push x parameter)
  - *IR*: `copy <long>` i.e. `copy 2`
- `pop` (10) pops x values off the stack
  - Useful for when making sure stack has space.
  - *IR*: `pop <long>` i.e. `pop 3` (pops the top 3 objects off the stack)
- `pushobj` (11) pushes object from register onto stack
  - *IR*: `pushobj <long>` i.e. `pushobj 0` (pushes object from register 0)
- `pushint` (12) pushes integer onto stack
  - *IR*: `pushint <long>` i.e. `pushint 59`
- `pushnum` (13) pushes floating point onto stack
  - *IR*: `pushnum <double>` i.e. `pushnum 32.59`
- `pushdec` (14) pushes decimal onto stack
  - *IR*: `pushdec <decimal>` i.e. `pushdec 59.56`
- `pushstr` (15) pushes string onto stack
  - *IR*: `pushstr <string>` i.e. `pushstr "Bob"`
- `pushbool` (16) pushes boolean onto stack
  - *IR*: `pushbool <bool>` i.e. `pushbool true`
- `push` (17) pushes the default of type
  - If no decimal point then integer, if decimal point then floating point, if true/false then bool, if double quote then string.
    - Therefore won't push decimal/object
  - *IR*: `push <bool/long/double/string>` i.e. `push true`
- `pushvec` (18) pushes vector onto stack (static array)
  - *Note:* changes push commands to index commands from 0 to length of vector.
    - Decides type after first 'push' therefore a length of 0 is invalid (can't determine type)!
      - i.e. `pushvec 0` is invalid and not an appropriate value.
  - *IR*: `pushvec <long>` i.e. `pushvec 10`
- `pushmap` (19) pushes map onto stack (dictionary)
  - *Note:* changes push commands to index commands up to the count of the map.
    - Though it will have to take two push commands at a time, one for the key; other for the value
      - Decides type after first two 'push's therefore a length of 0 is invalid (can't determine type)!
        - i.e. `pushmap 0` is invalid and not an appropriate value and will return an error if in safe mode and do un-defined behaviour in unsafe (most likely nothing as a map counter will probably be implemented).
  - *IR*: `pushmap <long>` i.e. `pushmap 10`

## Interfacing with DOML

There are a few ways to interface with *DOML*, each one **must** exist in either a static or reflected binding (offering both could allow users to choose one that is more applicable to them).

### Static vs Reflection Bindings

Both are valid, and both have their advantages (static often requires code generation but is fast whereas reflection is dynamic, doesn't require code generation, requires little to no input from 'users' but is often quite a significant amount slower).

### Constructors

A constructor call is expected to push a single object, the one the user wanted. It could be a 'new' one or could refer to an already created instance. This means that objects require an empty constructor to then later be initialised. This is utilised in the `new` opcode.

#### Parameters

Parameters can be passed through to a constructor through the following way;

```C
@ Test = System.Color (255, 125, 243) ...
// Which is equivalent to
@ Test = System.Color ...
       .ctor = 255, 125, 243

// Further more
@ Test = System.Color->Hex (1, 0.05, 0.39) ...
// Is equivalent to
@ Test = System.Color ...
       .ctor->Hex = 1, 0.05, 0.39
```

The `.ctor` has to be the first call if used else it can be part of the creation syntax (some may just prefer putting it on its own line). You can use the `->` operator to distinguish between the various constructors, since it won't automatically distinguish (you can choose to have one default one that doesn't require a `->` **only**).  The `(...)` syntax purely exists for those more used to a traditional style and is just a compiler 'trick'. The parameters are pushed before the constructor call, to be used during the constructor. The constructor should be stored as `ctor` and `ctorHex` in static/reflected bindings if done (or user side) rather than storing the function names as traditional constructors (i.e. instead of `public Color(int R, int G, int B);` you would have `public ctor(int R, int G, int B);`) no naming conflicts should arise since all fields/properties and even functions are named as `GetX` with X being the function name.

### Setters

A set call is expected to push *no* objects, and pull one object for every expected parameter and *one* object for the object to call the set on. A type is passed through the `...::` (i.e. in `System.Color::` the type is `System.Color`). This is utilised in the `set` opcode.

### Getters

A getter is expected to push objects, and pull only *one* object (for the object to get from). A type is passed through the `...::` (i.e. in `System.Color::` the type is `System.Color`). This is utilised in the `get` opcode.

### Sizeof

Both getters and setters have a sizeof operation (could be a string to int map for example) that refers to the *maximum* amount of objects they push or pull respectively. This should be handled at compile time making sure that setters get the correct amount of parameters and that there is enough space for getters.

- A value of > 0 refers to an exact amount of parameters
- 0 refers to no parameters
- < 0 refers to atleast x parameter/s but no limit (i.e. -1 refers to atleast 1 parameter, -10 refers to atleast 10 parameters)

## Binary Format

No parsers are required to support any of these variants they **could** support them but they are often only useful for embedded systems and short range communication as well as over the web (though that is still quite a large range of uses but its still up to the author).

*DOML* is also very useful for sending data to systems like robotics or other smaller 'computer systems' that rely on a small stack to make them cheaper and more efficient. This could also be used in applications like fitness watches or the like to send data. No parser is required to support this since no parser is required to support parsing bytecode.

Effectively the pro/con list is as such;

- Memory Efficient: Main Variant
  - Uses significantly less memory then the other methods
  - For example since numbers are generally small (and often take up less than a byte or two bytes) having a single field can save you 2/3 bytes in memory resulting well in just 1/2 saved bytes per number, this is even larger for floating point values and decimals. Though it does lose one byte per boolean.
- Simplicity: Native Variant, the removal of a length field for all but strings makes parsing it significantly easier since it doesn't have to pass the length field (which could result in a speed up), also simplifies the data sending procedure.
- Extremely Large Data: 7 bit length field with 1 bit decider (demonstrated in other variants)

### Main Variant

The stream looks something like this (*note: for simplicity I've not included a footer and a header which you may or may not want to include but it often is case by case*);
`| OP_CODE (1 Byte) | LENGTH_IN_BYTES (1 Byte) | DATA (n Bytes) |`
Now while have a length in bytes if we can already figure out how long the data is?  Well simply put it means we can make data shorter and more compact in a lot of scenarios for example if your sending a signed integer < 128 then we can just send it as a single byte of data with the 1 byte opcode and 1 byte length, thus resulting in a 3 byte message instead of a 9 byte message if we sent it as a 64 bit integer without the length. This does allow lengths up to 255 bytes or 2,040 bits for strings and comments which should suffice (though I don't see how one couldn't extend this length to two bytes if required), further more this even supports sending more complex messages over more packages.

Do note: that strings can be quite long and its suggested to replace the `LENGTH_IN_BYTES_` field with a 8 bit sequence (where the 8th bit determines if the sequence continues for another 8 bits, and you append the 7 bits to the last 7 if it does), however for the other types this is unnecessary and is only a **could** include due to the fact that having an integer/bool/floating point/decimal with more than 255 bytes is quite insane and an incredibly niche case whereas having a string of more than 255 bytes is quite possible as 255 bytes is only 127.5 characters (which is in fact only a sentence or two).

### Native Variant

Basically just doesn't use the length field (except for strings where it would use the 8 bit decider length as explained above), and figures out the length per opcode, this requires more complicated parsing but can result in more efficient packing (though it is often more memory efficient to use the main variant as you could compact bytes more).

### Other Variants

- You could have a 7 bit length field with the last 8th bit determining if the length continues (which would be appended to the total 14 bit byte sequence, though of course this could continue again and again).
