# Equality

In Kotlin, there are two types of equality:
- **Structural** equality (`==`) - a check for the `equals()` function
- **Referential** equality (`===`) - a check for two references pointing to the same object

## Structural equality

Structural equality verifies if two objects have the same content or structure.

Structural equality is checked by the `==` operation and its negated counterpart `!=`.

By convention, an expression like `a == b` is translated to:

```Kotlin
a?.equals(b) ?: (b === null)
```

If `a` is not `null`, it calls the `equals(Any?)` function. Otherwise (`a` is `null`), it checks that `b` is referentially equal to `null`:

```Kotlin
fun main() {
    var a = "hello"
    var b = "hello"
    var c = null
    var d = null
    var e = d

    println(a == b)
    // true
    println(a == c)
    // false
    println(c == e)
    // true
}
```

Note that there's no point in optimizing your code when comparing to `null` explicitly: `a == null` will be automatically translated to `a === null`.

n Kotlin, the `equals()` function is inherited by all classes from the `Any` class. By default, the `equals()` function implements referential equality. 

However, classes in Kotlin can override the `equals()` function to provide a custom equality logic and, in this way, implement structural equality.

[Value classes](Inline-Value-Classes.md) and [data classes](Data-Classes.md) are two specific Kotlin types that automatically override the `equals()` function. That's why they implement structural equality by default.

However, in the case of data classes, if the `equals()` function is marked as `final` in the parent class, its behaviour remains unchanged (behaviour of parent class will be used) .

Distinctly, non-data classes (those not declared with the `data` modifier) do not override the `equals()` function by default. Instead, non-data classes implement referential equality behavior inherited from the `Any` class. 

To implement structural equality, non-data classes require a custom equality logic to override the `equals()` function.

To provide a custom equals check implementation, override the `equals(other: Any?): Boolean` function:

```Kotlin
class Point(val x: Int, val y: Int) {
    override fun equals(other: Any?): Boolean {
        if (this === other) return true
        if (other !is Point) return false

        // Compares properties for structural equality
        return this.x == other.x && this.y == other.y
    }
}
```

<note>

When overriding the `equals()` function, you should also override the `hashCode()` function to keep consistency between equality and hashing and ensure a proper behavior of these functions.
</note>

Functions with the same name and other signatures (like `equals(other: Foo)`) don't affect equality checks with the operators `==` and `!=`.

Structural equality has nothing to do with comparison defined by the `Comparable<...>` interface, so only a custom `equals(Any?)` implementation may affect the behavior of the operator.

## Referential equality

Referential equality verifies the memory addresses of two objects to determine if they are the same instance.

Referential equality is checked by the `===` operation and its negated counterpart `!==`. 

`a === b` evaluates to true if and only if `a` and `b` point to the same object:

```Kotlin
fun main() {
    var a = "Hello"
    var b = a
    var c = "world"
    var d = "world"

    println(a === b)
    // true
    println(a === c)
    // false
    println(c === d)
    // true
}
```

For values represented by primitive types at runtime (for example, `Int`), the `===` equality check is equivalent to the `==` check.

## Array equality

To compare whether two arrays have the same elements in the same order, use `contentEquals()`.