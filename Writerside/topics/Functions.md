# Functions

<show-structure depth="2"/>

```Kotlin
fun double(x: Int): Int {
    return 2 * x
}
```

## Function usage

Functions are called using the standard approach:

```Kotlin
val result = double(2)
```

Calling member functions uses dot notation:

```Kotlin
Stream().read() // create instance of class Stream and call read()
```

### Parameters

Function parameters are defined using Pascal notation - **name: type**. 

Parameters are separated using commas, and each parameter must be explicitly typed:

```Kotlin
fun powerOf(number: Int, exponent: Int): Int { /*...*/ }
```

```Kotlin
fun powerOf(
    number: Int,
    exponent: Int, // trailing comma
) { /*...*/ }
```

### Default arguments

Function parameters can have default values, which are used when you skip the corresponding argument. 

This reduces the number of overloads:

```Kotlin
fun read(
    b: ByteArray,
    off: Int = 0,
    len: Int = b.size,
) { /*...*/ }
```

Overriding methods always use the base method's default parameter values. 

When overriding a method that has default parameter values, the default parameter values must be omitted from the signature:

```Kotlin
open class A {
    open fun foo(i: Int = 10) { /*...*/ }
    open fun bar(i: Int) { /*...*/ }
}

class B : A() {
    override fun foo(i: Int) { /*...*/ }  
    // No default value is allowed.
    
    open fun bar(i: Int = 10) { /*...*/ }
    // Error: No default value is allowed.
}
```

<note>
An overriding function is not allowed to specify default values for its parameters
</note>

If a default parameter precedes a parameter with no default value, the default value can only be used by calling the function with named arguments:

```Kotlin
fun foo(
    bar: Int = 0,
    baz: Int,
) { /*...*/ }

foo(baz = 1) // The default value bar = 0 is used
```

If the last argument after default parameters is a lambda, you can pass it either as a named argument or outside the parentheses:

```Kotlin
fun foo(
    bar: Int = 0,
    baz: Int = 1,
    qux: () -> Unit,
) { /*...*/ }


fun main() {
    foo(5,6, { println("Hello 1") })
    
    foo(1) { println("Hello 2") }     
    // Uses the default value baz = 1
    
    foo(qux = { println("Hello 3") }) 
    // Uses both default values bar = 0 and baz = 1
    
    foo { println("Hello 4") }        
    // Uses both default values bar = 0 and baz = 1
}
```

### Named arguments

You can name one or more of a function's arguments when calling it. 

This can be helpful when a function has many arguments and it's difficult to associate a value with an argument, especially if it's a boolean or null value.

When you use named arguments in a function call, you can freely change the order that they are listed in. If you want to use their default values, you can just leave these arguments out altogether.

Consider the `reformat()` function, which has 4 arguments with default values.

```Kotlin
fun reformat(
    str: String,
    normalizeCase: Boolean = true,
    upperCaseFirstLetter: Boolean = true,
    divideByCamelHumps: Boolean = false,
    wordSeparator: Char = ' ',
) { /*...*/ }
```

When calling this function, you don't have to name all its arguments:

```Kotlin
reformat(
    "String!",
    false,
    upperCaseFirstLetter = false,
    divideByCamelHumps = true,
    '_'
)
```

You can skip all the ones with default values:

```Kotlin
reformat("This is a long String!")
```

You are also able to skip specific arguments with default values, rather than omitting them all. However, after the first skipped argument, you must name all subsequent arguments:

```Kotlin
reformat("This is a short String!", upperCaseFirstLetter = false, wordSeparator = '_')
```

You can pass a variable number of arguments (`vararg`) with names using the `spread` operator:

```Kotlin
fun foo(vararg strings: String) { /*...*/ }

foo(strings = *arrayOf("a", "b", "c"))
```

### Unit-returning functions

If a function does not return a useful value, its return type is `Unit`. `Unit` is a type with only one value - `Unit`. 

This value does not have to be returned explicitly:

```Kotlin
fun printHello(name: String?): Unit {
    if (name != null)
        println("Hello $name")
    else
        println("Hi there!")
    // `return Unit` or `return` is optional
}
```

The Unit return type declaration is also optional. The above code is equivalent to:

```Kotlin
fun printHello(name: String?) { ... }
```

### Single-expression functions

When the function body consists of a single expression, the curly braces can be omitted and the body specified after an `=` symbol:

```Kotlin
fun double(x: Int): Int = x * 2
```

Explicitly declaring the return type is optional when this can be inferred by the compiler:

```Kotlin
fun double(x: Int) = x * 2
```

### Explicit return types

Functions with block body must always specify return types explicitly, unless it's intended for them to return Unit, in which case specifying the return type is optional.

Kotlin does not infer return types for functions with block bodies because such functions may have complex control flow in the body, and the return type will be non-obvious to the reader (and sometimes even for the compiler).

### Variable number of arguments (varargs)

You can mark a parameter of a function (usually the last one) with the `vararg` modifier:

```Kotlin
fun <T> asList(vararg ts: T): List<T> {
    val result = ArrayList<T>()
    for (t in ts) // ts is an Array
        result.add(t)
    return result
}
```

In this case, you can pass a variable number of arguments to the function:

```Kotlin
val list = asList(1, 2, 3)
```

Inside a function, a `vararg`-parameter of type `T` is visible as an array of `T`, as in the example above, where the `ts` variable has type `Array<out T>`.

Only one parameter can be marked as `vararg`. 

If a `vararg` parameter is not the last one in the list, values for the subsequent parameters can be passed using named argument syntax, or, if the parameter has a function type, by passing a lambda outside the parentheses.

When you call a `vararg`-function, you can pass arguments individually, for example `asList(1, 2, 3)`. If you already have an array and want to pass its contents to the function, use the **spread** operator (prefix the array with `*`):

```Kotlin
val a = arrayOf(1, 2, 3)
val list = asList(-1, 0, *a, 4)
```

If you want to pass a primitive type array into `vararg`, you need to convert it to a regular (typed) array using the `toTypedArray()` function:

```Kotlin
val a = intArrayOf(1, 2, 3) // IntArray is a primitive type array
val list = asList(-1, 0, *a.toTypedArray(), 4)
```

### Infix notation

Functions marked with the `infix` keyword can also be called using the infix notation (omitting the dot and the parentheses for the call). 

Infix functions must meet the following requirements:
- They must be member functions or [extension functions](Extensions.md#extension-functions).
- They must have a single parameter.
- The parameter must not accept variable number of arguments and must have no default value.

```Kotlin
infix fun Int.shl(x: Int): Int { ... }

// calling the function using the infix notation
1 shl 2

// is the same as
1.shl(2)
```

Infix function calls have lower precedence than arithmetic operators, type casts, and the `rangeTo` operator. 

The following expressions are equivalent:
- `1 shl 2 + 3` is equivalent to `1 shl (2 + 3)`
- `0 until n * 2` is equivalent to `0 until (n * 2)`
- `xs union ys as Set<*>` is equivalent to `xs union (ys as Set<*>)`

On the other hand, an infix function call's precedence is higher than that of the boolean operators `&&` and `||,` `is`- and `in`-checks, and some other operators. 

These expressions are equivalent as well:
- `a && b xor c` is equivalent to `a && (b xor c)`
- `a xor b in c` is equivalent to `(a xor b) in c`

Note that infix functions always require both the receiver and the parameter to be specified. When you're calling a method on the current receiver using the infix notation, use `this` explicitly. 

This is required to ensure unambiguous parsing.

```Kotlin
class MyStringCollection {
    infix fun add(s: String) { /*...*/ }

    fun build() {
        this add "abc"   // Correct
        add("abc")       // Correct
        add "abc"        // Incorrect: the receiver must be specified
    }
}
```

## Function scope

Kotlin functions can be declared at the top level in a file, meaning you do not need to create a class to hold a function.

n addition to top level functions, Kotlin functions can also be declared locally as member functions and extension functions.

### Local functions

Kotlin supports local functions, which are functions inside other functions:

```Kotlin
fun dfs(graph: Graph) {
    fun dfs(current: Vertex, visited: MutableSet<Vertex>) {
        if (!visited.add(current)) return
        for (v in current.neighbors)
            dfs(v, visited)
    }

    dfs(graph.vertices[0], HashSet())
}
```

A local function can access local variables of outer functions (the closure). In the case above, `visited` can be a local variable:

```Kotlin
fun dfs(graph: Graph) {
    val visited = HashSet<Vertex>()
    fun dfs(current: Vertex) {
        if (!visited.add(current)) return
        for (v in current.neighbors)
            dfs(v)
    }

    dfs(graph.vertices[0])
}
```

<note>
Other programming languages like C++ and Java doesn't support function declaration inside another functions, though they do support a function call inside another function
</note>

### Member functions

A member function is a function that is defined inside a class or object:

```Kotlin
class Sample {
    fun foo() { print("Foo") }
}
```

## Generic functions

Functions can have generic parameters, which are specified using angle brackets before the function name:

```Kotlin
fun <T> singletonList(item: T): List<T> { /*...*/ }
```

## Tail recursive functions

Kotlin supports a style of functional programming known as tail recursion. 

For some algorithms that would normally use loops, you can use a recursive function instead without the risk of stack overflow. 

When a function is marked with the `tailrec` modifier and meets the required formal conditions, the compiler optimizes out the recursion, leaving behind a fast and efficient loop based version instead:

```Kotlin
val eps = 1E-10 // "good enough", could be 10^-15

tailrec fun findFixPoint(x: Double = 1.0): Double =
    if (Math.abs(x - Math.cos(x)) < eps) 
        x 
    else 
        findFixPoint(Math.cos(x))
```

The resulting code is equivalent to this more traditional style:

```Kotlin
val eps = 1E-10 // "good enough", could be 10^-15

private fun findFixPoint(): Double {
    var x = 1.0
    while (true) {
        val y = Math.cos(x)
        if (Math.abs(x - y) < eps) return x
        x = Math.cos(x)
    }
}
```

To be eligible for the `tailrec` modifier, a function must call itself as the last operation it performs. 

You cannot use tail recursion when there is more code after the recursive call, within `try`/`catch`/`finally` blocks, or on open functions. 

Currently, tail recursion is supported by Kotlin for the JVM and Kotlin/Native.