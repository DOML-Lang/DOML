## What is this?
This is the formal specification document for the system library to exist to be compliant with this specification.  Note: you aren't required to include the system library to be compliant with the DOML specification as a whole, though you will only count as being partly compliant.

## Goals
- To provide functionality to reduce boiler plate code, effectively this should map to the standard library that comes with the language quite easily.
- Small and portable

## Terminology
- Setter refers to a function that takes parameters and performs some manipulation on data
- Getter refers to a function that returns a value or multiple
- Static refers to a function that doesn't require an object instance

## Table of Contents
- [Math](#math)
  - [IntMath](#intmath)
  - [DecMath](#decmath)

## Math
The math object allows you to perform various calculations on values.  It is created like `@ math = System.Math`.  Math has an internal storage of a double.

#### List of Commands
###### Setters
- `System.Math::Add` : Parameters can be numbers, integers, or decimals and size of is `-1` (at-least one parameters)
  - Adds all the parameters together and stores in internal storage (i.e. `internalStorage += sum(parameters)`).
- `System.Math::Subtract` : Parameters can be numbers, integers, or decimals and size of is `-1` (at-least one parameters)
  - Adds all the parameters together and then subtracts that value from the internal storage and places that result in internal storage (i.e. `internalStorage -= sum(parameters)`).
- `System.Math::Multiply` : Parameters can be numbers, integers, or decimals and size of is `-1` (at-least one parameters)
  - Multiplies all the parameters together and then multiplies that value by the internal storage and places that result in internal storage (i.e. `internalStorage *= parameters[0] * parameters[1] * ... * parameters[n]`).
- `System.Math::Divide` : Parameters can be numbers, integers, or decimals and size of is `-1` (at-least one parameters)
  - Multiplies all the parameters together and then divides the internal storage by that value and places that result in internal storage (i.e. `internalStorage /= parameters[0] * parameters[1] * ... * parameters[n]`).
    - Effectively equivalent to saying `x / a / b / c / d` which is just `x / (a * b * c * d)`
- `System.Math::Power` : Parameters can be numbers, integers, or decimals and size of is `-1` (at-least one parameters)
  - Places all the parameters to the power of the one on the right (till there is none left) and then places the internal storage to the power of that value and places that result in internal storage (i.e. `internalStorage = internalStorage ^ (parameters[0] ^ parameters[1] ^ ... ^ parameters[n])`).
- `System.Math::Modulus` : Parameters can be numbers, integers, or decimals and size of is `-1` (at-least one parameters)
  - Performs modulus on all the parameters together and then divides the internal storage by that value and places that result in internal storage (i.e. `internalStorage = internalStorage % (parameters[0] % parameters[1] % ... % parameters[n])`).

###### Getters
- `System.Math::Get.Number` : Returns a single double which represents the internal storage, no parameters.
- `System.Math::Get.String` : Returns a single string which represents the internal storage as a string, no parameters.
- `System.Math::Get.Int` : Returns a single integer which represents the internal storage as an int.
  - Note: Casting to integer is fine since `System.IntMath` is built for purely integer calculations.
- `System.Math::Get.Decimal` : Returns a single decimal which represents the internal storage as a decimal.
  - Note: Casting to decimal is fine since `System.DecMath` is built for purely decimal calculations.

###### Static Setters
- `System.Math::Random.Seed` : Parameter is a single integer value, thus size of is `1` (only one parameter).
  - Sets the random generator seed.
- `System.Math::Random.Range` : Parameter is two double values, thus size of is `2` (only two parameters).
  - Sets the range to generate numbers from.

###### Static Getters
- `System.Math::Random` : Returns a single double that holds a value between the lower and maximum bounds (defaulted to 0 and 1, though note you can change it through `System.Math::Random.Range`.  Effected by `System.Math::Random.Seed`.  Size of is `1` since it returns a single parameter.

## IntMath
For simplicity I won't repeat them here, though the commands are the exact same as `System.Math` except they have the `System.IntMath` prefix and it works on purely integers (it won't accept doubles or decimals), this means its internal storage is an integer.

## DecMath
For simplicity I won't repeat them here, though the commands are the exact same as `System.Math` except they have the `System.DecMath` prefix and it works on purely decimals (it won't accept doubles or integers), this means its internal storage is a decimal.
