# Loops
<show-structure depth="2"/>

## For loops

Use the `for` loop to iterate through a [collection](https://kotlinlang.org/docs/collections-overview.html), [array](06-arrays.md), or [range](https://kotlinlang.org/docs/ranges.html):

```kotlin
for (item in collection) print(item)
```

The body of a `for` loop can be a block with curly braces `{}`.

```kotlin
    val shoppingList = listOf("Milk", "Bananas", "Bread")

    println("Things to buy:")
    for (item in shoppingList) {
        println("- $item")
    }
```

```text
Things to buy:
- Milk
- Bananas
- Bread
```

### Ranges

To iterate over a range of numbers, use a [range expression](https://kotlinlang.org/docs/ranges.html) with `..` and `..<` operators:

```kotlin
    // Closed-ended range:
    for (i in 1..6) {
        print(i)
    }
    // 123456

    // Open-ended range:
    for (i in 1..<6) {
        print(i)
    }
    // 12345

    // Reverse order in steps of 2:
    for (i in 6 downTo 0 step 2) {
        print(i)
    }
    // 6420
```

### Arrays

If you want to iterate through an array or a list with an index, you can use the `indices` property:

```kotlin

    val routineSteps = arrayOf("Wake up", "Brush teeth", "Make coffee")

    for (i in routineSteps.indices) {
        println(routineSteps[i])
    }
```

```text
Wake up
Brush teeth
Make coffee
```

Alternatively, you can use the [`.withIndex()`](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin.collections/with-index.html) function from the standard library:

```kotlin
    val routineSteps = arrayOf("Wake up", "Brush teeth", "Make coffee")

    for ((index, value) in routineSteps.withIndex()) {
        println("The step at $index is \"$value\"")
    }
```

```text
The step at 0 is "Wake up"
The step at 1 is "Brush teeth"
The step at 2 is "Make coffee"
```

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
}
```

```text
Reading page 1
Reading page 2
Reading page 3
```

> Learn more about [interfaces](04-interfaces.md) and [inheritance](06-inheritance.md).
>
{style="tip"}

Alternatively, you can create the functions from scratch. In this case, add the `operator` keyword to the functions:

```kotlin
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

fun main() {
    val booklet = Booklet(3)
    for (page in booklet) {
        println("Reading page $page")
    }
}
```

```text
Reading page 1
Reading page 2
Reading page 3
```

## While loops

`while` and `do-while` loops run the code in their body continuously while the condition is satisfied.
The difference between them is the condition checking time:

* `while` checks the condition and, if it's satisfied, runs the code in its body and then returns to the condition check.
* `do-while` runs the code in its body and then checks the condition. If it's satisfied, the loop repeats. So, the body of `do-while`
  runs at least once, regardless of the condition.

For a `while` loop, place the condition to check in parentheses `()` and the body within curly braces `{}`:

```kotlin
    var carsInGarage = 0
    val maxCapacity = 3

    while (carsInGarage < maxCapacity) {
        println("Car entered. Cars now in garage: ${++carsInGarage}")
    }

    println("Garage is full!")
```

```text
Car entered. Cars now in garage: 1
Car entered. Cars now in garage: 2
Car entered. Cars now in garage: 3
Garage is full!
```

For a `do-while` loop, place the body within curly braces `{}` first before the condition to check in parentheses `()`:

```kotlin
    var roll: Int

    do {
        roll = Random.nextInt(1, 7)
        println("Rolled a $roll")
    } while (roll != 6)
    
    println("Got a 6! Game over.")
```

```text
Rolled a 2
Rolled a 6
Got a 6! Game over.
```

## Break and continue in loops

Kotlin supports traditional `break` and `continue` operators in loops. See [Returns and jumps](03-returns.md).
