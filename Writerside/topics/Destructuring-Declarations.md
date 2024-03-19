# Destructuring Declarations

Sometimes it is convenient to **destructure** an object into a number of variables, for example:

```Kotlin
val (name, age) = person
```

This syntax is called a **destructuring declaration**. 

A destructuring declaration creates multiple variables at once. You have declared two new variables: name and age, and can use them independently:

```Kotlin
println(name)
println(age)
```

A destructuring declaration is compiled down to the following code:

```Kotlin
val name = person.component1()
val age = person.component2()
```

The `component1()` and `component2()` functions are another example of the **principle of conventions** widely used in Kotlin (see operators like `+` and *`,` `for`-loops as an example). 

Anything can be on the right-hand side of a destructuring declaration, as long as the required number of component functions can be called on it. And, of course, there can be `component3()` and `component4()` and so on.

<note>

The `componentN()` functions need to be marked with the operator keyword to allow using them in a destructuring declaration.
</note>

Destructuring declarations also work in `for`-loops:

```Kotlin
for ((a, b) in collection) { ... }
```

Variables `a` and `b` get the values returned by `component1()` and `component2()` called on elements of the collection.

## Example: returning two values from a function

Assume that you need to return two things from a function - for example, a result object and a status of some sort. 

A compact way of doing this in Kotlin is to declare a data class and return its instance:

```Kotlin
data class Result(val result: Int, val status: Status)
fun function(...): Result {
    // computations

    return Result(result, status)
}

// Now, to use this function:
val (result, status) = function(...)
```

Since data classes automatically declare `componentN()` functions, destructuring declarations work here.

<note>

You could also use the standard class `Pair` and have `function()` return `Pair<Int, Status>`, but it's often better to have your data named properly.
</note>

## Example: destructuring declarations and maps

Probably the nicest way to traverse a map is this:

```Kotlin
for ((key, value) in map) {
   // do something with the key and the value
}
```

To make this work, you should
- Present the map as a sequence of values by providing an `iterator()` function.
- Present each of the elements as a pair by providing functions `component1()` and `component2()`.

And indeed, the standard library provides such extensions:

```Kotlin
operator fun <K, V> Map<K, V>.iterator(): Iterator<Map.Entry<K, V>> = entrySet().iterator()
operator fun <K, V> Map.Entry<K, V>.component1() = getKey()
operator fun <K, V> Map.Entry<K, V>.component2() = getValue()
```

So you can freely use destructuring declarations in `for`-loops with maps (as well as collections of data class instances or similar).

## Underscore for unused variables

If you don't need a variable in the destructuring declaration, you can place an underscore instead of its name:

```Kotlin
val (_, status) = getResult()
```

The `componentN()` operator functions are not called for the components that are skipped in this way.

## Destructuring in lambdas

You can use the destructuring declarations syntax for lambda parameters. 

If a lambda has a parameter of the `Pair` type (or `Map.Entry`, or any other type that has the appropriate `componentN` functions), you can introduce several new parameters instead of one by putting them in parentheses:

```Kotlin
map.mapValues { entry -> "${entry.value}!" }
map.mapValues { (key, value) -> "$value!" }
```

Note the difference between declaring two parameters and declaring a destructuring pair instead of a parameter:

```Kotlin
{ a -> ... } // one parameter
{ a, b -> ... } // two parameters
{ (a, b) -> ... } // a destructured pair
{ (a, b), c -> ... } // a destructured pair and another parameter
```

```Kotlin
val printSquare = { x: Int ->
    println("Square of $x is ${x * x}")
}

val sum = { a: Int, b: Int ->
    println("Sum of $a and $b is ${a + b}")
}

val printPair = { (a, b): Pair<Int, String> ->
    println("Pair is: $a and $b")
}

val printTriple = { (a, b): Pair<Int, String>, c: Double ->
    println("Pair is: $a and $b, and the additional parameter is $c")
}

fun main() {
    printSquare(4)
    sum(3, 5)
    
    val pair1 = Pair(1, "one")
    printPair(pair1)

    val pair2 = Pair(2, "two")
    printTriple(pair2, 3.14)
}
```

If a component of the destructured parameter is unused, you can replace it with the underscore to avoid inventing its name:

```Kotlin
map.mapValues { (_, value) -> "$value!" }
```

You can specify the type for the whole destructured parameter or for a specific component separately:

```Kotlin
map.mapValues { (_, value): Map.Entry<Int, String> -> "$value!" }

map.mapValues { (_, value: String) -> "$value!" }
```
