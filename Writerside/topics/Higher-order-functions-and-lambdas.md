# Higher-order functions and lambdas

Kotlin functions are first-class, which means they can be stored in variables and data structures, and can be passed as arguments to and returned from other higher-order functions. 

You can perform any operations on functions that are possible for other non-function values.

To facilitate this, Kotlin, as a statically typed programming language, uses a family of function types to represent functions, and provides a set of specialized language constructs, such as lambda expressions.

<note>

**First-Class Functions**: A Programming language is said to have first-class functions if it treats functions as first-class citizens.

**First-Class Citizen**:c A first-class citizen (sometimes called first-class objects) in a programming language is an entity which supports all the operations which are generally available to other entities. These operations typically include being passed as an argument, returned from a function, and assigned to a variable.
</note>

## Higher order functions

A higher-order function is a function that takes functions as parameters, or returns a function.

A good example of a higher-order function is the functional programming idiom `fold` for collections.

It takes an initial accumulator value and a combining function and builds its return value by consecutively combining the current accumulator value with each collection element, replacing the accumulator value each time:

```Kotlin
fun <T, R> Collection<T>.fold(
    initial: R,
    combine: (acc: R, nextElement: T) -> R
): R {
    var accumulator: R = initial
    for (element: T in this) {
        accumulator = combine(accumulator, element)
    }
    return accumulator
}
```

In the code above, the `combine` parameter has the function type `(R, T) -> R`, so it accepts a function that takes two arguments of types `R` and `T` and returns a value of type `R`. It is invoked inside the `for` loop, and the return value is then assigned to `accumulator`.

To call `fold`, you need to pass an instance of the function type to it as an argument, and lambda expressions (described in more detail below) are widely used for this purpose at higher-order function call sites:

```Kotlin
val items = listOf(2,4,6,8,10)

items.fold(0, {
    // When a lambda has parameters, they go first, followed by '->'
    acc: Int, i: Int ->
    print("acc = $acc, i = $i, ")
    val result = acc + i
    println("result = $result")
    result
    // The last expression in a lambda is considered the return value:
})

// Parameter types in a lambda are optional if they can be inferred:
val joinedToString = items.fold("Elements:", { 
    acc, i -> 
    acc + " " + i 
})

// Function references can also be used for higher-order function calls:
val product = items.fold(1, Int::times)
```

Output:

```Kotlin
acc = 0, i = 2, result = 2
acc = 2, i = 4, result = 6
acc = 6, i = 6, result = 12
acc = 12, i = 8, result = 20
acc = 20, i = 10, result = 30
    
Elements: 2 4 6 8 10
        
3840
```

## Function types




















