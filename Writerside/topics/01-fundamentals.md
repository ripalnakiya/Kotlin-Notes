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

```Kotlin
fun main() {
    println("Hello world!")
    
    print("Hello ")
    print("world!")
}
```

Another form of `main` accepts a variable number of String arguments:

```Kotlin
fun main(args: Array<String>) {
    println(args.contentToString())
}
```


## Functions

A function with two `Int` parameters and `Int` return type:

```Kotlin
fun sum(a: Int, b: Int): Int {
    return a + b
}
```

A function body can be an expression. Its return type is inferred:

```Kotlin
fun sum(a: Int, b: Int) = a + b
```

A function that returns no meaningful value:

```Kotlin
fun printSum(a: Int, b: Int): Unit {
    println("sum of $a and $b is ${a + b}")
}
```

`Unit` return type can be omitted:

```Kotlin
fun printSum(a: Int, b: Int) {
    println("sum of $a and $b is ${a + b}")
}
```

## Variables

The `val` keyword is used to declare immutable, 
read-only local variables that can’t be reassigned a different value after initialization.

```Kotlin
val x: Int = 5
```

The `var` keyword is used to mutable variables, 
and their values can be changed after initialization.

```Kotlin
var x: Int = 5
// Reassigns a new value of 6 to the variable x
x += 1
```

Kotlin supports type inference and 
automatically identifies the data type of a declared variable.

```Kotlin
// Declares the variable x with the value of 5;`Int` type is inferred
val x = 5
```

> Type Inference: Kotlin compiler can infer the type based on the type of the assigned value.
{style="note"}

Variables can also be declared and initialized separately

```Kotlin
val c: Int
// Initializes the variable c after declaration 
c = 3
```

## Console input

Read a full line:

```Kotlin
    val name = readLine()
    println("Hello $name")
```

Read an `Int`:

```Kotlin
    val num = readLine().toInt()
```

Read two numbers:

```Kotlin
    val a = readLine().toInt()
    val b = readLine().toInt()
```

Keep reading numbers until user enters `0`:

```Kotlin
    while (true) {
        val num = readLine().toInt()
        if (num == 0) break
        println(num)
    }
```

## Compilation

![Compilation](01-compilation.png)

## Recommended way to learn Kotlin

Complete the [Kotlin Tours](https://kotlinlang.org/docs/kotlin-tour-welcome.html),
and then read this documentation.
