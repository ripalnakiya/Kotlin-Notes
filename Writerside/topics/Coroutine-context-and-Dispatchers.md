# Coroutine context and Dispatchers

Coroutines always execute in some context represented by a value of the CoroutineContext type.

## Dispatchers and threads

The coroutine context includes a **coroutine dispatcher** that determines what thread or threads the corresponding coroutine uses for its execution. The coroutine dispatcher can confine coroutine execution to a specific thread, dispatch it to a thread pool, or let it run unconfined.

All coroutine builders like `launch` and `async` accept an optional `CoroutineContext` parameter that can be used to explicitly specify the dispatcher for the new coroutine and other context elements.

```Kotlin
launch { // context of the parent, main runBlocking coroutine
    println("main runBlocking      : I'm working in thread ${Thread.currentThread().name}")
}
launch(Dispatchers.Unconfined) { // not confined -- will work with main thread
    println("Unconfined            : I'm working in thread ${Thread.currentThread().name}")
}
launch(Dispatchers.Default) { // will get dispatched to DefaultDispatcher 
    println("Default               : I'm working in thread ${Thread.currentThread().name}")
}
launch(newSingleThreadContext("MyOwnThread")) { // will get its own new thread
    println("newSingleThreadContext: I'm working in thread ${Thread.currentThread().name}")
}
```

It produces the following output:

```
main runBlocking      : I'm working in thread main
Unconfined            : I'm working in thread main
Default               : I'm working in thread DefaultDispatcher-worker-1
newSingleThreadContext: I'm working in thread MyOwnThread
```

When` launch { ... }` is used without parameters, it inherits the context (and thus dispatcher) from the `CoroutineScope` it is being launched from. In this case, it inherits the context of the main `runBlocking` coroutine which runs in the `main` thread.

`Dispatchers.Unconfined` is a special dispatcher that also appears to run in the `main` thread, but it is, in fact, a different mechanism that is explained later.

The default dispatcher is used when no other dispatcher is explicitly specified in the scope. It is represented by `Dispatchers.Default` and uses a shared background pool of threads.

`newSingleThreadContext` creates a thread for the coroutine to run. A dedicated thread is a very expensive resource. In a real application it must be either released, when no longer needed, using the `close` function, or stored in a top-level variable and reused throughout the application.

## Unconfined vs confined dispatcher

The `Dispatchers.Unconfined` coroutine dispatcher starts a coroutine in the caller thread, but only until the first suspension point. After suspension it resumes the coroutine in the thread that is fully determined by the suspending function that was invoked. 

The unconfined dispatcher is appropriate for coroutines which neither consume CPU time nor update any shared data (like UI) confined to a specific thread.

On the other side, the dispatcher is inherited from the outer `CoroutineScope` by default. The default dispatcher for the `runBlocking` coroutine is confined to the invoker thread, so inheriting it has the effect of confining execution to this thread with predictable FIFO scheduling.

```Kotlin
fun main() = runBlocking {
    // not confined -- will work with main thread
    launch(Dispatchers.Unconfined) { 
        println("Unconfined      : I'm working in thread ${Thread.currentThread().name}")
        delay(500)
        println("Unconfined      : After delay in thread ${Thread.currentThread().name}")
    }
    
    // context of the parent, main runBlocking coroutine
    launch { 
        println("main runBlocking: I'm working in thread ${Thread.currentThread().name}")
        delay(1000)
        println("main runBlocking: After delay in thread ${Thread.currentThread().name}")
    }    
}
```

It produces the following output:

```
Unconfined      : I'm working in thread main
main runBlocking: I'm working in thread main
Unconfined      : After delay in thread kotlinx.coroutines.DefaultExecutor
main runBlocking: After delay in thread main
```

So, the coroutine with the context inherited from `runBlocking {...}` continues to execute in the `main` thread, while the unconfined one resumes in the default executor thread that the `delay` function is using.

## Jumping between threads

```Kotlin
fun log(msg: String) = println("[${Thread.currentThread().name}] $msg")

fun main() {
    newSingleThreadContext("Ctx1").use { ctx1 ->
        newSingleThreadContext("Ctx2").use { ctx2 ->
            runBlocking(ctx1) {
                log("Started in ctx1")
                withContext(ctx2) {
                    log("Working in ctx2")
                }
                log("Back to ctx1")
            }
        }
    }
}
```

It demonstrates several new techniques. One is using `runBlocking` with an explicitly specified context, and the other one is using the `withContext` function to change the context of a coroutine while still staying in the same coroutine, as you can see in the output below:

It produces the following output:

```
[Ctx1] Started in ctx1
[Ctx2] Working in ctx2
[Ctx1] Back to ctx1
```

Note that this example also uses the `use` function from the Kotlin standard library to release threads created with `newSingleThreadContext` when they are no longer needed.

## Job in the context

The coroutine's `Job` is part of its context, and can be retrieved from it using the `coroutineContext[Job]` expression:

```Kotlin
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> {
    println("My job is ${coroutineContext[Job]}")    
}

// My job is "coroutine#1":BlockingCoroutine{Active}@769c9116
```

Note that `isActive` in `CoroutineScope` is just a convenient shortcut for `coroutineContext[Job]?.isActive == true`.

## Children of a coroutine

When a coroutine is launched in the `CoroutineScope` of another coroutine, it inherits its context via` CoroutineScope.coroutineContext` and the `Job` of the new coroutine becomes a child of the parent coroutine's job. 

When the parent coroutine is cancelled, all its children are recursively cancelled, too.

However, this parent-child relation can be explicitly overriden in one of two ways:

1. When a different scope is explicitly specified when launching a coroutine (for example, `GlobalScope.launch`), then it does not inherit a `Job` from the parent scope.

2. When a different `Job` object is passed as the context for the new coroutine (as shown in the example below), then it overrides the `Job` of the parent scope.

In both cases, the launched coroutine is not tied to the scope it was launched from and operates independently.

```Kotlin
// launch a coroutine to process some kind of incoming request
val request = launch {
    // it spawns two other jobs
    launch(Job()) { 
        println("job1: I run in my own Job and execute independently!")
        delay(1000)
        println("job1: I am not affected by cancellation of the request")
    }
    // and the other inherits the parent context
    launch {
        delay(100)
        println("job2: I am a child of the request coroutine")
        delay(1000)
        println("job2: I will not execute this line if my parent request is cancelled")
    }
}
delay(500)
request.cancel() // cancel processing of the request
println("main: Who has survived request cancellation?")
delay(1000) // delay the main thread for a second to see what happens
```

It produces the following output:

```
job1: I run in my own Job and execute independently!
job2: I am a child of the request coroutine
main: Who has survived request cancellation?
job1: I am not affected by cancellation of the request
```

<note>

If we don't delay the `main` thread for a second in the last, then `main` thread won't find any waiting/blocked threads and program will finish, even though `job1` was still executing.
</note>

## Parental responsibilities

A parent coroutine always waits for completion of all its children. 

A parent does not have to explicitly track all the children it launches, and it does not have to use `Job.join` to wait for them at the end:

```Kotlin
fun main() = runBlocking {
    // This is the parent coroutine
    val job = launch {
        // Launch a child coroutine
        launch {
            delay(1000L)
            println("Child coroutine completed")
        }
        println("Parent coroutine is waiting for child to complete")
    }
    println("Main coroutine is waiting for the parent coroutine to complete")
}
```

It produces the following output:

```
Main coroutine is waiting for the parent coroutine to complete
Parent coroutine is waiting for child to complete
Child coroutine completed
```

### Other Scenario 1

```Kotlin
fun main() = runBlocking {
    // This is the parent coroutine
    val job = launch {
        // Launch a child coroutine
        launch {
            delay(1000L)
            println("Child coroutine completed")
        }
        println("Parent coroutine is waiting for child to complete")
    }
    job.join()
    println("Main coroutine is waiting for the parent coroutine to complete")
}
```

It produces the following output:

```
Parent coroutine is waiting for child to complete
Child coroutine completed
Main coroutine is waiting for the parent coroutine to complete
```

## Naming coroutines for debugging

When multiple coroutines are launched in the same context, it is useful to name them for debugging purposes.

The CoroutineName context element serves the same purpose as the thread name. It is included in the thread name that is executing this coroutine when the debugging mode is turned on.

## Combining context elements

Sometimes we need to define multiple elements for a coroutine context. We can use the `+` operator for that. 

For example, we can launch a coroutine with an explicitly specified dispatcher and an explicitly specified name at the same time:

```Kotlin
launch(Dispatchers.Default + CoroutineName("test")) {
    println("I'm working in thread ${Thread.currentThread().name}")
}

// I'm working in thread DefaultDispatcher-worker-1 @test#2
```

## Coroutine scope

Assume that our application has an object with a lifecycle, but that object is not a coroutine.

For example, we are writing an Android application and launch various coroutines in the context of an Android activity to perform asynchronous operations to fetch and update data, do animations, etc. All of these coroutines must be cancelled when the activity is destroyed to avoid memory leaks.

We, of course, can manipulate contexts and jobs manually to tie the lifecycles of the activity and its coroutines, but `kotlinx.coroutines` provides an abstraction encapsulating that: `CoroutineScope`. 

<note>

`CoroutineScope` a scope for new coroutines. Every **coroutine builder** (like launch, async, etc.) is an extension on CoroutineScope and inherits its coroutineContext to automatically propagate all its elements and cancellation.

The best ways to obtain a standalone instance of the scope are `CoroutineScope()` and `MainScope()` factory functions, taking care to cancel these coroutine scopes when they are no longer needed.
</note>

We manage the lifecycles of our coroutines by creating an instance of CoroutineScope tied to the lifecycle of our activity.

- `CoroutineScope()` : This creates a general-purpose scope
- `MainScope()` : This creates a scope for UI applications and uses Dispatchers.Main as the default dispatcher

```Kotlin
class Activity {
    private val mainScope = MainScope()

    fun destroy() {
        mainScope.cancel()
    }
    // to be continued ...
```

Now, we can launch coroutines in the scope of this `Activity` using the defined `mainScope`. For the demo, we launch ten coroutines that delay for a different time:

```Kotlin
// class Activity continues
    fun doSomething() {
        // launch ten coroutines for a demo, each working for a different time
        repeat(10) { i ->
            mainScope.launch {
                delay((i + 1) * 200L) // variable delay 200ms, 400ms, ... etc
                println("Coroutine $i is done")
            }
        }
    }
} // class Activity ends
```

In our main function we create the activity, call our test `doSomething` function, and destroy the activity after 500ms. This cancels all the coroutines that were launched from `doSomething`. We can see that because after the destruction of the activity no more messages are printed, even if we wait a little longer.

```Kotlin
val activity = Activity()
activity.doSomething() // run test function
println("Launched coroutines")
delay(500L) // delay for half a second
println("Destroying activity!")
activity.destroy() // cancels all coroutines
delay(1000) // visually confirm that they don't work
```

It produces the following output:

```
Launched coroutines
Coroutine 0 is done
Coroutine 1 is done
Destroying activity!
```

As you can see, only the first two coroutines print a message and the others are cancelled by a single invocation of `job.cancel()` in `Activity.destroy()`.