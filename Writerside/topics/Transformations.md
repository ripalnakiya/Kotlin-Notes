# Transformations

The Kotlin standard library provides a set of extension functions for collection transformations. 

These functions build new collections from existing ones based on the transformation rules provided.

## Map

The **mapping** transformation creates a collection from the results of a function on the elements of another collection.

The basic mapping function is `map()`. It applies the given lambda function to each subsequent element and returns the list of the lambda results. The order of results is the same as the original order of elements.

To apply a transformation that additionally uses the element index as an argument, use `mapIndexed()`.

```Kotlin
val numbers = setOf(1, 2, 3)
println(numbers.map { it * 3 })
println(numbers.mapIndexed { idx, value -> value * idx })

// Output:
[3, 6, 9]
[0, 2, 6]
```

If the transformation produces `null` on certain elements, you can filter out the `null`s from the result collection by calling the `mapNotNull()` function instead of `map()`, or `mapIndexedNotNull()` instead of `mapIndexed()`.

```Kotlin
val numbers = setOf(1, 2, 3)
println(numbers.mapNotNull { if ( it == 2) null else it * 3 })
println(numbers.mapIndexedNotNull { idx, value -> if (idx == 0) null else value * idx })

// Output:
[3, 9]
[2, 6]
```

When transforming maps, you have two options: transform keys leaving values unchanged and vice versa. To apply a given transformation to keys, use `mapKeys()`; or use `mapValues()` to transforms values. 

Both functions use the transformations that take a map entry as an argument, so you can operate both its key and value.

```Kotlin
val numbersMap = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key11" to 11)
println(numbersMap.mapKeys { it.key.uppercase() })
println(numbersMap.mapValues { it.value + it.key.length })

// Output:
{KEY1=1, KEY2=2, KEY3=3, KEY11=11}
{key1=5, key2=6, key3=7, key11=16}
```

## Zip

Zipping transformation is building pairs from elements with the same positions in both collections. In the Kotlin standard library, this is done by the `zip()` extension function.

When called on a collection or an array with another collection (or array) as an argument, `zip()` returns the `List` of `Pair` objects. 

The elements of the receiver collection are the first elements in these pairs.

If the collections have different sizes, the result of the `zip()` is the smaller size; the last elements of the larger collection are not included in the result.

`zip()` can also be called in the infix form `a zip b`.

```Kotlin
val colors = listOf("red", "brown", "grey")
val animals = listOf("fox", "bear", "wolf")
println(colors zip animals)

val twoAnimals = listOf("fox", "bear")
println(colors.zip(twoAnimals))

// Output:
[(red, fox), (brown, bear), (grey, wolf)]
[(red, fox), (brown, bear)]
```

You can also call `zip()` with a transformation function that takes two parameters: the receiver element and the argument element. 

In this case, the result `List` contains the return values of the transformation function called on pairs of the receiver and the argument elements with the same positions.

```Kotlin
fun main() {
    val list1 = listOf(1, 2, 3)
    val list2 = listOf(4, 5, 6)

    // Using zip with a transformation function
    val result = list1.zip(list2) { a, b -> a + b }

    println(result) // Output: [5, 7, 9]
}
```

```Kotlin
val colors = listOf("red", "brown", "grey")
val animals = listOf("fox", "bear", "wolf")

println(colors.zip(animals) { color, animal -> "The ${animal.replaceFirstChar { it.uppercase() }} is $color"})

// Output:
[The Fox is red, The Bear is brown, The Wolf is grey]
```

When you have a `List` of `Pairs`, you can do the reverse transformation – **unzipping** – that builds two lists from these pairs:

To unzip a list of pairs, call `unzip()`.

```Kotlin
val numberPairs = listOf("one" to 1, "two" to 2, "three" to 3, "four" to 4)
println(numberPairs.unzip())
// ([one, two, three, four], [1, 2, 3, 4])
```

## Associate

**Association** transformations allow building maps from the collection elements and certain values associated with them. In different association types, the elements can be either keys or values in the association map.

The basic association function `associateWith()` creates a `Map` in which the elements of the original collection are keys, and values are produced from them by the given transformation function. If two elements are equal, only the last one remains in the map.

```Kotlin
val numbers = listOf("one", "two", "three", "four")
println(numbers.associateWith { it.length })
// {one=3, two=3, three=5, four=4}
```

For building maps with collection elements as values, there is the function `associateBy()`. It takes a function that returns a key based on an element's value. If two elements' keys are equal, only the last one remains in the map.

`associateBy()` can also be called with a value transformation function.

```Kotlin
val numbers = listOf("one", "two", "three", "four")

println(numbers.associateBy { it.first().uppercaseChar() })
println(numbers.associateBy(keySelector = { it.first().uppercaseChar() }, valueTransform = { it.length }))

// Output:
{O=one, T=three, F=four}
{O=3, T=5, F=4}
```

Another way to build maps in which both keys and values are somehow produced from collection elements is the function `associate()`. It takes a lambda function that returns a `Pair`: the key and the value of the corresponding map entry.

Note that `associate()` produces short-living `Pair` objects which may affect the performance. Thus, `associate()` should be used when the performance isn't critical or it's more preferable than other options.

An example of the latter is when a key and the corresponding value are produced from an element together.

```Kotlin
val names = listOf("Alice Adams", "Brian Brown", "Clara Campbell")
println(names.associate { name -> parseFullName(name).let { it.lastName to it.firstName } })  
// {Adams=Alice, Brown=Brian, Campbell=Clara}
```

Here we call a transform function on an element first, and then build a pair from the properties of that function's result.

## Flatten

If you operate nested collections, you may find the standard library functions that provide flat access to nested collection elements useful.

The first function is `flatten()`. You can call it on a collection of collections, for example, a `List` of `Sets`. The function returns a single `List` of all the elements of the nested collections.

```Kotlin
val numberSets = listOf(setOf(1, 2, 3), setOf(4, 5, 6), setOf(1, 2))
println(numberSets.flatten())
// [1, 2, 3, 4, 5, 6, 1, 2]
```

Another function – `flatMap()` provides a flexible way to process nested collections. It takes a function that maps a collection element to another collection. As a result, `flatMap()` returns a single list of its return values on all the elements. 

So, `flatMap()` behaves as a subsequent call of `map()` (with a collection as a mapping result) and `flatten()`.

```Kotlin
val containers = listOf(
    StringContainer(listOf("one", "two", "three")),
    StringContainer(listOf("four", "five", "six")),
    StringContainer(listOf("seven", "eight"))
)

val newContainers = containers.flatMap { it.values }
println(newContainers)
// [one, two, three, four, five, six, seven, eight]
```

## String representation

If you need to retrieve the collection content in a readable format, use functions that transform the collections to strings: `joinToString()` and `joinTo()`.

`joinToString()` builds a single `String` from the collection elements based on the provided arguments. `joinTo()` does the same but appends the result to the given `Appendable` object.

When called with the default arguments, the functions return the result similar to calling `toString()` on the collection: a `String` of elements' string representations separated by commas with spaces.

```Kotlin
val numbers = listOf("one", "two", "three", "four")

println(numbers)    // [one, two, three, four]         
println(numbers.joinToString())     // one, two, three, four

val listString = StringBuffer("The list of numbers: ")
numbers.joinTo(listString)
println(listString)     // The list of numbers: one, two, three, four
```

To build a custom string representation, you can specify its parameters in function arguments `separator`, `prefix`, and `postfix`. The resulting string will start with the `prefix` and end with the `postfix`. The `separator` will come after each element except the last.

```Kotlin
val numbers = listOf("one", "two", "three", "four")    
println(numbers.joinToString(separator = " | ", prefix = "start: ", postfix = ": end"))
// start: one | two | three | four: end
```

For bigger collections, you may want to specify the `limit` – a number of elements that will be included into result. If the collection size exceeds the `limit`, all the other elements will be replaced with a single value of the `truncated` argument.

```Kotlin
val numbers = (1..100).toList()
println(numbers.joinToString(limit = 10, truncated = "<...>"))
// 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, <...>
```

Finally, to customize the representation of elements themselves, provide the transform function.

```Kotlin
val numbers = listOf("one", "two", "three", "four")
println(numbers.joinToString { "Element: ${it.uppercase()}"})
// Element: ONE, Element: TWO, Element: THREE, Element: FOUR
```