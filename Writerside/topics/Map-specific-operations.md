# Map-specific operations

<show-structure depth="2"/>

In maps, types of both keys and values are user-defined. 

Key-based access to map entries enables various map-specific processing capabilities from getting a value by key to separate filtering of keys and values. 

## Retrieve keys and values

For retrieving a value from a map, you must provide its key as an argument of the `get()` function. The shorthand `[key]` syntax is also supported. If the given key is not found, it returns `null`. 

There is also the function `getValue()` which has slightly different behavior: it throws an exception if the key is not found in the map.

Additionally, you have two more options to handle the key absence:

- `getOrElse()` works the same way as for lists: the values for non-existent keys are returned from the given lambda function.

- `getOrDefault()` returns the specified default value if the key is not found.

```Kotlin
val numbersMap = mapOf("one" to 1, "two" to 2, "three" to 3)
println(numbersMap.get("one"))                  // 1
println(numbersMap["one"])                      // 1
println(numbersMap.getOrDefault("four", 10))    // 10
println(numbersMap["five"])                     // null
numbersMap.getValue("six")                      // exception!
```

To perform operations on all keys or all values of a map, you can retrieve them from the properties `keys` and `values` accordingly. `keys` is a set of all map keys and `values` is a collection of all map values.

```Kotlin
val numbersMap = mapOf("one" to 1, "two" to 2, "three" to 3)
println(numbersMap.keys)                        // [one, two, three]
println(numbersMap.values)                      // [1, 2, 3]
```

## Filter

You can filter maps with the `filter()` function as well as other collections. 

When calling `filter()` on a map, pass to it a predicate with a `Pair` as an argument. This enables you to use both the key and the value in the filtering predicate.

```Kotlin
val numbersMap = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key11" to 11)
val filteredMap = numbersMap.filter { (key, value) -> key.endsWith("1") && value > 10}
println(filteredMap)
// {key11=11}
```

There are also two specific ways for filtering maps: by keys and by values. 

For each way, there is a function: `filterKeys()` and `filterValues()`. Both return a **new map** of entries which match the given predicate. The predicate for `filterKeys()` checks only the element keys, the one for `filterValues()` checks only values.

```Kotlin
val numbersMap = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key11" to 11)
val filteredKeysMap = numbersMap.filterKeys { it.endsWith("1") }
val filteredValuesMap = numbersMap.filterValues { it < 10 }

println(filteredKeysMap)            // {key1=1, key11=11}
println(filteredValuesMap)          // {key1=1, key2=2, key3=3}
```

## Plus and minus operators

Due to the key access to elements, `plus` (`+`) and `minus` (`-`) operators work for maps differently than for other collections. 

`plus` returns a `Map` that contains elements of its both operands: a `Map` on the left and a `Pair` or another `Map` on the right. 

When the right-hand side operand contains entries with keys present in the left-hand side `Map`, the result map contains the entries from the right side.

```Kotlin
val numbersMap = mapOf("one" to 1, "two" to 2, "three" to 3)

println(numbersMap + Pair("four", 4))
// {one=1, two=2, three=3, four=4}

println(numbersMap + Pair("one", 10))
// {one=10, two=2, three=3}

println(numbersMap + mapOf("five" to 5, "one" to 11))
// {one=11, two=2, three=3, five=5}
```

`minus` creates a `Map` from entries of a `Map` on the left except those with keys from the right-hand side operand. So, the right-hand side operand can be either a single key or a collection of keys: list, set, and so on.

```Kotlin
val numbersMap = mapOf("one" to 1, "two" to 2, "three" to 3)
println(numbersMap - "one")                         // {two=2, three=3}
println(numbersMap - listOf("two", "four"))         // {one=1, three=3}
```

## Map write operations

There are certain rules that define write operations on maps:

- Values can be updated. In turn, keys never change: once you add an entry, its key is constant.

- For each key, there is always a single value associated with it. You can add and remove whole entries.

### Add and update entries

To add a new key-value pair to a mutable map, use `put()`. When a new entry is put into a `LinkedHashMap` (the default map implementation), it is added so that it comes last when iterating the map. 

In sorted maps, the positions of new elements are defined by the order of their keys.

```Kotlin
val numbersMap = mutableMapOf("one" to 1, "two" to 2)
numbersMap.put("three", 3)
println(numbersMap)                 // {one=1, two=2, three=3}
```

To add multiple entries at a time, use `putAll()`. Its argument can be a `Map` or a group of `Pair`s: `Iterable`, `Sequence`, or `Array`.

```Kotlin
val numbersMap = mutableMapOf("one" to 1, "two" to 2, "three" to 3)
numbersMap.putAll(setOf("four" to 4, "five" to 5))
println(numbersMap)                 
// {one=1, two=2, three=3, four=4, five=5}
```

Both `put()` and `putAll()` overwrite the values if the given keys already exist in the map. Thus, you can use them to update values of map entries.

```Kotlin
val numbersMap = mutableMapOf("one" to 1, "two" to 2)
val previousValue = numbersMap.put("one", 11)
println("value associated with 'one', before: $previousValue, after: ${numbersMap["one"]}")
println(numbersMap)

// Output:
value associated with 'one', before: 1, after: 11
{one=11, two=2}
```

You can also add new entries to maps using the shorthand operator form. There are two ways:

- `plusAssign` (`+=`) operator.

- the `[]` operator alias for `set()`

```Kotlin
val numbersMap = mutableMapOf("one" to 1, "two" to 2)
numbersMap["three"] = 3     // calls numbersMap.put("three", 3)
numbersMap += mapOf("four" to 4, "five" to 5)
println(numbersMap)             
// {one=1, two=2, three=3, four=4, five=5}
```

When called with the key present in the map, operators overwrite the values of the corresponding entries.

### Remove entries

To remove an entry from a mutable map, use the `remove()` function. When calling `remove()`, you can pass either a key or a whole key-value-pair. 

If you specify both the key and value, the element with this key will be removed only if its value matches the second argument.

```Kotlin
val numbersMap = mutableMapOf("one" to 1, "two" to 2, "three" to 3)
numbersMap.remove("one")
println(numbersMap)                             // {two=2, three=3}
numbersMap.remove("three", 4)   //doesn't remove anything
println(numbersMap)                             // {two=2, three=3}
``` 

You can also remove entries from a mutable map by their keys or values. To do this, call `remove()` on the map's keys or values providing the key or the value of an entry. When called on values, `remove()` removes only the first entry with the given value.

```Kotlin
val numbersMap = mutableMapOf("one" to 1, "two" to 2, "three" to 3, "threeAgain" to 3)
numbersMap.keys.remove("one")
println(numbersMap)                         // {two=2, three=3, threeAgain=3}
numbersMap.values.remove(3)
println(numbersMap)                         // {two=2, threeAgain=3}
```

The `minusAssign` (`-=`) operator is also available for mutable maps.

```Kotlin
val numbersMap = mutableMapOf("one" to 1, "two" to 2, "three" to 3)
numbersMap -= "two"
println(numbersMap)                                 // {one=1, three=3}
numbersMap -= "five"    //doesn't remove anything
println(numbersMap)                                 // {one=1, three=3}
```