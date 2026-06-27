# Numbers
<show-structure depth="2"/>

## Integer types

For integer numbers, there are four types with different sizes and value ranges:

| Type    | Size (bits) | Min Value | Max Value |
|---------|-------------|-----------|-----------|
| `Byte`  | 8           | -128      | 127       |
| `Short` | 16          | -32768    | 32767     |
| `Int`   | 32          |           |           |
| `Long`  | 64          |           |           |

When you initialize a variable with no explicit type specification, 
the compiler automatically infers the type with 
the smallest range enough to represent the value starting from `Int`.

If it is not exceeding the range of `Int`, the type is `Int`. 

If it exceeds, the type is `Long`.

```Kotlin
val one = 1 // Int
val threeBillion = 3000000000 // Long
```

To specify the `Long` value explicitly,
append the suffix `L` to the value.

To use the `Byte` or `Short` type, specify it explicitly in the declaration.

> Explicit type specification triggers the compiler
to check that the value doesn't exceed the range of the specified type.
{style="note"}

```Kotlin
val oneLong = 1L // Long
val oneByte: Byte = 1
```

## Floating-point types

For real numbers, 
Kotlin provides floating-point types `Float` and `Double` 
that adhere to the IEEE 754 standard.

These types differ in their size and 
provide storage for floating-point numbers with different precision:

| Type     | Size (bits) | Decimal Digits |
|----------|-------------|----------------|
| `Float`  | 32          | 6-7            |
| `Double` | 64          | 15-16          |

You can initialize Double and Float variables 
only with numbers that have a fractional part. 
Separate the fractional part from the integer part by a period (`.`)

For variables initialized with fractional numbers, 
the compiler infers the Double type:

```Kotlin
val pi = 3.14          // Double

val one: Double = 1    // Int is inferred
// Initializer type mismatch

val oneDouble = 1.0    // Double
```

To explicitly specify the Float type for a value, add the suffix `f` or `F`. 
If a value provided in this way contains more than 7 decimal digits, 
it is rounded:

```Kotlin
val e = 2.7182818284          // Double
val eFloat = 2.7182818284f    // Float, actual value is 2.7182817
```


For variables initialized with fractional numbers, 
the compiler infers the Double type:

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

> Unlike in some other languages, 
> there are no implicit widening conversions for numbers in Kotlin. 
> 
> For example, a function with a `Double` parameter can be called only 
> on `Double` values, but not `Float`, `Int`, or other numeric values:
{style="warning"}

## Literal constants for numbers

There are the following kinds of literal constants for integral values:

- Decimals: `123`
- Longs are tagged by a capital L: `123L`
- Hexadecimals: `0x0F`
- Binaries: `0b00001011`

> Octal literals are not supported in Kotlin.
{style="note"}

Underscores can be used to make number constants more readable:

```Kotlin
val oneMillion = 1_000_000
val creditCardNumber = 1234_5678_9012_3456L
val socialSecurityNumber = 999_99_9999L
val hexBytes = 0xFF_EC_DE_5E
val bytes = 0b11010010_01101001_10010100_10010010
val bigFractional = 1_234_567.7182818284
```

## Explicit number conversions

Due to different representations, number types **are not subtypes** of each other. 

As a consequence, smaller types are not implicitly converted to 
bigger types and vice versa. 

For example, assigning a value of type `Byte` to an `Int` variable 
requires an explicit conversion:

```Kotlin
val byte: Byte = 1
// OK, literals are checked statically

val intAssignedByte: Int = byte 
// Initializer type mismatch

val intConvertedByte: Int = byte.toInt()
```

- All number types support conversions to other types:
  - `toByte() : Byte`
  - `toShort() : Short`
  - `toInt() : Int`
  - `toLong() : Long`
  - `toFloat() : Float`
  - `toDouble() : Double`

In many cases, there is no need for explicit conversion 
because the type is inferred from the context, 
and arithmetical operators are overloaded to handle conversions automatically. 
For example:

```Kotlin
val l = 1L + 3       // Long + Int => Long
println(l is Long)   // true
```

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

## Operations on numbers

Kotlin supports the standard set of arithmetical operations 
over numbers: `+`, `-`, `*`, `/`, `%`.



```Kotlin
println(1 + 2)                  // 3
println(2_500_000_000L - 1L)    // 2499999999
println(3.14 * 2.71)            // 8.5094
println(10.0 / 3)               // 3.3333333333333335
```

### Division of integers

Division between integers numbers always returns an integer number. 
Any fractional part is discarded.

```Kotlin
val x = 5 / 2
println(x == 2.5) 
// Operator '==' cannot be applied to 'Int' and 'Double'

println(x == 2)   
// true
```

This is true for a division between any two integer types:

```Kotlin
val x = 5L / 2
println (x == 2)
// Error, as Long (x) cannot be compared to Int (2)

println(x == 2L)
// true
```

To return a division result with the fractional part, 
explicitly convert one of the arguments to a floating-point type:


```Kotlin
val x = 5 / 2.toDouble()
println(x == 2.5)   // true
```

### Bitwise operations

Kotlin provides a set of bitwise operations on integer numbers. 
They operate on the binary level directly with bits of the numbers' representation. 
Bitwise operations are represented by functions that can be called in infix form. 

They can be applied only to `Int` and `Long`:

```Kotlin
val x = 1
val xShiftedLeft = (x shl 2)
println(xShiftedLeft)  
// 4

val xAnd = x and 0x000FF000
println(xAnd)          
// 0
```


- The complete list of bitwise operations:
  - `shl(bits)` – signed shift left
  - `shr(bits)` – signed shift right
  - `ushr(bits)` – unsigned shift right
  - `and(bits)` – bitwise **AND**
  - `or(bits)` – bitwise **OR**
  - `xor(bits)` – bitwise **XOR**
  - `inv()` – bitwise inversion

### Floating-point numbers comparison

- Equality checks: `a == b` and `a != b`
- Comparison operators: `a < b`, `a > b`, `a <= b`, `a >= b`
- Range instantiation and range checks: `a..b`, `x in a..b`, `x !in a..b`
