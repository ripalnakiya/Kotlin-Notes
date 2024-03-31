# Retrieve Collection Parts

The Kotlin standard library contains extension functions for retrieving parts of a collection.

These functions provide a variety of ways to select elements for the result collection: listing their positions explicitly, specifying the result size, and others.

## Slice

`slice()` returns a list of the collection elements with given indices. The indices may be passed either as a range or as a collection of integer values.

```Kotlin
val numbers = listOf("one", "two", "three", "four", "five", "six")  
  
println(numbers.slice(1..3))            // [two, three, four]
println(numbers.slice(0..4 step 2))     // [one, three, five]
println(numbers.slice(setOf(3, 5, 0)))  // [four, six, one]
```

## Take and Drop

To get the specified number of elements starting from the first, use the `take()` function. For getting the last elements, use `takeLast()`. When called with a number larger than the collection size, both functions return the whole collection.

To take all the elements except a given number of first or last elements, call the `drop()` and `dropLast()` functions respectively.

```Kotlin
val numbers = listOf("one", "two", "three", "four", "five", "six")

println(numbers.take(3))        // [one, two, three]
println(numbers.takeLast(3))    // [four, five, six]
println(numbers.drop(1))        // [two, three, four, five, six]
println(numbers.dropLast(5))    // [one]
```

You can also use predicates to define the number of elements for taking or dropping. There are four functions similar to the ones described above:

- `takeWhile()` takes elements from the beginning of the collection as long as they match a given condition (predicate). As soon as it finds an element that doesn't match, it stops.

- `takeLastWhile()` takes elements from the end of the collection as long as they match a given condition (predicate). It stops when it finds an element that doesn't match.

- `dropWhile()` drops elements from the beginning of the collection as long as they match a given condition (predicate). It returns the rest of the elements starting from the last one that doesn't match the condition.

- `dropLastWhile()` drops elements from the end of the collection as long as they match a given condition (predicate). It returns the rest of the elements up to the last one that doesn't match the condition.

```Kotlin
val numbers = listOf("one", "two", "three", "four", "five", "six")

println(numbers.takeWhile { !it.startsWith('f') }) // [one, two, three]
println(numbers.takeLastWhile { it != "three" })   // [four, five, six]
println(numbers.dropWhile { it.length == 3 })      // [three, four, five, six]
println(numbers.dropLastWhile { it.contains('i') })// [one, two, three, four]
```

## Chunked

To break a collection into parts of a given size, use the `chunked() `function. 

`chunked()` takes a single argument – the size of the chunk – and returns a `List` of `Lists` of the given size. 

The first chunk starts from the first element and contains the `size` elements, the second chunk holds the next `size` elements, and so on. The last chunk may have a smaller size.

```Kotlin
val numbers = (0..13).toList()
println(numbers.chunked(3))

// Output:
[[0, 1, 2], [3, 4, 5], [6, 7, 8], [9, 10, 11], [12, 13]]
```

You can also apply a transformation for the returned chunks right away. 

To do this, provide the transformation as a lambda function when calling `chunked()`. The lambda argument is a chunk of the collection. 

When `chunked()` is called with a transformation, the chunks are short-living `List`s that should be consumed right in that lambda.

```Kotlin
val numbers = (0..13).toList() 
println(numbers.chunked(3) { it.sum() })  
// `it` is a chunk of the original collection

// Output:
[3, 12, 21, 30, 25]
```

## Windowed

You can retrieve all possible ranges of the collection elements of a given size.

The function for getting them is called `windowed()`: it returns a list of element ranges that you would see if you were looking at the collection through a sliding window of the given size.

Unlike `chunked()`, `windowed()` returns element ranges (windows) starting from each collection element. All the windows are returned as elements of a single `List`.

```Kotlin
val numbers = listOf("one", "two", "three", "four", "five")    
println(numbers.windowed(3))
// [[one, two, three], [two, three, four], [three, four, five]]
```

`windowed()` provides more flexibility with optional parameters:

- `step` defines a distance between first elements of two adjacent windows. By default the value is 1, so the result contains windows starting from all elements. If you increase the step to 2, you will receive only windows starting from odd elements: first, third, and so on.

- `partialWindows` includes windows of smaller sizes that start from the elements at the end of the collection. For example, if you request windows of three elements, you can't build them for the last two elements. Enabling `partialWindows` in this case includes **two** more lists of sizes 2 and 1.

```Kotlin
val numbers = (1..10).toList()
println(numbers.windowed(3, step = 2, partialWindows = true))
println(numbers.windowed(3) { it.sum() })

// Output:
[[1, 2, 3], [3, 4, 5], [5, 6, 7], [7, 8, 9], [9, 10]]
[6, 9, 12, 15, 18, 21, 24, 27]
```

To build two-element windows, there is a separate function - `zipWithNext()`. It creates pairs of adjacent elements of the receiver collection. 

Note that `zipWithNext()` doesn't break the collection into pairs; it creates a `Pair` for each element except the last one, so its result on `[1, 2, 3, 4]` is `[[1, 2], [2, 3], [3, 4]]`, not `[[1, 2], [3, 4]]`. 

`zipWithNext()` can be called with a transformation function as well; it should take two elements of the receiver collection as arguments.

```Kotlin
val numbers = listOf("one", "two", "three", "four", "five")    
println(numbers.zipWithNext())
println(numbers.zipWithNext() { s1, s2 -> s1.length > s2.length})

// Output:
[(one, two), (two, three), (three, four), (four, five)]
[false, false, true, false]
```