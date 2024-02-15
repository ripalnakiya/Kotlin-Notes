# Numbers

<show-structure depth="2"/>

## Integer types

For integer numbers, there are four types:

| Type    | Size (bits) | Min Value | Max Value |
|---------|-------------|-----------|-----------|
| `Byte`  | 8           | -128      | 127       |
| `Short` | 16          | -32768    | 32767     |
| `Int`   | 32          |           |           |
| `Long`  | 64          |           |           |

When you initialize a variable with no explicit type specification, the compiler automatically infers the type with the smallest range enough to represent the value starting from `Int`.

If it is not exceeding the range of `Int`, the type is `Int`. If it exceeds, the type is `Long`.

```Kotlin
val one = 1 // Int
val threeBillion = 3000000000 // Long
```

Explicit type specification triggers the compiler to check the value not to exceed the range of the specified type.

```Kotlin
val oneLong = 1L // Long
val oneByte: Byte = 1
```

## Unsigned Integer types

| Type     | Size (bits) | Min value | Max value |
|----------|-------------|-----------|-----------|
| `UByte`  | 8           | 0         | 255       |
| `UShort` | 16          | 0         | 65535     |
| `UInt`   | 32          | 0         |           |
| `ULong`  | 64          | 0         |           |

`u` and `U` tag is for unsigned literals. 

The exact type is determined based on the expected type. 

If no expected type is provided, compiler will use `UInt` or `ULong` depending on the size of literal:

```Kotlin
val b: UByte = 1u  // UByte, expected type provided
val s: UShort = 1u // UShort, expected type provided
val l: ULong = 1u  // ULong, expected type provided

val a1 = 42u // UInt
 // no expected type provided, constant fits in UInt
 
val a2 = 0xFFFF_FFFF_FFFFu // ULong
// no expected type provided, constant doesn't fit in UInt
```

`uL` and `UL` explicitly tag literal as unsigned long:

```Kotlin
val a = 1UL // ULong
// even though no expected type provided and constant fits into UInt
```

## Floating Point types

| Type     | Size (bits) |
|----------|-------------|
| `Float`  | 32          |
| `Double` | 64          |

<note>
Float reflects the IEEE 754 single precision, while Double reflects double precision.
</note>

For variables initialized with fractional numbers, the compiler infers the Double type:

```Kotlin
val pi = 3.14 // Double
val one: Double = 1 // Error: type mismatch
val oneDouble = 1.0 // Double
```

To explicitly specify the Float type for a value, add the suffix f or F.

If such a value contains more than 6-7 decimal digits, it will be rounded:

```Kotlin
val num = 2.7182818284f // Float
// rounded value is 2.7182817
```

<tip>
Unlike some other languages, there are no implicit widening conversions for numbers in Kotlin.

For example, a function with a Double parameter can be called only on Double values, but not Float, Int
</tip>

## Literal constants for numbers

There are the following kinds of literal constants for integral values:

- Decimals: `123`
- Longs are tagged by a capital L: `123L`
- Hexadecimals: `0x0F`
- Binaries: `0b00001011`

<note>Octal literals are not supported in Kotlin.</note>

Underscores can be used to make number constants more readable:

```Kotlin
val oneMillion = 1_000_000
val creditCardNumber = 1234_5678_9012_3456L
val hexBytes = 0xFF_EC_DE_5E
```

## Numbers representation on the JVM

On the JVM platform, numbers are stored as primitive types: `int`, `double`, and so on.

Exceptions are cases when you create a nullable number reference such as `Int?` or use generics.

In these cases numbers are boxed in Java classes `Integer`, `Double`, and so on.

Nullable references to the same number can refer to different objects:

```Kotlin
val a: Int = 127
val boxedA: Int? = a
val anotherBoxedA: Int? = a

val b: Int = 128
val boxedB: Int? = b
val anotherBoxedB: Int? = b

println(boxedA === anotherBoxedA) // true
println(boxedB === anotherBoxedB) // false
```

All nullable references to `a` are actually the same object because of the memory optimization that JVM applies to Integers between `-128` and `127`.

Memory optimization doesn't apply to the `b` references, so they are different objects.

## Explicit number conversions

Smaller types are NOT implicitly converted to bigger types.

```Kotlin
val b: Byte = 1 // OK, literals are checked statically
val i: Int = b // ERROR
val i1: Int = b.toInt()
```

All number types support conversions to other types:
- `toByte() : Byte`
- `toShort() : Short`
- `toInt() : Int`
- `toLong() : Long`
- `toFloat() : Float`
- `toDouble() : Double`

In many cases, there is no need for explicit conversions because the type is inferred from the context.

```Kotlin
val num = 1L + 3 
       // Long + Int => Long
```

## Operations on numbers

Kotlin supports the standard set of arithmetical operations over numbers: `+`, `-`, `*`, `/`, `%`.

```Kotlin
println(1 + 2)                  // 3
println(2_500_000_000L - 1L)    // 2499999999
println(3.14 * 2.71)            // 8.5094
println(10.0 / 3)               // 3.3333333333333335
```

Division between integers numbers always returns an integer number. Any fractional part is discarded.

```Kotlin
val x = 5 / 2
println(x == 2.5) // ERROR: Operator '==' cannot be applied to 'Int' and 'Double'
println(x == 2)     // true
```

This is true for a division between any two integer types:

```Kotlin
val x = 5L / 2
println(x == 2L)    // true
```

To return a floating-point type, explicitly convert one of the arguments to a floating-point type:

```Kotlin
val x = 5 / 2.toDouble()
println(x == 2.5)   // true
```

### Bitwise Operations

They can be applied only to `Int` and `Long`.

Bitwise operations:

- `shl(bits)` – signed shift left
- `shr(bits)` – signed shift right
- `ushr(bits)` – unsigned shift right
- `and(bits)` – bitwise AND
- `or(bits)` – bitwise OR
- `xor(bits)` – bitwise XOR
- `inv()` – bitwise inversion

### Floating-point numbers comparison

- Equality checks: `a == b` and `a != b`
- Comparison operators: `a < b`, `a > b`, `a <= b`, `a >= b`
- Range instantiation and range checks: `a..b`, `x in a..b`, `x !in a..b`


