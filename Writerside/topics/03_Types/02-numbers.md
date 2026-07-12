# Numbers
<show-structure depth="2"/>

The Kotlin number types represent:
* Integer values ([Byte](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin/-byte/),
  [Short](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin/-short/),
  [Int](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin/-int/),
  and [Long](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin/-long/))
* Floating-point values ([Float](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin/-float/)
  and [Double](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin/-double/))

Use number types to store and process numeric data, for example, in arithmetic, counters, measurements,
and other calculations.

## Choose a number type

In most cases, you can refer to the following rules to determine the
correct number type for your task:

* Use `Int` for whole numbers.
* Use `Long` for whole numbers outside the `Int` range.
* Use `Double` for decimal numbers.
* Use `Float` when lower precision is acceptable or required.
* Use `Byte` and `Short` when an API or data format requires them.

## Integer types

For integer numbers, there are four types with different sizes and value ranges:

| Type	    | Size (bits) | Min value                                    | Max value                                      |
|----------|-------------|----------------------------------------------|------------------------------------------------|
| `Byte`	  | 8           | -128                                         | 127                                            |
| `Short`	 | 16          | -32768                                       | 32767                                          |
| `Int`	   | 32          | -2,147,483,648 (-2<sup>31</sup>)             | 2,147,483,647 (2<sup>31</sup> - 1)             |
| `Long`	  | 64          | -9,223,372,036,854,775,808 (-2<sup>63</sup>) | 9,223,372,036,854,775,807 (2<sup>63</sup> - 1) |

### Declare integer values

Kotlin supports the following literal forms for integer values:

* Decimals: `123`
* Hexadecimals: `0x0F`
* Binaries: `0b00001011`

> Kotlin does not support octal literals.
>
{style="note"}

To declare a numeric value, specify the type explicitly:

```kotlin
val one: Int = 1

// Use underscores to improve readability
val oneBillion: Long = 1_000_000_000
val hexBytes: Int = 0x7F_EC_DE_5E
val bytes: Int = 0b01010010_01101001_10010100_10010010

val oneByte: Byte = 1
val oneShort: Short = 1
```

You can also append the `L` suffix, to declare a `Long` value:

```kotlin
val oneLong = 1L
```

When you declare a numeric type explicitly, the compiler checks that the value
fits in the range of that type:

```kotlin
// Value fits in Byte
val oneByte: Byte = 1

// Error: the value does not fit in Byte
val tooBig: Byte = 128
```

When you do not specify a numeric type, Kotlin infers `Int` if the
value fits in the `Int` range. Otherwise, Kotlin infers `Long`:

```kotlin
val million = 1_000_000 // Int
val threeBillion = 3_000_000_000 // Long
```

If a value can be absent, use nullable types:

```kotlin
val maybeAbsent: Int? = null
```

## Floating-point types

For numbers with a fractional part, Kotlin provides `Float` and `Double`.

Floating-point types follow
the [IEEE 754 standard](https://en.wikipedia.org/wiki/IEEE_754).
`Float` reflects the _single precision_. `Double` reflects the _double precision_.

Floating-point types differ in size and precision:

| Type	    | Size (bits) | Significant bits | Exponent bits | Decimal digits |
|----------|-------------|------------------|---------------|----------------|
| `Float`	 | 32          | 24               | 8             | 6-7            |
| `Double` | 64          | 53               | 11            | 15-16          |    

### Declare floating-point values

To declare a floating-point literal, include a decimal point (`.`) or use exponent notation:

```kotlin
val pi = 3.14
val avogadro = 6.02214076e23
```

By default, Kotlin infers floating-point literals as `Double`.
To declare a `Float`, add the `f` or `F` suffix:

```kotlin
val pi = 3.14 // Double
val eFloat = 2.7182817f // Float
```

> Kotlin rounds a `Float` literal that contains more precision than `Float` can store.
>
{style="note"}

If such a value contains more than 6-7 decimal digits, it will be rounded:

```Kotlin
val num = 2.7182818284f // Float
// rounded value is 2.7182817
```

> Unlike in some other languages, 
> there are no implicit widening conversions for numbers in Kotlin. 
> 
> For example, a function with a `Double` parameter can be called only 
> on `Double` values, but not `Float`, `Int`, or other numeric values:
> {style="warning"}

If a value can be absent, use nullable types:

```kotlin
val maybeAbsent: Double? = null
```

## Arithmetic operations

Kotlin supports the standard arithmetic operations on numbers: `+`, `-`, `*`, `/`, and `%`.

Use these operators to perform common calculations:

```kotlin
    println(1 + 2) // 3
    println(2_500_000_000L - 1L) // 2499999999
    println(3.14 * 2.71) // 8.5094
    println(10.0 / 3) // 3.3333333333333335
```

The result type depends on the types of the operands. Learn more in [](#mixed-numeric-expressions).

> You can override these operators in custom number classes.
> For more information, see [Operator overloading](06-operator-overloading.md).
>
{style="tip"}

### Integer division

Division between integer values always returns an integer result. The compiler discards the fractional part:

```kotlin
    val intValue = 5 / 2
    println(intValue) // 2
    
    val longValue = 5L / 2
    println(longValue) // 2
```

To return a floating-point result, make at least one operand a `Float` or `Double`:

```kotlin
    val a = 5 / 2.0
    println(a) // 2.5
    
    val b = 5 / 2.toDouble()
    println(b) // 2.5
```

## Type conversion

Numeric types are not subtypes of one another. Kotlin requires explicit
conversions to avoid silent data loss and unexpected behavior.

For example, a function that expects `Double` cannot accept an `Int` or a `Float` value without conversion:

```kotlin
fun main() {
    fun printDouble(x: Double) { 
        print(x) 
    }

    val x = 1.0
    val xInt = 1
    val xFloat = 1.0f
    val one: Double = 1 // Error: initializer type mismatch

    printDouble(x) // OK
    printDouble(xInt) // Error: argument type mismatch
    printDouble(xFloat) // Error: argument type mismatch
}
```

All number types support conversions to other number types.
To convert a number to another type, use an explicit conversion function:

* `toByte()`
* `toShort()`
* `toInt()`
* `toLong()`
* `toFloat()`
* `toDouble()`

For example, the following code converts an `Int` value to `Double`:

```kotlin
    val intValue: Int = 1
    val doubleValue = intValue.toDouble()
    
    println(doubleValue) // 1.0
```

When you convert a floating-point value to an integer type, the compiler discards the fractional part:

```kotlin
    val d: Double = 1.5
    val l: Long = d.toLong()
    
    println(l) // 1
```

### Mixed numeric expressions

Kotlin does not support implicit conversion for assignments or function arguments.
However, you can combine different numeric types in arithmetic expressions. In such cases,
Kotlin determines a result type based on the operand types,
and arithmetic operators handle the conversion automatically:

```kotlin
val intNumber: Int = 1
val longNumber: Long = 1000
val result = intNumber + longNumber // 1001, Long
```

If you try to assign the result to a smaller type, the compiler reports an error:

```kotlin
val intNumber: Int = 1
val longNumber: Long = 1000
val result: Int = intNumber + longNumber 
// Error: Initializer type mismatch
```

### Integer literal types

During type inference, Kotlin treats unsuffixed integer literals as a special [Integer Literal Type (ILT)](https://kotlinlang.org/spec/type-system.html#integer-literal-types)
until the surrounding context determines a specific type:

```kotlin
fun List<Any>.log() {
    println(joinToString(" | ") { it::class.simpleName ?: "Unknown" })
}

fun main() {
    listOf(1, 2).log()
    // Int | Int
    
    listOf(1L, 2L).log()
    // Long | Long
    
    // Compiler interprets 1 as an ILT and resolves it to Long
    listOf(1, 2L).log()
    // Long | Long
    
    // .toInt() converts the literal to Int
    listOf(1.toInt(), 2L).log()
    // Int | Long
}
```

It's especially easy to miss with the `Int` and `Long` values because they have the same string representation
at runtime. To avoid this, specify the expected type or convert values explicitly:

```kotlin
fun List<Any>.log() {
    println(joinToString(" | ") { it::class.simpleName ?: "Unknown" })
}

fun main() {
    val longValues: List<Long> = listOf(1, 2L)
    longValues.log()
    // Long | Long

    val numberValues: List<Number> = listOf(1.toInt(), 2L)
    numberValues.log()
    // Int | Long
}
```

You can also use an explicit type to catch unintended type inference:

```kotlin
    val intValues: List<Int> = listOf(1, 2L)
    // Error: initializer type mismatch
```

> Learn more about [Integer literal types](https://kotlinlang.org/spec/type-system.html#integer-literal-types).
>
{style="tip"}

### Reasoning against implicit conversions

Kotlin doesn't support implicit conversions 
because they can lead to unexpected behavior.

If numbers of different types were converted implicitly, 
we could sometimes lose equality and identity silently. 

For example, imagine if `Int` was a subtype of `Long`:

```Kotlin
// Hypothetical code, does not actually compile:

val a: Int? = 1    // A boxed Int (java.lang.Integer)
val b: Long? = a   // Implicit conversion yields a boxed Long (java.lang.Long)

print(b == a)      
// Prints "false" as Long.equals() checks not only the value but whether the other number is Long as well
```

## Data overflow

Numeric types can represent only values within their defined ranges.

If the result of an operation falls outside that range, overflow occurs.
If you convert a value to a smaller numeric type, the converted value may not preserve
the original numeric value.

This behavior can affect the result of your code even when the compiler accepts it.

### Overflow in operations

Each integer type can store only values within its defined range. When the result of an
arithmetic operation exceeds that range, _data overflow_ occurs:

```kotlin
    val intNumber: Int = 2147483647
    // Max Int value is 2147483647
    println(intNumber + 1) // -2147483648
```

Here, the result wraps around because the value no longer fits in `Int`.

> The compiler does not automatically produce an error when integer overflow occurs.
>
{style="note"}

### Overflow in negation

Overflow can also occur during negation.
For example, you cannot represent the positive counterpart of `Int.MIN_VALUE` as an `Int`.

```kotlin
    val min = Int.MIN_VALUE
    println(-min) // -2147483648
```

### Narrowing conversions

When you convert a value to a smaller integer type,
the result may not preserve the original numeric value:

```kotlin
    val large: Int = 130
    val narrowed: Byte = large.toByte()

    println(narrowed) // -126
```

However, since floating-point types follow the
[IEEE 754 Standard](https://en.wikipedia.org/wiki/IEEE_754), very large results can become `Infinity`:

```kotlin
    println(Double.MAX_VALUE * 2) // Infinity
```

## Bitwise operations

Kotlin provides _bitwise operations_ for `Int` and `Long`. These operations are represented by
a set of [infix functions](01-functions.md#infix-notation) and `inv()`.

```kotlin
    val x = 1
    
    println(x shl 2) // 4
    println(x and 0x000FF000) // 0
```

Bitwise operations include:

* `shl()` – signed shift left
* `shr()` – signed shift right
* `ushr()` – unsigned shift right
* `and()` – bitwise AND
* `or()` – bitwise OR
* `xor()` – bitwise XOR
* `inv()` – bitwise inversion

## Boxing and caching numbers on the JVM

On the JVM, non-nullable numeric values are usually stored using primitive types, such as `int`, `long`, or `double`.
However, when you use [generic types](10-generics.md) or nullable numeric types like `Int?`, the value is boxed and
represented as an object.

The JVM applies a [memory optimization technique](https://docs.oracle.com/javase/specs/jls/se22/html/jls-5.html#jls-5.1.7)
to small numbers by caching their boxed representations. As a result,
boxed numbers with the same value can be [referentially equal](09-equality.md#referential-equality).

For example, the JVM caches boxed `Integer` values in the range `-128` to `127`. Therefore, the following
code returns `true`:

```kotlin
    val score: Int = 100
    val savedScore: Int? = score
    val displayedScore: Int? = score
    
    println(savedScore === displayedScore) // true
```

For values outside the cached range, boxed values are separate objects. In that case,
they are not referentially equal, even if their values are [structurally equal](09-equality.md#structural-equality).
For this reason, use `==` to compare numeric values:

```kotlin
    val score: Int = 10000
    val savedScore: Int? = score
    val displayedScore: Int? = score

    println(savedScore === displayedScore) // false
    println(savedScore == displayedScore) // true
```
