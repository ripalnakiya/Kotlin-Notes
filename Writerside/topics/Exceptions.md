# Exceptions

All exception classes in Kotlin inherit the `Throwable` class.

Every exception has a message, a stack trace, and an optional cause.

To throw an exception object, use the `throw` expression:

```Kotlin
throw Exception("Hi There!")
```

To catch an exception, use the `try`...`catch` expression:

```Kotlin
try {
    // some code
} catch (e: SomeException) {
    // handler
} finally {
    // optional finally block
}
```

There may be zero or more `catch` blocks, and the `finally` block may be omitted.

However, at least one `catch` or `finally` block is required.

## try is expression

`try` is an expression, which means it can have a return value:

```Kotlin
val a: Int? = 
try { 
    input.toInt() 
} catch (e: NumberFormatException) { 
    null 
}
```

The returned value of a `try` expression is either the last expression in the `try` block or the last expression in the `catch` block (or blocks).

The contents of the `finally` block don't affect the result of the expression.

## Checked exceptions

Kotlin does not have checked exceptions, unlike Java.

```Kotlin
Appendable append(CharSequence csq) throws IOException;
```

## The Nothing type

`throw` is an expression in Kotlin, so you can use it, for example, as part of an Elvis expression:

```Kotlin
val str = person.name ?: throw IllegalArgumentException("Name required")
```

> Elvis operator (?:) is used to provide a default value(`throw`) when a nullable reference(`person.name`) is assigned to a non-nullable type(`str`).
>
{style="note"}

The `throw` expression has the type `Nothing`.

This type has no values and is used to mark code locations that can never be reached.

In your own code, you can use `Nothing` to mark a function that never returns:

```Kotlin
fun fail(message: String): Nothing {
    throw IllegalArgumentException(message)
}
```

When you call this function, the compiler will know that the execution doesn't continue beyond the call:

```Kotlin
val s = person.name ?: fail("Name required")
println(s)     // 's' is known to be initialized at this point
```

Since `fail` has a return type of `Nothing`, the compiler knows that the program cannot continue beyond the call to `fail`. Therefore, the `println(s)` statement will not be executed if `person?.name` is `null`.

Code comment means that, When `println(s)` is executed, it's known that `s` has been initialized and no exception was thrown.