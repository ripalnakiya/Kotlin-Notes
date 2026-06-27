# Higher-order functions
<show-structure depth="2"/>

Kotlin functions are first-class, which means they can be stored in variables 
and data structures, and can be passed as arguments to and returned from other
higher-order functions. 

You can perform any operations on functions that are possible for other non-function values.

To facilitate this, Kotlin, as a statically typed programming language, 
uses a family of [function types](#function-types) to represent functions, 
and provides a set of specialized language constructs, such as
[lambda expressions](#lambda-expressions-and-anonymous-functions).

> **First-Class Functions**: A Programming language is said to have first-class functions if it treats functions as first-class citizens.
>
> **First-Class Citizen**: A first-class citizen (sometimes called first-class objects) in a programming language is an entity which supports all the operations which are generally available to other entities. These operations typically include being passed as an argument, returned from a function, and assigned to a variable.
{style="note"}

## Higher order functions

A higher-order function is a function that takes functions as parameters, 
or returns a function.

## Function types

Kotlin uses function types, such as `(Int) -> String`, 
for declarations that deal with functions: `val onClick: () -> Unit = ...`.

These types have a special notation that corresponds 
to the signatures of the functions - their parameters and return values:

* All function types have a parenthesized list of parameter types and a return type: `(A, B) -> C` denotes a type that
  represents functions that take two arguments of types `A` and `B` and return a value of type `C`.
  The list of parameter types may be empty, as in `() -> A`. The `Unit` return type cannot be omitted.

* Function types can optionally have an additional *receiver* type, which is specified before the dot in the notation:
  the type `A.(B) -> C` represents functions that can be called on a receiver object `A` with a parameter `B` and
  return a value `C`.
  Function literals with receiver are often used along with these types.

* Suspending functions belong to a special kind of function type that have
  a *suspend* modifier in their notation, such as `suspend () -> Unit` or `suspend A.(B) -> C`.

The function type notation can optionally include names for the function parameters: `(x: Int, y: Int) -> Point`.
These names can be used for documenting the meaning of the parameters.

To specify that a function type is nullable, use parentheses as follows: `((Int, Int) -> Int)?`.

Function types can also be combined using parentheses: `(Int) -> ((Int) -> Unit)`.

> The arrow notation is right-associative, `(Int) -> (Int) -> Unit` is equivalent 
> to the previous example, but not to `((Int) -> (Int)) -> Unit`.
>
{style="note"}

You can also give a function type an alternative name by using a type alias:

```kotlin
typealias ClickHandler = (Button, ClickEvent) -> Unit
```

### Instantiating a function type

There are several ways to obtain an instance of a function type:

1. Use a code block within a function literal, in one of the following forms:
   - A lambda expression: `{ a, b -> a + b }`
   - An anonymous function: `fun(s: String): Int { return s.toIntOrNull() ?: 0 }`

    **Function literals with receiver** can be used as values of function types with receiver.

2. Use a callable reference to an existing declaration:
    - A top-level, local, member, or extension function: `::isOdd`, `String::toInt`
    - A top-level, member, or extension property: `List<Int>::size`
    - A constructor: `::Regex`

   These include bound callable references that point to a member of a particular instance: `foo::toString`.

3. Use instances of a custom class that implements a function type as an interface:
    ```Kotlin
    // Function types are actually interfaces under the hood here
   
    class IntTransformer : (Int) -> Int {
        override operator fun invoke(x: Int): Int {
            return x * 2
        }
    }
    
    val transformer: (Int) -> Int = IntTransformer()
    println(transformer(5)) // 10
    ```

The compiler can infer the function types for variables if there is enough information:

```Kotlin
val a = { i: Int -> i + 1 } // The inferred type is (Int) -> Int
```

**Non-literal** values of function types with and without a receiver are interchangeable,  so the receiver can stand in for the first parameter, and vice versa

For instance, a value of type `(A, B) -> C` can be passed or assigned where a value of type `A.(B) -> C` is expected, and the other way around:

```Kotlin
    val repeatFun: String.(Int) -> String = { times -> this.repeat(times) }
    val twoParameters: (String, Int) -> String = repeatFun // OK

    fun runTransformation(f: (String, Int) -> String): String {
        return f("hello", 3)
    }
    val result = runTransformation(repeatFun) // OK

    println("result = $result")
    // result = hellohellohello
```

<note>
Kotlin treats an extension function both as a regular function type and as an extension function, 
depending on how you specify the type.

If you assign an extension function (`String.(Int) -> String`) to a variable, 
Kotlin will still assume it's a regular function (`(String, Int) -> String`) unless you specify otherwise.

To change the default behavior, you need to declare the type of the variable 
as an extension function type explicitly `String.(Int) -> String`.
</note>

### Invoking a function type instance

A value of a function type can be invoked by using its `invoke(...)` operator: `f.invoke(x)` or just `f(x)`.

If the value has a receiver type, the receiver object should be passed as the first argument.

Another way to invoke a value of a function type with receiver is 
to prepend it with the receiver object, 
as if the value were an extension function: `1.foo(2)`.

```Kotlin
val repeatTimes: String.(Int) -> String = { times -> this.repeat(times) }

println(repeatTimes("Hello", 3))    // HelloHelloHello

println("Hello".repeatTimes(3))     // HelloHelloHello
```

```Kotlin
    val stringPlus: (String, String) -> String = String::plus
    val intPlus: Int.(Int) -> Int = Int::plus

    println(stringPlus.invoke("<-", "->"))
    // <-->
    println(stringPlus("Hello, ", "world!"))
    // Hello, world!

    println(intPlus.invoke(1, 1))
    // 2
    println(intPlus(1, 2))
    // 3
    println(2.intPlus(3)) // extension-like call
    // 5
```

### Inline functions

Sometimes it is beneficial to use inline functions, which provide flexible control flow, for higher-order functions.
