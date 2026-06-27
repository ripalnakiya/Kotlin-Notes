# Exceptions
<show-structure depth="2"/>

Exceptions help your code run more predictably, 
even when runtime errors occur that could disrupt program execution.

Kotlin treats all exceptions as **unchecked** by default.

Unchecked exceptions simplify the exception handling process: 
you can catch exceptions, but you don't need to explicitly handle or declare them. 

- Working with exceptions consists of two primary actions:
  - **Throwing exceptions:** indicate when a problem occurs.
  - **Catching exceptions:** handle the unexpected exception manually 
    by resolving the issue or notifying the developer or application user.

Exceptions are represented by subclasses of the `Exception` class, 
which is a subclass of the `Throwable` class. 

For more information about the hierarchy, see the [Exception hierarchy](#exception-hierarchy) section. 

Since `Exception` is an `open class`, 
you can create [custom exceptions](https://kotlinlang.org/docs/extensions.html#generic-extension-functions) 
to suit your application's specific needs.

## Throw exceptions

You can manually throw exceptions with the `throw` keyword.

Throwing an exception indicates that an 
unexpected runtime error has occurred in the code.

Exceptions are objects, and throwing one creates an instance of an exception class.

You can throw an exception without any parameters:

```kotlin
throw IllegalArgumentException()
```

To better understand the source of the problem, 
include additional information, such as a custom message and the original cause:

```kotlin
val cause = IllegalStateException("Original cause: illegal state")

// Throws an IllegalArgumentException if userInput is negative 
// Additionally, it shows the original cause, represented by the cause IllegalStateException
if (userInput < 0) {
    throw IllegalArgumentException("Input must be non-negative", cause)
}
```

In this example, an `IllegalArgumentException` is thrown 
when the user inputs a negative value.

You can create custom error messages and 
keep the original cause (`cause`) of the exception, 
which will be included in the stack trace.

### Throw exceptions with precondition functions

Kotlin offers additional ways to automatically throw exceptions 
using precondition functions. 

Precondition functions include:

| Precondition function            | Use case                                 | Exception thrown           |
|----------------------------------|------------------------------------------|----------------------------|
| [`require()`](#require-function) | Checks user input validity               | `IllegalArgumentException` |
| [`check()`](#check-function)     | Checks object or variable state validity | `IllegalStateException`    |
| [`error()`](#error-function)     | Indicates an illegal state or condition  | `IllegalStateException`    |

These functions are suitable for situations 
where the program's flow cannot continue if specific conditions aren't met.

This streamlines your code and makes handling these checks efficient.

#### require() function

Use the `require()` function to validate input arguments 
when they are crucial for the function's operation,
and the function can't proceed if these arguments are invalid.

If the condition in `require()` is not met, it throws an `IllegalArgumentException`:

```kotlin
fun getIndices(count: Int): List<Int> {
    require(count >= 0) { "Count must be non-negative. You set count to $count." }
    return List(count) { it + 1 }
}

fun main() {
    // This fails with an IllegalArgumentException
    println(getIndices(-1))
}
```

> The `require()` function allows the compiler to perform smart casting.
> After a successful check, the variable is automatically cast to a non-nullable type.
> 
> These functions are often used for nullability checks 
> to ensure that the variable is not null before proceeding. 
> For example:
>
> ```kotlin
> fun printNonNullString(str: String?) {
>     // Nullability check
>     require(str != null) 
>     // After this successful check, 'str' is guaranteed to be 
>     // non-null and is automatically smart cast to non-nullable String
>     println(str.length)
> }
> ```
>
{style="note"}

#### check() function

Use the `check()` function to validate the state of an object or variable.

If the check fails, it indicates a logic error that needs to be addressed.

If the condition specified in the `check()` function is `false`, 
it throws an `IllegalStateException`:

```kotlin
fun main() {
    var someState: String? = null

    fun getStateValue(): String {

        val state = checkNotNull(someState) { "State must be set beforehand!" }
        check(state.isNotEmpty()) { "State must be non-empty!" }
        return state
    }
    // Program fails with IllegalStateException
    getStateValue()
    // State must be set beforehand!

    someState = ""

    // Program fails with IllegalStateException
    getStateValue() 
    // State must be non-empty!
    
    someState = "non-empty-state"

    // This prints "non-empty-state"
    println(getStateValue())
}
```

> The `check()` function allows the compiler to perform smart casting.
> After a successful check, the variable is automatically cast to a non-nullable type.
> 
> These functions are often used for nullability checks 
> to ensure that the variable is not null before proceeding. 
> For example:
>
> ```kotlin
> fun printNonNullString(str: String?) {
>     // Nullability check
>     check(str != null) 
>     // After this successful check, 'str' is guaranteed to be 
>     // non-null and is automatically smart cast to non-nullable String
>     println(str.length)
> }
> ```
>
{style="note"}

#### error() function

The `error()` function is used to signal an illegal state or a condition in the code 
that logically should not occur.

It's suitable for scenarios when you want to throw an exception 
intentionally in your code, such as when the code encounters an unexpected state.

This function is particularly useful in `when` expressions, 
providing a clear way to handle cases that shouldn't logically happen.

In the following example, 
the `error()` function is used to handle an undefined user role.

If the role is not one of the predefined ones, an `IllegalStateException` is thrown:

```kotlin
class User(val name: String, val role: String)

fun processUserRole(user: User) {
    when (user.role) {
        "admin" -> println("${user.name} is an admin.")
        "editor" -> println("${user.name} is an editor.")
        "viewer" -> println("${user.name} is a viewer.")
        else -> error("Undefined role: ${user.role}")
    }
}

fun main() {
    // This works as expected
    val user1 = User("Alice", "admin")
    processUserRole(user1)
    // Alice is an admin.

    // This throws an IllegalStateException
    val user2 = User("Bob", "guest")
    processUserRole(user2)
}
```

## Handle exceptions using try-catch blocks

When an exception is thrown, it interrupts the normal execution of the program.

You can handle exceptions gracefully with the `try` and `catch` keywords 
to keep your program stable.

The `try` block contains the code that might throw an exception, 
while the `catch` block catches and handles the exception if it occurs.

The exception is caught by the first `catch` block 
that matches its specific type or a superclass of the exception.

Here's how you can use the `try` and `catch` keywords together:

```kotlin
try {
    // Code that may throw an exception
} catch (e: SomeException) {
    // Code for handling the exception
}
```

It's a common approach to use `try-catch` as an expression, 
so it can return a value from either the `try` block or the `catch` block:

```kotlin
fun main() {
    val num: Int = try {

        // If count() completes successfully, its return value is assigned to num
        count()
        
    } catch (e: ArithmeticException) {
        
        // If count() throws an exception, the catch block returns -1, which is assigned to num
        -1
    }
    println("Result: $num")
}

// Simulates a function that might throw ArithmeticException
fun count(): Int {
    val a = 0
    return 10 / a
}
```

Output:

```
Result: -1
```

### The finally block

The `finally` block contains code that always executes, 
regardless of whether the `try` block completes successfully or throws an exception.

With the `finally` block you can clean up code 
after the execution of `try` and `catch` blocks.

This is especially important when working with resources 
like files or network connections, as `finally` guarantees 
they are properly closed or released.

Here is how you would typically use the `try-catch-finally` blocks together:

```kotlin
try {
    // Code that may throw an exception
}
catch (e: YourException) {
    // Exception handler
}
finally {
    // Code that is always executed
}
```

The returned value of a `try` expression is determined by the 
last executed expression in either the `try` or `catch` block.

If no exceptions occur, the result comes from the `try` block; 
if an exception is handled, it comes from the `catch` block.

The `finally` block is always executed, 
but it doesn't change the result of the `try-catch` block.

Let's look at an example to demonstrate:

```kotlin
fun divideOrNull(a: Int): Int {
    
    // The try block is always executed
    // An exception here (division by zero) causes an immediate jump to the catch block
    try {
        val b = 44 / a
        println("try block: Executing division: $b")
        return b
    }
    
    // The catch block is executed due to the ArithmeticException
    catch (e: ArithmeticException) {
        println("catch block: Encountered ArithmeticException $e")
        return -1
    }
    
    finally {
        println("finally block: The finally block is always executed")
    }
}

fun main() {
    divideOrNull(0)
}
```

Output:
```
catch block: Encountered ArithmeticException java.lang.ArithmeticException: / by zero
finally block: The finally block is always executed
```

> In Kotlin, the idiomatic way to manage resources that 
> implement the `AutoClosable` interface,
> such as file streams like `FileInputStream` or `FileOutputStream`, 
> is to use the `.use()` function.
> 
> This function automatically closes the resource when the block of code completes, 
> regardless of whether an exception is thrown, 
> thereby eliminating the need for a `finally` block.
> 
> Consequently, Kotlin does not require a special syntax like 
> Java's try-with-resources for resource management.
>
> ```kotlin
> FileWriter("test.txt").use { writer ->
>     writer.write("some text")
>     // After this block, the .use function automatically calls writer.close(), similar to a finally block
> }
> ```
>
{style="note"}

In Kotlin, you have the flexibility to use 
only a `catch` block, only a `finally` block, or both, depending on your specific needs, 

but a `try` block must always be accompanied by 
at least one `catch` block or a `finally` block.

## The Nothing type

In Kotlin, every expression has a type.

The type of the expression `throw IllegalArgumentException()` is `Nothing`, 
a built-in type that is a subtype of all other types, 
also known as [the bottom type](https://en.wikipedia.org/wiki/Bottom_type).

This means `Nothing` can be used as a return type or generic type 
where any other type is expected, without causing type errors.

`Nothing` is a special type in Kotlin used to represent functions or expressions 
that never complete successfully, either because they always throw an exception 
or enter an endless execution path like an infinite loop.

You can use `Nothing` to mark functions that are not yet implemented 
or are designed to always throw an exception,
clearly indicating your intentions to both the compiler and code readers.

If the compiler infers a `Nothing` type in a function signature, it will warn you.
Explicitly defining `Nothing` as the return type can eliminate this warning.

This Kotlin code demonstrates the use of the `Nothing` type, 
where the compiler marks the code following the function call as unreachable:

```kotlin
class Person(val name: String?)

fun fail(message: String): Nothing {
    throw IllegalArgumentException(message)
    // This function will never return successfully.
    // It will always throw an exception.
}

fun main() {
    // Creates an instance of Person with 'name' as null
    val person = Person(name = null)
    
    val s: String = person.name ?: fail("Name required")
}
```

Kotlin's `TODO()` function, which also uses the `Nothing` type, 
serves as a placeholder to highlight areas of the code that need future implementation:

```kotlin
fun notImplementedFunction(): Int {
    TODO("This function is not yet implemented")
}

fun main() {
    val result = notImplementedFunction()
    // This throws a NotImplementedError
}
```

The `TODO()` function always throws a `NotImplementedError` exception.

## Exception hierarchy

![Throwable](02-throwable.svg)

