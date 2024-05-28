# Grouping

The Kotlin standard library provides extension functions for grouping collection elements.

- The basic function `groupBy()` takes a lambda function and returns a `Map`. 
  - In this map, each key is the lambda result and the corresponding value is the `List` of elements on which this result is returned.

This function can be used, for example, to group a list of `String`s by their first letter.

- You can also call `groupBy()` with a second lambda argument – a value transformation function. 
    - In the result map of `groupBy()` with two lambdas, the keys produced by `keySelector` function are mapped to the results of the value transformation function instead of the original elements.

```Kotlin
val numbers = listOf("one", "two", "three", "four", "five")

println(numbers.groupBy { it.first().uppercase() })
println(numbers.groupBy(keySelector = { it.first() }, valueTransform = { it.uppercase() }))

// Output:
{O=[one], T=[two, three], F=[four, five]}
{o=[ONE], t=[TWO, THREE], f=[FOUR, FIVE]}
```

## GroupingBy

If you want to group elements and then apply an operation to all groups at one time, use the function `groupingBy()`. It returns an instance of the `Grouping` type. 

The `Grouping` instance lets you apply operations to all groups in a lazy manner: the groups are actually built right before the operation execution.

<note>

Lazy manner: When you use the `groupingBy()` function, it creates a `Grouping` instance. This instance represents the concept of grouping but doesn't actually create the groups yet. The groups are built only when you call an operation like `eachCount`, `fold`, `reduce`, or `aggregate`.

- Efficiency: This approach is more efficient because it avoids creating intermediate collections unnecessarily.
- Performance: It saves memory and processing time since it only processes the elements when needed. 
</note>

`Grouping` supports the following operations:

- `eachCount()` counts the elements in each group.

- `fold()` and `reduce()` perform fold and reduce operations on each group as a separate collection and return the results.

```Kotlin
val numbers = listOf("one", "two", "three", "four", "five", "six")

// Create a Grouping instance, no groups are built yet
val grouping = numbers.groupingBy { it.first() }

// Now we apply an operation that will trigger the actual grouping
val result = grouping.eachCount()

// Print the result, which now contains the counts of each group
println(result) 

// Output: 
{o=1, t=2, f=2, s=1}
```

- `aggregate()` applies a given operation subsequently to all the elements in each group and returns the result. This is the generic way to perform any operations on a `Grouping`.
  - Use it to implement custom operations, when fold or reduce are not enough.

```Kotlin
val words = listOf("apple", "apricot", "banana", "blueberry", "cherry")

// The aggregate function takes four parameters:
val result = words.groupingBy { it.first() }.aggregate { key, accumulator: StringBuilder?, element, first ->
    if (first) {
        // Initialize the accumulator with the first element
        StringBuilder(element)
    } else {
        // Append the element to the accumulator
        accumulator!!.append(", ").append(element)
    }
}

println(result)

// Output: 
{a=apple, apricot, b=banana, blueberry, c=cherry}
```

The `aggregate` function applies a given operation to each element in each group. You can specify how to initialize the accumulator, how to accumulate values, and how to combine results from different elements.