# Loops
<show-structure depth="2"/>

## For loops

Use the `for` loop to iterate through a collection, array, or range:

```Kotlin
for (item in collection) print(item)
```

The body of a `for` loop can be a block with curly braces `{}`.

```Kotlin
    val shoppingList = listOf("Milk", "Bananas", "Bread")

    println("Things to buy:")
    for (item in shoppingList) {
        println("- $item")
    }
    // Things to buy:
    // - Milk
    // - Bananas
    // - Bread
```

### Ranges

To iterate over a range of numbers, 
use a range expression with `..` and `..<` operators:

```Kotlin
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
```

### Arrays

If you want to iterate through an array or a list with an index, 
you can use the `indices` property:

```Kotlin
    val routineSteps = arrayOf("Wake up", "Brush teeth", "Make coffee")

    for (i in routineSteps.indices) {
        println(routineSteps[i])
    }
    // Wake up
    // Brush teeth
    // Make coffee
```

Alternatively, you can use the `.withIndex()` function from the standard library:

```Kotlin
    val routineSteps = arrayOf("Wake up", "Brush teeth", "Make coffee")

    for ((index, value) in routineSteps.withIndex()) {
        println("The step at $index is \"$value\"")
    }
    // The step at 0 is "Wake up"
    // The step at 1 is "Brush teeth"
    // The step at 2 is "Make coffee"
```

## While loops

`while` and `do-while` loops run the code in their body 
continuously while the condition is satisfied.
The difference between them is the condition checking time:

* `while` checks the condition and, if it's satisfied, 
  runs the code in its body and then returns to the condition check.
* `do-while` runs the code in its body and then checks the condition. 
  If it's satisfied, the loop repeats. So, the body of `do-while`
  runs at least once, regardless of the condition.
