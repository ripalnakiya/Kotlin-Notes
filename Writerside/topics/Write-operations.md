# Write operations

Mutable collections support operations for changing the collection contents, for example, adding or removing elements. On this page, we'll describe write operations available for all implementations of `MutableCollection`.

## Adding elements

To add a single element to a list or a set, use the `add()` function. The specified object is appended to the end of the collection.

```Kotlin
val numbers = mutableListOf(1, 2, 3, 4)
numbers.add(5)
println(numbers)                // [1, 2, 3, 4, 5]
```

`addAll()` adds every element of the argument object to a list or a set. The argument can be an `Iterable`, a `Sequence`, or an `Array`. 

The types of the receiver and the argument may be different, for example, you can add all items from a `Set` to a `List`.

When called on lists, `addAll()` adds new elements in the same order as they go in the argument. 

You can also call `addAll()` specifying an element position as the first argument. The first element of the argument collection will be inserted at this position. Other elements of the argument collection will follow it, shifting the receiver elements to the end.

```Kotlin
val numbers = mutableListOf(1, 2, 5, 6)
numbers.addAll(arrayOf(7, 8))
println(numbers)                // [1, 2, 5, 6, 7, 8]
numbers.addAll(2, setOf(3, 4))
println(numbers)                // [1, 2, 3, 4, 5, 6, 7, 8]
```

You can also add elements using the in-place version of the `plus` operator -` plusAssign` (`+=`) When applied to a mutable collection, `+=` appends the second operand (an element or another collection) to the end of the collection.

```Kotlin
val numbers = mutableListOf("one", "two")
numbers += "three"
println(numbers)                // [one, two, three]
numbers += listOf("four", "five")    
println(numbers)                // [one, two, three, four, five]
```

## Removing elements

To remove an element from a mutable collection, use the `remove()` function. `remove()` accepts the element value and removes one occurrence of this value.

```Kotlin
val numbers = mutableListOf(1, 2, 3, 4, 3)
numbers.remove(3)   // removes the first `3`
println(numbers)                // [1, 2, 4, 3]
numbers.remove(5)   // removes nothing
println(numbers)                // [1, 2, 4, 3]
```

For removing multiple elements at once, there are the following functions :

- `removeAll()` removes all elements that are present in the argument collection. Alternatively, you can call it with a predicate as an argument; in this case the function removes all elements for which the predicate yields `true`.

- `retainAll()` is the opposite of `removeAll()`: it removes all elements except the ones from the argument collection. When used with a predicate, it leaves only elements that match it.

- `clear()` removes all elements from a list and leaves it empty.

```Kotlin
val numbers = mutableListOf(1, 2, 3, 4)
println(numbers)                // [1, 2, 3, 4]
numbers.retainAll { it >= 3 }
println(numbers)                // [3, 4]
numbers.clear()
println(numbers)                // []

val numbersSet = mutableSetOf("one", "two", "three", "four")
numbersSet.removeAll(setOf("one", "two"))
println(numbersSet)             // [three, four]
```

Another way to remove elements from a collection is with the `minusAssign` (`-=`) operator – the in-place version of `minus`. 

The second argument can be a single instance of the element type or another collection. With a single element on the right-hand side, `-=` removes the first occurrence of it. 

In turn, if it's a collection, all occurrences of its elements are removed. For example, if a list contains duplicate elements, they are removed at once. 

The second operand can contain elements that are not present in the collection. Such elements don't affect the operation execution.

```Kotlin
val numbers = mutableListOf("one", "two", "three", "three", "four", "four")
numbers -= "three"
println(numbers)                // [one, two, three, four, four]

numbers -= listOf("four", "five")    
numbers -= listOf("four")       // does the same as above
println(numbers)                // [one, two, three]
```

## Updating elements

Lists and maps also provide operations for updating elements. 

For sets, updating doesn't make sense since it's actually removing an element and adding another one.