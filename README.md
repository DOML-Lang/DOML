# Data Oriented Markup Language - DOML (.Net)
> By Braedon Wooding

> Latest Version N/A (Approaching 0.1)

> *Note: This spec is changing a whole lot!  Until its marked at-least 1.0 you should regard it as unstable*

## Brief Introduction
DOML (Data Oriented Markup Language) is a language that enacts to stop re-inventing the wheel and start inventing wings, effectively it enacts to solve the 'problem' (as in software problem) of serialization in a different way then all other popular languages like XML/JSON/YAML/TOML/...

Its entire ABNF is only 74 lines compared to TOMLs 219 line grammar (which is ~3x larger), this is including comments and nice spacing in both.  DOML has two rules; creating new objects, and manipulating parts of those objects.  

DOML also erases the need for a middleman implicitly, no longer do you have to convert the JSON to a map then convert it to your data types, or try to use an automatic method (which often requires reflection).  DOML parses to an IR format which is ran, this means that if you have a parser that is compliant you could get IR code from it and then use it in another parser and further more it means that parsers could add new features and those new features would work in other parsers since there is a restricted set of IR commands.

All together it makes it significantly faster than JSON/XML/TOML/... parsing (in basic testing, I'll do some serious benchmarks later).  This is mainly due to the fact that the syntax and parsing doesn't have to do any look-aheads, and we create IR because its easier to parse and transfer over the web (example being that you convert your source into IR and transfer it in binary format to the server to run).  Having a computer run the code as it reads it could also be a possibility.

Also I should add that it is safe (in comparison to formats like YAML) this is mainly because it doesn't allow arbitary creation of objects, you can add new objects that it can create but it can't define them for you.

## Objectives
- Efficient (cost of speeds have to provide a high benefit of utility)
- Integrated into your project (no longer a separate part)
- Simple (only two 'expressions' to parse)

## A quick overview
```C
// This is a comment
// Construct a new System.Color
@ Test        = System.Color ...
;             .RGB             = 255, 64, 128 // Implicit 'array'

@ TheSame     = System.Color ...
;             .RGB(Normalised) = 1, 0.25, 0.5, // You can have trailing commas

@ AgainSame   = System.Color ...
;             .RGB(Hex)        = 0xFF4080
;             .Name            = "OtherName"

/* Multi Line Comment Blocks are great */
@ Copy        = System.Color ...
;             .RGB             = Test.R, Test.G, Test.B
;             .Name            = "Copy"
```
> Note: Nested multi line comments are allowed (I didn't put it in the example since github doesn't recognise DOML yet so I'm just using `C` in the meantime which doesn't support nesting).
> > Further Note: The copy example also shows you that you can nest objects like `; Child.Parent = Parent`.

When you put this into a parser you'll get the below output (its standidized so you **will** get the below output - though the supplementary comments for each line may differ, though I've not included all the comments to keep it more concise and short);
```assembly
; This is the resulting bytecode from the file given
; This bytecode will be overriden if new bytecode is generated.
MAKE_SPACE      4                                                  ; Reserves 4 spaces on the stack.
MAKE_REG        4                                                  ; Reserves 4 spaces on the register.
COMMENT         "This is a comment"                                ; USER COMMENT
COMMENT         "Construct a new System.Color"                     ; USER COMMENT
NEW             System.Color                                       ; Performs a constructor call on System.Color and pushes the new object onto the stack
REG_OBJ         0                                                  ; Registers top object to index 0 after popping it off the stack
PUSH_INT        255                                                ; Pushes long integer 255 onto the stack
PUSH_INT        64                                                 ; Pushes long integer 64 onto the stack
PUSH_INT        128                                                ; Pushes long integer 128 onto the stack
PUSH_OBJ        0                                                  ; Pushes object in register ID: 0 onto the stack
SET             System.Color::RGB                                  ; Runs the System.Color::RGB function
COMMENT         "Implicit 'array'"                                 ; USER COMMENT
NEW             System.Color                                       ; Performs a constructor call on System.Color and pushes the new object onto the stack
REG_OBJ         1                                                  ; Registers top object to index 1 after popping it off the stack
PUSH_INT        1                                                  ; Pushes long integer 1 onto the stack
PUSH_NUM        0.25                                               ; Pushes double 0.25 onto the stack
PUSH_NUM        0.5                                                ; Pushes double 0.5 onto the stack
PUSH_OBJ        1                                                  ; Pushes object in register ID: 1 onto the stack
SET             System.Color::RGB.Normalised                       ; Runs the System.Color::RGB.Normalised function
COMMENT         "You can have trailing commas"                     ; USER COMMENT
NEW             System.Color                                       ; Performs a constructor call on System.Color and pushes the new object onto the stack
REG_OBJ         2                                                  ; Registers top object to index 2 after popping it off the stack
PUSH_INT        16728192                                           ; Pushes long integer 16728192 onto the stack
PUSH_OBJ        2                                                  ; Pushes object in register ID: 2 onto the stack
SET             System.Color::RGB.Hex                              ; Runs the System.Color::RGB.Hex function
PUSH_STR        "OtherName"                                        ; Pushes string ""OtherName"" onto the stack
PUSH_OBJ        2                                                  ; Pushes object in register ID: 2 onto the stack
SET             System.Color::Name                                 ; Runs the System.Color::Name function
COMMENT         " Multi Line Comment Blocks are great "            ; USER COMMENT
NEW             System.Color                                       ; Performs a constructor call on System.Color and pushes the new object onto the stack
REG_OBJ         3                                                  ; Registers top object to index 3 after popping it off the stack
PUSH_OBJ        0                                                  ; Pushes object in register ID: 0 onto the stack
CALL            System.Color::R                                    ; Performs a getter call on System.Color::R and pushes the values onto the stack
PUSH_OBJ        0                                                  ; Pushes object in register ID: 0 onto the stack
CALL            System.Color::G                                    ; Performs a getter call on System.Color::G and pushes the values onto the stack
PUSH_OBJ        0                                                  ; Pushes object in register ID: 0 onto the stack
CALL            System.Color::B                                    ; Performs a getter call on System.Color::B and pushes the values onto the stack
PUSH_OBJ        3                                                  ; Pushes object in register ID: 3 onto the stack
SET             System.Color::RGB                                  ; Runs the System.Color::RGB function
PUSH_STR        "Copy"                                             ; Pushes string ""Copy"" onto the stack
PUSH_OBJ        3                                                  ; Pushes object in register ID: 3 onto the stack
SET             System.Color::Name                                 ; Runs the System.Color::Name function
```
I won't go into great detail about the IR, but effectively it is similar to assembly; the `;` is a line comment, each command is seperated by a line and has one parameter (only one).  Its stack based but uses a statically defined stack for efficiency (literally the speed comparison is insane).

## Types

| Type          | Example Values                        | Details (all suffixes are case insensitive)        |
| ------------- | ------------------------------------- | -------------------------------------------------- |
| Integer       | 12, -40, +2, 01010b, 0x40FF, 0o42310  | 0b for binary, 0x for hex, 0o for octal            |
| Double        | 10.5, 20, 5e+22, 1e6, -2.54E-2        | E for exponent                                     |
| Decimals      | $40, +$99.05, -$4e+22                 | Can use E for exponent and '$' refers to decimal   |
| String        | "This contains a \" escaped quote"    | "...", you can escape `"` with `\`                 |
| Char          | 'C', '5'                              | Maps to a character                                |
| Boolean       | true, false                           | The boolean values                                 |
| Object        | Test, X, MyColor                      | Refers to a previously defined object              |

> Note: decimals have standidised for `$` though many parsers will probably allow the various other currency signs.

## Comparison with other formats
> I've re-written this so many times because DOML is so different it almost accomplishes the same goal completely differently.  So its hard to compare.

Effectively DOML is simpler and more efficient then the other data formats.  XML is hard for machines to parse, JSON is overly complex and hard to write, YAML is way to complex (no parser is fully 1.1 compliant and there are basically no 1.2 parsers at all), and INI isn't standidised.  TOML is closer to DOML I think then most other languages (and well it does share 3/4 of the same letters though I would argue TOML's acronym is less informative), they both try to be simple but I feel that TOML falls into the trap of trying to make the current solution as nice as possible where as DOML tries to solve the problem at a new angle, TOML is more akined for small config files where as DOML is nicer for files with multiple objects.

Effectively DOML is a scripting language that is constrained to purely object creation and 'editing', this allows it to be effective at what it does and makes it as low level as possible (IR brings it even closer).  I would argue its still a markup language since that's what it does it notates objects.  Arguably a different name could have been DOON or Data Oriented Object Notation.

## Get Involved!
Anyways, if you have any new changes you may wish to add please ask in the issues!  I will be more cautious to accept PRs if the changes aren't talked about in an issue (though the obvious exception is if you are fixing up typos/errors or if its a super small thing, OR if you are adding your implementation / project to the list all these don't require issues of course).  Further more I'll open up a discord chat somewhat soon (and maybe a mailing list, though nowadays they seem a little archaic).

## Projects using DOML
- None yet, if you know of one please send a PR adding to this list!

## Implementations of DOML
If you have an implementation, send a pull request adding to this list. Please note the version tag that your parser supports in your Readme.

## v0.1 Compatible
- Version v0.1 isn't out yet so no parsers will support it (inherently).

## In Progress
- [.Net (C#/F#/...)](https://github.com/DOML-DataOrientedMarkupLanguage/DOML.net)
- [C++](https://github.com/DOML-DataOrientedMarkupLanguage/DOML-Cxx)

## Editor Support
- [Notepad++](https://github.com/DOML-DataOrientedMarkupLanguage/Notepad-Syntax)
- vim/EMACS/Sublime Text/VS Code are all in development
