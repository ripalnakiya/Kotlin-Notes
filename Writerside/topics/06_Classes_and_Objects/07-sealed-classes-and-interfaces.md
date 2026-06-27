# Sealed Classes and Interfaces
<show-structure depth="2"/>

**Sealed** classes and interfaces provide **controlled inheritance** of your class hierarchies.

All direct subclasses of a sealed class are known at compile time. 
No other subclasses may appear outside the module and package within which the sealed class is defined. 

The same logic applies to sealed interfaces and their implementations: 
once a module with a sealed interface is compiled, no new implementations can be created.

> Direct subclasses are classes that immediately inherit from their superclass.
>
> Indirect subclasses are classes that inherit from more than one level down from their superclass.
>
{style="note"}

When you combine sealed classes and interfaces with the `when` expression, 
you can cover the behavior of all possible subclasses and ensure that 
no new subclasses are created to affect your code adversely.

Sealed classes are best used for scenarios when:

* **Limited class inheritance is desired:** You have a predefined, 
  finite set of subclasses that extend a class, all of which are known at compile time.
* **Type-safe design is required:** Safety and pattern matching are crucial in your project. 
  Particularly for state management or handling complex conditional logic. 
  For an example, check out [Use sealed classes with when expressions](#use-sealed-classes-with-when-expression).
* **Working with closed APIs:** You want robust and maintainable public APIs 
  for libraries that ensure that third-party clients use the APIs as intended.

## Declare a sealed class or interface

To declare a sealed class or interface, use the `sealed` modifier:

```kotlin
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

### Constructors

A sealed class itself is always an abstract class, and as a result, can't be instantiated directly.

However, it may contain or inherit constructors. 

These constructors aren't for creating instances of the sealed class itself but for its subclasses. 

Consider the following example with a sealed class called `Error` and its several subclasses,
which we instantiate:

```kotlin
sealed class Error(val message: String) {
    class NetworkError : Error("Network failure")
    class DatabaseError : Error("Database cannot be reached")
    class UnknownError : Error("An unknown error has occurred")
}

fun main() {
    val errors = listOf(Error.NetworkError(), Error.DatabaseError(), Error.UnknownError())
    errors.forEach { println(it.message) }
}
// Network failure 
// Database cannot be reached 
// An unknown error has occurred
```

You can use `enum` classes within your sealed classes to use enum constants 
to represent states and provide additional detail. 

Each enum constant exists only as a **single** instance, 
while subclasses of a sealed class may have **multiple** instances.

In the example, the `sealed class Error` along with its several subclasses, 
employs an `enum` to denote error severity.
Each subclass constructor initializes the `severity` and can alter its state:

```kotlin
enum class ErrorSeverity { MINOR, MAJOR, CRITICAL }

sealed class Error(val severity: ErrorSeverity) {
    class FileReadError(val file: File): Error(ErrorSeverity.MAJOR)
    class DatabaseError(val source: DataSource): Error(ErrorSeverity.CRITICAL)
    object RuntimeError : Error(ErrorSeverity.CRITICAL)
    // Additional error types can be added here
}
```

Constructors of sealed classes can have one of two visibilities: 
`protected` (by default) or `private`:

```kotlin
sealed class IOError {
    // A sealed class constructor has protected visibility by default. It's visible inside this class and its subclasses 
    constructor() { /*...*/ }

    // Private constructor, visible inside this class only. 
    // Using a private constructor in a sealed class allows for even stricter control over instantiation, enabling specific initialization procedures within the class.
    private constructor(description: String): this() { /*...*/ }

    // This will raise an error because public and internal constructors are not allowed in sealed classes
    // public constructor(code: Int): this() {} 
}
```


## Use sealed classes with when expression

The key benefit of using sealed classes comes into play when you use them in a `when` expression.

The `when` expression, used with a sealed class, 
allows the Kotlin compiler to check exhaustively that all possible cases are covered.

In such cases, you don't need to add an `else` clause:

```kotlin
// Sealed class and its subclasses
sealed class Error {
    class FileReadError(val file: String): Error()
    class DatabaseError(val source: String): Error()
    object RuntimeError : Error()
}

// Function to log errors
fun log(e: Error) = when(e) {
    is Error.FileReadError -> println("Error while reading file ${e.file}")
    is Error.DatabaseError -> println("Error while reading from database ${e.source}")
    Error.RuntimeError -> println("Runtime error")
    // No `else` clause is required because all the cases are covered
}

// List all errors
fun main() {
    val errors = listOf(
        Error.FileReadError("example.txt"),
        Error.DatabaseError("usersDatabase"),
        Error.RuntimeError
    )

    errors.forEach { log(it) }
}
```

When using sealed classes with `when` expressions, 
you can also add guard conditions to include additional checks in a single branch.


## Use case scenarios

Let's explore some practical scenarios where sealed classes and interfaces can be particularly useful.

[Visit Kotlin Documentation](https://kotlinlang.org/docs/sealed-classes.html#use-case-scenarios)
