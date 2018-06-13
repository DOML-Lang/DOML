# Data Oriented Markup Language - DOML

![DOML](./DOML.png=250x250)

> By Braedon Wooding
> Latest Version 0.3
>> The spec could change and break previous code however this will be avoided (i.e. semi-stable), in saying that 0.2 -> 0.3 was a completely breaking set of changes that invalidated all old DOML code, however in saying that I don't intend to do that again :D.

## Introduction

DOML is more like a scripting language then it is like a markup language like JSON/XML/TOML/...  Essentially DOML works as such; a base markup like document that looks like a mix between a simpler JSON and something like python/lua then that gets converted into an IR format which looks quite similar to something like a higher level assembly.

#### The main goals of DOML are;

- To define objects rather than describe them, that is the objects are created natively rather than requiring some transposation of the 'map'
- The converted IR is quicker and easier to pass as well as being insanely efficient for a markup language, it has a binary stream format allowing it to be passed efficiently.
- DOML is aimed to be both simple, easy to read, and easy to write.  It is aimed towards coders rather than towards those who haven't coded before.
- DOML is 100% whitespace insensitive
- DOML has no keywords with true and false being the only 'keywords' and they are context sensitive, and you can use `#` to refer to the object over the value i.e. `#true.x` refers to an object you created call true.
- DOML is completely safe since it relies on static/reflective bindings to build the linkages to your own code and you have the control of what objects can be created and what functions can be run, it disallows anonymous object creation.
- DOML has no requirement for the user to create parsing code, one of the key aspects of the spec is to emphasise allowing users to link DOML into their code with no parsing functions required often utilising static/reflective bindings, on some systems where this isn't possible it is suggested to try to build static analysers instead of asking users to give function ptrs/equivalent.

## A quick overview

```C
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
```

When you put this into a parser you'll get the below output (its standidized so you **will** get the below output - though the supplementary comments for each line may differ, though I've not included all the comments to keep it more concise and short);

```assembly
; This is the resulting bytecode from the file given
; This bytecode will be overriden if new bytecode is generated.
#IR simple {
  init 4 4
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

  ; Since declaring the array here will be annoying we can just
  ; dumbly get the value then later tell its type (not very efficient but its simple)
  dumbget #MyTags Tags GetTags
  quickindexarray str 0
  quickcall #MyTags Tags Name 1

  push flt 3 0.5 1.2 3.5
  newobj Color Normalized #MyDictionary__Bob 3
  quickpush str "Bob's Color"
  quickcall #MyDictionary__Bob Color Name 1
}
```
> Unlike the previous format of DOML the resultant assembly from commands isn't always set in stone the compiler is free to perform some more creative optimisations with data, for example a compiler could support you ensuring that the `SetTags == GetTags` that is no manipulation is occurring there then it could simply just do a;
```assembly
quickpush str 1 "Hello"
quickcall #MyTags Tags Name 1
```
Rather than that slower `dumbget` which effectively doesn't perform type safety checks this does mean that your code isn't as safe as it would be with a normal `get` however in this case its fine since we are writing it, the compiler shouldn't typically write code like this; and would probably either use `createtype` and a `getcollection` or a miriad of other things that it could perform that would generally either be more efficient or more safe, however a lot of those aren't very friendly to read so I opted just for the much more readable `dumbget`.

## Shortened Format

Sometimes the problem with JSON is that it just bulks up so much, so DOML provides a few ways to shorten your scripts;

An initial doml script;
```C
Wizard : Character {
  .Name = "Wizard the Great",
  .Stats = {
    { Character.Stat.HP : 2 },
    { Character.Stat.AP : 9 },
    { Character.Stat.ST : 3 },
    // And so on
  },
  .Spells = [
      Spell::Fireball(),
      Spell::New() {
        .Name = "Polymorphism",
        .EffectScript = "Polymorphism.lua",
      }
  ],
}
```
You could reduce this down to;
```C
Wizard = Character::New("Wizard the Great") {
  // If the object is a enum like in this case, you can scope it like (works with some other things too)
  .Stats : [Character.Stat : Int] = { HP : 4 }, { AP : 9 }, { ST : 3 }
  .Spells : [Spell] = [Fireball(), NewLua(name: "Polymorphism", script: "Polymorphism.Lua")]
}
```
As you can see it is partly due to building a good API and partly due to a mix of other tools even if NewLua didn't exist that spell call would just be;
```C
  .Spells : [Spell] = [Fireball(), New() { .Name = "Polymorphism, .EffectScript = "Polymorphism.Lua" }]
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

> Note: decimals have standidised for `$` though many parsers will probably allow the various other currency signs.

## Collections


| Type          | Example                               | Details (all suffixes are case insensitive)        |
| ------------- | ------------------------------------- | -------------------------------------------------- |
| Arrays        | \[1, 2, 3, 4\]                        | All have to be of the same type                    |
| Dictionary    | { { "X" : 2 }, { "Y" : 9 } }          | All keys/values have to be same type (key != value)|

## Comparison with other formats

> I've re-written this so many times because DOML is so different it almost accomplishes the same goal completely differently. So its hard to compare.

Effectively DOML is simpler and more efficient then the other data formats. XML is hard for machines to parse, JSON is overly complex and hard to write, YAML is way to complex (no parser is fully 1.1 compliant and there are basically no 1.2 parsers at all), and INI isn't standidised. TOML is closer to DOML I think then most other languages (and well it does share 3/4 of the same letters though I would argue TOML's acronym is less informative and serves an 'ego' inflating purpose), they both try to be simple but I feel that TOML falls into the trap of trying to make the current solution as nice as possible where as DOML tries to solve the problem at a new angle, TOML is more akined for small config files where as DOML is nicer for files with multiple objects.

Effectively DOML is a scripting language that is constrained to purely object creation and 'editing', this allows it to be effective at what it does and makes it as low level as possible (IR brings it even closer). I would argue its still a markup language since that's what it does it notates objects. Arguably a different name could have been DOON or Data Oriented Object Notation.

## Get Involved

Anyways, if you have any new changes you may wish to add please ask in the issues!  I will be more cautious to accept PRs if the changes aren't talked about in an issue (though the obvious exception is if you are fixing up typos/errors or if its a super small thing, OR if you are adding your implementation / project to the list all these don't require issues of course). Further more I'll open up a discord chat somewhat soon (and maybe a mailing list), though this discord chat will most likely cover multiple of my projects for my sake.

## Roadmap

> TODO

## Projects using DOML

- If you know of any please send a PR adding to this list!

## Implementations of DOML

If you have an implementation, send a pull request adding to this list. Please note the version tag that your parser supports in your Readme.

## v0.3 Compatible Compilers

- None yet, C#, GO, and Zig compatible ones will be updated soon.

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
