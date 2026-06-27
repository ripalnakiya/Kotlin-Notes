# Enum Classes
<show-structure depth="2"/>

The most basic use case for enum classes is the implementation of type-safe enums:

```Kotlin
enum class Direction {
    NORTH, SOUTH, WEST, EAST
}
```

Each enum constant is an object. Enum constants are separated by commas.

Since each enum is an instance of the enum class, it can be initialized as:

```Kotlin
enum class Color(val rgb: Int) {
    RED(0xFF0000),
    GREEN(0x00FF00),
    BLUE(0x0000FF)
}
```

## Anonymous Classes

Enum constants can declare their own anonymous classes with their corresponding methods, 
as well as with overriding base methods.

```Kotlin
enum class ProtocolState {
    WAITING {
        override fun signal() = TALKING
    },

    TALKING {
        override fun signal() = WAITING
    };

    abstract fun signal(): ProtocolState
}
```

If the enum class defines any members, 
separate the constant definitions from the member definitions with a semicolon.

## Implementing interfaces in enum classes

An enum class can implement an interface (but it cannot derive from a class), 
providing either a common implementation of interface members for all the entries, 
or separate implementations for each entry within its anonymous class.

```Kotlin
interface Printable {
    fun print()
}

interface Log {
    fun log()
}

enum class Priority : Printable, Log {
    HIGH {
        override fun print() {
            println("High priority")
        }
    },
    MEDIUM {
        override fun print() {
            println("Medium priority")
        }
    },
    LOW {
        override fun print() {
            println("Low priority")
        }
    };

    override fun log() = print()
}

fun main() {
    Priority.HIGH.print()  
    Priority.MEDIUM.print()
    Priority.LOW.print()    

    Priority.LOW.log()
}
```

Output: 
```
High priority
Medium priority
Low priority
Low priority
```

## Working with enum constants

Enum classes in Kotlin have synthetic properties and methods for listing the defined enum constants and getting an enum constant by its name. 

The signatures of these methods are as follows (assuming the name of the enum class is `EnumClass`):

```Kotlin
EnumClass.valueOf(value: String): EnumClass

EnumClass.entries: EnumEntries<EnumClass> 
// specialized List<EnumClass>
```

Below is an example of them in action:

```Kotlin
enum class RGB { RED, GREEN, BLUE }

fun main() {
    for (color in RGB.entries) 
        println(color.toString()) 
    // prints RED, GREEN, BLUE
    
    val first: RGB = RGB.valueOf("RED")
    println("The first color is: $first")
    // prints "The first color is: RED"
}
```

The `valueOf()` method throws an `IllegalArgumentException` if the specified name does not match any of the enum constants defined in the class.

Every enum constant also has properties: `name` and `ordinal`, for obtaining its name and position (starting from 0) in the enum class declaration:

```Kotlin
println(RGB.RED.name)    // prints RED
println(RGB.RED.ordinal) // prints 0
```
