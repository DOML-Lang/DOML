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
- [Collection Types](#collection-types)
- [DOML Specifics](#doml-specifics)
  - [Constructors](#constructors)
  - [Assignments](#assignments)
    - [Short Form Set Assignments](#short-form-set-assignments)
    - [Arrays](#arrays)
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

The file ending **must** be `.doml` for *DOML* files, and if a file is purely IR it can either have `.doml` or `.odoml`. Furthermore, they **must** be encoded with standard UTF-8, though parsers **could** support other formats.

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
// A
X : Y() // B
```

The following comment **must** be supported for all *IR* code;

```Assembly
; A
25 59 48 32 ; B
```

### Types

Both *DOML* and *IR* share the same type system, all the following types **must** be implemented.

- Integers (typeID: 0)
  - signed 64 bit (8 bytes) by default
  - 0x prefix to make it hexadecimal, 0b prefix to make it binary, and 0o to make it octal (case insensitive)
  - Can have `_` in numbers. Though they can't however occur at the beginning or ending of a number and you can't have two next to each other.
- Floating Point (typeID: 1)
  - Double Precision 64 bits (8 bytes), by default
  - IEEE 754
  - Can have `_` in numbers. Though they can't however occur at the beginning or ending of a number and you can't have two next to each other and can't occur next to the `.` (if there is one).
- Decimals (typeID: 2)
  - `$` prefix (note the `+` or `-` goes before the `$` prefix i.e. `-$40.95` or `+$59.54`/`$59.54`).
  - Double Precision 64 bit (8 bytes) by default.
  - Decimal typed, C# shows it well [here](https://docs.microsoft.com/en-us/dotnet/api/system.decimal?view=netframework-4.7)
  - Can have `_` in numbers. Though they can't however occur at the beginning or ending of a number and you can't have two next to each other, also have to occur after the `$` and can't occur next to the `.` (if there is one).
- String (typeID: 3)
  - Begins with `"` ends with `"`
  - You can escape quotes only using `\"`
  - You can insert a unicode character like `\u4e00` (insert unicode character 4e00 which is; 一)
- Boolean (typeID: 4)
  - `true` and `false`
  - Thus **can't** have any objects with names that match `true` or `false`.
- Objects (typeID: 5)
  - Represents any creation object for *DOML* and for *IR* refers to a register ID.
  - **Should** be represented as a pointer or similar (i.e. a reference in C#).
  
### Collection Types

Both *DOML* and *IR* have support for collections and thus the following types **must** be implemented.

- Dictionaries/Maps
  - The key type **must** remain constant, and so **must** the value type.
- Arrays
  - Array elements **must** all have the same type.

### DOML Specifics

The following sets of rules apply only to *DOML* code.

#### Style of syntax

All DOML sections below will follow the following style;
```C
// #. <Description>
<Example1>; <Example2>; ...; <ExampleN>;
```
Then below each number will be explained thorougly, the `;` is to indicate a new way to present that example that has the SAME result.

All IR sections below will follow the following style;
```assembly
; #. <Description>
<IR Code> <Parameters>
```
There is typically just one way to do things in IR, so no need to chain multiple to show off the syntatical stuff.  Also a few symbols may pop up on the IR that mean different things;

- `#`: means a register ID (dependent on the previous IR commands before)

#### Constructors

*DOML* has the following syntax for constructors;
```C
// 1. Standard
A : B::(); A : B; A : B::B();

// 2. Using a constructor
A : B::C(n1: p1, n2: p2, n3: p3, ..., nn: pn);
```
1) Creates an object A of type B with standard constructor.
2) Creates an object A of type B with constructor C (with parameters a...n, with labels n1..nn).

Notes:
  - Constructor overloading is NOT supported.
  
##### Resultant IR

The above code generates the following IR;

```assembly
; 1. Standard
newobj B B # 0
; 2. Using a constructor
newobj B C # n p1 p2 p3 ... pn
```
1) Creates an object of type B calling constructor B (aka the default) at register '#' (at runtime register '#' or -1 means any, but in this case it does mean dependent on previous values).  With 0 parameters.
2) Creates an object of type B calling constructor C at register '#' (same reason as above).  'n' parameters, with the parameters following it (note: `...` isn't a parameter it is just to show that there are pn parameters).
  - note: the labels aren't expressed in IR.

#### Assignments

```C
// 1. Accessing and setting an element
A.B = p1, p2, ..., pn
A.{ B = p1, p2, ..., pn }
A : C() { B = p1, p2, ..., pn }

// 2. Performing a single assignment
A.B = p1
A.{ B = p1 }
A : C() { B = p1 }
```
1) Assigns objects p1, p2, ..., pn to B of A.  They all do the same thing in the end (that is if A is always of type D).
  - Note: p1..pn don't have to be of the same type.

##### Resultant IR

```assembly
; 1. Accessing and setting an element
push typeID 1 p1
push typeID 1 p2
...
push typeID 1 pn
calln # C B n

; 2. Performing a single assignment
quickpush typeID 1 p1
quickcall # C B 1
```
1) Pushes 'n' objects onto the stack before collapsing 'n' objects to perform call.
2) Performs a smart push on an object then performs a quick call with just one previous object.

A few notes about this structure;
- The type ID follows the ones stated in the [Types](#types) section, note: this doesn't work for arrays/maps a different command is for that.
- Furthernote: `quickpush`/`quickcall` are efficient ways to perform small calls on objects, they avoid pushing onto the stack and can short circuit any indirection calls to properties/fields by directly editing the value there; often resulting in a speed increase.  Quickpush can be modified by the user to expand its size as to allow larger functions and can also split up calls like `RGB = p1, p2, p3` into a series of 3 quick pushes and quick calls.
- push calls can have multiple parameters the `1` just means a single object, but they all have to have the same type.

#### Arrays
There are two cases where arrays come into form in DOML; the first is arrays of objects and the second is parameter arrays.

The first, arrays of objects;
```C
// 1. Implicit Size
A : []B { 
  ::(){ D = a1, ..., an },
  ::C(p1, ..., pn){ D = a1, ..., an },
  ...
  { D = a1, ..., an },
}

// 2. Explicit Size and assignment
A : [3]B

A[0] : B::() {
  D = a1, ..., an
}
A[1] : B::C(p1, ..., pn) {
  D = a1, ..., an
}
A[2] : B {
  D = a1, ..., an
}
```
1) Assigns an implicitly sized array and assigns each index to a member; you can also assign a length if you want and set just some of the elements.  Elements are typesafe and bounded, so you can't go outside the array or any silliness like that.

2) Assigns an explicit size and the assigns each element follows same rules as above.

##### Resultant IR

Now interestingly enough the IR produced by above actually doesn't create an array, and follows the same as if you defined something like `A1, A2, A3, ... An`.  HOWEVER when referenced it is still treated like an array just the IR has no concept of object arrays (does have concept of array objects however) this means that something like `D = A` where A is defined as something like above, would decompose to `D = [A1, A2, ..., An]` in terms of the IR (below will be how arrays exist in IR).

Now this is done because in reality array objects don't really need to be an array since one it complicates the IR and two its just a nice way to express multiple objects as belonging to a single 'set'.  When referring to each object in IR comments you should reffer to it as `A-x` or `A[x]` with the dash/brackets to signify it is an array.

#### Maps


#### Accessing Values


#### Calls



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
