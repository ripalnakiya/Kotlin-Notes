# Arrays

<show-structure depth="2"/>

Internal working of Array

```Kotlin
var riversArray = arrayOf("Nile", "Amazon", "Yangtze")

// Using the += assignment operation creates a new riversArray,
// copies over the original elements and adds "Mississippi"
riversArray += "Mississippi"
println(riversArray.joinToString())
// Nile, Amazon, Yangtze, Mississippi
```

The most common type of array in Kotlin is the object-type array.

<note>
If you use primitives in an object-type array, this has a performance impact because your primitives are boxed into objects.

To avoid boxing overhead, use primitive-type arrays instead.
</note>

## Create Array

Arrays can be created using
- functions: `arrayOf()`, `arrayOfNulls()`, `emptyArray()`
- constructor: `Array`

```Kotlin
val simpleArray = arrayOf(1, 2, 3)
println(simpleArray.joinToString())     // 1, 2, 3
```

```Kotlin
// Creates an array with values [null, null, null]
val nullArray: Array<Int?> = arrayOfNulls(3)
println(nullArray.joinToString())       // null, null, null
```

You can specify the type of the `empty array` on the left-hand or right-hand side of the assignment due to Kotlin's type inference.

```Kotlin
var exampleArray = emptyArray<String>()

var exampleArray: Array<String> = emptyArray()
```

The `Array` constructor takes the array size and a function that returns values for array elements given its index:

```Kotlin
// Creates an Array<Int> that initializes with zeros [0, 0, 0]
val initArray = Array<Int>(3) { 0 }
println(initArray.joinToString())           // 0, 0, 0

// Creates an Array<String> with values ["0", "1", "4", "9", "16"]
val asc = Array(5) { i -> (i * i).toString() }
asc.forEach { print(it) }                   // 014916
```

## Nested Arrays

```Kotlin
// Creates a two-dimensional array
val twoDArray = Array(2) { Array<Int>(2) { 0 } }
println(twoDArray.contentDeepToString())
// [[0, 0], [0, 0]]

// Creates a three-dimensional array
val threeDArray = Array(3) { Array(3) { Array<Int>(3) { 0 } } }
println(threeDArray.contentDeepToString())
// [[[0, 0, 0], [0, 0, 0], [0, 0, 0]], [[0, 0, 0], [0, 0, 0], [0, 0, 0]], [[0, 0, 0], [0, 0, 0], [0, 0, 0]]]
```

## Access and Modify elements

```Kotlin
val simpleArray = arrayOf(1, 2, 3)
val twoDArray = Array(2) { Array<Int>(2) { 0 } }

// Accesses the element and modifies it
simpleArray[0] = 10
twoDArray[0][0] = 2

// Prints the modified element
println(simpleArray[0].toString()) // 10
println(twoDArray[0][0].toString()) // 2
```

## Variable Arguments

To pass an array containing a variable number of arguments to a function, use the **spread operator** (*). 

The spread operator passes each element of the array as individual arguments to your chosen function:

```Kotlin
fun main() {
    val lettersArray = arrayOf("c", "d")
    printAllStrings("a", "b", *lettersArray)
}

fun printAllStrings(vararg strings: String) {
    for (string in strings) {
        print(string)                       // abcd
    }
}
```

## Compare Arrays

```Kotlin
val simpleArray = arrayOf(1, 2, 3)
val anotherArray = arrayOf(1, 2, 3)

// Compares contents of arrays
println(simpleArray.contentEquals(anotherArray))        // true

// Using infix notation, 
// compares contents of arrays after an element is changed
simpleArray[0] = 10
println(simpleArray contentEquals anotherArray)         // false
```

## Transform Arrays

To return the sum of all elements in an array, use the `.sum()` function:

```Kotlin
val sumArray = arrayOf(1, 2, 3)

// Sums array elements
println(sumArray.sum())                                 // 6
```

To randomly shuffle the elements in an array, use the `.shuffle()` function:

```Kotlin
val simpleArray = arrayOf(1, 2, 3)

// Shuffles elements [3, 2, 1]
simpleArray.shuffle()
println(simpleArray.joinToString())

// Shuffles elements again [2, 3, 1]
simpleArray.shuffle()
println(simpleArray.joinToString())
```

## Convert arrays to collections

### Convert to List or Set

```Kotlin
val simpleArray = arrayOf("a", "b", "c", "c")

// Converts to a Set
println(simpleArray.toSet())                // [a, b, c]

// Converts to a List
println(simpleArray.toList())               // [a, b, c, c]
```

### Convert to Map

Only an array of `Pair<K,V>` can be converted to a `Map`. 

The first value of a `Pair` instance becomes a key, and the second becomes a value.

```Kotlin
val pairArray = arrayOf("apple" to 120, "banana" to 150, "cherry" to 90, "apple" to 140)

// Converts to a Map
// The keys are fruits and the values are their number of calories
println(pairArray.toMap())     // {apple=140, banana=150, cherry=90}
// Note how keys must be unique, 
// so the latest value of "apple" overwrites the first
```

This example uses the infix notation to call the `to` function that creates tuples of `Pair`.

## Primitive-type arrays

If you use the `Array` class with primitive values, these values are boxed into objects. 

| Primitive-type array | Equivalent in Java |
|----------------------|--------------------|
| `BooleanArray`       | `boolean[]`        |
| `ByteArray`          | `byte[]`           |
| `CharArray`          | `char[]`           |
| `DoubleArray`        | `double[]`         |
| `FloatArray`         | `float[]`          |
| `IntArray`           | `int[]`            |
| `LongArray`          | `long[]`           |
| `ShortArray`         | `short[]`          |

These classes have no inheritance relation to the `Array` class, but they have the same set of functions and properties.

```Kotlin
// Creates an array of Int of size 5 with the values initialized to zero
val exampleArray = IntArray(5)
println(exampleArray.joinToString())            // 0, 0, 0, 0, 0
```

> To convert primitive-type arrays to object-type arrays, use the `.toTypedArray()` function.
> 
> To convert object-type arrays to primitive-type arrays, use `.toBooleanArray()`, `.toByteArray()`, `.toCharArray()`, and so on.
> 
{style="note"}