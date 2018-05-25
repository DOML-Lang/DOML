# Data Oriented Markup Language - DOML

> By Braedon Wooding
> Latest Version 0.3
>> The spec could change and break previous code however this will be avoided (i.e. semi-stable)

## Introduction

The single line to sum up DOML is; DOML will create the objects defined, as if you were utilising a scripting language, it has the speed of a compiled language, it parses faster than any alternative due to its simplicity, it will result in actual objects rather than just a map of generalizations, and finally it is easy to write and read.

Since I want to keep this introduction short (the code examples will act as better guides anyway) I'll just put a TLDR, with some explanation;

- DOML (Data Oriented Markup Language) is less of a markup langauge and more of a scripting language
  - This is cause it defines objects rather than describes them, and when run will actually allocate the memory and the objects (often done through reflected/static bindings).
  - This is contrasted to JSON/XML/YAML/TOML/... which require you to interpret the resulting map thus making using DOML that much simpler and easier.
- It parses down to an IR which is simpler and quicker to parse and allows you to effectively 'bake' your configuration scripts, or even pass DOML as a binary stream without using plain text.
  - The IR isn't meant to be writable by humans though it easily could be, it consists of a series of lines with each line having an opcode value and a parameter.
  - Parsing IR is about 3.333...x faster (or rather only 30% of the original value)
    - That is if parsing plain text IR, if you parse the binary stream version it is significantly faster
    - This is seen [here](https://github.com/DOML-DataOrientedMarkupLanguage/DOML.net#benchmarks)
  - This IR is transferable to any DOML parser (allowing you to create the produced IR through some web API when you push to your remote, then place the IR along with the repo for compiling + packaging)
- It's extremely easy to read and write (compared to JSON being easy to read but hard to write, and XML being hard to read/write)
- Whitespace insensitive (you can remove every single whitespace if wished, there is no required whitespace since there is no keywords)
- No keywords only symbols
- Safe from anonymous object creation
  - This is because the objects created are restricted to what is designated by the user, and a static/reflected binding system is implemented to provide ways to link in with the user's code and functions.
- No requirement for the user to create any parsing code
  - For example in C# all you need to do is add the `[DOMLInclude("System")]` attribute (you can replace System with whatever you want the namespace to be called), you can also ignore and customise the names of functions, properties, and fields through attributes.

## A quick overview

```C
// Construct a new System.Color
Test = Color {
  .RGB = 255, 64, 128,
}

// Constructors do exist
TheSame = Color::Normalized(1, 0.25, 0.5) {
  .Name = "Bob"
}

// You can also just declare an object without scoping it
Other = Color
Other.Name = "X"

// You can also edit the original Test at any point EITHER by doing
Test.R = 50
// Or by doing
Test.{
  .G = 128
}

// You can declare arrays like
ArrayObject = [Color] {
  ::Normalized(0.95, 0.55, 0.22){
    .Name = "Other", // Trailing commas are always allowed
  },
  // You can still do an empty construction
  ::() {
    .RGB = 50, 25, 125,
  },
  // And thus you can leave out the ::()
  {
    .RGB = 50, 25, 125,
  },
}

// You can also copy objects by doing
NewObj = Other

// Or can do something like
NewObj.Name = ArrayObject[0].Name

// You can also declare arrays inside object definitions
MyTags = Tags {
  // Note: all have to be of the same type
  .Tags = ["Hello", "Other", "bits", "bobs", "kick"]
  .Name = MyTags.Tags[0] // And indexing them works like you would think
}

// You can declare dictionaries like
// Dictionaries within objects can also be created similarly
MyDictionary = [string : Color] {
    { 
      "Bob" : Color::Normalzed(0.5, 1.2, 3.5)
      {
        .Name = "Bob's Color"
      }
    },
}
```

When you put this into a parser you'll get the below output (its standidized so you **will** get the below output - though the supplementary comments for each line may differ, though I've not included all the comments to keep it more concise and short);

> NOTE: this resultant IR is from the old version the new version will have a similar format but IR will be a little more complicated.
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

I won't go into great detail about the IR, but effectively it is similar to assembly; the `;` is a line comment, each command is seperated by a line and has one parameter (only one). You can place multiple statements on a line by using a `,` to separate them.

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

## Collections


| Type          | Example                               | Details (all suffixes are case insensitive)        |
| ------------- | ------------------------------------- | -------------------------------------------------- |
| Arrays        | \[1, 2, 3, 4\]                        | All have to be of the same type                    |
| Dictionary    | { { "X", 2 }, { "Y", 9 } }            | All keys/values have to be same type (key != value)|

## Comparison with other formats

> I've re-written this so many times because DOML is so different it almost accomplishes the same goal completely differently. So its hard to compare.

Effectively DOML is simpler and more efficient then the other data formats. XML is hard for machines to parse, JSON is overly complex and hard to write, YAML is way to complex (no parser is fully 1.1 compliant and there are basically no 1.2 parsers at all), and INI isn't standidised. TOML is closer to DOML I think then most other languages (and well it does share 3/4 of the same letters though I would argue TOML's acronym is less informative and serves an 'ego' inflating purpose), they both try to be simple but I feel that TOML falls into the trap of trying to make the current solution as nice as possible where as DOML tries to solve the problem at a new angle, TOML is more akined for small config files where as DOML is nicer for files with multiple objects.

Effectively DOML is a scripting language that is constrained to purely object creation and 'editing', this allows it to be effective at what it does and makes it as low level as possible (IR brings it even closer). I would argue its still a markup language since that's what it does it notates objects. Arguably a different name could have been DOON or Data Oriented Object Notation.

## Get Involved

Anyways, if you have any new changes you may wish to add please ask in the issues!  I will be more cautious to accept PRs if the changes aren't talked about in an issue (though the obvious exception is if you are fixing up typos/errors or if its a super small thing, OR if you are adding your implementation / project to the list all these don't require issues of course). Further more I'll open up a discord chat somewhat soon (and maybe a mailing list), though this discord chat will most likely cover multiple of my projects for my sake.

## Roadmap

### Soon (within a month)

- Arrays/Dictionaries [#1]
  - Done in Version 0.2
- Constructors [#2]
  - Done in Version 0.2
- Standidize current syntax [#3]
  - Done in Version 0.2

### This Year

- Writing into IR/DOML code from already existing objects
  - I.e. serialization rather than just de-serialization
    - Though it would have to be a nice implementation
- Optimising the required IR code output (though this could become a breaking change)
  - Such as not requiring a push of the same object multiple times in a row
    - Probably could just be done by keeping the object at the bottom rather than the top and not popping the object and having the system pop it

## Projects using DOML

- Project Splinter-Grim (TBA)
  - Personal project
- If you know of any please send a PR adding to this list!

## Implementations of DOML

If you have an implementation, send a pull request adding to this list. Please note the version tag that your parser supports in your Readme.

## v0.2 Compatible

- None yet, the v0.1 compatible ones will be updated soon.

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
  - No progress has started (but it will start soon)
- [Zig]()
  - Next project

## Editor Support

> These will be missing a considerable amount of features as the language has recently changed, I'll fix them up when I get time.
- [Notepad++](https://github.com/DOML-DataOrientedMarkupLanguage/Notepad-Syntax)
- [VIM](https://github.com/DOML-DataOrientedMarkupLanguage/DOML-VIM)
- EMACS/Sublime Text/VS Code are all in development
