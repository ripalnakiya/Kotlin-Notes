# Lambdas
<show-structure depth="2"/>

## Lambda expressions

Lambda expressions and anonymous functions are **function literals**.

> Function literals are functions that are not declared but are passed immediately as an expression.
{style="note"}

Consider the following example:

```Kotlin
max(strings, { a, b -> a.length < b.length })
```

The function `max` is a higher-order function, as it takes a function value as its second argument.

This second argument is an expression that is itself a function, called a function literal, 
which is equivalent to the following named function:

```Kotlin
fun compare(a: String, b: String): Boolean = a.length < b.length
```

You can also create a **suspending lambda expression** using the `suspend` keyword.

A suspending lambda has the function type `suspend () -> Unit` and can call other suspending functions:

```kotlin
val suspendingTask = suspend { doSuspendingWork() }
```

### Lambda expression syntax

The full syntactic form of lambda expressions is as follows:

```Kotlin
val sum: (Int, Int) -> Int = { x: Int, y: Int -> x + y }
```

- A lambda expression is always surrounded by curly braces.
- Parameter declarations in the full syntactic form go inside curly braces and have optional type annotations.
- The body goes after the `->`.
- If the inferred return type of the lambda is not `Unit`, the last (or possibly single) expression inside the lambda body is treated as the return value.

If you leave all the optional annotations out, what's left looks like this:

```Kotlin
val sum = { x: Int, y: Int -> x + y }
```

### Passing trailing lambdas

According to Kotlin convention, if the last parameter of a function is a function, then a lambda expression passed as the corresponding argument can be placed outside the parentheses:

```Kotlin
val product = items.fold(1) { acc, e -> acc * e }
```

Such syntax is also known as **trailing lambda**.

If the lambda is the only argument in that call, the parentheses can be omitted entirely:

```Kotlin
fun doSomething(action: () -> Unit) {
    action()
}

fun main() {
    // Without trailing lambda syntax:
    doSomething({
        println("Hello, world!")
    })

    // With trailing lambda syntax:
    doSomething {
        println("Hello, world!")
    }
}
```

### it: implicit name of a single parameter

It's very common for a lambda expression to have only one parameter.

If the compiler can parse the signature without any parameters, 
the parameter does not need to be declared and `->` can be omitted. 

The parameter will be implicitly declared under the name `it`:

```kotlin
ints.filter { it > 0 } 
// this literal is of type '(it: Int) -> Boolean'
```

```Kotlin
fun processList(items: List<Int>, action: (Int) -> Unit) {
    for (item in items) {
        action(item)
    }
}

fun main() {
    val numbers = listOf(1, 2, 3, 4, 5)

    // With explicit parameter name
    processList(numbers) { number -> 
        println("Processing number: $number")
    }
    
    // With implicit parameter name
    processList(numbers) {
        println("Processing number: $it")
    }
}
```

### Returning a value from a lambda expression

You can explicitly return a value from the lambda using the qualified return syntax.

Otherwise, the value of the last expression is implicitly returned.

Therefore, the two following snippets are equivalent:

```Kotlin
ints.filter {
    val shouldFilter = it > 0
    shouldFilter
}

ints.filter {
    val shouldFilter = it > 0
    return@filter shouldFilter
}
```

### Underscore for unused variables

If the lambda parameter is unused, you can place an underscore instead of its name:

```Kotlin
map.forEach { (_, value) -> println("$value!") }
```

```Kotlin
fun performOperation(x: Int, operation: (Int) -> Unit) {
    operation(x)
}

fun main() {
    // Call performOperation with a lambda that ignores the parameter
    performOperation(5) { _ ->
        println("Operation performed, but the parameter is not used.")
    }
}
```

### Destructuring in lambdas

Destructuring in lambdas is described as a part of destructuring declarations.

### Anonymous functions

The lambda expression syntax above is missing one thing – the ability to specify the function's return type.

In most cases, this is unnecessary because the return type can be inferred automatically.

However, if you do need to specify it explicitly, you can use an alternative syntax: an **anonymous function**.

```Kotlin
fun(x: Int, y: Int): Int = x + y
```

An anonymous function looks very much like a regular function declaration, except its name is omitted. Its body can be either an expression (as shown above) or a block:

```Kotlin
fun(x: Int, y: Int): Int {
    return x + y
}
```

The parameters and the return type are specified in the same way as for regular functions, except the parameter types can be omitted if they can be inferred from the context:

```Kotlin
ints.filter(fun(item) = item > 0)
```

- The return type inference for anonymous functions works just like for normal functions:
  - the return type is inferred automatically for anonymous functions with an **expression body**,
  - but it has to be specified explicitly (or is assumed to be `Unit`) for anonymous functions with a **block body**.

> When passing anonymous functions as parameters, place them inside the parentheses. 
> 
> The shorthand syntax that allows you to leave the function 
> outside the parentheses works only for lambda expressions.
{style="warning"}

Another difference between lambda expressions and anonymous functions 
is the behavior of non-local returns.

A `return` statement without a label always returns from the function 
declared with the `fun` keyword.

This means that a `return` inside a lambda expression will return 
from the enclosing function, whereas a `return` inside an anonymous function 
will return from the anonymous function itself.

### Closures

A lambda expression or anonymous function (as well as a local function and an object expression) can access its **closure**, which includes the variables declared in the outer scope.

The variables captured in the closure can be modified in the lambda:

```Kotlin
var sum = 0
ints.filter { it > 0 }.forEach {
    sum += it
}
print(sum)
```

### Function literals with receiver

Function types with receiver, such as `A.(B) -> C`, 
can be instantiated with a special form of function literals – function literals with receiver.

As mentioned, Kotlin provides the ability to call an instance of a function type with receiver 
while providing the **receiver object**.

Inside the body of the function literal, 
the receiver object passed to a call becomes an **implicit** `this`, 
so that you can access the members of that receiver object 
without any additional qualifiers, or access the receiver object using a `this` expression.

This behavior is similar to that of extension functions, which also allow you to access the members of
the receiver object inside the function body.

Here is an example of a function literal with receiver along with its type, 
where `plus` is called on the receiver object:

```kotlin
val sum: Int.(Int) -> Int = { other -> plus(other) }
```

The anonymous function syntax allows you to specify the receiver type of a function literal directly.
This can be useful if you need to declare a variable of a function type with receiver, and then to use it later.

```kotlin
val sum = fun Int.(other: Int): Int = this + other
```

Lambda expressions can be used as function literals with receiver 
when the receiver type can be inferred from the context.
One of the most important examples of their usage is type-safe builders:

```kotlin
class HTML {
    fun body() { ... }
}

fun html(init: HTML.() -> Unit): HTML {
    val html = HTML()  // create the receiver object
    html.init()        // pass the receiver object to the lambda
    return html
}

html {       // lambda with receiver begins here
    body()   // calling a method on the receiver object
}
```
