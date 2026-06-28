# Loops
<show-structure depth="2"/>

## For loops

Use the `for` loop to iterate through a [collection](https://kotlinlang.org/docs/collections-overview.html), [array](06-arrays.md), or [range](https://kotlinlang.org/docs/ranges.html):

```kotlin
for (item in collection) print(item)
```

The body of a `for` loop can be a block with curly braces `{}`.

```kotlin
fun main() {
    val shoppingList = listOf("Milk", "Bananas", "Bread")
    //sampleStart
    println("Things to buy:")
    for (item in shoppingList) {
        println("- $item")
    }
    // Things to buy:
    // - Milk
    // - Bananas
    // - Bread
    //sampleEnd
}
```
{kotlin-runnable="true" kotlin-min-compiler-version="1.3" id="kotlin-for-loop"}

### Ranges

To iterate over a range of numbers, use a [range expression](https://kotlinlang.org/docs/ranges.html) with `..` and `..<` operators:

```kotlin
fun main() {
//sampleStart
    println("Closed-ended range:")
    for (i in 1..6) {
        print(i)
    }
    // Closed-ended range:
    // 123456
  
    println("\nOpen-ended range:")
    for (i in 1..<6) {
        print(i)
    }
    // Open-ended range:
    // 12345
  
    println("\nReverse order in steps of 2:")
    for (i in 6 downTo 0 step 2) {
        print(i)
    }
    // Reverse order in steps of 2:
    // 6420
//sampleEnd
}
```
{kotlin-runnable="true" kotlin-min-compiler-version="1.3" id="kotlin-for-loop-range"}

### Arrays

If you want to iterate through an array or a list with an index, you can use the `indices` property:

```kotlin
fun main() {
    val routineSteps = arrayOf("Wake up", "Brush teeth", "Make coffee")
    //sampleStart
    for (i in routineSteps.indices) {
        println(routineSteps[i])
    }
    // Wake up
    // Brush teeth
    // Make coffee
    //sampleEnd
}
```
{kotlin-runnable="true" kotlin-min-compiler-version="1.3" id="kotlin-for-loop-array"}

Alternatively, you can use the [`.withIndex()`](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin.collections/with-index.html) function from the standard library:

```kotlin
fun main() {
    val routineSteps = arrayOf("Wake up", "Brush teeth", "Make coffee")
    //sampleStart
    for ((index, value) in routineSteps.withIndex()) {
        println("The step at $index is \"$value\"")
    }
    // The step at 0 is "Wake up"
    // The step at 1 is "Brush teeth"
    // The step at 2 is "Make coffee"
    //sampleEnd
}
```
{kotlin-runnable="true" kotlin-min-compiler-version="1.3" id="kotlin-for-loop-array-index"}

### Iterators

The `for` loop iterates through anything that provides an [iterator](https://kotlinlang.org/docs/iterators.html). Collections provide iterators by
default, whereas ranges and arrays are compiled into index-based loops.

You can create your own iterators by providing a member or extension function called `iterator()` that returns an `Iterator<>`.
The `iterator()` function must have a `next()` function and a `hasNext()` function that returns a `Boolean`.

The easiest way to create your own iterator for a class is to inherit from the [`Iterable<T>`](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin.collections/-iterable/) interface and override the
`iterator()`, `next()`, and `hasNext()` functions that are already there. For example:

```kotlin
class Booklet(val totalPages: Int) : Iterable<Int> {
    override fun iterator(): Iterator<Int> {
        return object : Iterator<Int> {
            var current = 1
            override fun hasNext() = current <= totalPages
            override fun next() = current++
        }
    }
}

fun main() {
    val booklet = Booklet(3)
    for (page in booklet) {
        println("Reading page $page")
    }
    // Reading page 1
    // Reading page 2
    // Reading page 3
}
```
{kotlin-runnable="true" kotlin-min-compiler-version="1.3" id="kotlin-for-loop-inherit-iterator"}

> Learn more about [interfaces](04-interfaces.md) and [inheritance](06-inheritance.md).
>
{style="tip"}

Alternatively, you can create the functions from scratch. In this case, add the `operator` keyword to the functions:

```kotlin
//sampleStart
class Booklet(val totalPages: Int) {
    operator fun iterator(): Iterator<Int> {
        return object {
            var current = 1

            operator fun hasNext() = current <= totalPages
            operator fun next() = current++
        }.let {
            object : Iterator<Int> {
                override fun hasNext() = it.hasNext()
                override fun next() = it.next()
            }
        }
    }
}
//sampleEnd

fun main() {
    val booklet = Booklet(3)
    for (page in booklet) {
        println("Reading page $page")
    }
    // Reading page 1
    // Reading page 2
    // Reading page 3
}
```
{kotlin-runnable="true" kotlin-min-compiler-version="1.3" id="kotlin-for-loop-iterator-from-scratch"}

## While loops

`while` and `do-while` loops run the code in their body continuously while the condition is satisfied.
The difference between them is the condition checking time:

* `while` checks the condition and, if it's satisfied, runs the code in its body and then returns to the condition check.
* `do-while` runs the code in its body and then checks the condition. If it's satisfied, the loop repeats. So, the body of `do-while`
  runs at least once, regardless of the condition.

For a `while` loop, place the condition to check in parentheses `()` and the body within curly braces `{}`:

```kotlin
fun main() {
    var carsInGarage = 0
    val maxCapacity = 3
//sampleStart
    while (carsInGarage < maxCapacity) {
        println("Car entered. Cars now in garage: ${++carsInGarage}")
    }
    // Car entered. Cars now in garage: 1
    // Car entered. Cars now in garage: 2
    // Car entered. Cars now in garage: 3

    println("Garage is full!")
    // Garage is full!
//sampleEnd
}
```
{kotlin-runnable="true" kotlin-min-compiler-version="1.3" id="kotlin-while-loop"}

For a `do-while` loop, place the body within curly braces `{}` first before the condition to check in parentheses `()`:

```kotlin
import kotlin.random.Random

fun main() {
    var roll: Int
//sampleStart
    do {
        roll = Random.nextInt(1, 7)
        println("Rolled a $roll")
    } while (roll != 6)
    // Rolled a 2
    // Rolled a 6
    
    println("Got a 6! Game over.")
    // Got a 6! Game over.
//sampleEnd
}
```
{kotlin-runnable="true" kotlin-min-compiler-version="1.3" id="kotlin-do-while-loop"}

## Break and continue in loops

Kotlin supports traditional `break` and `continue` operators in loops. See [Returns and jumps](03-returns.md).
