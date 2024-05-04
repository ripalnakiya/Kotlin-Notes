# Kotlin Basics

Kotlin is a statically-typed language.

>A programming language is said to use static typing when type checking is performed during compile-time as opposed to run-time.
{style="note"}

This means that the variable type is resolved at compile time and never changes.

## Type Inference

When you assign an initial value to languageName, the Kotlin compiler can infer the type based off of the type of the assigned value.

In the following example, languageName is inferred as a String, so you can't call any functions that aren't part of the String class:

```Kotlin
val languageName = "Kotlin"
val upperCaseName = languageName.toUpperCase() // Compiles

languageName.inc() // Fails to compile - inc() is `Int` operator function
```

## Null Values

Kotlin variables can't hold null values by default.

```Kotlin
val languageName: String = null // Fails to compile
```

For a variable to hold a null value, it must be of a **nullable type**.

```kotlin
val languageName: String? = null
```

With a `String?` type, you can assign either a `String` value or `null` to `languageName`.


## Conditional Expressions

```kotlin
val answerString: String = if (count == 42) {
    "I have the answer."
} else if (count > 35) {
    "The answer is close."
} else {
    "The answer eludes me."
}

println(answerString)
```

replacing your `if-else` expression with a `when` expression

```kotlin
val answerString = when {
    count == 42 -> "I have the answer."
    count > 35 -> "The answer is close."
    else -> "The answer eludes me."
}

println(answerString)
```
