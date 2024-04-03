# Aggregate operations

Kotlin collections contain functions for commonly used **aggregate operations** – operations that return a single value based on the collection content. Most of them are well known and work the same way as they do in other languages:

- `minOrNull()` and `maxOrNull()` return the smallest and the largest element respectively. On empty collections, they return `null`.

- `average()` returns the average value of elements in the collection of numbers.

- `sum()` returns the sum of elements in the collection of numbers.

- `count()` returns the number of elements in a collection.

```Kotlin
fun main() {
    val numbers = listOf(6, 42, 10, 4)

    println("Count: ${numbers.count()}")        // 4
    println("Max: ${numbers.maxOrNull()}")      // 42
    println("Min: ${numbers.minOrNull()}")      // 4
    println("Average: ${numbers.average()}")    // 15.5
    println("Sum: ${numbers.sum()}")            // 62
}
```

There are also functions for retrieving the smallest and the largest elements by certain selector function or custom `Comparator`:

- `maxByOrNull()` and `minByOrNull()` Find the element for which the selector function returns the largest or smallest value.
    ```Kotlin
    val numbers = listOf(1, 2, 3, 4, 5)
    
    val maxByOrNullExample = numbers.maxByOrNull { it }
    println("Max by: $maxByOrNullExample")  // Max by: 5
    
    val minByOrNullExample = numbers.minByOrNull { it }
    println("Min by: $minByOrNullExample")  // Min by: 1
    
    ```

- `maxWithOrNull()` and `minWithOrNull()` Find the largest or smallest element based on a provided Comparator.
  ```Kotlin
  val words = listOf("one", "two", "three", "four")

  val maxWithOrNullExample = words.maxWithOrNull(compareBy { it.length })
  println("Max with: $maxWithOrNullExample")  // Max with: three

  val minWithOrNullExample = words.minWithOrNull(compareBy { it.length })
  println("Min with: $minWithOrNullExample")  // Min with: one 
  ```

- `maxOfOrNull()` and `minOfOrNull()` Find the largest or smallest return value of the selector function itself.
  ```Kotlin
  val numbers = listOf(1, 2, 3, 4, 5)

  val maxOfOrNullExample = numbers.maxOfOrNull { it * 2 }
  println("Max of: $maxOfOrNullExample")  // Max of: 10

  val minOfOrNullExample = numbers.minOfOrNull { it * 2 }
  println("Min of: $minOfOrNullExample")  // Min of: 2
  ```

- `maxOfWithOrNull()` and `minOfWithOrNull()` Find the largest or smallest return value of the selector function based on a provided Comparator.
  ```Kotlin
  val words = listOf("one", "two", "three", "four")

  val maxOfWithOrNullExample = words.maxOfWithOrNull(compareBy { it.length }) { it.length }
  println("Max of with: $maxOfWithOrNullExample")  // Max of with: 5

  val minOfWithOrNullExample = words.minOfWithOrNull(compareBy { it.length }) { it.length }
  println("Min of with: $minOfWithOrNullExample")  // Min of with: 3 
  
  // Note:
  compareBy { it.length }: Comparator used to compare strings by their length.
  { it.length }: Selector function to extract the length of each string.
  ```

These functions return `null` on empty collections. 

There are also alternatives – `maxOf`, `minOf`, `maxOfWith`, and `minOfWith` – which do the same as their counterparts but throw a `NoSuchElementException` on empty collections.

```Kotlin
val numbers = listOf(5, 42, 10, 4)
val min3Remainder = numbers.minByOrNull { it % 3 }
println(min3Remainder)      // 42

val strings = listOf("one", "two", "three", "four")
val longestString = strings.maxWithOrNull(compareBy { it.length })
println(longestString)      // four
```

Besides regular `sum()`, there is an advanced summation function `sumOf()` that takes a selector function and returns the sum of its application to all collection elements. Selector can return different numeric types: `Int`, `Long`, `Double`, `UInt`, and `ULong` (also `BigInteger` and `BigDecimal` on the JVM).

```Kotlin
val numbers = listOf(5, 42, 10, 4)
println(numbers.sumOf { it * 2 })               // 122
println(numbers.sumOf { it.toDouble() / 2 })    // 30.5
```

## Fold and reduce

The `fold()` and `reduce()` functions in Kotlin are higher-order functions that allow you to perform accumulation operations on collections. 

Both functions traverse a collection and combine elements into a single result, but they differ in how they handle the initial value and the accumulator.

- The `fold()` function takes an initial value and a lambda function that operates on the accumulator and each element of the collection. The initial value is used as the starting point for the accumulation. 

  ```Kotlin
  val numbers = listOf(1, 2, 3, 4, 5)
  val sum = numbers.fold(0) { acc, num -> acc + num }
  println("Sum: $sum")  // Sum: 15
  ```

  ```Kotlin
  val numbers = listOf(1, 2, 3, 4, 5)
  val (sum, product) = numbers.fold(0 to 1) { acc, num ->
      acc.first + num to acc.second * num
  }
  println("Sum: $sum, Product: $product")  // Sum: 15, Product: 120
  ```
  - `fold()` can operate on an empty collection because it has an initial value.

- The `reduce()` function does not take an initial value. Instead, it uses the first element of the collection as the initial value and starts the accumulation from the second element. 

  ```Kotlin
  val numbers = listOf(1, 2, 3, 4, 5)
  val sum = numbers.reduce { acc, num -> acc + num }
  println("Sum: $sum")  // Sum: 15
  ```

  ```Kotlin
  val numbers = listOf(1, 2, 3, 4, 5)
  val product = numbers.reduce { acc, num -> acc * num }
  println("Product: $product")  // Product: 120
  ```

  - `reduce()` will throw an exception if the collection is empty because there is no initial value.

```Kotlin
val numbers = listOf(5, 2, 10, 4)

val sumDoubled = numbers.fold(0) { sum, element -> sum + element * 2 }
println(sumDoubled)         // 42

val simpleSum = numbers.reduce { sum, element -> sum + element }
println(simpleSum)          // 21

//incorrect: the first element isn't doubled in the result
val sumDoubledReduce = numbers.reduce { sum, element -> sum + element * 2 } 
println(sumDoubledReduce)     // 37
```

`fold()` is used for calculating the sum of doubled elements. If you pass the same function to `reduce()`, it will return another result because it uses the list's first and second elements as arguments on the first step, so the first element won't be doubled.

To apply a function to elements in the reverse order, use functions `reduceRight()` and `foldRight()`. They work in a way similar to `fold()` and `reduce()` but start from the last element and then continue to previous.

Note that when folding or reducing right, the operation arguments change their order: first goes the element, and then the accumulated value.

```Kotlin
val numbers = listOf(5, 2, 10, 4)
val sumDoubledRight = numbers.foldRight(0) { element, sum -> sum + element * 2 }
println(sumDoubledRight)    // 42
```

You can also apply operations that take element indices as parameters. For this purpose, use functions `reduceIndexed()` and `foldIndexed()` passing element index as the first argument of the operation.

Finally, there are functions that apply such operations to collection elements from right to left - `reduceRightIndexed()` and `foldRightIndexed()`.

```Kotlin
val numbers = listOf(5, 2, 10, 4)
val sumEven = numbers.foldIndexed(0) { idx, sum, element -> if (idx % 2 == 0) sum + element else sum }
println(sumEven)

val sumEvenRight = numbers.foldRightIndexed(0) { idx, element, sum -> if (idx % 2 == 0) sum + element else sum }
println(sumEvenRight)
```

All reduce operations throw an exception on empty collections. To receive `null` instead, use their `*OrNull()` counterparts:
- `reduceOrNull()`
- `reduceRightOrNull()`
- `reduceIndexedOrNull()`
- `reduceRightIndexedOrNull()`

For cases where you want to save intermediate accumulator values, there are functions `runningFold()` (or its synonym `scan()`) and `runningReduce()`.

```Kotlin
fun main() {
    val numbers = listOf(0, 1, 2, 3, 4, 5)
    
    val runningReduceSum = numbers.runningReduce { sum, item -> sum + item }
    val runningFoldSum = numbers.runningFold(10) { sum, item -> sum + item }
    
    val transform = { index: Int, element: Int -> "N = ${index + 1}: $element" }
    
    println(runningReduceSum.mapIndexed(transform).joinToString("\n", "Sum of first N elements with runningReduce:\n"))
    println(runningFoldSum.mapIndexed(transform).joinToString("\n", "Sum of first N elements with runningFold:\n"))
}

// Output:
Sum of first N elements with runningReduce:
N = 1: 0
N = 2: 1
N = 3: 3
N = 4: 6
N = 5: 10
N = 6: 15
Sum of first N elements with runningFold:
N = 1: 10
N = 2: 10
N = 3: 11
N = 4: 13
N = 5: 16
N = 6: 20
N = 7: 25
```

If you need an index in the operation parameter, use `runningFoldIndexed()` or `runningReduceIndexed()`.