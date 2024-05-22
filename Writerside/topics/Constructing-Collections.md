# Constructing Collections

## Construct from elements

The most common way to create a collection is with the standard library functions `listOf<T>()`, `setOf<T>()`, `mutableListOf<T>()`, `mutableSetOf<T>()`. 

If you provide a comma-separated list of collection elements as arguments, the compiler detects the element type automatically. When creating empty collections, specify the type explicitly.

```Kotlin
val numbersSet = setOf("one", "two", "three", "four")
val emptySet = mutableSetOf<String>()
```

The same is available for maps with the functions `mapOf()` and `mutableMapOf()`. The map's keys and values are passed as `Pair` objects (usually created with `to` infix function).

```Kotlin
val numbersMap = mapOf("key1" to 1, "key2" to 2, "key3" to 3)
```
Note that the `to` notation creates a short-living Pair object, so it's recommended that you use it only if performance isn't critical. To avoid excessive memory usage, use alternative ways. 

For example, you can create a mutable map and populate it using the write operations. The `apply()` function can help to keep the initialization fluent here.

```Kotlin
val numbersMap = mutableMapOf<String, String>()
                    .apply { this["one"] = "1"; this["two"] = "2" }
```

## Create with collection builder functions

Another way of creating a collection is to call a builder function – `buildList()`, `buildSet()`, or `buildMap()`. 

They create a new, mutable collection of the corresponding type, populate it using write operations, and return a read-only collection with the same elements:

```Kotlin
val map = buildMap { // this is MutableMap<String, Int>
// types of key and value are inferred from the `put()` calls below
    put("a", 1)
    put("b", 0)
    put("c", 4)
}

println(map) // {a=1, b=0, c=4}
```

## Empty collections

There are also functions for creating collections without any elements: `emptyList()`, `emptySet()`, and `emptyMap()`. When creating empty collections, you should specify the type of elements that the collection will hold.

```Kotlin
val empty = emptyList<String>()
```

## Initializer functions for lists

For lists, there is a constructor-like function that takes the list size and the initializer function that defines the element value based on its index.

```Kotlin
val doubled = List(3, { it * 2 })  // or MutableList if you want to change its content later
println(doubled)
```

## Concrete type constructors

To create a concrete type collection, such as an `ArrayList` or `LinkedList`, you can use the available constructors for these types. Similar constructors are available for implementations of `Set` and `Map`.

```Kotlin
val linkedList = LinkedList<String>(listOf("one", "two", "three"))
val presizedSet = HashSet<Int>(32)
```

## Copy

To create a collection with the same elements as an existing collection, you can use copying functions. 

Collection copying functions from the standard library **create shallow copy** collections with references to the same elements. Thus, a change made to a collection element reflects in all its copies.

Collection copying functions, such as `toList()`, `toMutableList()`, `toSet()` and others, create a snapshot of a collection at a specific moment. Their result is a new collection of the same elements. 

If you add or remove elements from the original collection, this won't affect the copies. Copies may be changed independently of the source as well.

```Kotlin
val alice = Person("Alice")
val sourceList = mutableListOf(alice, Person("Bob"))
val copyList = sourceList.toList()
sourceList.add(Person("Charles"))
alice.name = "Alicia"

println("First item's name is: ${sourceList[0].name} in source and ${copyList[0].name} in copy")
println("List size is: ${sourceList.size} in source and ${copyList.size} in copy")

// Output:
First item's name is: Alicia in source and Alicia in copy
List size is: 3 in source and 2 in copy
```

These functions can also be used for converting collections to other types, for example, build a set from a list or vice versa.

```Kotlin
val sourceList = mutableListOf(1, 2, 3)    
val copySet = sourceList.toMutableSet()
copySet.add(3)
copySet.add(4)    
println(copySet)    // [1, 2, 3, 4]
```

Alternatively, you can create new references to the same collection instance. New references are created when you initialize a collection variable with an existing collection. So, when the collection instance is altered through a reference, the changes are reflected in all its references.

```Kotlin
val sourceList = mutableListOf(1, 2, 3)
val referenceList = sourceList
referenceList.add(4)
println("Source size: ${sourceList.size}")      // Source size: 4
```

Collection initialization can be used for restricting mutability. For example, if you create a `List` reference to a `MutableList`, the compiler will produce errors if you try to modify the collection through this reference.

```Kotlin

val sourceList = mutableListOf(1, 2, 3)
val referenceList: List<Int> = sourceList

sourceList.add(4)
referenceList.add(4)            //compilation error

println(referenceList) // [1, 2, 3, 4]
```

## Invoke functions on other collections

Collections can be created as a result of various operations on other collections. 

For example, filtering a list creates a new list of elements that match the filter:

```Kotlin
val numbers = listOf("one", "two", "three", "four")  
val longerThan3 = numbers.filter { it.length > 3 }
println(longerThan3) 

// Output:
[three, four] 
```

[Mapping](Transformations.md#map) produces a list from a transformation's results:

```Kotlin
val numbers = setOf(1, 2, 3)
println(numbers.map { it * 3 })
println(numbers.mapIndexed { idx, value -> value * idx })

// Output:
[3, 6, 9]
[0, 2, 6]
```

[Association](Transformations.md#associate) produces maps:

```Kotlin
val numbers = listOf("one", "two", "three", "four")
println(numbers.associateWith { it.length })

// Output:
{one=3, two=3, three=5, four=4}
```







