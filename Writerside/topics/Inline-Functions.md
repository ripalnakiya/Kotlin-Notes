# Inline Functions

Using higher-order functions imposes certain runtime penalties: each function is an object, and it captures a closure.

Memory allocations (both for function objects and classes) and virtual calls introduce runtime overhead.

```Kotlin
// Define a higher-order function that takes a function as a parameter
fun doOperation(x: Int, operation: (Int) -> Int): Int {
    return operation(x)
}

fun main() {
    val multiplier = 10

    // Define a lambda that captures a variable from its enclosing scope
    val multiplyByMultiplier: (Int) -> Int = { it * multiplier }

    // Call the higher-order function with the lambda
    val result = doOperation(5, multiplyByMultiplier)

    println("Result: $result")  // Output: Result: 50
}
```

The lambda `multiplyByMultiplier` is an object. When you pass it to `doOperation`, an object is created.

Invoking the function object inside `doOperation` involves a virtual call, which is generally slower than direct method calls.

But it appears that in many cases this kind of overhead can be eliminated by inlining the lambda expressions.

<note>
Inlining is a compiler optimization that replaces a function call with the actual code of the function. This can eliminate the overhead of the function call and potentially allow for further optimizations.
</note>

Suppose you have a `lock` function that takes a lock and a lambda, like this:

```Kotlin
fun <T> lock(lock: Lock, body: () -> T): T {
    lock.lock()
    try {
        return body()
    } finally {
        lock.unlock()
    }
}
```

You might call it like this:

```Kotlin
lock(l) { foo() }
```

Normally, this would create a function object for `{ foo() }` and make a call to `lock`

By adding the inline modifier, you tell the compiler to replace the lock function call with its actual code at each call site:

```Kotlin
inline fun <T> lock(lock: Lock, body: () -> T): T {
    lock.lock()
    try {
        return body()
    } finally {
        lock.unlock()
    }
}
```

Instead of generating a function call, the compiler will emit the code directly:

```Kotlin
l.lock()
try {
    foo()
} finally {
    l.unlock()
}
```

This way, no function object is created, and the function call overhead is eliminated.

Inlining may cause the generated code to grow. However, if you do it in a reasonable way (avoiding inlining large functions), it will pay off in performance, especially at "megamorphic" call-sites inside loops.

## noinline

If you don't want all the lambdas passed to an inline function to be inlined, mark some of your function parameters with the `noinline` modifier:

```Kotlin
inline fun foo(inlined: () -> Unit, noinline notInlined: () -> Unit) {
    // Call the inlined lambda directly
    inlined()
    
    // Store the notInlined lambda for later use
    val storedLambda = notInlined
    storedLambda()
}

fun main() {
    foo(
        inlined = { println("This is inlined") },
        notInlined = { println("This is not inlined") }
    )
}
```

The lambda passed as `inlined` is inlined, meaning its code is directly inserted into the `foo` function at the call site. No function object is created for it.

The lambda passed as `noinline` is not inlined. Instead, it is treated like a regular lambda: a function object is created, and it can be stored or passed around.

<note>

`inline` lambdas can only be called inside inline functions or passed as inlinable arguments.

However, if you need to store a lambda for later execution or pass it to another function, you would mark it as `noinline`.
</note>

## Non-local returns

In Kotlin, you can only use a normal, unqualified `return` to exit a named function or an anonymous function.

To exit a lambda, use a label. A bare `return` is forbidden inside a lambda because a lambda cannot make the enclosing function `return`:

```Kotlin
fun foo() {
    ordinaryFunction {
        return // ERROR: cannot make `foo` return here
    }
}
```

But if the function the lambda is passed to is inlined, the return can be inlined, as well. So it is allowed:

```Kotlin
fun foo() {
    inlined {
        return // OK: the lambda is inlined
    }
}
```

Such returns (located in a lambda, but exiting the enclosing function) are called **non-local returns**.

<note>
Non-Local Control Flow: Allows a lambda to return from the enclosing function.
</note>

This sort of construct usually occurs in loops, which inline functions often enclose:

```Kotlin
fun hasZeros(ints: List<Int>): Boolean {
    ints.forEach {
        if (it == 0) return true // returns from hasZeros
    }
    return false
}
```

Note that some inline functions may call the lambdas passed to them as parameters not directly from the function body, but from another execution context, such as a local object or a nested function. 

In such cases, non-local control flow is also not allowed in the lambdas. 

To indicate that the lambda parameter of the inline function cannot use non-local returns, mark the lambda parameter with the `crossinline` modifier:

```Kotlin
inline fun executeTask(crossinline task: () -> Unit) {
    val runnable = object : Runnable {
        override fun run() {
            task()  // Calling the lambda from a different context
        }
    }
    Thread(runnable).start()
}

fun main() {
    executeTask {
        println("Task is running")
        // Uncommenting the line below would cause a compilation error
        // return  // Non-local return is not allowed
    }
    println("Main function continues...")
}
```

## Reified type parameters

Generics are usually erased at runtime, meaning that the type information is not available during the execution of the program. 

However, Kotlin provides a feature called "reified type parameters" that allows us to retain the type information at runtime. 

This is particularly useful when you need to perform operations based on the type information, such as type checks or casting.

TODO:

[Kotlin Docs](https://kotlinlang.org/docs/inline-functions.html#reified-type-parameters)

## Inline properties

The `inline` modifier can be used on accessors of properties that don't have backing fields. You can annotate individual property accessors:

```Kotlin
val foo: Foo
    inline get() = Foo()

var bar: Bar
    get() = ...
    inline set(v) { ... }
```

You can also annotate an entire property, which marks both of its accessors as `inline`:

```Kotlin
inline var bar: Bar
    get() = ...
    set(v) { ... }
```

At the call site, inline accessors are inlined as regular inline functions.

## Restrictions for public API inline functions

When an inline function is `public` or `protected` but is not a part of a `private` or `internal` declaration, it is considered a module's public API. 

It can be called in other modules and is inlined at such call sites as well.

This imposes certain risks of binary incompatibility caused by changes in the module that declares an inline function in case the calling module is not re-compiled after the change.

To eliminate the risk of such incompatibility being introduced by a change in a non-public API of a module, public API inline functions are not allowed to use non-public-API declarations, i.e. `private` and `internal` declarations and their parts, in their bodies.

An `internal` declaration can be annotated with `@PublishedApi`, which allows its use in public API inline functions. 

When an `internal` inline function is marked as `@PublishedApi`, its body is checked too, as if it were public.
