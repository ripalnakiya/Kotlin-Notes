# Returns and jumps

Kotlin has three structural jump expressions:
- `return` by default returns from the nearest enclosing function or anonymous function.
- `break` terminates the nearest enclosing loop.
- `continue` proceeds to the next step of the nearest enclosing loop.

## break and continue labels

Any expression in Kotlin may be marked with a label.

Labels have the form of an identifier followed by the `@` sign, such as `abc@` or `fooBar@`.

To label an expression, just add a label in front of it.

```Kotlin
loop@ for (i in 1..100) {
    // ...
}
```

Now, we can qualify a break or a continue with a label:

```Kotlin
loop@ for (i in 1..100) {
    for (j in 1..100) {
        if (...) break@loop
    }
}
```

A break qualified with a label jumps to the execution **point right after the loop** marked with that label.

A continue proceeds to the next iteration of that loop.

## return to labels

In Kotlin, functions can be nested using function literals(lambda expressions), local functions, and object expressions(inline objects).

Qualified `return`s allow us to return from an outer function.

The most important use case is returning from a lambda expression.

Recall that when we write the following, the `return` expression returns from the nearest enclosing function `foo`:

```Kotlin
fun foo() {
    listOf(1, 2, 3, 4, 5).forEach {
        if (it == 3) return // non-local return directly to the caller of foo()
        print(it)
    }
    println("this point is unreachable")
}

// Output: 12
```

<note>
Note that such non-local returns are supported only for lambda expressions passed to inline functions. 

`forEach` function is an example of an inline function that can accept lambda expressions and allow non-local returns
</note>

To return from a lambda expression, label it and qualify the `return`:

```Kotlin
fun foo() {
    listOf(1, 2, 3, 4, 5).forEach lit@{
        if (it == 3) return@lit // local return to the caller of the lambda - the forEach loop
        print(it)
    }
    print(" done with explicit label")
}

// Output: 1245 done with explicit label
```

Now, it returns only from the lambda expression.

Often it is more convenient to use implicit labels, because such a label has the same name as the function to which the lambda is passed.

```Kotlin
fun foo() {
    listOf(1, 2, 3, 4, 5).forEach {
        if (it == 3) return@forEach // local return to the caller of the lambda - the forEach loop
        print(it)
    }
    print(" done with implicit label")
}
// Output: 1245 done with implicit label
```

Alternatively, you can replace the lambda expression with an anonymous function.

A return statement in an anonymous function will return from the anonymous function itself.

```Kotlin
fun foo() {
    listOf(1, 2, 3, 4, 5).forEach(fun(value: Int) {
        if (value == 3) return  // local return to the caller of the anonymous function - the forEach loop
        print(value)
    })
    print(" done with anonymous function")
}
// Output: 1245 done with anonymous function
```

<note>
Note that the use of local returns in the previous three examples is similar to the use of continue in regular loops.
</note>

There is no direct equivalent for break, but it can be simulated by adding another nesting lambda and non-locally returning from it:

```Kotlin
fun foo() {
    run loop@{
        listOf(1, 2, 3, 4, 5).forEach {
            if (it == 3) return@loop // non-local return from the lambda passed to run
            print(it)
        }
    }
    print(" done with nested loop")
}
// Output: 12 done with nested loop
```

> `run` is a function in Kotlin that takes a lambda expression as an argument and executes the code inside the lambda.
>
{style="note"}

Therefore, code can be rewritten as (using the implicit label):

```Kotlin
fun foo() {
    run{
        listOf(1, 2, 3, 4, 5).forEach {
            if (it == 3) return@run // non-local return from the lambda passed to run
            print(it)
        }
    }
    print(" done with nested loop")
}
// Output: 12 done with nested loop
```