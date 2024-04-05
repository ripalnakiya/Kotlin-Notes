# List-specific operations

<show-structure depth="2"/>

## Retrieve elements by index

Lists support all common operations for element retrieval: `elementAt()`, `first()`, `last()`, and others listed in [Retrieve single elements](Retrieve-Single-Elements.md). 

What is specific for lists is index access to the elements, so the simplest way to read an element is retrieving it by index. That is done with the `get()` function with the index passed in the argument or the shorthand `[index]` syntax.

If the list size is less than the specified index, an exception is thrown. There are two other functions that help you avoid such exceptions:

- `getOrElse()` lets you provide the function for calculating the default value to return if the index isn't present in the collection.

- `getOrNull()` returns `null` as the default value.

```Kotlin
val numbers = listOf(1, 2, 3, 4)
println(numbers.get(0))                 // 1
println(numbers[0])                     // 1
numbers.get(5)                          // exception!
println(numbers.getOrNull(5))           // null
println(numbers.getOrElse(5, {it}))     // 5
```

## Retrieve list parts

In addition to common operations for [Retrieving Collection Parts](Retrieve-Collection-Parts.md), lists provide the `subList()` function that returns a view of the specified elements range as a list. Thus, if an element of the original collection changes, it also changes in the previously created sublists and vice versa.

```Kotlin
val numbers = (0..13).toList()
println(numbers.subList(3, 6))          // [3, 4, 5]
```

## Find element positions

### Linear search

In any lists, you can find the position of an element using the functions `indexOf()` and `lastIndexOf()`. They return the first and the last position of an element equal to the given argument in the list. If there are no such elements, both functions return `-1`.

```Kotlin
val numbers = listOf(1, 2, 3, 4, 2, 5)
println(numbers.indexOf(2))             // 1
println(numbers.lastIndexOf(2))         // 4
```

There is also a pair of functions that take a predicate and search for elements matching it:

- `indexOfFirst()` returns the **index of the first** element matching the predicate or `-1` if there are no such elements.

- `indexOfLast()` returns the **index of the last** element matching the predicate or `-1` if there are no such elements.

```Kotlin
val numbers = mutableListOf(1, 2, 3, 4)
println(numbers.indexOfFirst { it > 2})         // 2
println(numbers.indexOfLast { it % 2 == 1})     // 2
```

### Binary search in sorted lists

It works significantly faster than other built-in search functions but **requires the list to be sorted** in ascending order according to a certain ordering: natural or another one provided in the function parameter. Otherwise, the result is undefined.

To search an element in a sorted list, call the `binarySearch()` function passing the value as an argument. If such an element exists, the function returns its index; otherwise, it returns (`-insertionPoint - 1`) where `insertionPoint` is the index where this element should be inserted so that the list remains sorted.

If there is more than one element with the given value, the search can return any of their indices.

You can also specify an index range to search in: in this case, the function searches only between two provided indices.

```Kotlin
val numbers = mutableListOf("one", "two", "three", "four")
numbers.sort()
println(numbers)
println(numbers.binarySearch("two"))            // 3
println(numbers.binarySearch("z"))              // -5
println(numbers.binarySearch("two", 0, 2))      // -3
```

#### Comparator binary search

When list elements aren't `Comparable`, you should provide a `Comparator` to use in the binary search. The list must be sorted in ascending order according to this `Comparator`.

```Kotlin
val productList = listOf(
    Product("WebStorm", 49.0),
    Product("AppCode", 99.0),
    Product("DotTrace", 129.0),
    Product("ReSharper", 149.0))

println(productList.binarySearch(Product("AppCode", 99.0), compareBy<Product> { it.price }.thenBy { it.name })) // 1
```

Here's a list of `Product` instances that aren't `Comparable` and a `Comparator` that defines the order: product `p1` precedes product `p2` if `p1`'s price is less than `p2`'s price. So, having a list sorted ascending according to this order, we use `binarySearch()` to find the index of the specified Product.

<note>

The `compareBy` function is used to create a comparator that initially compares `Product` objects by their `price` property. However, if two `Product` objects have the same `price`, the `thenBy` function is used to provide a secondary comparison based on the `name` property.
</note>

Custom comparators are also handy when a list uses an order different from natural one, for example, a case-insensitive order for `String` elements.

```Kotlin
val colors = listOf("Blue", "green", "ORANGE", "Red", "yellow")
println(colors.binarySearch("RED", String.CASE_INSENSITIVE_ORDER)) // 3
```

#### Comparison binary search

Binary search with **comparison function** lets you find elements without providing explicit search values. 

Instead, it takes a comparison function mapping elements to `Int` values and searches for the element where the function returns zero. 

The list must be sorted in the ascending order according to the provided function; in other words, the return values of comparison must grow from one list element to the next one.

```Kotlin
data class Product(val name: String, val price: Double)

fun priceComparison(product: Product, price: Double) = sign(product.price - price).toInt()

fun main() {
    val productList = listOf(
        Product("WebStorm", 49.0),
        Product("AppCode", 99.0),
        Product("DotTrace", 129.0),
        Product("ReSharper", 149.0))

    println(productList.binarySearch { priceComparison(it, 99.0) })
}
```

<note>

The `sign` function in Kotlin is used to determine the sign of a numeric value. It returns:
- `-1.0` if the value is negative
- `0.0` if the value is zero
- `1.0` if the value is positive
</note>

Both comparator and comparison binary search can be performed for list ranges as well.

## List write operations

In addition to the collection modification operations described in [Collection write operations](Write-operations.md), mutable lists support specific write operations.

### Add

To add elements to a specific position in a list, use `add()` and `addAll()` providing the position for element insertion as an additional argument. All elements that come after the position shift to the right.

```Kotlin
val numbers = mutableListOf("one", "five", "six")
numbers.add(1, "two")
numbers.addAll(2, listOf("three", "four"))
println(numbers)
// [one, two, three, four, five, six]
```

### Update

Lists also offer a function to replace an element at a given position - `set()` and its operator form `[]`. `set()` doesn't change the indexes of other elements.

```Kotlin
val numbers = mutableListOf("one", "five", "three")
numbers[1] =  "two"
println(numbers)                // [one, two, three]
```

`fill()` simply replaces all the collection elements with the specified value.

```Kotlin
val numbers = mutableListOf(1, 2, 3, 4)
numbers.fill(3)
println(numbers)                // [3, 3, 3, 3]
```

### Remove

To remove an element at a specific position from a list, use the `removeAt()` function providing the position as an argument. All indices of elements that come after the element being removed will decrease by one.

```Kotlin
val numbers = mutableListOf(1, 2, 3, 4, 3)    
numbers.removeAt(1)
println(numbers)                // [1, 3, 4, 3]
```

### Sort

In [Collection Ordering](Ordering.md), we describe operations that retrieve collection elements in specific orders. 

For mutable lists, the standard library offers similar extension functions that perform the same ordering operations in place. When you apply such an operation to a list instance, it changes the order of elements in that exact instance.

The in-place sorting functions have similar names to the functions that apply to read-only lists, but without the `ed`/`d` suffix:

- `sort*` instead of `sorted*` in the names of all sorting functions: `sort()`, `sortDescending()`, `sortBy()`, and so on.

- `shuffle()` instead of `shuffled()`.

- `reverse()` instead of `reversed()`.

`asReversed()` called on a mutable list returns another mutable list which is a reversed view of the original list. Changes in that view are reflected in the original list.

```Kotlin
val numbers = mutableListOf("one", "two", "three", "four")

numbers.sort()
println("Sort into ascending: $numbers")        
// [four, one, three, two]

numbers.sortDescending()
println("Sort into descending: $numbers")
// [two, three, one, four]


numbers.sortBy { it.length }
println("Sort into ascending by length: $numbers")
// [two, one, four, three]

numbers.sortByDescending { it.last() }
println("Sort into descending by the last letter: $numbers")
// [four, two, one, three]

numbers.sortWith(compareBy<String> { it.length }.thenBy { it })
println("Sort by Comparator: $numbers")
// [one, two, four, three]

numbers.shuffle()
println("Shuffle: $numbers")
// [two, one, four, three]

numbers.reverse()
println("Reverse: $numbers")
// [three, four, one, two]
```








