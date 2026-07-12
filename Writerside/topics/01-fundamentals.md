# Fundamentals
<show-structure depth="2"/>

In a statically typed language, 
the type of every variable, expression, and function is known at compile time.

> C++, Java, Kotlin are statically typed languages
{style="note"}

```Java
// Java
int x = 10;
x = "hi"; // compile-time error
```

```Kotlin
// Kotlin
val x = 10   // type inferred as Int
x = "hi"     // compile-time error
```

```Python
# Python
x = 10
x = "hello"  # totally fine
```

## Program entry point

An entry point of a Kotlin application is the `main` function:

```kotlin
fun main() {
    println("Hello world!")
}
```
{kotlin-runnable="true" kotlin-min-compiler-version="1.3" id="kotlin-basic-syntax-hello-world"}

Another form of `main` accepts a variable number of `String` arguments:

```kotlin
fun main(args: Array<String>) {
    println(args.contentToString())
}
```
{kotlin-runnable="true" kotlin-min-compiler-version="1.3"}

## Print to the standard output

`print` prints its argument to the standard output:

```kotlin
fun main() {
    print("Hello ")
    print("world!")
}
```

`println` prints its arguments and adds a line break, so that the next thing you print appears on the next line:

```kotlin
fun main() {
    println("Hello world!")
    println(42)
}
```

## Read from the standard input

The `readln()` function reads from the standard input. This function reads the entire line the user enters as a string.

You can use the `println()`, `readln()`, and `print()` functions together to print messages requesting
and showing user input:

```kotlin
// Prints a message to request input
println("Enter any word: ")

// Reads and stores the user input. For example: Happiness
val yourWord = readln()

// Prints a message with the input
print("You entered the word: ")
print(yourWord)
// You entered the word: Happiness
```

For more information, see [Read standard input](https://kotlinlang.org/docs/read-standard-input.html).

## Functions

A function with two `Int` parameters and `Int` return type:

```kotlin
fun sum(a: Int, b: Int): Int {
    return a + b
}

fun main() {
    print("sum of 3 and 5 is ")
    println(sum(3, 5))
}
```

A function body can be an expression. Its return type is inferred:

```kotlin
fun sum(a: Int, b: Int) = a + b
```

A function that returns no meaningful value:

```kotlin
fun printSum(a: Int, b: Int): Unit {
    println("sum of $a and $b is ${a + b}")
}
```

`Unit` return type can be omitted:

```kotlin
fun printSum(a: Int, b: Int) {
    println("sum of $a and $b is ${a + b}")
}
```

See [Functions](01-functions.md).

## Variables

In Kotlin, you declare a variable starting with a keyword, `val` or `var`, followed by the name of the variable.

Use the `val` keyword to declare variables that are assigned a value only once. These are immutable, read-only local variables that can't be reassigned a different value
after initialization:

```kotlin
    // Declares the variable x and initializes it with the value of 5
    val x: Int = 5
    println(x) // 5
```

Use the `var` keyword to declare variables that can be reassigned. These are mutable variables, and you can change their values after initialization:

```kotlin
    // Declares the variable x and initializes it with the value of 5
    var x: Int = 5
    // Reassigns a new value of 6 to the variable x
    x += 1
    println(x) // 6
```

Kotlin supports type inference and automatically identifies the data type of a declared variable. 
When declaring a variable, you can omit the type after the variable name:

```kotlin
    // Declares the variable x with the value of 5;`Int` type is inferred
    val x = 5
    // 5
```

You can use variables only after initializing them. 
You can either initialize a variable at the moment of declaration 
or declare a variable first and initialize it later.
In the second case, you must specify the data type:

```kotlin
    // Initializes the variable x at the moment of declaration; type is not required
    val x = 5
    
    // Declares the variable c without initialization; type is required
    val c: Int
    // Initializes the variable c after declaration 
    c = 3
```

You can declare variables at the top level:

```kotlin
val PI = 3.14
var x = 0

fun incrementX() {
    x += 1
}

fun main() {
    println("x = $x; PI = $PI")
    incrementX()
    println("incrementX()")
    println("x = $x; PI = $PI")
}

// x = 0; PI = 3.14
// incrementX()
// x = 1; PI = 3.14
```

For information about declaring properties, see [Properties](13-properties.md).

## Compilation

![Compilation](compilation.png)

## Recommended way to learn Kotlin

Complete the [Kotlin Tours](https://kotlinlang.org/docs/kotlin-tour-welcome.html),
and then read this documentation.
