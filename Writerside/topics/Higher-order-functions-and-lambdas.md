# Higher-order functions and lambdas

<show-structure depth="2"/>

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

Kotlin uses **function types**, such as `(Int) -> String`, for declarations that deal with functions: `val onClick: () -> Unit = ...`.

These **types** have a special notation that corresponds to the signatures of the functions 
1. their parameters 
2. return values

- All function types have a parenthesized list of parameter types and a return type:
  - `(A, B) -> C` denotes a **type** that represents functions that take two arguments of types `A` and `B` and return a value of type `C`.
  - The list of parameter types may be empty, as in `() -> A`. The `Unit` return type cannot be omitted.

- Function types can optionally have an additional receiver type, which is specified before the dot in the notation: 
  - the type `A.(B) -> C` represents functions that can be called on a receiver object `A` with a parameter `B` and return a value `C`. 
  - [Function literals with receiver](#function-literals-with-receiver) are often used along with these types.

- **Suspending functions** belong to a special kind of function type that have a suspend modifier in their notation, 
  - such as `suspend () -> Unit` or `suspend A.(B) -> C`.

The function type notation can optionally include names for the function parameters: `(x: Int, y: Int) -> Point`.

These names can be used for documenting the meaning of the parameters.

<note>

To specify that a function type is nullable, use parentheses as follows: `((Int, Int) -> Int)?`
</note>

Function types can also be combined using parentheses: `(Int) -> ((Int) -> Unit)`.

<tip>

The arrow notation is right-associative, `(Int) -> (Int) -> Unit` is equivalent to the previous example, but not to `((Int) -> (Int)) -> Unit`.
</tip>

You can also give a function type an alternative name by using a type alias:

```Kotlin
typealias ClickHandler = (Button, ClickEvent) -> Unit
```

### Instantiating a function type

There are several ways to obtain an instance of a function type:

1. Use a code block within a function literal, in one of the following forms:
   - A lambda expression: `{ a, b -> a + b }`
   - An anonymous function: `fun(s: String): Int { return s.toIntOrNull() ?: 0 }`

    [Function literals with receiver](#function-literals-with-receiver) can be used as values of function types with receiver.

2. Use a callable reference to an existing declaration:
    - A top-level, local, member, or extension function: `::isOdd`, `String::toInt`
    - A top-level, member, or extension property: `List<Int>::size`
    - A constructor: `::Regex`

   These include bound callable references that point to a member of a particular instance: `foo::toString`.

3. Use instances of a custom class that implements a function type as an interface:
   ```Kotlin
   class IntTransformer: (Int) -> Int {
    override operator fun invoke(x: Int): Int = TODO()
   }
   
    val intFunction: (Int) -> Int = IntTransformer()
    ```

The compiler can infer the function types for variables if there is enough information:

```Kotlin
val a = { i: Int -> i + 1 } // The inferred type is (Int) -> Int
```

**Non-literal** values of function types with and without a receiver are interchangeable,  so the receiver can stand in for the first parameter, and vice versa

For instance, a value of type `(A, B) -> C` can be passed or assigned where a value of type `A.(B) -> C` is expected, and the other way around:

```Kotlin
fun main() {
    val repeatFun: String.(Int) -> String = { times -> this.repeat(times) }
    val twoParameters: (String, Int) -> String = repeatFun // OK

    fun runTransformation(f: (String, Int) -> String): String {
        return f("hello", 3)
    }
    val result = runTransformation(repeatFun) // OK

    println("result = $result")
    // result = hellohellohello
}
```

<note>
Kotlin treats an extension function both as a regular function type and as an extension function, depending on how you specify the type.

If you assign an extension function (`String.(Int) -> String`) to a variable, Kotlin will still assume it's a regular function (`(String, Int) -> String`) unless you specify otherwise.

To change the default behavior, you need to declare the type of the variable as an extension function type explicitly `String.(Int) -> String`.
</note>

### Invoking a function type instance

A value of a function type can be invoked by using its `invoke(...)` operator: 
- `f.invoke(x) `
- or just `f(x)`

If the value has a receiver type, the receiver object should be passed as the first argument.

Another way to invoke a value of a function type with receiver is to prepend it with the receiver object, as if the value were an extension function: `1.foo(2)`.

```Kotlin
val repeatTimes: String.(Int) -> String = { times -> this.repeat(times) }

println(repeatTimes("Hello", 3))    // HelloHelloHello

println("Hello".repeatTimes(3))     // HelloHelloHello
```

```Kotlin
fun main() {
    val stringPlus: (String, String) -> String = String::plus
    val intPlus: Int.(Int) -> Int = Int::plus

    println(stringPlus.invoke("<-", "->"))
    println(stringPlus("Hello, ", "world!"))

    println(intPlus.invoke(1, 1))
    println(intPlus(1, 2))
    println(2.intPlus(3)) // extension-like call
}

// Output:
<-->
Hello, world!
2
3
5
```

## Lambda expressions and anonymous functions

Lambda expressions and anonymous functions are **function literals**.

<note>
Function literals are functions that are not declared but are passed immediately as an expression.
</note>

```Kotlin
max(strings, { a, b -> a.length < b.length })
```

The function `max` is a higher-order function, as it takes a function value as its second argument. 

This second argument is an expression that is itself a function, called a function literal, which is equivalent to the following named function:

```Kotlin
fun compare(a: String, b: String): Boolean = a.length < b.length
```

### Lambda expression syntax

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

If the compiler can parse the signature without any parameters, the parameter does not need to be declared and `->` can be omitted. 

The parameter will be implicitly declared under the name `it`:

```Kotlin
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

The return type inference for anonymous functions works just like for normal functions: 
- the return type is inferred automatically for anonymous functions with an **expression body**, 
- but it has to be specified explicitly (or is assumed to be `Unit`) for anonymous functions with a **block body**.

<tip>
When passing anonymous functions as parameters, place them inside the parentheses. The shorthand syntax that allows you to leave the function outside the parentheses works only for lambda expressions.
</tip>

Another difference between lambda expressions and anonymous functions is the behavior of non-local returns. 

A `return` statement without a label always returns from the function declared with the `fun` keyword.

This means that a `return` inside a lambda expression will return from the enclosing function, whereas a 1 inside an anonymous function will return from the anonymous function itself.

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

Function types with receiver, such as `A.(B) -> C`, can be instantiated with a special form of function literals – function literals with receiver.

Inside the body of the function literal, the receiver object passed to a call becomes an **implicit** `this`, so that you can access the members of that receiver object without any additional qualifiers, or access the receiver object using a `this` expression.

This allows you to access the members of the receiver object without needing to qualify them with `this.` explicitly.

This behavior is similar to that of extension functions, which also allow you to access the members of the receiver object inside the function body.

```Kotlin
class Rectangle(val width: Int, val height: Int)

// Define an extension function that takes a lambda with receiver
fun Rectangle.describe(description: Rectangle.() -> String): String {
    return description()
}

fun main() {
    val rectangle = Rectangle(5, 10)

    val description = rectangle.describe {
        "Rectangle of width ${this.width}, height $height"
        // `this` refers to the Rectangle instance
    }

    println(description)
    // Rectangle of width 5, height 10
}
```

Here is an example of a function literal with receiver along with its type, where plus is called on the receiver object:

```Kotlin
val sum: Int.(Int) -> Int = { other -> plus(other) }
```

The anonymous function syntax allows you to specify the **receiver type** of a function literal directly. 

This can be useful if you need to declare a variable of a function type with receiver, and then to use it later.

```Kotlin
val sum = fun Int.(other: Int): Int = this + other
```

```Kotlin
// Declare a variable of a function type with receiver using 
// anonymous function syntax
val describePerson: Person.() -> String = fun Person.(): String {
    return "Person(name: $name, age: $age)"
}
```