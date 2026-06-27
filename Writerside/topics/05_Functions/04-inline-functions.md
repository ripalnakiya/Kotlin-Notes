# Inline functions
<show-structure depth="2"/>

Using higher-order functions imposes certain runtime penalties: each function is an object, and it captures a closure.
A closure is a scope of variables that can be accessed in the body of the function.

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

Note that some inline functions may call the lambdas passed to them 
as parameters not directly from the function body, 
but from another execution context, 
such as a local object or a nested function. 
