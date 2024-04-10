# Coroutines Basics

<show-structure depth="2"/>

<note>

For coroutine docs, check [API Reference](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/)
</note>

A **coroutine** is an instance of a suspendable computation. 

It is conceptually similar to a thread, in the sense that it takes a block of code to run that works concurrently with the rest of the code.

However, a coroutine is not bound to any particular thread. It may suspend its execution in one thread and resume in another one.

- Single Threaded Coroutine

  ![SingleThreadedCoroutine.png](single-threaded-coroutine.png)

- Multi Threaded Coroutine

  ![MultiThreadedCoroutine.png](multi-threaded-coroutine.png)

<tip>
Coroutines are programming construct. That's why all coroutines actually may end up sharing same thread by default.
</tip>

Coroutines can be _thought_ of as **light-weight threads**, but there is a number of important differences that make their real-life usage very different from threads.

```Kotlin
fun main() = runBlocking { // this: CoroutineScope
    launch { // launch a new coroutine and continue
        delay(1000L) // non-blocking delay for 1 second (default time unit is ms)
        println("World!") // print after delay
    }
    println("Hello") // main coroutine continues while a previous one is delayed
}

// Output:
Hello
World!
```

`launch` is a **coroutine builder**. It launches a new coroutine concurrently with the rest of the code, which continues to work independently. That's why `Hello` has been printed first.

`delay` is a special **suspending function**. It suspends the coroutine for a specific time. 

<note>
Suspending a coroutine does not block the underlying thread, but allows other coroutines to run and use the underlying thread for their code.
</note>

`runBlocking` is also a **coroutine builder** that bridges the non-coroutine world of a regular `fun main()` and the code with coroutines inside of `runBlocking { ... }` curly braces. 

This is highlighted in an IDE by `this: CoroutineScope` hint right after the `runBlocking` opening curly brace.

> If you remove or forget `runBlocking` in this code, you'll get an error on the launch call, since `launch` is declared only on the CoroutineScope.

The name of `runBlocking` means that the thread that runs it (in this case — the main thread) gets blocked for the duration of the call, until all the coroutines inside `runBlocking { ... }` complete their execution. 

<note>

When you call `runBlocking`, it blocks the thread it runs on. In simple terms, this means that the thread stops doing anything else until all the work inside runBlocking is finished.

If you use `runBlocking` in the main thread (the thread that runs your app's main code), the main thread will pause until everything inside `runBlocking { ... }` is done.

Inside the `runBlocking { ... }`, you can start other coroutines (lightweight threads) to do work concurrently.

Once all those coroutines inside the `runBlocking` complete their tasks, the main thread will continue with its next task.
</note>

You will often see `runBlocking` used like that at the very top-level of the application and quite rarely inside the real code, as threads are expensive resources and blocking them is inefficient and is often not desired.

## Structured concurrency

Coroutines follow a principle of **structured concurrency** which means that new coroutines can only be launched in a specific `CoroutineScope` which delimits the lifetime of the coroutine.

The above example shows that `runBlocking` establishes the corresponding scope and that is why the previous example waits until `World!` is printed after a second's delay and only then exits.

In a real application, you will be launching a lot of coroutines. Structured concurrency ensures that they are not lost and do not leak. An outer scope cannot complete until all its children coroutines complete.

## Extract function refactoring

Let's extract the block of code inside `launch { ... }` into a separate function. 

When you perform "Extract function" refactoring on this code, you get a new function with the **suspend** modifier. This is your first **suspending function**. 

Suspending functions can be used inside coroutines just like regular functions, but their additional feature is that they can, in turn, use other suspending functions (like `delay` in this example) to **suspend** execution of a coroutine.

```Kotlin
fun main() = runBlocking { // this: CoroutineScope
    launch { doWorld() }
    println("Hello")
}

suspend fun doWorld() {
    delay(1000L)
    println("World!")
}

// Output:
Hello
World!
```

## Scope builder

In addition to the coroutine scope provided by different builders, it is possible to declare your own scope using the `coroutineScope` builder. 

It creates a coroutine scope and does not complete until all launched children complete.

`runBlocking` and `coroutineScope` builders may look similar because they both wait for their body and all its children to complete. 

The main difference is that the `runBlocking` method **blocks** the current thread for waiting, while `coroutineScope` just suspends, releasing the underlying thread for other usages. 

Because of that difference, `runBlocking` is a regular function and `coroutineScope` is a suspending function.

You can use `coroutineScope` from any suspending function. For example, you can move the concurrent printing of `Hello` and `World` into a `suspend fun doWorld()` function:

```Kotlin
fun main() = runBlocking {
    doWorld()
}

suspend fun doWorld() = coroutineScope {  // this: CoroutineScope
    launch {
        delay(1000L)
        println("World!")
    }
    println("Hello")
}
```

## Scope builder and concurrency

A `coroutineScope` builder can be used inside any suspending function to perform multiple concurrent operations. Let's launch two concurrent coroutines inside a `doWorld` suspending function:

```Kotlin
// Sequentially executes doWorld followed by "Done"
fun main() = runBlocking {
    doWorld()
    println("Done")
}

// Concurrently executes both sections
suspend fun doWorld() = coroutineScope { // this: CoroutineScope
    launch {
        delay(2000L)
        println("World 2")
    }
    launch {
        delay(1000L)
        println("World 1")
    }
    println("Hello")
}

// Output:
Hello
World 1
World 2
Done
```
A coroutineScope in doWorld completes only after both are complete, so doWorld returns and allows Done string to be printed only after that.

## An explicit job

A `launch` coroutine builder returns a `Job` object that is a handle to the launched coroutine and can be used to explicitly wait for its completion. For example, you can wait for completion of the child coroutine and then print "Done" string:

```Kotlin
val job = launch { // launch a new coroutine and keep a reference to its Job
    delay(1000L)
    println("World!")
}
println("Hello")
job.join() // wait until child coroutine completes
println("Done") 

// Output:
Hello
World!
Done
```

## Coroutines are light-weight

Coroutines are less resource-intensive than JVM threads. 

Code that exhausts the JVM's available memory when using threads can be expressed using coroutines without hitting resource limits.

<tip>
Creating multiple threads means actually sharing limited number of CPU cores between threads.
</tip>

<note>
Coroutines are not bound to physical cores of CPU like threads. Hence the name light-weight threads.
</note>