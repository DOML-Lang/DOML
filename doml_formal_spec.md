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
  - [Arrays](#arrays-of-objects)
  - [Maps](#maps-of-objects)
  - [Calls](#calls)
  - [Accessing Values](#accessing-values)
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
  
Throughout this document I'll use the word equivalent for IR, word equivalents don't 'have' to be supported and just the opcode variants have to, but it is much harder to show complex IR without the word values.

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
push 0 50 ; B
```

### Types

Both *DOML* and *IR* share the same type system, all the following types **must** be implemented.

- Integers (type ID: `0`, type name: `Integer`)
  - signed 64 bit (8 bytes) by default
  - 0x prefix to make it hexadecimal, 0b prefix to make it binary, and 0o to make it octal (case insensitive)
  - Can have `_` in numbers. Though they can't however occur at the beginning or ending of a number and you can't have two next to each other.
- Floating Point (type ID: `1`, type name: `Float`)
  - Double Precision 64 bits (8 bytes), by default
  - IEEE 754
  - Can have `_` in numbers. Though they can't however occur at the beginning or ending of a number and you can't have two next to each other and can't occur next to the `.` (if there is one).
- Decimals (type ID: `2`, type name: `Decimal`)
  - `$` prefix (note the `+` or `-` goes before the `$` prefix i.e. `-$40.95` or `+$59.54`/`$59.54`).
  - Double Precision 64 bit (8 bytes) by default.
  - Decimal typed, C# shows it well [here](https://docs.microsoft.com/en-us/dotnet/api/system.decimal?view=netframework-4.7)
  - Can have `_` in numbers. Though they can't however occur at the beginning or ending of a number and you can't have two next to each other, also have to occur after the `$` and can't occur next to the `.` (if there is one).
- String (type ID: `3`, type name: `String`)
  - Begins with `"` ends with `"`
  - You can escape quotes only using `\"`
  - You can insert a unicode character like `\u4e00` (insert unicode character 4e00 which is; 一)
- Boolean (type ID: `4`, type name: `Bool`)
  - `true` and `false`
  - Thus **can't** have any objects with names that match `true` or `false`.
- Objects (type ID: `5`, type name: `Object`)
  - Represents any creation object for *DOML* and for *IR* refers to a register ID.
  - **Should** be represented as a pointer or similar (i.e. a reference in C#).
  
### Collection Types

Both *DOML* and *IR* have support for collections and thus the following types **must** be implemented.

- Arrays (collection ID: `0`, collection name: `array`)
  - Array elements **must** all have the same type.
- Dictionaries/Maps (collection ID: `1`, collection name: `map`)
  - The key type **must** remain constant, and so **must** the value type.

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
; ... Pushes
newobj B C # n
```
1) Creates an object of type B calling constructor B (aka the default) at register '#' (at runtime register '#' or -1 means any, but in this case it does mean dependent on previous values).  With 0 parameters.
2) Creates an object of type B calling constructor C at register '#' (same reason as above).  'n' parameters.
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

#### Arrays of Objects
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

#### Object Arrays
You can create an object array by declaring it as such;
```C
// 1. Declaring implicit size
A.B = [o1, ..., on]
// 2. Declaring an explicit size
A.B : [n]Type {
  o1, ..., on
}
A.B : [n]Type
A.B[0] = o1
...
A.B[n] = on
```

Both declares an array of objects of the same type, this **should** implemented as a static array but in languages like Python which don't understand the concept of static arrays implementing it as a dynamic is still compliant.  However, going outside the explicit bounds or implicit ones (n in this case) **must** not be supported.

Note: they should decompose to the same code however optimisations may be performed converting the first option to a single push call that can be cached till you push the array saving a set of copies, or that you can perform array like copies that are more efficient then typical ways.  The second one could be optimised.  Optimisations don't interact with compliance so you are free to optimise calls as long as the result is the same.

You can edit elements of an array like;
```C
A.B[0] = 2
A.B[1] = A.B[2]
```

##### Resultant IR

This is a little more complicated then other IR due to the level of optimisation possible SO I'll outline the basic structure, and in the IR section I'll cover some of the optimisations possible;

```assembly
pusharray typeID n
setarray typeID 0 o1
...
setarray typeID n on
```

You can perform a wide 'setarray' by doing a 'arraycpy' like, which is significantly more efficient;
```assembly
pusharray typeID n
arraycpy typeID n o1 o2 ... on
```

Note: if you want array of arrays that are n dimensional and square then you should either use the `setarraynth` command (like `setarraynth typeID x y val`) or the `arraynthcpy` command (like `arraynthcpy typeID N x1 ... xn y1 ... yn z1 ... zn` where N is the row count * height count).  You can also use the compacting method that is (this is required for arrays of maps);
```assembly
arraycpy typeID n o1 o2 ... on
arraycpy typeID n p1 p2 ... pn
... ; 'N' times
compact typeID N
```
Compact takes type and number of arguments, it then pops those arguments from the stack and places those into the array.  This is the way to build jagged arrays.

Accessing arrays are discussed under accessing values.

#### Maps of objects
You can define maps of objects like;
```C
A : [B : C] {
  {
    b : C::E(k1, ..., kn) {
      X = x1, ..., xn
    }
  },
  ...
  // And so on
}

// Accessing elements
P : C = A[b]
```

This is extremely simular to how arrays of objects works so I won't redescribe, it should produce NO IR and purely should be for the user to simplify their code.  Note: the key can only a literal type (excluding objects) so like a string, int, float and so on (using the type names specified in the type section).

#### Map Objects
You can define map objects in code like;
```C
// Implicit
A.B = {
  {
    c1 : D::E(p1, ..., pn) {
      F = f1, ..., fn
    },
    ...,
    c2 : D::G(k1, ..., kn) {
      F = f2_1, ..., f2_n
    },
  }
}

// Explicit
A.B : [C : D]
```
Note: you can combine the explicit and implicit into one.

##### Resultant IR
Maps have somewhat similar IR to arrays.

```assembly
pushmap typeID typeID
setmap typeID typeID key value
```
Setting multiple values can be shorted to
```assembly
pushmap typeID typeID
quicksetmap typeID typeID n k1 v1 ... kn vn
```

typeID can only contain the types stated in the type section.  If you want a map of arrays (and same for map of maps) then you have to do the following;

For map of maps that are one dimensional
```assembly
pushmap typeID typeID
quicksetmap typeID typeID n k1 v1 ... kn vn
push typeID new_key1
pushmap typeID typeID
push typeID new_key2
quicksetmap typeID typeID n k2_1 v2_1 ... k2_n v2_n
... ; 'N' times
zipmap 1 typeID typeID typeID N
```
The above takes a collection type along with the new key type and the old map types that is you can view it like `zipmap 1 [typeID : [typeID : typeID]] N`, note how I had to push elements after each one, if you don't want to do that then you can create a complex map.

Complex maps are actually quite hard to express so there are two commands, the first is through `createtype` to express your map type then later on you can refer to that type by its ID you provide.

That is to create a string map of a int to array of floats map that is `[string : [int : []float]]` you would do;
```assembly
createtype 2 1 3 1 1 0 1 0 0 1
; Or in word mode
createtype 2 3 map str map int array float
```
That is you create a type and call it ID `2` with depth `3` and that it is a map (1) then you state its a `string` (id 3) and its collection type is map (1) then you state the key value is int (0) and the value is array (0) then you state the type is float (1).  So yeh... it is quite complex to define them but once they are defined you can just use them like any other collection type i.e.;
```assembly
pushcollection 2
setcollection 2 string_value int_value n float_0 ... float_n
```
where `n` is the number of items in the array.  As you can see the type system is quite sophisticated, once more try not to abuse collections, and if you can completely avoid them that is preferred.  This is more down to the user then you but still trying to expand the uses of zip map to more types if you can is important.  Doing maps of maps just gets icky and for most users they will just define simple map structures, though maps of arrays are also a bit icky so in the future having a nicer command for them could exist.

#### Accessing Values


#### Calls

> NOTE: do this please, allow function calls to have args

#### Embedding IR

Embedding *IR* **should** be supported by a parser to allow for more complex operations to occur that aren't supported by the grammar (and in most cases will eventually become supported).
> I'm not particularly happy with any suggestion I've seen so far/thought of so this will remain empty till one is approved.

Currently my favourite thought for embedding IR is;

```C
Example : Color {
  RGB = 4, 55, 255
}
// `RGB = 4, 55, 255` is equivalent to:
#IR simple {
  #IR_obj Color @Color;
  #IR_ctor Color @Color::Color;
  #IR_set RGB @Color.RGB;
  newobj Color Color #Example;
  push int 4, 55, 255;
  call #Example Color RGB; '0' as in register '0'
}
```
Noting the use of `simple` which allows me to use words instead of integer values (as well as using `#` for registers allowing alphanumeric) for almost everything, making the code look very readable.  This is covered in the [attributes](#attributes) section.

#### Attributes

You can apply a few attributes to your DOML code to provide various different benefits;
- `#Version X.Y.Z` you can state your version like `#Version 5` or `#Version 5.1` or even `#Version 5.5.9` compilers are required to check this and make sure that your version will work with the current compiler this will help people upgrade their DOML code if any compatability stuff breaks.  Lowest version allowed is `#Version 0.3` as before that the language was different (in really just terms of syntax and complexity of IR) and very few compilers ever fully supported `0.2`.  Compilers **must** support this.
- `#IR { ... }` allows you to embed IR, an optional mode is `#IR simple` which allows you to use simple words such as `push` or `int` instead of their value equivalents, noting of course that the extra work to analyse will slow down parsing.  Compilers **should** support this and **could** support simple mode.
- `#Strict true/false` supplying a true value should require the compiler enforce only **must** and **should** requirements and definitely turn off all **could** or any ones not stated in this document.  Compilers **should** support `#Strict true` and **must** support `#Strict false`.
- `#NoKeywords true/false` compilers **must** support `#NoKeywords false` and **should** support `#NoKeywords true` which turns true and false into `#true` and `#false` for all cases except for attribute statements.

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
  - It **shouldn't** be dynamically allocated since the `init` command initialize the stack and registers.
  - You **must** expose both the current size of the stack and the max size.
    - The current size represents how many elements have currently been pushed through `cursize` (pushing size onto stack)
    - The max size represents the total space available for elements (equal to `init` parameter) through `maxsize` (pushing size onto stack)
  - It **should** *ONLY* allow pushing/popping/peeking operations (though note : hat a peek can be implemented through popping and then push that value as well as return it, though this is inefficient and a more native method should be implemented)
  - You **could** implement it using either a `stack` or a static array with pointer/index
- `call` ooperations operate on a register based model
  - It **shouldn't** be dynamically allocated since the `init` command grants the size.
  - You **must** expose the current size of the registers which represents the total size available for objects (equal to `init` parameter) through `regsize` (pushing size onto stack)
  - It **should** *ONLY* allow indexing operations (both to set and to 'unset' - or set to null)
- All the commands **should** be implemented through a byte value (which is defined in this spec as an unsigned 8-bit integer type) and **could** be defined as an enum.

#### Required Commands

The following are all the required commands.  All parsers **need** to support the following.
> I'll include both the name and opcode but you only **need** to support the opcode the name is optional.

They will be in the format `<command>(opcode) < < parameterName: parameter >, < ... > >`.

> This list may be changed up overtime, so take care!

- `nop(00)`: does explicitly nothing
- `init(01) <Stack Size: int> <Register Size: int>`: sets up the stack and registers
  - The parameter represents the new size not the difference
  - Objects aren't carried across so effectively a wipe
  - If either size < current size then that initilization doesn't occur.
- `deinit(02)` useful in some rare cases, just deinitialize all the memory freeing it.
- `createtype(03) <ID: int> < <CollectionID: TypeID> <Type: TypeID> ... >` creates a complex type useful for maps of maps of arrays.
  - i.e. `createtype 2 1 3 1 0 0 1` makes a map of string to another map which is int key to a float array.  This can be read like `[string : [int : []float]]`, and create type would often be expressed like; `createtype 2 map str map int array float` (and perhaps even a `true/false`)
- `newobj(10) <Type: Object> <Constructor: Function> <Register: int>`: creates a new object 
  - The register refers to what register this object is created in.
  - The count refers to how many parameters there are.
  - To default constructor the type and constructor are the same.
- `push(11) <Type: TypeID> < <Parameter: Type> >`: pushes objects of the same type onto the stack
  - All parameters have to match the type given.
- `calln(12) <Register: int> <Type: ObjectID> <Setter: FunctionID>`: calls a register object of a certain type along with the number of stack objects given.
  - If you want to call an object on the stack you have to use `callstack` or simply `regobj` to register an object to a register then you can follow up with a `calln` (which is faster if you are doing multiple calls).
- `callstack(13) <Type: Object> <Setter: Function> <N: int>`: calls a function of type given on the top object of the stack (doesn't pop it).
- `pop(14) <N: int>`: pops number of objects off the stack.
- `getn(15) <Register: int> <Type: Object> <Getter: Function> <ReturnType: TypeID> <N: int>` calls a function and places value onto stack.
- `getstack(16) <Type: Object> <Getter: Function> <ReturnType: TypeID> <N: int>` sames as `getn` but for the stack.
- `quickpush(20) <Type: TypeID> <N: int> < <Parameter: Type> >`: Quick push doesn't push to the stack but rather stores it in a smaller 'register' like addressable state.
  - Often implemented as a long integer (8 bytes) meaning it'll work either as a ptr, an integer, a floating point value, a boolean, a string would be done with a ptr to the instruction data commonly though of course this is an unstable 'string' (will be free'd on completion of program).
  - The speed improvement comes from avoiding touching the stack and compilers can also convert `quickpush`'es to `pcall`'s either ahead of time or as the code executes.
  - You can enforce a maximum amount for `n`, which can be set to a number even as low as '1' and still be compliant as typical uses of quickpush are for easy string generation where a ptr is more efficient.
  - Note: normal calls won't work with quick pushes, only quick calls work.
- `quickcall(21) <Register: int> <Type: Object> <Setter: Function> <N: int>`: Performs a call with quick push'd variables.
- `pcall(22) <Register: int> <Type: Object> <Setter: Function> <N: int> < <Obj Type: type> <Parameter: Obj Type> ... >`: performs a call with the parameters in the call, very similar to quick call however isn't typically supported with a wide range of parameters.
- `pnewobj(23) <Type: Object> <Constructor: Function> <Register: int> <Count: int> < <Parameters> >` Allows you to supply parameters in call.  Parameters have to be calculated at runtime so it is less efficient.
  - Note: this IR code has not been confirmed yet, so one should be hesitant to use it.
- `quickget(24) <Register: int> <Type: Object> <Setter: Function> <ReturnType: TypeID> <N: int>` very similar in nature to `quickcall` but functions like `getn`
- `dumbget(25) <Register: int> <Type: Object> <Setter: Function> <N: int>` 'dumbly' gets a value by effectively calling it like a void* function (or `Object`), then not implementing the type, meaning that the type is set next time it is in use not in a dumb context.
- `pusharray(30) <Type: TypeID> <Len: int>`: pushes an array of length and type given onto the stack.
- `setarray(31) <Type: TypeID> <Index: int> <Obj: Type>`: indexes and sets an object.
- `getarray(32) <Type: TypeID> <Index: int>`: indexes an object and pushes value onto stack.
- `arraycpy(33) <Type: TypeID> <Len: int> < <Obj: Type> ... >`: basically builds the array through using a memcpy if it can as in the case of binary streams and in other cases also I'm sure.  Should be more efficient anyway as cuts down number of instructions.
- `compact(34) <Type: TypeID> <N Dimension: int>` compacts above arrays into dimensions given, i.e. if you give it two arrays will build a 2D array, the order is from top down not down up.
- `pushmap(40) <KeyType: TypeID> <ValueType: TypeID>` pushes a map onto the stack.
- `pushcollection(41) <CreatedType: TypeID>` pushes a 'createtype' map
- `setmap(42) <KeyType: TypeID> <ValueType: TypeID> <Key: KeyType> <Value: ValueType>` indexes and sets map for key and value given.
- `setcollection(43) <CreatedType: TypeID> < <Key: KeyType> ... > <Value: ValueType>` same as `setmap` but for created types.
- `quicksetmap(44) <KeyType: TypeID> <ValueType: TypeID> <N: int> < <Key: KeyType> <Value: ValueType> >` just applies multiple set maps in a row, more efficient as less IR but doesn't have any benefits like memcpy.
- `zipmap(45) <Depth: int> < <KeyType: TypeID> ... > <ValueType: TypeID> <Count: int>` can be an efficient way to mass create a complex map, will zip maps together as per your depth, WON'T however allow you to have arrays for either keys or values.
  - When calling it, it will pop off items, you give it on the stack like the key is the top (or last pushed) and then next is the 'deeper' map i.e. if you have `[int : [string : float]]` you would have the `[string : float]` map at the bottom with the `int` key above each of its corresponding map and if you had `[int : [int : [string : int]]]` you would have the first int key at the top then the second int key with each of its corresponding map i.e. `map01 int1 map02 int2 ... top_int1 map1y inty ... top_int2`.
- `getmap(46) <KeyType: TypeID> <ValueType: TypeID> <Key: KeyType>` pushes value at key onto stack.  If value doesn't exist for key should push 'NULL'
- `getcollection(47) <CreatedType: TypeID> < <Key: KeyType> ... >` same as getmap but for created types.

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
