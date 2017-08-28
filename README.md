# Data Oriented Markup Language - DOML (.Net)
> By Braedon Wooding

> Latest Version N/A

> *Note: This spec is changing a whole lot!  Until its marked at-least 1.0 you should regard it as unstable*

This is the Specification document for DOML (Data Oriented Markup Language), which is a 'new' markup language that takes a different approach then most. It enacts to simulate a call-stack rather than simulate data structures, this allows it to represent a constructor like look rather than the usual {...} mess.  Its simplicity can't be ignored, the entire ABNF is 80 lines including comments and nice indentation which is insanely small (especially considering TOML another 'minimalistic language' has a 219 line grammar which around ~2.74x larger)!

## Why DOML?
DOML was born out of a need, a need to have a markup language that stops functioning like just a text file and starts functioning like an integral part of the program, it was born out of an annoyance at writing code of loading a file in and either having to manually de-serialize/serialize or having to fiddle with an automatic serializer, and never knowing if the solution had a small typo in it.  I finally 'cracked' and brought out the playground project and whiteboard when I saw someone building an entire application whose whole purpose was to just take data and build a JSON object, since it was easier then writing 50 billion `"` and braces, and was validated (unlike every other markup language on the planet).  So I sat down at my desk and crafted DOML, a solution which I aimed to be efficient and to be practical to allow you to do whatever you need without much intrusion.  Hopefully I'm correct.  Further more DOML is simple, the first parser was < 300 lines, and today its < 2000 (due to the addition of bytecode and security), the entire ABNF fits in 80 lines and there are only TWO main rules (excluding comments) that is `creation` or `set` which either perform a constructor or set a variable within a variable created in a creation call.  It has 0 keywords and only 5 operators (`...`, `=`, `,`, `@`, `;`)!  Operators like `.` are part of the name, and other operators like `(`, `)`, `{`, `[`... (effectively any open/close brace/bracket/arrow/...) are replaced with `.` so the following are all the SAME call (simplicity mhmmmm); `.RGB(Normalised)`, `.RGB[Normalised]`, `.RGB<Normalised>` are all just `.RGB.Normalised`!

## Objectives
Efficient data serialization.  Not only be fast, but also the avoidance of wasting memory.  Removal of the separation between markup languages and your project, the burning of the obscure and ugly bridge to join the two islands together using a nice steel well constructed bridge that is easy to traverse and actually supports vehicles (maybe I went too far on this analogy?)...  Further more it aims to be simple, the avoidance of 'arrays' and complex structures for implicit collections is key!

## A quick overview
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
  // Note: nesting is allowed its just till I make a github syntax and it gets accepted I won't include it in examples, since I'll use the C one in the meantime and that doesn't allow nesting multi line comment blocks
  Anyways lets go and copy another previous one by just copying over the values.
*/
@ Copy        = System.Color ...
;             .RGB             = Test.RGB
;             .Name            = "Copy"
```

Effectively the heirachy looks something like this (outdated slightly update it);
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

## Quick Spec Details
- DOML is case sensitive
- DOML must be a valid UTF-8 encoded unicode document (though I don't see why you can't support other formats)
- DOML refers to the initial source code not to the produced bytecode, parsers need to support ONLY DOML and don't need to support parsing bytecode, though due to its simplicity it may be easy to support.

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

## Syntax
I'll cover this quickly here you can view an indepth either under [indepth syntax](syntax.md) or at by viewing the [abnf](doml.abnf).

#### Comments
C-Styled comments either `//` or `/*` nesting is allowed for both;
```
// This is a comment
/* This is a block and /* this is a nested block */ */
```

#### Structure
The structure is simple you have two types of 'calls' you have either 'creation calls' or 'set calls' they look like either;
```C
@ X = Y.Z
// OR
; X.A = B, C, D
```
Where B/C/D can either be a creation call variable (like X in this instance) can be a lookup of any root object (like Y in this instance, though you couldn't just use `Y` you would have to do something like `Y.MyValue` since 'Y' refers more to a collection of functions then an object).  B/C/D can also be any value (listed in the type table) they can go forever, you have no limits on how many values you set.

You can shorten the structure a little using the `...` operator such as;
```C
@ X   = Y.Z ...
;     .A = B, C, D
```
Note that the second X wasn't required, this is because the `...` means just presume that every thing that starts with `.` is referring to `X` UNTIL you meet another `@` (regardless if that `@` has a `...`) you can always just refer to other objects like;
```C
@ B   = W.V
@ X   = Y.Z ...
;     .A = B, C, D
;    B.E = F, G, H
```

#### That's it!
> Wait what?
Yes it may seem weird but the syntax is THAT simple its meant to be!  It will probably expand a little, like I would like the choice to have an inline setter so you could do something like;
```C
@ point = PointHandler.NewPoint ...
;       .{ x = 2; y = 3; z = 9 }
```
But I guess you could always just do;
```C
@ point = PointHandler.NewPoint ...
;       .x = 2; .y = 3; .z = 9
```
But eh the other one looks a little cleaner, regardless we will see.

#### But no arrays...?
There are still arrays remember `B, C, D` you passing multiple values to `.A` your effectively passing a heterogeneous array (allows any type), at its core C arrays are just this (well that and they are homogeneous or the same type) modern arrays often tack on a little length variable and I guess you could always do that yourself, and the argument may be that we should handle that, but I'm still not convinced on that front since the ONLY case where that would be useful would be when you are passing multiple values to a function and you want one of the values to be an array and the others to not be an array.  Otherwise you can always just either keep popping till you get a 'false' (run out of elements) or be a good person and do a for loop using a variable that says how big the 'stack' is.

Note: this is where I should say that all the DOML implementations by me will have the same API (with only minor differences) and I suggest if other people make new implementations they follow the same API guidelines, though I'm not going to enforce that so the whole array thing is more of a parser implemented thing then anything.

## Comparison with other formats
Firstly I'll list the problems with other formats; XML has always been hard to read, hard to write (though I would claim its still easier to write then JSON by quite a landslide) and especially hard to parse and its spec is a complete mess.  JSON is easy to parse, but still suffers from being annoyingly verbose.  YAML is much improved in this manner, but however due to its tight bindings to JSON (all JSON works in YAML parsers) it stretches out its SPEC even further (and YAML's spec is sooo large that no parser is capable of supporting it all), and even further more its reliance on JSON hinders whitespace meaning that if you choose indentation you need to like Python keep everything indented right which means that its not compact enough for web use (and JSON isn't either though its often used due to its native compatibility with Javascript).

A few others;
- INI is old, and suffers from a lot (though I quite like it for its simplicity)

So what one is like DOML the most?  Well I guess I would say TOML?  TOML is a great step forward in the advancement of markup languages though I still feel it suffers from trying to create a new thing from what already is here rather than create a new thing from the core fundamentals of what it should be (which DOML was created out of), this is why I can't really compare it greatly with other formats...  Because it was built to be a different sort of tool, a different sort of markup language it accomplishes the same goal but in a way that I would *claim* has never been done before (succesfully at least).  This isn't just another markup language (damn should have included a YAML pun), but rather I'm sure some will claim isn't really even a 'markup language', and shares more similiarity with scripting languages then actual markup languages.

Though I would claim differently, the clear difference between a scripting language and a programming language is *how* you use it and in what context, and I would further claim that the biggest difference between a markup language and a scripting language is how you *can* you use (or rather *should*), since while DOML is 'turing complete' (though you would have to add functions like math and basic drawing for anyone to be like 'yeh now its turing complete') it purposefully limits actions like math and so on, since its built to represent data thus its a data oriented language.  Every action you perform is around the manipulation of data (again the clear distinction is that the 'purpose' of the language is to create data the purpose of other languages is to creation applications and while both may 'manipulate data' its clear to see what is different).

So to 'wrap up', DOML is more than just a simplistic alternative, its a 'scripting language' with the whole goal of creating data, not with using it!  Thus I would claim its a markup-language due to the similar goals and the restrictions placed on it, though I'm sure others will disagree.

## Get Involved!
Anyways, if you have any new changes you may wish to add please ask in the issues!  I will be more cautious to accept PRs if the changes aren't talked about in an issue (though the obvious exception is if you are fixing up typos/errors or if its a super small thing, OR if you are adding your implementation / project to the list all these don't require issues of course).  Further more I'll open up a discord chat somewhat soon (and maybe a mailing list, though nowadays they seem a little archaic).

## Projects using DOML
- None yet, if you know of one please send a PR adding to this list!

## Implementations of DOML
If you have an implementation, send a pull request adding to this list. Please note the version tag that your parser supports in your Readme.

## v0.1 Compatible

## In Progress
- [.Net (C#/F#/...)](https://github.com/DOML-DataOrientedMarkupLanguage/DOML.net)
- [C++](https://github.com/DOML-DataOrientedMarkupLanguage/DOML-Cxx)

## Editor Support
- None yet, but VIM/Notepad++/EMACS/Sublime Text/VS Code are all in development
