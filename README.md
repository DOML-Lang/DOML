# Data Oriented Markup Language - DOML
> By Braedon Wooding

> Latest Version 0.1

> *Note: This spec is changing a whole lot!  Until its marked at-least 1.0 you should regard it as unstable*

## TLDR
- DOML is more like a scripting language
- It parses down to a IR (effectively the same look as assembly but simpler - one parameter) that can be passed efficiently across devices
- It's grammar is dead simple (71 lines of ABNF compared to TOML's 219 line grammar)
- It's easy to write and read (and parse)
- It's fast to parse (and IR is even faster, especially if passed as byte stream)
    - As more versions of the language appear I'll have a benchmark section for each version.
- All the code you write will actually create real objects (though it is safe, and you define what objects can be created)
    - Not only this but you can edit properties of those objects and reference those objects both their properties and them
    - Don't fear the anonymous object creation that YAML had, this doesn't allow random object creation it only allows you to call your own code and which parts are completely up to the author.
- Callback system, with an aim of both static and reflective binding support.
- Whitespace insensitive

## Brief Introduction
DOML (Data Oriented Markup Language) is a language that enacts to stop re-inventing the wheel and start inventing wings, effectively it enacts to solve the 'problem' (as in software problem) of serialization in a different way then all other popular languages like XML/JSON/YAML/TOML/...

Its entire ABNF is only 71 lines compared to TOMLs 219 line grammar (which is ~3x larger), this is including comments and nice spacing in both.  DOML has two rules; creating new objects, and manipulating parts of those objects.  

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
              .RGB             = 255, 64, 128 // Implicit 'array'

@ TheSame     = System.Color ...
              .RGB(Normalised) = 1, 0.25, 0.5, // You can have trailing commas

@ AgainSame   = System.Color ...
              .RGB(Hex)        = 0xFF4080
              .Name            = "OtherName"

/* Multi Line Comment Blocks are great */
@ Copy        = System.Color ...
              .RGB             = Test.R, Test.G, Test.B // Reference other objects
              .Name            = "Copy"
```
> Note: Nested multi line comments are allowed (I didn't put it in the example since github doesn't recognise DOML yet so I'm just using `C` in the meantime which doesn't support nesting).
> > Further Note: The copy example also shows you that you can nest objects like `; Child.Parent = Parent`.

When you put this into a parser you'll get the below output (its standidized so you **will** get the below output - though the supplementary comments for each line may differ, though I've not included all the comments to keep it more concise and short);
```assembly
; This is the resulting bytecode from the file given
; This bytecode will be overriden if new bytecode is generated.
02 4                                                  ; Reserves 4 spaces on the stack.
03 4                                                  ; Reserves 4 spaces on the register.
01 "This is a comment"                                ; USER COMMENT
01 "Construct a new System.Color"                     ; USER COMMENT
06 System.Color                                       ; Performs a constructor call on System.Color and pushes the new object onto the stack
07 0                                                  ; Registers top object to index 0 after popping it off the stack
12 255                                                ; Pushes long integer 255 onto the stack
12 64                                                 ; Pushes long integer 64 onto the stack
12 128                                                ; Pushes long integer 128 onto the stack
11 0                                                  ; Pushes object in register ID: 0 onto the stack
04 System.Color::RGB                                  ; Runs the System.Color::RGB function
01 "Implicit 'array'"                                 ; USER COMMENT
06 System.Color                                       ; Performs a constructor call on System.Color and pushes the new object onto the stack
07 1                                                  ; Registers top object to index 1 after popping it off the stack
12 1                                                  ; Pushes long integer 1 onto the stack
13 0.25                                               ; Pushes double 0.25 onto the stack
13 0.5                                                ; Pushes double 0.5 onto the stack
01 "You can have trailing commas"                     ; USER COMMENT
11 1                                                  ; Pushes object in register ID: 1 onto the stack
04 System.Color::RGB.Normalised                       ; Runs the System.Color::RGB.Normalised function
06 System.Color                                       ; Performs a constructor call on System.Color and pushes the new object onto the stack
07 2                                                  ; Registers top object to index 2 after popping it off the stack
12 16728192                                           ; Pushes long integer 16728192 onto the stack
11 2                                                  ; Pushes object in register ID: 2 onto the stack
04 System.Color::RGB.Hex                              ; Runs the System.Color::RGB.Hex function
15 "OtherName"                                        ; Pushes string "OtherName" onto the stack
11 2                                                  ; Pushes object in register ID: 2 onto the stack
04 System.Color::Name                                 ; Runs the System.Color::Name function
01 " Multi Line Comment Blocks are great "            ; USER COMMENT
06 System.Color                                       ; Performs a constructor call on System.Color and pushes the new object onto the stack
07 3                                                  ; Registers top object to index 3 after popping it off the stack
11 0                                                  ; Pushes object in register ID: 0 onto the stack
05 System.Color::R                                    ; Performs a getter call on System.Color::R and pushes the values onto the stack
11 0                                                  ; Pushes object in register ID: 0 onto the stack
05 System.Color::G                                    ; Performs a getter call on System.Color::G and pushes the values onto the stack
11 0                                                  ; Pushes object in register ID: 0 onto the stack
05 System.Color::B                                    ; Performs a getter call on System.Color::B and pushes the values onto the stack
11 3                                                  ; Pushes object in register ID: 3 onto the stack
04 System.Color::RGB                                  ; Runs the System.Color::RGB function
15 "Copy"                                             ; Pushes string "Copy" onto the stack
11 3                                                  ; Pushes object in register ID: 3 onto the stack
04 System.Color::Name                                 ; Runs the System.Color::Name function
```
I won't go into great detail about the IR, but effectively it is similar to assembly; the `;` is a line comment, each command is seperated by a line and has one parameter (only one).  Its stack based but uses a statically defined stack for efficiency (literally the speed comparison is insane).

## Types

| Type          | Example Values                        | Details (all suffixes are case insensitive)        |
| ------------- | ------------------------------------- | -------------------------------------------------- |
| Integer       | 12, -40, +2, 01010b, 0x40FF, 0o42310  | 0b for binary, 0x for hex, 0o for octal            |
| Double        | 10.5, 20, 5e+22, 1e6, -2.54E-2        | E for exponent                                     |
| Decimals      | $40, +$99.05, -$4e+22                 | Can use E for exponent and '$' refers to decimal   |
| String        | "This contains a \" escaped quote"    | "...", you can escape `"` with `\`                 |
| Boolean       | true, false                           | The boolean values                                 |
| Object        | Test, X, MyColor                      | Refers to a previously defined object              |

> Note: decimals have standidised for `$` though many parsers will probably allow the various other currency signs.

## Comparison with other formats
> I've re-written this so many times because DOML is so different it almost accomplishes the same goal completely differently.  So its hard to compare.

Effectively DOML is simpler and more efficient then the other data formats.  XML is hard for machines to parse, JSON is overly complex and hard to write, YAML is way to complex (no parser is fully 1.1 compliant and there are basically no 1.2 parsers at all), and INI isn't standidised.  TOML is closer to DOML I think then most other languages (and well it does share 3/4 of the same letters though I would argue TOML's acronym is less informative and serves an 'ego' inflating purpose), they both try to be simple but I feel that TOML falls into the trap of trying to make the current solution as nice as possible where as DOML tries to solve the problem at a new angle, TOML is more akined for small config files where as DOML is nicer for files with multiple objects.

Effectively DOML is a scripting language that is constrained to purely object creation and 'editing', this allows it to be effective at what it does and makes it as low level as possible (IR brings it even closer).  I would argue its still a markup language since that's what it does it notates objects.  Arguably a different name could have been DOON or Data Oriented Object Notation.

## Get Involved!
Anyways, if you have any new changes you may wish to add please ask in the issues!  I will be more cautious to accept PRs if the changes aren't talked about in an issue (though the obvious exception is if you are fixing up typos/errors or if its a super small thing, OR if you are adding your implementation / project to the list all these don't require issues of course).  Further more I'll open up a discord chat somewhat soon (and maybe a mailing list, though nowadays they seem a little archaic).

## Projects using DOML
- None yet, if you know of one please send a PR adding to this list!

## Implementations of DOML
If you have an implementation, send a pull request adding to this list. Please note the version tag that your parser supports in your Readme.

## v0.1 Compatible
- [.Net (C#/F#/...)](https://github.com/DOML-DataOrientedMarkupLanguage/DOML.net)
    - Supports reflective and static bindings, supports parsing IR by byte stream in both native and main variants.
    - Uses .Net Standard 1.3 (thus supporting Core 1.0+, Unity if using 4.6 compiler option, .Net Framework 4.6+, and UWP 10.0)
        - This is the lowest we can have the standard (since we require certain libraries that exist at 1.3+), thus we are sorry if this doesn't suit your particular project but we can't push back the standard any further.
- [GO](https://github.com/DOML-DataOrientedMarkupLanguage/DOML-GO)
    - Progress has recently begun
    - Buggy but allows reading DOML into IR only

## In Progress
- [C++](https://github.com/DOML-DataOrientedMarkupLanguage/DOML-Cxx)
    - No progress has started

## Editor Support
- [Notepad++](https://github.com/DOML-DataOrientedMarkupLanguage/Notepad-Syntax)
- vim/EMACS/Sublime Text/VS Code are all in development
