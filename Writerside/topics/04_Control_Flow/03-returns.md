# Returns and jumps
<show-structure depth="2"/>

Kotlin has three structural jump expressions:

* `return` by default returns from the nearest enclosing function or [anonymous function](03-lambda-expressions-and-anonymous-functions.md#anonymous-functions).
* `break` terminates the nearest enclosing loop.
* `continue` proceeds to the next step of the nearest enclosing loop.

All of these expressions can be used as part of larger expressions:

```kotlin
val s = person.name ?: return
```

The type of these expressions is the [Nothing type](05-exceptions.md#the-nothing-type).

## Break and continue labels

Any expression in Kotlin may be marked with a _label_.
Labels have the form of an identifier followed by the `@` sign, such as `abc@` or `fooBar@`.
To label an expression, just add a label in front of it.

```kotlin
loop@ for (i in 1..100) {
    // ...
}
```

Now, you can qualify a `break` or a `continue` with a label:

```kotlin
loop@ for (i in 1..100) {
    for (j in 1..100) {
        if (...) break@loop
    }
}
```

A `break` qualified with a label jumps to the execution point right after the loop marked with that label.
A `continue` proceeds to the next iteration of that loop.

> In some cases, you can apply `break` and `continue` *non-locally* without explicitly defining labels.
> Such non-local usages are valid in lambda expressions used in enclosing [inline functions](05-inline-functions.md#break-and-continue).
>
{style="note"}

## Return to labels

In Kotlin, functions can be nested using function literals, local functions, and object expressions.
A qualified `return` allows you to return from an outer function.

The most important use case is returning from a lambda expression. 


Recall that when we write the following,
the `return` expression returns from the nearest enclosing function `foo`:

```Kotlin
fun foo() {
    listOf(1, 2, 3, 4, 5).forEach {
        if (it == 3) return // non-local return directly to the caller of foo()
        print(it)
    }
    println("this point is unreachable")
}

// 12
```

> Note that such non-local returns are supported only for lambda expressions
> passed to inline functions.
>
> `forEach` function is an example of an inline function that
> can accept lambda expressions and allow non-local returns
{style="note"}

To return from a lambda expression, label it and qualify the `return`:

```kotlin
fun foo() {
    listOf(1, 2, 3, 4, 5).forEach lit@{
        if (it == 3) return@lit // local return to the caller of the lambda - the forEach loop
        print(it)
    }
    print(" done with explicit label")
}

// 1245 done with explicit label
```

Now, it returns only from the lambda expression. Often it is more convenient to use _implicit labels_, because such a label
has the same name as the function to which the lambda is passed.

```kotlin
fun foo() {
    listOf(1, 2, 3, 4, 5).forEach {
        if (it == 3) return@forEach // local return to the caller of the lambda - the forEach loop
        print(it)
    }
    print(" done with implicit label")
}

// 1245 done with implicit label
```

Alternatively, you can replace the lambda expression with an [anonymous function](03-lambda-expressions-and-anonymous-functions.md#anonymous-functions).
A `return` statement in an anonymous function will return from the anonymous function itself.

```kotlin
fun foo() {
    listOf(1, 2, 3, 4, 5).forEach(fun(value: Int) {
        if (value == 3) return // local return to the caller of the anonymous function - the forEach loop
        print(value)
    })
    print(" done with anonymous function")
}

// 1245 done with anonymous function
```

> Note that the use of local returns in the previous three examples
> is similar to the use of `continue` in regular loops.
{style="note"}

There is no direct equivalent for `break`,
but it can be simulated by adding another nesting lambda and non-locally returning from it:

```kotlin
fun foo() {
    run loop@{
        listOf(1, 2, 3, 4, 5).forEach {
            if (it == 3) return@loop // non-local return from the lambda passed to run
            print(it)
        }
    }
    print(" done with nested loop")
}

// 12 done with nested loop
```

> `run` is a function in Kotlin that takes a lambda expression as an argument and executes the code inside the lambda.
>
{style="note"}

Therefore, code can be rewritten as (using the implicit label):

```kotlin
fun foo() {
    run{
        listOf(1, 2, 3, 4, 5).forEach {
            if (it == 3) return@run // non-local return from the lambda passed to run
            print(it)
        }
    }
    print(" done with nested loop")
}

// 12 done with nested loop
```

The non-local return here is possible since the nested `forEach()` lambda acts as an [inline function](05-inline-functions.md).

When returning a value, the parser gives preference to the qualified return:

```kotlin
return@a 1
```

This means "return `1` at label `@a`" rather than "return a labeled expression `(@a 1)`".

> In some cases, you can return from a lambda expression without using labels. Such *non-local* returns are located in a
> lambda but exit the enclosing [inline function](05-inline-functions.md#returns).
>
{style="note"}
