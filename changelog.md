# Changelog

This changelog will contain all the various changes to the grammar and syntax of DOML from v0.1 onwards.

> *Note:* There are a few large steps such as Version 0.1->0.2, this was mainly done when there was very few people working on the project (originally only one) and it was easier then having to constantly release new versions.

## Version 0.2

Version 0.2 didn't bring any massive changes but it did bring arrays and dictionaries.

- Arrays
  - `X.Y = [1, 2, 3]`
  - `18/pushvec (int)` command added
  - `19/pushmap (int)` command added
- Dictionaries
  - `X.Y = ["K": 1, "B": 2, "C": 3]`
- Short Function calls
  - `@ X->Y = 2` which translates to `@ _ = X.X ... .Y = 2`
    - i.e. `@ System->CallWithInt = 9` is equivalent to `@ _ = System.System ... .CallWithInt = 9`
    - The double X may be removed in the future but really it exists just to make sure you don't pollute the namespace and that you specify these calls can only exist for certain things.
      - The `_` object will only be created once despite how many calls you perform to save memory (this should be fine since its only meant to register things and no state should really be kept)
- A standidized syntax (really existed in like 0.15 or whatever)

## Pre-Version (leading up to and including) 0.1

There were many changes made here, I'm highlighting a few just to demonstrate the various ways the language has evolved, many of these changes lasted less than a week and were effectively 'litmus' tests.

- Syntax changes
  - Originally you had a `;` at the beginning of each 'set' statement (i.e. `; X.Y = 2`),
  this was done to make parsing easier and I felt it made the language simpler, it was only once I started writing out DOML I found how annoying it was to maintain the `;` and forgetting one was infuriating, so I removed this; you can still place one at the end to put multiple on the same line.
    - This is actually why the implementation of C# still treats the `;` as the start, and just allows you to also use a newline to denote the start just a side effect of this shift that hasn't been changed (due to not needing to be changed).
  - Removal of the shortcut; `; .{x = 4; y = 2; z = 9}`
    - The current way which is `.x = 4; .y = 2; .z = 9` is only more work when you can't just use `.` which is rare, and not worth introducing braces for, furthermore embedding IR uses `{ ... }` so basically it just became useless
- IR
  - So many commands have been removed, previously there were `CheckIsInt` and so on for each of the types (with ones for 8, 16, 32, and 64 bit integers along with all the sizes for the other types), there was also panic ones to produce an error, effectively they just checked you had the right number of parameters this was later done by the compiler so there was no need for commands.
- Multiple type sizes
  - From byte sized integers to 128 bit sized integers (8, 16, 32, 64, and 128), floats went from half to quad, there was characters (8 and 16), strings came in narrow and wide, decimals also had 64 and 128.
  - Removed because of the complexity, the newer system keeps the flexibility but removes all the complexity for just a small cast cost (which is relatively minor and the impact is more the increased memory usage).
- Arrays and Dictionaries were removed and explicity stated to become never a thing
  - This changed once I saw the evidence that it was very clear that arrays should exist (and same for dictionaries), I came across this evidence myself when I built the C# parser (and it was even more clear with the GO one)
  - Mainly I had declared them explicity gone to maintain simplicity, I still doubt that dictionaries really 'need' to exist, however since arrays will exist I might as well do both at the same time (they can always be removed later).
    - The reason they need to exist however is that to maintain compile time reliability we require the number of parameters (it also helps with parsing), later on most parsers will also store the types of the parameters to further help with producing compile time safety (i.e. if the code compiles the IR will 100% run with no errors unless the user's code has specific range requirements like a float between 0 and 1).  This means that you need a specific number of arguments (you can declare a negative number/null parameter list to obtain any amount but this basically just disables checking except for type), so arrays came in as a solution.
