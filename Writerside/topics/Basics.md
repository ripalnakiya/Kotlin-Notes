# Basics

## Program entry point

An entry point of a Kotlin application is the `main` function:

```Kotlin
fun main() {
    println("Hello world!")
}
```

Another form of `main` accepts a variable number of String arguments:

```Kotlin
fun main(args: Array<String>) {
    println(args.contentToString())
}
```

## Print to the standard output

```Kotlin
print("Hello ")
print("world!")

println("Kotlin")
println(42)
```

Output:

```Console
Hello world!
Kotlin
42
```

## Functions

A function with two Int parameters and Int return type:

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

The `val` keyword is used to declare immutable, read-only local variables that can’t be reassigned a different value
after initialization.

```Kotlin
val x: Int = 5
```

The `var` keyword is used to mutable variables, and their values can be changed after initialization.

```Kotlin
var x: Int = 5
// Reassigns a new value of 6 to the variable x
x += 1
```

Kotlin supports type inference and automatically identifies the data type of a declared variable.

```Kotlin
// Declares the variable x with the value of 5;`Int` type is inferred
val x = 5
```

> Type Inference : Kotlin compiler can infer the type based on the type of the assigned value.
{style="note"}

Variables can also be declared and initialised separately

```Kotlin
val NUM: Int
// Initializes the variable c after declaration 
NUM = 3
```

