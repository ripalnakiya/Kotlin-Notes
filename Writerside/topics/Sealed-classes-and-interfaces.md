# Sealed Classes and Interfaces

**Sealed** classes and interfaces provide **controlled inheritance** of your class hierarchies.

All direct subclasses of a sealed class are known at compile time. 

No other subclasses may appear outside the module and package within which the sealed class is defined. 

The same logic applies to sealed interfaces and their implementations: once a module with a sealed interface is compiled, no new implementations can be created.

<note>
Direct subclasses are classes that immediately inherit from their superclass.

Indirect subclasses are classes that inherit from more than one level down from their superclass.
</note>

When you combine sealed classes and interfaces with the `when` expression, you can cover the behavior of all possible subclasses and ensure that no new subclasses are created to affect your code adversely.

Sealed classes are best used for scenarios when:

- **Limited class inheritance is desired:** You have a predefined, finite set of subclasses that extend a class, all of which are known at compile time.

- **Type-safe design is required:** Safety and pattern matching are crucial in your project. Particularly for state management or handling complex conditional logic. For an example, check out Use sealed classes with when expressions.

- **Working with closed APIs:** You want robust and maintainable public APIs for libraries that ensure that third-party clients use the APIs as intended.

## Declare a sealed class or interface

```Kotlin
// Create a sealed interface
sealed interface Error

// Create a sealed class that implements sealed interface Error
sealed class IOError(): Error

// Define subclasses that extend sealed class 'IOError'
class FileReadError(val file: File): IOError()
class DatabaseError(val source: DataSource): IOError()

// Create a singleton object implementing the 'Error' sealed interface
object RuntimeError : Error
```

## Constructors

A sealed class itself is always an abstract class, and as a result, can't be instantiated directly.

However, it may contain or inherit constructors.

These constructors aren't for creating instances of the sealed class itself but for its subclasses.

Consider the following example with a sealed class called Error and its several subclasses, which we instantiate:

```Kotlin
sealed class Error(val message: String) {
    class NetworkError : Error("Network failure")
    class DatabaseError : Error("Database cannot be reached")
    class UnknownError : Error("An unknown error has occurred")
}

fun main() {
    // Creating instances of the subclasses
    val errors = listOf(Error.NetworkError(), 
                        Error.DatabaseError(), 
                        Error.UnknownError())
    errors.forEach { println(it.message) }
}

// Output:
Network failure 
Database cannot be reached 
An unknown error has occurred
```

You can use `enum` classes within your sealed classes to use enum constants to represent states and provide additional detail. 

Each enum constant exists only as a **single instance**, while subclasses of a sealed class may have **multiple instances**. 

In the example, the `sealed class Error` along with its several subclasses, employs an enum to denote error severity. Each subclass constructor initializes the severity and can alter its state:

```Kotlin
enum class ErrorSeverity { MINOR, MAJOR, CRITICAL }

sealed class Error(val severity: ErrorSeverity) {
    class FileReadError(val file: File): Error(ErrorSeverity.MAJOR)
    class DatabaseError(val source: DataSource): Error(ErrorSeverity.CRITICAL)
    object RuntimeError : Error(ErrorSeverity.CRITICAL)
    // Additional error types can be added here
}
```

Constructors of sealed classes can have one of two visibilities: protected (by default) or private:

```Kotlin
sealed class IOError {
    // A sealed class constructor has protected visibility by default. 
    // It's visible inside this class and its subclasses
    constructor() { /*...*/ }

    // Private constructor, visible inside this class only.
    // Using a private constructor in a sealed class
    // allows for even stricter control over instantiation, 
    // enabling specific initialization procedures within the class.
    private constructor(description: String): this() { /*...*/ }

    // This will raise an error because 
    // public and internal constructors are 
    // not allowed in sealed classes
    public constructor(code: Int): this() {}
}
```

## Inheritance

Direct subclasses of sealed classes and interfaces must be declared in the same package. 

They may be top-level or nested inside any number of other named classes, named interfaces, or named objects. 

Subclasses can have any visibility as long as they are compatible with normal inheritance rules in Kotlin.

Subclasses of sealed classes must have a properly qualified name. They can't be local or anonymous objects.

<note>

enum classes can't extend a sealed class, or any other class. However, they can implement sealed interfaces:
```Kotlin
sealed interface Error

// enum class extending the sealed interface Error
enum class ErrorType : Error {
    FILE_ERROR, DATABASE_ERROR
}
```
</note>

These restrictions don't apply to indirect subclasses. If a direct subclass of a sealed class is not marked as sealed, it can be extended in any way that its modifiers allow:

```Kotlin
// Sealed interface 'Error' has implementations only 
// in the same package and module
sealed interface Error

// Sealed class 'IOError' extends 'Error' and 
// is extendable only within the same package
sealed class IOError(): Error

// Open class 'CustomError' extends 'Error' and 
// can be extended anywhere it's visible
open class CustomError(): Error
```

## Use sealed classes with when expressions

The key benefit of using sealed classes comes into play when you use them in a when expression. The when expression, used with a sealed class, allows the Kotlin compiler to check exhaustively that all possible cases are covered. In such cases, you don't need to add an else clause:

```Kotlin
// Function to log errors
fun log(e: Error) = when(e) {
    is Error.FileReadError -> println("Error while reading file ${e.file}")
    is Error.DatabaseError -> println("Error while reading from database ${e.source}")
    Error.RuntimeError -> println("Runtime error")
    // No `else` clause is required because all the cases are covered
}
```

## Use case scenarios

TODO:

[Kotlin Docs](https://kotlinlang.org/docs/sealed-classes.html#use-case-scenarios)