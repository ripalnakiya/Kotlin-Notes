# Iterators

For traversing collection elements, the Kotlin standard library supports the commonly used mechanism of **iterators** – objects that provide access to the elements sequentially without exposing the underlying structure of the collection. 

Iterators are useful when you need to process all the elements of a collection one-by-one, for example, print values or make similar updates to them.

Iterators can be obtained for inheritors of the `Iterable<T>` interface, including `Set` and `List`, by calling the `iterator()` function.

Once you obtain an iterator, it points to the first element of a collection; calling the `next()` function returns this element and moves the iterator position to the following element if it exists.

Once the iterator passes through the last element, it can no longer be used for retrieving elements; neither can it be reset to any previous position. To iterate through the collection again, create a new iterator.

```Kotlin
val numbers = listOf("one", "two", "three", "four")
val numbersIterator = numbers.iterator()
while (numbersIterator.hasNext()) {
    println(numbersIterator.next())
}

// Output:
one
two
three
four
```

Another way to go through an `Iterable` collection is the well-known `for` loop. When using `for` on a collection, you obtain the iterator implicitly. So, the following code is equivalent to the example above:

```Kotlin
val numbers = listOf("one", "two", "three", "four")
for (item in numbers) {
    println(item)
}
```

Finally, there is a useful `forEach()` function that lets you automatically iterate a collection and execute the given code for each element. So, the same example would look like this:

```Kotlin
val numbers = listOf("one", "two", "three", "four")
numbers.forEach {
    println(it)
}
```

## List iterators

For lists, there is a special iterator implementation: `ListIterator`. It supports iterating lists in both directions: forwards and backwards.

Backward iteration is implemented by the functions `hasPrevious()` and `previous()`. 

Additionally, the ListIterator provides information about the element indices with the functions `nextIndex()` and `previousIndex()`.

```Kotlin
val numbers = listOf("one", "two", "three", "four")
val listIterator = numbers.listIterator()
while (listIterator.hasNext()) listIterator.next()

println("Iterating backwards:")
while (listIterator.hasPrevious()) {
    print("Index: ${listIterator.previousIndex()}")
    println(", value: ${listIterator.previous()}")
}

// Output:
Iterating backwards:
Index: 3, value: four
Index: 2, value: three
Index: 1, value: two
Index: 0, value: one
```

Having the ability to iterate in both directions, means the `ListIterator` can still be used after it reaches the last element.

## Mutable iterators

For iterating mutable collections, there is `MutableIterator` that extends Iterator with the element removal function `remove()`. So, you can remove elements from a collection while iterating it.

```Kotlin
val numbers = mutableListOf("one", "two", "three", "four") 
val mutableIterator = numbers.iterator()

mutableIterator.next()
mutableIterator.remove()    
println("After removal: $numbers")
// After removal: [two, three, four]
```

In addition to removing elements, the MutableListIterator can also insert and replace elements while iterating the list by using the `add()` and `set()` functions.

```Kotlin
val numbers = mutableListOf("one", "four", "four") 
val mutableListIterator = numbers.listIterator()

mutableListIterator.next()
mutableListIterator.add("two")
println(numbers)
// [one, two, four, four]

mutableListIterator.next()
mutableListIterator.set("three")   
println(numbers)
// [one, two, three, four]
```
