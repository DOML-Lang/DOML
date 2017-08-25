# Data Oriented Markup Language - DOML (.Net)
This is the Specification document for DOML (Data Oriented Markup Language), which is a 'new' markup language that takes a different approach then most. It enacts to simulate a call-stack rather than simulate data structures, this allows it to represent a constructor like look rather than the usual {...} mess.

## A quick overview
Till the website goes online I'll give a quick explanation;
```C
// This is a comment
@ Test        = System.Color ... // System is just a random name represents a 'root object', System.Color represents a 'object'
;             .RGB             = 255, 64, 128

@ TheSame     = System.Color ...
;             .RGB(Normalised) = 1, 0.25, 0.5 // 'ish' normally I would round down not up but eh

@ AgainSame   = System.Color ...
;             .RGB(Hex)        = 0xFF4080
;             .Name            = "OtherName

/* Multi Line Comment Blocks are great
  /* Especially when nesting is allowed */
  
  Anyways lets go and copy another previous one by just copying over the values.
*/
@ Copy        = System.Color ...
;             .RGB             = Test.RGB
;             .Name            = "Copy"
```

Effectively the heirachy looks something like this;
`RootObject.CreateObject("Color")->Object.RunFunction("RGB", [255, 64, 128])` which is broken down into a series of byte code instructions.  Below is the bytecode representation of the above code;
```assembly
; This is a comment
initspace 3 &0
initspace 3 &1
initspace 1 &2
initspace 1 &2 ; Two seperate spaces that belong to the same object
initspace 3 &3 
initspace 1 &3 ; Two seperate spaces that belong to the same object

new System.Color @0       ; new System.Color into variable @0
tiespace &0 @0            ; ties the space &0 to variable @0
; System is just a random name representing a 'root object', System.Color reprseents an 'object'
pushint 255 &0            ; pushes integer 255, referencing spot &0
pushint 64  &0            ; pushes integer 64, referencing spot &0
pushint 128 &0            ; pushes integer 128, referencing spot &0
set RGB @0                ; runs the .RGB function from variable @0 using variables &0
new System.Color @1       ; new System.Color into variable @1
tiespace &1 @1            ; ties the space &1 to variable @1
pushfloat 1 &1            ; pushes float 1, referencing spot &1
pushfloat 0.25 &1         ; pushes float 0.25, referencing spot &1
pushfloat 0.5 &1          ; pushes float 0.5, referencing spot &1
set RGB.Normalised @1     ; runs the .RGB(Normalised) function from variable @1 using variables &1
new System.Color @2       ; new System.Color into variable @2
tiespace &2 @2            ; ties the space &2 to variable @2
pushhex FF4080 &2         ; pushes hex value FF4080, referencing spot &2
set RGB.Hex @2            ; runs the .RGB(Hex) function from variable @2 using variables &2
pushstring 'OtherName' &2 ; pushes string value 'OtherName', referencing spot &2
set Name @2               ; runs the .Name function from variable @2 using variables &2
; Multi Line Comment Blocks are great
;  Especially when nesting is allowed
;
;  Anyways lets go and copy another previous one by just copying over the values.
;
new System.Color @3     ; new System.Color into variable @3
tiespace &3 @3          ; ties the space &3 to variable @3
flushspace &0 0         ; flush space leaving 0 remaining (top down)
pushspace RGB @0        ; gets RGB from variable @0 and puts the result into space &0
move &0 &3              ; moves all variables from &0 to &3 (doesn't do a copy assign just moves ownership)
set RGB @3              ; runs the .RGB function from variable @3 using variables &3
pushstring 'Copy' &3    ; pushes string value 'Copy', referencing spot &3
set Name @3             ; runs the .Name function from variable @3 using variables &3
```
The use of `;` as a comment is purely just due to its comparison to assembly, I haven't actually decided on whether or not comments will 1) be allowed and 2) have the character `;` or `//` or even `#` though there will be no block comments and as you can see above the block comment is converted into a list of comments.  Comments will be reserved as you can see if they are allowed and extra ones can be toggled to display exactly what is occuring (incase one has to debug).

Effectively the 'compiler' will convert your DOML markup to this set of commands which will be executed by the system.  Note: this is done because the actual conversion to bytecode is effectively nothing in terms of speed and the output won't be emitted unless that is wanted by the user.  This also allows the actual runner to run the bytecode rather than interpret the markup allowing it to execute faster.  Sending bytecode over the internet rather than a JSON output is also faster and more compact.

Another thing to note is that while in the bytecode I used things like `System.Color`, `RGB`, `RGB.Normalised` and so on in the actual implementation it will use an index value like `#0`/`0`/`*0` the exact prefix (if any) will be determined later, and it'll map to the implementation.

## Types
This will be covered in more detail elsewhere but here are all the types

| Type          | Example Values                        | Details (all suffixes are case insensitive)        |
| ------------- | ------------------------------------- | -------------------------------------------------- |
| Integer       | 12, -40, +2, 40L, 40s, 01010b, 0x40FF | Add s/l for short/long  b for binary, 0x for hex   |
| Unsigned      | 40u, -40uL, 80Su                      | Add u for unsigned                                 |
| Float         | 10.0009, -0.05f, 5e+22, 1e6, -2.54E-2 | E/optional f for exponent/float (needs format)     |
| Double        | 10.5d, 20D, 5e+22d, 1e6d, -2.54E-2d   | E/required d for exponent/double (needs format)    |
| String        | "This contains a \" escaped quote"    | "...", you can escape `"` with `\`                 |
| Char          | 'C', '5'                              | Maps to a character (Ambigious needs format        |
| Boolean       | true, false                           | The boolean values                                 |
You can also use underscores in any number (integers, unsigned, floats, and doubles).
