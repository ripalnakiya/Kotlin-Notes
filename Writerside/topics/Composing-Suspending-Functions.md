# Composing Suspending Functions

A suspendable function is a function that can be paused and resumed later, allowing for asynchronous operations without blocking a thread.

```Kotlin
suspend fun fetchData(): String {
    delay(1000) // Simulates a network request
    return "Data fetched!"
}

fun main() = runBlocking {
    val result = fetchData()
    println(result)
}
```

## Sequential by default

Assume that we have two suspending functions defined elsewhere that do something useful like some kind of remote service call or computation. 

```Kotlin
suspend fun doSomethingUsefulOne(): Int {
    delay(1000L) // pretend we are doing something useful here
    return 13
}

suspend fun doSomethingUsefulTwo(): Int {
    delay(1000L) // pretend we are doing something useful here, too
    return 29
}
```

What do we do if we need them to be invoked **sequentially** — first `doSomethingUsefulOne` and then `doSomethingUsefulTwo`, and compute the sum of their results?

In practice, we do this if we use the result of the first function to make a decision on whether we need to invoke the second one or to decide on how to invoke it.

We use a normal sequential invocation, because the code in the coroutine, just like in the regular code, is sequential by default. The following example demonstrates it by measuring the total time it takes to execute both suspending functions:

```Kotlin
fun main() = runBlocking {
    val time = measureTimeMillis {
        val one = doSomethingUsefulOne()
        val two = doSomethingUsefulTwo()
        println("The answer is ${one + two}")
    }
    println("Completed in $time ms")    
}

// Output:
The answer is 42
Completed in 2027 ms
```

## Concurrent using async

What if there are no dependencies between invocations of `doSomethingUsefulOne` and `doSomethingUsefulTwo` and we want to get the answer faster, by doing both **concurrently**?

Conceptually, `async` is just like `launch`. It starts a separate coroutine which is a light-weight thread that works concurrently with all the other coroutines. 

The difference is that `launch` returns a `Job` and does not carry any resulting value, while `async` returns a `Deferred` — a light-weight non-blocking future that represents a promise to provide a result later. 

You can use `.await()` on a deferred value to get its eventual result, but `Deferred` is also a `Job`, so you can cancel it if needed.

```Kotlin
fun main() = runBlocking<Unit> {
    val time = measureTimeMillis {
        val one = async { doSomethingUsefulOne() }
        val two = async { doSomethingUsefulTwo() }
        println("The answer is ${one.await() + two.await()}")
    }
    println("Completed in $time ms")    
}

// Output:
The answer is 42
Completed in 1017 ms
```

This is twice as fast, because the two coroutines execute concurrently.

<note>
Note that concurrency with coroutines is always explicit.
</note>

## Lazily started async

Optionally, `async` can be made lazy by setting its `start` parameter to `CoroutineStart.LAZY`. In this mode it only starts the coroutine when its result is required by `await`, or if its `Job`'s start function is invoked.

```Kotlin
fun main() = runBlocking {
    val time = measureTimeMillis {
        val one = async(start = CoroutineStart.LAZY) { doSomethingUsefulOne() }
        val two = async(start = CoroutineStart.LAZY) { doSomethingUsefulTwo() }
        
        // some computation
        one.start() // start the first one
        two.start() // start the second one
        println("The answer is ${one.await() + two.await()}")
    }
    println("Completed in $time ms")    
}

// Output:
The answer is 42
Completed in 1017 ms
```

So, here the two coroutines are defined but not executed as in the previous example, but the control is given to the programmer on when exactly to start the execution by calling `start`. We first start `one`, then start `two`, and then await for the individual coroutines to finish.

Note that if we just call `await` in `println` without first calling `start` on individual coroutines, this will lead to **sequential behavior**, since `await` starts the coroutine execution and waits for its finish, which is not the intended use-case for laziness.

We manually start both tasks by calling `one.start()` and `two.start()`. If we didn't do this, calling `await()` would start the tasks one after the other, making them run sequentially instead of concurrently (at the same time).

`one.await()` and `two.await()` wait for the tasks to finish and return their results. Since we started the tasks manually, they run concurrently, and we get the results faster.

If you don't call `start()` and only use `await()`, the tasks will run one after the other, which is slower.

## Inheritance



























