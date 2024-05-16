# Functions

<show-structure levels="2"/>

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


















