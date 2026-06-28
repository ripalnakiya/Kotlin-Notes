# Types Overview

In Kotlin, everything is an object in the sense that 
you can call member functions and properties on any variable. 

While certain types have an optimized internal representation 
as primitive values at runtime (such as numbers, characters, and booleans), 
they appear and behave like regular classes to you.

- This section describes the **basic types used in Kotlin**:
  - [](02-numbers.md)
  - [](03-booleans.md)
  - [](04-characters.md)
  - [](05-strings.md)
  - [](06-arrays.md)

- To learn about other **Kotlin types:**
  - [**`Any`**](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-any/): 
      The root of the Kotlin class hierarchy. 
      Every Kotlin class has Any as a superclass.
  - [**`Nothing`**](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-nothing.html):
      Nothing has no instances. 
      You can use Nothing to represent "a value that never exists": 
      for example, if a function has the return type of Nothing, 
      it means that it never returns (always throws an exception).
  - [**`Unit`** ](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-unit/):
      The type with only one value: the `Unit` object. 
      This type corresponds to the `void` type in Java.

> [Learn how to perform type checks and casts in Kotlin](07-typecasts.md).
>
{style="tip"}
