# Types Overview

In Kotlin, everything is an object in the sense that 
you can call member functions and properties on any variable. 

While certain types have an optimized internal representation 
as primitive values at runtime (such as numbers, characters, and booleans), 
they appear and behave like regular classes to you.

- This section describes the **basic types used in Kotlin**:
  - Numbers
  - Booleans
  - Characters
  - Strings
  - Arrays

- To learn about other **Kotlin types:**
  - **`Any`:** The root of the Kotlin class hierarchy. 
           Every Kotlin class has Any as a superclass.
  - **`Nothing`:** Nothing has no instances. 
               You can use Nothing to represent "a value that never exists": 
               for example, if a function has the return type of Nothing, 
               it means that it never returns (always throws an exception).
  - **`Unit`:** The type with only one value: the `Unit` object. 
            This type corresponds to the `void` type in Java.
