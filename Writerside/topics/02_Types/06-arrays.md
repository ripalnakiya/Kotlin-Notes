# Arrays
<show-structure depth="2"/>

An array is a data structure 
that holds a fixed number of values of the same type or its subtypes. 

The most common type of array in Kotlin is the object-type array, 
represented by the `Array` class.

> If you use primitives in an object-type array, 
> this has a performance impact because your primitives are boxed into objects.
> 
> To avoid boxing overhead, use primitive-type arrays instead.
{style="note"}

## When to use arrays

Use arrays in Kotlin when you have specialized low-level requirements 
that you need to meet. 

For example, if you have performance requirements beyond 
what is needed for regular applications, 
or you need to build custom data structures. 

If you don't have these sorts of restrictions, use collections instead.

- Collections have the following benefits compared to arrays:
  - Collections can be read-only, 
    which gives you more control and allows you to write robust code 
    that has a clear intent.
  - It is easy to add or remove elements from collections. 
    In comparison, arrays are fixed in size. 
    The only way to add or remove elements from an array is to 
    create a new array each time, which is very inefficient:
    ```Kotlin
    var riversArray = arrayOf("Nile", "Amazon", "Yangtze")

    // Using the += assignment operation creates a new riversArray,
    // copies over the original elements and adds "Mississippi"
    riversArray += "Mississippi"
    println(riversArray.joinToString())
    // Nile, Amazon, Yangtze, Mississippi
    ```
  - You can use the equality operator (`==`) to check if collections are structurally equal. 
    You can't use this operator for arrays. 
    Instead, you have to use a special function, 
    which you can read more about in Compare arrays section.

## Create arrays

To create arrays in Kotlin, you can use:
- functions, such as `arrayOf()`, `arrayOfNulls()` or `emptyArray()`.
- the `Array` constructor.

This example uses the `arrayOf()` function and passes item values to it:

```Kotlin
// Creates an array with values [1, 2, 3]
val simpleArray = arrayOf(1, 2, 3)
println(simpleArray.joinToString())
// 1, 2, 3
```

This example uses the `arrayOfNulls()` function 
to create an array of a given size filled with null elements:

```Kotlin
// Creates an array with values [null, null, null]
val nullArray: Array<Int?> = arrayOfNulls(3)
println(nullArray.joinToString())
// null, null, null
```

This example uses the `emptyArray()` function to create an empty array :

```Kotlin
var exampleArray = emptyArray<String>()
```

> You can specify the type of the empty array on the left-hand or right-hand side 
> of the assignment due to Kotlin's type inference.
> 
> ```Kotlin
> var exampleArray = emptyArray<String>()
> 
> var exampleArray: Array<String> = emptyArray()
> ```
{style="note"}

The `Array` constructor takes the array size 
and a function that returns values for array elements given its index:

```Kotlin
// Creates an Array<Int> that initializes with zeros [0, 0, 0]
val initArray = Array<Int>(3) { 0 }
println(initArray.joinToString())           
// 0, 0, 0

// Creates an Array<String> with values ["0", "1", "4", "9", "16"]
val asc = Array(5) { i -> (i * i).toString() }
asc.forEach { print(it) }                   
// 014916
```

### Nested arrays

```Kotlin
// Creates a two-dimensional array
val twoDArray = Array(2) { Array<Int>(2) { 0 } }
println(twoDArray.contentDeepToString())
// [[0, 0], [0, 0]]

// Creates a three-dimensional array
val threeDArray = Array(3) { Array(3) { Array<Int>(3) { 0 } } }
println(threeDArray.contentDeepToString())
// [[[0, 0, 0], [0, 0, 0], [0, 0, 0]], 
// [[0, 0, 0], [0, 0, 0], [0, 0, 0]], 
// [[0, 0, 0], [0, 0, 0], [0, 0, 0]]]
```

## Access and modify elements

Arrays are always mutable. 

To access and modify elements in an array, use the indexed access operator `[]`:


```Kotlin
    val arr = arrayOf(1, 2, 3, 4)
    
    val a = arr[0]
    val b = arr.get(1)

    arr[2] = 30
    arr.set(3, 40)
```

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

## Work with arrays

### Pass variable number of arguments to a function

In Kotlin, 
you can pass a variable number of arguments to a function via the `vararg` parameter. 

This is useful when you don't know the number of arguments in advance, 
like when formatting a message or creating an SQL query.

To pass an array containing a variable number of arguments to a function, 
use the spread operator (`*`). 

The spread operator passes each element of the array 
as individual arguments to your chosen function:

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

### Compare arrays

To compare whether two arrays have the same elements in the same order, 
use the `.contentEquals()` and `.contentDeepEquals()` functions:

```Kotlin
val simpleArray = arrayOf(1, 2, 3)
val anotherArray = arrayOf(1, 2, 3)

// Compares contents of arrays
println(simpleArray.contentEquals(anotherArray))
// true

// Using infix notation, compares contents of arrays after an element is changed
simpleArray[0] = 10
println(simpleArray contentEquals anotherArray)
// false
```

> Don't use equality (`==`) and inequality (`!=`) operators to 
> compare the contents of arrays. 
> 
> These operators check whether the assigned variables point to the same object.
{style="warning"}

### Transform arrays

Kotlin has many useful functions to transform arrays. 

This document highlights a few but this isn't an exhaustive list. 
For the full list of functions, see [API reference](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin/-array/).

#### Sum

To return the sum of all elements in an array, use the `.sum()` function:

```Kotlin
val sumArray = arrayOf(1, 2, 3)

// Sums array elements
println(sumArray.sum())
// 6
```

> The `.sum()` function can only be used with arrays of numeric data types, such as `Int`.
{style="note"}

#### Shuffle

To randomly shuffle the elements in an array, use the `.shuffle()` function:

```Kotlin
val simpleArray = arrayOf(1, 2, 3)

// Suffles elements
simpleArray.shuffle()
println(simpleArray.joinToString())
// 1, 3, 2

// Shuffles elements again
simpleArray.shuffle()
println(simpleArray.joinToString())
// 3, 1, 2
```

### Convert arrays to collections

If you work with different APIs where some use arrays and some use collections, 
then you can convert your arrays to collections and vice versa.

#### Convert to `List` or `Set`

To convert an array to a `List` or `Set`, use the `.toList()` and `.toSet()` functions.

```Kotlin
val simpleArray = arrayOf("a", "b", "c", "c")

// Converts to a Set
println(simpleArray.toSet())
// [a, b, c]

// Converts to a List
println(simpleArray.toList())
// [a, b, c, c]
```

#### Convert to `Map`

To convert an array to a `Map`, use the `.toMap()` function.

Only an array of `Pair<K,V>` can be converted to a `Map`. 

The first value of a `Pair` instance becomes a key, and the second becomes a value.

This example uses the infix notation to call the `to` function to create tuples of `Pair`:

```Kotlin
val pairArray = 
    arrayOf("apple" to 120, "banana" to 150, "cherry" to 90, "apple" to 140)

// Converts to a Map
// The keys are fruits and the values are their number of calories
println(pairArray.toMap())     
// {apple=140, banana=150, cherry=90}

// Note how keys must be unique, 
// so the latest value of "apple" overwrites the first
```

## Primitive-type arrays

If you use the `Array` class with primitive values, 
these values are boxed into objects. 

As an alternative, you can use primitive-type arrays, 
which allow you to store primitives in an array 
without the side effect of boxing overhead:

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

These classes have no inheritance relation to the `Array` class, 
but they have the same set of functions and properties.

This example creates an instance of the `IntArray` class:

```Kotlin
// Creates an array of Int of size 5 with the values initialized to zero
val exampleArray = IntArray(5)
println(exampleArray.joinToString())
// 0, 0, 0, 0, 0
```

> To convert primitive-type arrays to object-type arrays, 
> use the `.toTypedArray()` function.
> 
> To convert object-type arrays to primitive-type arrays, 
> use `.toBooleanArray()`, `.toByteArray()`, `.toCharArray()`, and so on.
{style="note"}
