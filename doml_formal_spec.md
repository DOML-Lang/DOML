## What is this?
This is the formal specification document for DOML, mainly aimed at developers who wish to implement a parser.

## Terminology
The following are key words and their definitions;
- **must**/**need** : For a parser to be clarified as 'mostly compliant' with this specification document, it must/needs to implement this 'feature'
- **should** : For a parser to be clarified as 'completely compliant' with this specification document, it should implement this 'feature'
- **could**/**may** : A parser could support these features but they are not relevant to be 'compliant' with this specification document, there are many additional options they could add and I won't be even try to list them all.
- *text* : Refers to any unicode readable printable character including language characters, and 'emojis' (though a parser **could** warn against the use of using 'emojis') and underscore.
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

## File Format
The file ending **must** be `.doml` for *DOML* files and `.odoml` for the IR (similarity to `.o` files).  Furthermore they **should** be encoded with standard UTF-8, though other parsers **could** support other formats.

## Other Details
- *DOML*
  - **Must** be case sensitive
  - **Must** be whitespace insensitive
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
/* A block commment that can be nested! */
```
The following comment **must** be supported for all *IR* code;
```Assembly
; A line comment that terminates at the end of the line
```

### Types
Both DOML and IR share the same type system, all the following types **must** be implemented.
- Integers
  - signed 32 bit (4 bytes) by default
  - 16 bit (2 bytes) and 64 bit (8 bytes) variants through the S/L postfixes (case insensitive)
    - Standing for Short and Long
  - By adding a U postfix (case insensitive) you can get unsigned integers
- Floating Point
  - Single Precision (IEEE 754) by default
  - You can make it Double Precision (again IEEE 754) by adding a D postfix (case insensitive)
- String
  - Begins with `"` ends with `"`
  - You can escape quotes only using `\"`
- Character
  - Begins with `\`` ends with `\``
  - You can escape quotes only using `\\\``
- Boolean
  - `true` and `false`
- Objects
  - Represents any creation object for DOML and for ODOML refers to a register ID.
  
### DOML Specifics
#### Creation Assignments
Creation assignments are one of t

#### Set Assignments

#### Embed IR
Embedding IR **should** be supported by a parser to allow for more complex operations to occur that aren't supported by the grammar (and in most cases will eventually become supported).

The syntax for embedding IR is;
```c
IR {
  push32i   59    ; pushes a 32 bit integer (59) onto the stack
  pusho     2     ; push object from register 2 onto the stack
  set Color.Hex   ; runs the Color.Hex command
}
```

### IR Specifics
#### Whitespace
- Whitespace is important in IR
- Atleast one space has to exist between the IR command and the parameter.
- Each line can only contain one IR command and a parameter (i.e. there must be a newline between each 'Instruction')
- Comments can only appear between 'Instructions' and at the top/bottom, i.e. they can't exist after the IR command and before the parameter.

#### Architecture
The Architecture of IR has been standidised just to maintain consistency.
- All pushing/popping operations should operate on a stack based model
  - It **shouldn't** be dynamically allocated since the `makespace` command should be akin to a malloc.
  - You **must** expose both the current size of the stack, and the max size.
    - The current size represents how many elements have currently been pushed
    - The max size represents the total space available for elements (equal to `makespace` parameter)
  - It **should** *ONLY* allow pushing/popping/peeking operations (though note: that a peek can be implemented through popping and then pushing that value as well as returning it, though this is inefficient and a more native method should be implemented)
  - You **could** implement it using either a `stack` or a static array with pointer/index
- `pusho` and all register object operations operate on a register based model
  - It **shouldn't** be dynamically allocated since the `makereg` command should be akin to a malloc.
  - You **must** expose the current size of the registers which represents the total size available for objects (equal to `makereg` parameter)
  - It **should** *ONLY* allow indexing operations (both to set and to 'unset' - or set to null)
- All the commands **should** be implemented through a byte value (which is an unsigned 8 bit integer) and **could** be placed into an enum.
