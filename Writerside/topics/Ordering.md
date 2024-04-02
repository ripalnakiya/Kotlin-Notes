# Ordering

The order of elements is an important aspect of certain collection types. For example, two lists of the same elements are not equal if their elements are ordered differently.

In Kotlin, the orders of objects can be defined in several ways.

First, there is **natural** order. It is defined for implementations of the Comparable interface. Natural order is used for sorting them when no other order is specified.

Most built-in types are comparable:

- Numeric types use the traditional numerical order: `1` is greater than `0`; `-3.4f` is greater than `-5f`, and so on.

- `Char` and `String` use the lexicographical order: `b` is greater than `a`; `world` is greater than `hello`.

To define a natural order for a user-defined type, make the type an implementer of `Comparable`. This requires implementing the `compareTo()` function. `compareTo()` must take another object of the same type as an argument and return an integer value showing which object is greater:

- Positive values show that the receiver object is greater.
- Negative values show that it's less than the argument.
- Zero shows that the objects are equal.

Below is a class for ordering versions that consist of the major and the minor part.

```Kotlin
class Version(val major: Int, val minor: Int): Comparable<Version> {
    override fun compareTo(other: Version): Int = when {
        this.major != other.major -> this.major compareTo other.major 
        this.minor != other.minor -> this.minor compareTo other.minor
                                    // compareTo() in the infix form 
        else -> 0
    }
}

fun main() {    
    println(Version(1, 2) > Version(1, 3))      // false
    println(Version(2, 0) > Version(1, 5))      // true
}
```

**Custom orders** let you sort instances of any type in a way you like. Particularly, you can define an order for non-comparable objects or define an order other than natural for a comparable type.

To define a custom order for a type, create a `Comparator` for it. `Comparator` contains the `compare()` function: it takes two instances of a class and returns the integer result of the comparison between them. The result is interpreted in the same way as the result of a `compareTo()` as is described above.

```Kotlin
val lengthComparator = Comparator { str1: String, str2: String -> str1.length - str2.length }
println(listOf("aaa", "bb", "c").sortedWith(lengthComparator))

// [c, bb, aaa]
```

Having the `lengthComparator`, you are able to arrange strings by their length instead of the default lexicographical order.


A shorter way to define a `Comparator` is the `compareBy()` function from the standard library. `compareBy()` takes a lambda function that produces a `Comparable` value from an instance and defines the custom order as the natural order of the produced values.

With `compareBy()`, the length comparator from the example above looks like this:

```Kotlin
println(listOf("aaa", "bb", "c").sortedWith(compareBy { it.length }))

// [c, bb, aaa]
```

On this page, we'll describe sorting functions that apply to [read-only](Collections-Overview.md#collection-types) collections. These functions return their result as a new collection containing the elements of the original collection in the requested order. 

To learn about functions for sorting mutable collections in place, see the [List-specific operations](List-specific-operations.md).

## Natural order

The basic functions `sorted()` and `sortedDescending()` return elements of a collection sorted into ascending and descending sequence according to their natural order. These functions apply to collections of `Comparable` elements.

```Kotlin
val numbers = listOf("one", "two", "three", "four")

println("Sorted ascending: ${numbers.sorted()}")
println("Sorted descending: ${numbers.sortedDescending()}")

// Output:
Sorted ascending: [four, one, three, two]
Sorted descending: [two, three, one, four]
```

## Custom orders

For sorting in custom orders or sorting non-comparable objects, there are the functions `sortedBy()` and `sortedByDescending()`.

They take a [selector function](Retrieve-Single-Elements.md#selector-function) that maps collection elements to `Comparable` values and sort the collection in natural order of that values.

```Kotlin
val numbers = listOf("one", "two", "three", "four")

val sortedNumbers = numbers.sortedBy { it.length }
println("Sorted by length ascending: $sortedNumbers")
// Sorted by length ascending: [one, two, four, three]

val sortedByLast = numbers.sortedByDescending { it.last() }
println("Sorted by the last letter descending: $sortedByLast")
// Sorted by the last letter descending: [four, two, one, three]
```

To define a custom order for the collection sorting, you can provide your own `Comparator`. To do this, call the `sortedWith()` function passing in your `Comparator`. 

With this function, sorting strings by their length looks like this:

```Kotlin
val numbers = listOf("one", "two", "three", "four")
val sortedNumbers = numbers.sortedWith(compareBy { it.length })
println("Sorted by length ascending: $sortedNumbers")
// Sorted by length ascending: [one, two, four, three]
```

```Kotlin
val numbers = listOf("one", "two", "three", "four")

// Define a custom comparator to sort by the length of each string
val comparator = compareBy<String> { it.length }
val sortedNumbers = numbers.sortedWith(comparator)
println("Sorted by length ascending: $sortedNumbers") 
// Sorted by length ascending: [one, two, four, three]
```

- Use `sortedBy()` when you have a straightforward sorting requirement based on a single property or value that is naturally comparable (e.g., length of strings, numeric values).
- Use `sortedWith()` when you need to sort based on more complex criteria, multiple properties, or when you need a custom comparison logic that can't be easily expressed with a single selector function.

## Reverse order

You can retrieve the collection in the reversed order using the `reversed()` function.

```Kotlin
val numbers = listOf("one", "two", "three", "four")
println(numbers.reversed()) // [four, three, two, one]
```

`reversed()` returns a new collection with the copies of the elements. So, if you change the original collection later, this won't affect the previously obtained results of` reversed()`.

Another reversing function - `asReversed()` - returns a reversed view of the same collection instance, so it may be more lightweight and preferable than `reversed()` if the original list is not going to change.

```Kotlin
val numbers = listOf("one", "two", "three", "four")
val reversedNumbers = numbers.asReversed()
println(reversedNumbers) // [four, three, two, one]
```

If the original list is mutable, all its changes reflect in its reversed views and vice versa.

```Kotlin
val numbers = mutableListOf("one", "two", "three", "four")
val reversedNumbers = numbers.asReversed()
println(reversedNumbers)    // [four, three, two, one]
numbers.add("five")
println(reversedNumbers)    // [five, four, three, two, one]
```

However, if the mutability of the list is unknown or the source is not a list at all, `reversed()` is more preferable since its result is a copy that won't change in the future.

## Random order

Finally, there is a function that returns a new List containing the collection elements in a random order - `shuffled()`. You can call it without arguments or with a `Random` object.

```Kotlin
val numbers = listOf("one", "two", "three", "four")
println(numbers.shuffled())     // [one, four, two, three]
```

Without passing a `Random` object, calling `shuffled()` will use the default random generator, which produces different results each time you run the program.

```Kotlin
import kotlin.random.Random

fun main() {
    val numbers = listOf(1, 2, 3, 4, 5)

    // Create a Random object with a specific seed for reproducibility
    val random = Random(42)
    
    // Create a Random object without specifying a seed
    val random = Random.Default

    // Shuffle the list using the Random object
    val shuffledNumbers = numbers.shuffled(random)
}
```

We create a `Random` object with a specific seed (42 in this case) for reproducibility. This means that every time we run the program with the same seed, the shuffled result will be the same.

`Random.Default` is used, which provides a random generator without a fixed seed.
The output will be different each time you run the program since the shuffling is truly random.
