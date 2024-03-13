# Operator Overloading

<show-structure depth="2"/>

Kotlin allows you to provide custom implementations for the predefined set of operators on types. 

To implement an operator, provide a member function or an extension function with a specific name for the corresponding type. 

To overload an operator, mark the corresponding function with the `operator` modifier:

```Kotlin
interface IndexedContainer {
    operator fun get(index: Int)
}
```

When overriding your operator overloads, you can omit operator:

```Kotlin
class OrdersList: IndexedContainer {
    override fun get(index: Int) { /*...*/ }
}
```

## Unary operations

### Unary prefix operators

| Expression | Translated to    |
|------------|------------------|
| `+a`       | `a.unaryPlus()`  |
| `-a`       | `a.unaryMinus()` |
| `!a`       | `a.not()`        |


```Kotlin
data class Point(val x: Int, val y: Int)

operator fun Point.unaryMinus() = Point(-x, -y)

val point = Point(10, 20)

fun main() {
   println(-point)  // prints "Point(x=-10, y=-20)"
}
```

### Increments and decrements

| Expression | Translated to |
|------------|---------------|
| `a++`      | `a.inc()`     |
| `a--`      | `a.dec()`     |

The compiler performs the following steps for resolution of an operator in the postfix form, for example `a++`:
- Store the initial value of `a` to a temporary storage `a0`.
- Assign the result of `a0.inc()` to `a`.
- Return `a0` as the result of the expression.

For the prefix forms ++a, the effect is:
- Assign the result of `a.inc()` to `a`.
- Return the new value of `a` as a result of the expression.

## Binary operations

### Arithmetic operators

| Expression | Translated to     |
|------------|-------------------|
| `a + b`    | `a.plus(b)`       |
| `a - b`    | `a.minus(b)`      |
| `a * b`    | `a.times(b)`      |
| `a / b`    | `a.div(b)`        |
| `a % b`    | `a.rem(b)`        |
| `a..b`     | `a.rangeTo(b)`    |
| `a..<b`    | `a.rangeUntil(b)` |

For the operations in this table, the compiler just resolves the expression in the Translated to column.

Below is an example Counter class that starts at a given value and can be incremented using the overloaded + operator:

```Kotlin
data class Counter(val dayIndex: Int) {
    operator fun plus(increment: Int): Counter {
        return Counter(dayIndex + increment)
    }
}

fun main() {
    val obj = Counter(5)
    print(obj + 10)         // 15
}
```

### in operator

| Expression | Translated to    |
|------------|------------------|
| `a in b`   | `b.contains(a)`  |
| `a !in b`  | `!b.contains(a)` |

For `in` and `!in` the procedure is the same, but the order of arguments is reversed.

### Indexed access operator

| Expression             | Translated to             |
|------------------------|---------------------------|
| `a[i]`                 | `a.get(i)`                |
| `a[i, j]`              | `a.get(i, j)`             |
| `a[i_1, ..., i_n]`     | `a.get(i_1, ..., i_n)`    |
| `a[i] = b`             | `a.set(i, b)`             |
| `a[i, j] = b`          | `a.set(i, j, b)`          |
| `a[i_1, ..., i_n] = b` | `a.set(i_1, ..., i_n, b)` |

### invoke operator

| Expression         | Translated to             |
|--------------------|---------------------------|
| `a()`              | `a.invoke()`              |
| `a(i)`             | `a.invoke(i)`             |
| `a(i, j)`          | `a.invoke(i, j)`          |
| `a(i_1, ..., i_n)` | `a.invoke(i_1, ..., i_n)` |

### Augmented assignments

| Expression | Translated to    |
|------------|------------------|
| a += b     | a.plusAssign(b)  |
| a -= b     | a.minusAssign(b) |
| a *= b     | a.timesAssign(b) |
| a /= b     | a.divAssign(b)   |
| a %= b     | a.remAssign(b)   |

<note>
Assignments are NOT expressions in Kotlin.
</note>

### Equality and inequality operators

| Expression | Translated to                     |
|------------|-----------------------------------|
| `a == b`   | `a?.equals(b) ?: (b === null)`    |
| `a != b`   | `!(a?.equals(b) ?: (b === null))` |

<note>

`===` and `!==` (identity checks) are not overloadable, so no conventions exist for them.
</note>

The `==` operation is special: it is translated to a complex expression that screens for `null`'s.

- If both things are `null`, `==` will return `true`.
- If one thing is `null` and the other isn't, `==` will return `false` without calling the `equals()` method on the non-null object.

This way, you don't have to worry about getting an error when comparing something to `null`.

### Comparison operators

| Expression | Translated to         |
|------------|-----------------------|
| `a > b`    | `a.compareTo(b) > 0`  |
| `a < b`    | `a.compareTo(b) < 0`  |
| `a >= b`   | `a.compareTo(b) >= 0` |
| `a <= b`   | `a.compareTo(b) <= 0` |

All comparisons are translated into calls to `compareTo`, that is required to return `Int`.





























































