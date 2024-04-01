# Retrieve Single Elements

Kotlin collections provide a set of functions for retrieving single elements from collections. Functions described on this page apply to both lists and sets.

As the definition of `List` says, a list is an ordered collection. Hence, every element of a list has its position that you can use for referring.

In turn, `Set` is not an ordered collection by definition. However, the Kotlin Set stores elements in certain orders. These can be the order of insertion (in `LinkedHashSet`), natural sorting order (in `SortedSet`), or another order. The order of a set of elements can also be unknown. In such cases, the elements are still ordered somehow, so the functions that rely on the element positions still return their results. However, such results are unpredictable to the caller unless they know the specific implementation of `Set` used.

## Retrieve by position

For retrieving an element at a specific position, there is the function `elementAt()`. Call it with the integer number as an argument, and you'll receive the collection element at the given position. 

The first element has the position `0`, and the last one is `(size - 1)`.

`elementAt()` is useful for collections that do not provide indexed access, or are not statically known to provide one. In case of `List`, it's more idiomatic to use indexed access operator (`get()` or `[]`).

```Kotlin
val numbers = linkedSetOf("one", "two", "three", "four", "five")
println(numbers.elementAt(3))                   // four

val numbersSortedSet = sortedSetOf("one", "two", "three", "four")
println(numbersSortedSet.elementAt(0))          // four
// elements are stored in the ascending order
```

There are also useful aliases for retrieving the first and the last element of the collection: `first()` and `last()`.

```Kotlin
val numbers = listOf("one", "two", "three", "four", "five")
println(numbers.first())        // one
println(numbers.last())         // five
```

To avoid exceptions when retrieving element with non-existing positions, use safe variations of `elementAt()`:

- `elementAtOrNull()` returns null when the specified position is out of the collection bounds.

- `elementAtOrElse()` additionally takes a lambda function that maps an `Int` argument to an instance of the collection element type. When called with an out-of-bounds position, the `elementAtOrElse()` returns the result of the lambda on the given value.

```Kotlin
val numbers = listOf("one", "two", "three", "four", "five")
println(numbers.elementAtOrNull(5))
println(numbers.elementAtOrElse(5) { index -> "The value for index $index is undefined"})

// Output:
null
The value for index 5 is undefined
```

## Retrieve by condition

Functions `first()` and `last()` also let you search a collection for elements matching a given predicate. 

When you call `first()` with a predicate that tests a collection element, you'll receive the first element on which the predicate yields true. In turn, `last()` with a predicate returns the last element matching it.

```Kotlin
val numbers = listOf("one", "two", "three", "four", "five", "six")
println(numbers.first { it.length > 3 })        // three
println(numbers.last { it.startsWith("f") })    // five
```

If no elements match the predicate, both functions throw exceptions. To avoid them, use `firstOrNull()` and `lastOrNull()` instead: they return `null` if no matching elements are found.

```Kotlin
val numbers = listOf("one", "two", "three", "four", "five", "six")
println(numbers.firstOrNull { it.length > 6 })      // null
```

Use the aliases if their names suit your situation better:

- `find()` instead of `firstOrNull()`

- `findLast()` instead of `lastOrNull()`

```Kotlin
val numbers = listOf(1, 2, 3, 4)
println(numbers.find { it % 2 == 0 })           // 2
println(numbers.findLast { it % 2 == 0 })       // 4
```

###  Selector Function

To map each element of a collection to a specific value or property, often for the purpose of sorting or filtering.

Commonly used in functions like `sortedBy()`, `groupBy()`, `associateBy()`, etc.

The value returned by the selector function should be comparable if used for sorting.

```Kotlin
val words = listOf("apple", "banana", "cherry", "date")

// Define a selector function to extract the length of each string
val sortedByLength = words.sortedBy { it.length }

println("Sorted by length: $sortedByLength")
// Sorted by length: [date, apple, cherry, banana]
```

```Kotlin
data class Person(val name: String, val age: Int)

val people = listOf(
    Person("Alice", 30),
    Person("Bob", 25),
    Person("Charlie", 35)
)

// Define a selector function to extract the age of each person
val sortedByAge = people.sortedBy { it.age }

println("Sorted by age: $sortedByAge")
// Sorted by age: [Person(name=Bob, age=25), Person(name=Alice, age=30), Person(name=Charlie, age=35)]
```

## Retrieve with selector

If you need to map the collection before retrieving the element, there is a function `firstNotNullOf()`. It combines 2 actions:
- Maps the collection with the selector function
- Returns the first non-null value in the result

`firstNotNullOf()` throws the `NoSuchElementException` if the resulting collection doesn't have a non-nullable element. Use the counterpart `firstNotNullOfOrNull()` to return null in this case.

```Kotlin
val list = listOf<Any>(0, "true", false)
// Converts each element to string and 
// returns the first one that has required length
val longEnough = list.firstNotNullOf { item -> item.toString().takeIf { it.length >= 4 } }
println(longEnough)
```

## Random element

If you need to retrieve an arbitrary element of a collection, call the `random()` function. You can call it without arguments or with a `Random` object as a source of the randomness.

```Kotlin
val numbers = listOf(1, 2, 3, 4)
println(numbers.random())           // 2
```

On empty collections, `random()` throws an exception. To receive `null` instead, use `randomOrNull()`.

## Check element existence

To check the presence of an element in a collection, use the `contains()` function. 

It returns `true` if there is a collection element that `equals()` the function argument. You can call `contains()` in the operator form with the `in` keyword.

To check the presence of multiple instances together at once, call `containsAll()` with a collection of these instances as an argument.

```Kotlin
val numbers = listOf("one", "two", "three", "four", "five", "six")
println(numbers.contains("four"))                       // true
println("zero" in numbers)                              // false

println(numbers.containsAll(listOf("four", "two")))     // true
println(numbers.containsAll(listOf("one", "zero")))     // false
```

Additionally, you can check if the collection contains any elements by calling `isEmpty()` or `isNotEmpty()`.

```Kotlin
val numbers = listOf("one", "two", "three", "four", "five", "six")
println(numbers.isEmpty())              // false
println(numbers.isNotEmpty())           // true

val empty = emptyList<String>()
println(empty.isEmpty())                // true
println(empty.isNotEmpty())             // false
```








