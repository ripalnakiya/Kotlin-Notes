# Filtering Collections

In Kotlin, filtering conditions are defined by **predicates** – lambda functions that take a collection element and return a boolean value: `true` means that the given element matches the predicate, `false` means the opposite.

These functions leave the original collection unchanged, so they are available for both mutable and read-only collections. 

To operate the filtering result, you should assign it to a variable or chain the functions after filtering.

## Filter by Predicate

The basic filtering function is `filter()`. When called with a predicate, `filter()` returns the collection elements that match it. 

For both `List` and `Set`, the resulting collection is a `List`, for `Map` it's a `Map`.

```Kotlin
val numbers = listOf("one", "two", "three", "four")  
val longerThan3 = numbers.filter { it.length > 3 }
println(longerThan3)

val numbersMap = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key11" to 11)
val filteredMap = numbersMap.filter { (key, value) -> key.endsWith("1") && value > 10}
println(filteredMap)

// Output:
[three, four]
{key11=11}
```

The predicates in `filter()` can only check the values of the elements. If you want to use element positions in the filter, use `filterIndexed()`. It takes a predicate with two arguments: the index and the value of an element.

To filter collections by negative conditions, use `filterNot()`. It returns a list of elements for which the predicate yields `false`.

```Kotlin
val numbers = listOf("one", "two", "three", "four")

val filteredIdx = numbers.filterIndexed { index, s -> (index != 0) && (s.length < 5)  }
val filteredNot = numbers.filterNot { it.length <= 3 }

println(filteredIdx)
println(filteredNot)

// Output:
[two, four]
[three, four]
```

There are also functions that narrow the element type by filtering elements of a given type:

- `filterIsInstance()` returns collection elements of a given type. Being called on a `List<Any>`, `filterIsInstance<T>()` returns a `List<T>`, thus allowing you to call functions of the `T` type on its items.

    ```Kotlin
    val numbers = listOf(null, 1, "two", 3.0, "four")
    println("All String elements in upper case:")
    numbers.filterIsInstance<String>().forEach {
        println(it.uppercase())
    }
    
    // Output:
    All String elements in upper case:
    TWO
    FOUR
    ```

- `filterNotNull()` returns all non-nullable elements. Being called on a `List<T?>`, `filterNotNull()` returns a `List<T: Any>`, thus allowing you to treat the elements as non-nullable objects.

    ```Kotlin
    val numbers = listOf(null, "one", "two", null)
    numbers.filterNotNull().forEach {
        println(it.length)   // length is unavailable for nullable Strings
    }
    
    // Output:
    3
    3
    ```

## Partition

Another filtering function – `partition()` – filters a collection by a predicate and keeps the elements that don't match it in a separate list. 

So, you have a `Pair` of `Lists` as a return value: the first list containing elements that match the predicate and the second one containing everything else from the original collection.

```Kotlin
val numbers = listOf("one", "two", "three", "four")
val (match, rest) = numbers.partition { it.length > 3 }

println(match)
println(rest)

// Output:
[three, four]
[one, two]
```

## Test predicates

Finally, there are functions that simply test a predicate against collection elements:

- `any()` returns `true` if at least one element matches the given predicate.

- `none()` returns `true` if none of the elements match the given predicate.

- `all()` returns `true` if all elements match the given predicate. Note that `all()` returns true when called with any valid predicate on an empty collection. Such behavior is known as **vacuous truth**.

```Kotlin
val numbers = listOf("one", "two", "three", "four")

println(numbers.any { it.endsWith("e") })   // true
println(numbers.none { it.endsWith("a") })  // true
println(numbers.all { it.endsWith("e") })   // false

println(emptyList<Int>().all { it > 5 })   // vacuous truth
// true
```

`any()` and `none()` can also be used without a predicate: in this case they just check the collection emptiness. `any()` returns true if there are elements and false if there aren't; `none()` does the opposite.

```Kotlin
val numbers = listOf("one", "two", "three", "four")
val empty = emptyList<String>()

println(numbers.any())      // true
println(empty.any())        // false

println(numbers.none())     // false
println(empty.none())       // true
```