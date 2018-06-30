# Data Oriented Markup Language - DOML

<img src="./DOML.png" width="100">

> By Braedon Wooding
> Latest Version 0.3
>> The spec could change and break previous code however this will be avoided (i.e. semi-stable), in saying that 0.2 -> 0.3 was a completely breaking set of changes that invalidated all old DOML code, however in saying that I don't intend to do that again :D.

## What is DOML?

DOML is simple - it's a markup language akin to JSON/YAML/XML/TOML/.../ with the usability of a language like LUA.

## A quick overview

```C
# Version 0.3
// Construct a new Color
Test : Color {
  RGB = 255, 64, 128,
}

// Constructors do exist
// the parameter names are purely for your own merit, they will check if its possible however (will be possible on most systems)
TheSame : Color::Normalized(r: 1, g: 0.25, b: 0.5) {
  Name = "Bob"
}

// You can also just declare an object without scoping it
Other : Color
Other.Name = "X"

// You can also edit the original Test at any point EITHER by doing
Test.R = 50
// Or by doing
Test.{
  G = 128
}

// You can declare arrays like
ArrayObject : []Color {
  ::Normalized(0.95, 0.55, 0.22){
    Name = "Other", // Trailing commas are always allowed
  },
  // You can still do an empty construction
  ::() {
    RGB = 50, 25, 125,
  },
  // And thus you can leave out the ::()
  {
    RGB = 50, 25, 125,
  },
}

// You can also copy objects by doing
NewObj : Color = Other

// Or can do something like
NewObj.Name = ArrayObject[0].Name

// You can also declare arrays inside object definitions
MyTags : Tags {
  // Note: all have to be of the same type
  SetTags = ["Hello", "Other", "bits", "bobs", "kick"]
  Name = MyTags.GetTags[0] // And indexing them works like you would think
}

// You can declare dictionaries like
// Dictionaries within objects can also be created similarly
MyDictionary : [String : Color] {
  { 
    "Bob" : Color::Normalized(0.5, 1.2, 3.5) {
      Name = "Bob's Color"
    }
  },
}
// No need to keep classes around in this example
# Deinit all
```

When you put this into a parser you'll get something like this; compilers are free to optimise by implementing quick calls and other methods, also almost all compilers suppport adding additional comments to each line to clearly demonstrate what each line does (for when you want to inspect the produced code).  Furthermore the actual IR that you will use won't be in `simple` mode (which is meant mainly for when you want to read/tweak it) since the excessive use of strings is inefficient.

```assembly
; This is the resulting bytecode from the file given
; This bytecode will be overriden if new bytecode is generated.
# IR simple {
  ; Sets up a cache
  # IR_obj Color
  # IR_obj Tags
  # IR_ctor Tags::Tags
  # IR_ctor Color::Color
  # IR_ctor Color::Normalized
  # IR_set Color.RGB
  # IR_set Color.Name
  # IR_get Color.Name
  # IR_set Color.R
  # IR_set Color.G
  # IR_set Tags.SetTags
  # IR_get Tags.GetTags
  # IR_set Tags.Name
  init 4 4 ; Initialises the stack and registers
  ; This is a comment
  ; Construct a new Color
  newobj Color Color #Test 0
  push int 3 255 64 128
  calln #Test Color RGB 3

  push flt 3 1 0.25 0.5
  newobj Color Normalized #TheSame 3
  quickpush str 1 "Bob"
  quickcall #Test Color Name 1

  newobj Color Color #Other 0
  quickpush str 1 "X"
  quickcall #Other Color Name 1

  quickpush int 1 50
  quickcall #Test Color R 1

  quickpush int 1 128
  quickcall #Test Color G 1

  push flt 3 0.95 0.55 0.22
  newobj Color Normalized #ArrayObject__0 3
  quickpush str 1 "Other"
  quickcall #ArrayObject__0 Color Name 1

  newobj Color Color #ArrayObject__1 0
  push int 3 50 25 125
  calln #ArrayObject__1 Color RGB 3

  newobj Color Color #ArrayObject__2 0
  push int 3 50 25 125
  calln #ArrayObject__2 Color RGB 3

  ; In some cases a `copyobj` call be be done in this case we can just copy the IR
  newobj Color Color #NewObj 0
  quickpush str 1 "X"
  quickcall #NewObj Color Name 1

  quickget str #ArrayObject__0 Color Name
  quickcall #NewObj Color Name 1

  newobj Tags Tags #MyTags 0
  pusharray str 5
  arraycpy str 5 "Hello" "Other" "bits" "bobs" "kick"
  calln #MyTags Tags SetTags 1

  ; This often will be optimised away as dumbget is less safe and less efficient
  dumbget #MyTags Tags GetTags
  quickindexarray str 0
  quickcall #MyTags Tags Name 1

  push flt 3 0.5 1.2 3.5
  newobj Color Normalized #MyDictionary__Bob 3
  quickpush str "Bob's Color"
  quickcall #MyDictionary__Bob Color Name 1
}
```

## Shortened Format

Sometimes the problem with JSON is that it just bulks up so much, so DOML provides a few ways to shorten your scripts;

An initial doml script;
```C
Wizard : Character {
  Name = "Wizard the Great",
  Stats = {
    { Character.Stat.HP : 2 },
    { Character.Stat.AP : 9 },
    { Character.Stat.ST : 3 }
    // And so on
  },
  Spells = [
      Spell::Fireball(),
      Spell::New() {
        Name = "Polymorphism",
        EffectScript = "Polymorphism.lua"
      }
  ]
}
```
You could reduce this down to (using the idea of scoping variables)
```C
Wizard = Character {
  Name = "Wizard the Great",
  Stats : [Character.Stat : Int] = { HP : 4 }, { AP : 9 }, { ST : 3 }
  Spells : [Spell] = [Fireball(), New() { Name = "Polymorphism", EffectScript = "Polymorphism.Lua" }]
}
```
Basically as vertical space often makes things seem longer than horizontal we allow you to expand horizontally quite nicely.

## Types

| Type          | Example Values                        | Details (all suffixes are case insensitive)        |
| ------------- | ------------------------------------- | -------------------------------------------------- |
| Integer       | 12, -40, +2, 01010b, 0x40FF, 0o42310  | 0b for binary, 0x for hex, 0o for octal            |
| Double        | 10.5, 20, 5e+22, 1e6, -2.54E-2        | E for exponent                                     |
| Decimals      | $40, +$99.05, -$4e+22                 | Can use E for exponent and '$' refers to decimal   |
| String        | "This contains a \" escaped quote"    | "...", you can escape `"` with `\`                 |
| Boolean       | true, false                           | The boolean values                                 |
| Object        | Test, X, MyColor                      | Refers to a previously defined object              |

> Note: decimals have standidized for `$` though many parsers will probably allow the various other currency signs.

## Collections


| Type          | Example                               | Details (all suffixes are case insensitive)        |
| ------------- | ------------------------------------- | -------------------------------------------------- |
| Arrays        | \[1, 2, 3, 4\]                        | All have to be of the same type                    |
| Dictionary    | { { "X" : 2 }, { "Y" : 9 } }          | All keys/values have to be same type (key != value)|

## Actually using objects created

There are a multitude of ways to use the objects i.e. actually give them to the application, often applications have some kind of 'registerX' function that registers the objects so I'll follow that in this example.
```C
// First you could just do it on a base by base basis
Wizard = Character {
  Name = "Wizard the Great",
  Stats : [Character.Stat : Int] = { HP : 4 }, { AP : 9 }, { ST : 3 }
  Spells : [Spell] = [Fireball(), New() { Name = "Polymorphism", EffectScript = "Polymorphism.Lua" }]
}.Register();
// Can also put it on its own line
Wizard.Register();
// Or do the identical call
Character.Register(Wizard); // Presuming that it is a static method
// Or do a localized call (calls Register on all characters)
#LocalizedCall Character Register
// Or just a mass call (calls Register on every object)
#MassCall Register
```

## Serialization

Compilers will support serialization, however they may just support it through a binary serialization (i.e. an outputted DOML IR file which isn't text readable), all 'official' compilers will however support it in any format both binary, text readable and actual 'DOML' output.

## Comparison with other formats

Hopefully the code examples clearly demonstrate DOML's strengths, it is built for programmers as a de-serializable format to enable them to store data efficiently and effectively; it excels at being readable and simple (it's grammar is extremely simple compared to any language out there), while it is similar to markup languages and similar to scripting languages I feel it sits somewhat in the middle, it is as useful as using LUA for your data management but as simple as JSON.

## Get Involved

Anyways, if you have any new changes you may wish to add please ask in the issues!  I will be more cautious to accept PRs if the changes aren't talked about in an issue (though the obvious exception is if you are fixing up typos/errors or if its a super small thing, OR if you are adding your implementation / project to the list all these don't require issues of course).

Join the discord here; https://discord.gg/hWyGJVQ.  This will be shared for a series of my various projects not just DOML.

## Roadmap

- V0.3.

## Projects using DOML

- If you know of any please send a PR adding to this list!

## Implementations of DOML

If you have an implementation, send a pull request adding to this list. Please note the version tag that your parser supports in your Readme.

## v0.3 Compatible Compilers

- [C#](https://github.com/DOML-Lang/DOML.net)
  - Currently is about 65% complete.

## In Progress

- [C++](https://github.com/DOML-Lang/DOML-Cxx)
  - No progress has started (but it will start soon)
- [Go](https://github.com/DOML-Lang/DOML-GO)
  - Slightly v0.2 compatible but not v0.3 compatible yet.
- [Zig]()
  - Currently not possible to implement to the standard I require as the language is too 'young' as is evolving too rapidly.

## Do you have a compiler in a language that you want to work on?

- Go ahead!  I won't make any compiler 'official' (i.e. under the DOML-Lang 'company') till it is finished but it can definitely go on this list till then!

## Editor Support

> These will be missing a considerable amount of features as the language has recently changed, I'll fix them up when I get time.
- [Notepad++](https://github.com/DOML-Lang/Notepad-Syntax) OUTDATED
- [VIM](https://github.com/DOML-Lang/DOML-VIM) OUTDATED
- [VS-Code](https://github.com/DOML-Lang/vscode-DOML)
- EMACS/Sublime Text/... are all in development (and will be released soonish)
