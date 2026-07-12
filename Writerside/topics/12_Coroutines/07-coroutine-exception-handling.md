# Coroutine Exception Handling
<show-structure depth="2"/>

This section covers exception handling and cancellation on exceptions.

We already know that a cancelled coroutine throws [CancellationException] in suspension points and that it
is ignored by the coroutines' machinery. 
Here we look at what happens if an exception is thrown during cancellation or multiple children of the same
coroutine throw an exception.

## Exception propagation

Coroutine builders come in two flavors: propagating exceptions automatically ([launch]) or
exposing them to users ([async] and [produce]).

When these builders are used to create a **root** coroutine, that is not a **child** of another coroutine,
the former builders treat exceptions as **uncaught** exceptions, similar to Java's `Thread.uncaughtExceptionHandler`,
while the latter are relying on the user to consume the final
exception, for example via [await][Deferred.await] or [receive][ReceiveChannel.receive]
([produce] and [receive][ReceiveChannel.receive] are covered in [Channels](https://github.com/Kotlin/kotlinx.coroutines/blob/master/docs/channels.md) section).

It can be demonstrated by a simple example that creates root coroutines using the [GlobalScope]:

> [GlobalScope] is a delicate API that can backfire in non-trivial ways. Creating a root coroutine for the
> whole application is one of the rare legitimate uses for `GlobalScope`, so you must explicitly opt-in into
> using `GlobalScope` with `@OptIn(DelicateCoroutinesApi::class)`.
>
{style="note"}

```kotlin
@OptIn(DelicateCoroutinesApi::class)
fun main() = runBlocking {
    
    val job = GlobalScope.launch { // root coroutine with launch
        println("Throwing exception from launch")
        throw IndexOutOfBoundsException() // Will be printed to the console by Thread.defaultUncaughtExceptionHandler
    }
    job.join()
        
    println("Joined failed job")
        
    val deferred = GlobalScope.async { // root coroutine with async
        println("Throwing exception from async")
        throw ArithmeticException() // Nothing is printed, relying on user to call await
    }
        
    try {
        deferred.await()
        println("Unreached")
    } catch (e: ArithmeticException) {
        println("Caught ArithmeticException")
    }
}
```

The output of this code is (with [debug](https://github.com/Kotlin/kotlinx.coroutines/blob/master/docs/coroutine-context-and-dispatchers.md#debugging-coroutines-and-threads)):

```text
Throwing exception from launch
Exception in thread "DefaultDispatcher-worker-1 @coroutine#2" java.lang.IndexOutOfBoundsException
Joined failed job
Throwing exception from async
Caught ArithmeticException
```

## CoroutineExceptionHandler

It is possible to customize the default behavior of printing **uncaught** exceptions to the console.

[CoroutineExceptionHandler] context element on a **root** coroutine can be used as a generic `catch` block for
this root coroutine and all its children where custom exception handling may take place.
It is similar to [`Thread.uncaughtExceptionHandler`](https://docs.oracle.com/javase/8/docs/api/java/lang/Thread.html#setUncaughtExceptionHandler-java.lang.Thread.UncaughtExceptionHandler-).

You cannot recover from the exception in the `CoroutineExceptionHandler`. The coroutine had already completed
with the corresponding exception when the handler is called. Normally, the handler is used to
log the exception, show some kind of error message, terminate, and/or restart the application.


`CoroutineExceptionHandler` is invoked only on **uncaught** exceptions &mdash; exceptions that were not handled in any other way.

In particular, all **children** coroutines (coroutines created in the context of another [Job]) delegate handling of
their exceptions to their parent coroutine, which also delegates to the parent, and so on until the root,
so the `CoroutineExceptionHandler` installed in their context is never used.

In addition to that, [async] builder always catches all exceptions and represents them in the resulting [Deferred] object,
so its `CoroutineExceptionHandler` has no effect either.

> Coroutines running in supervision scope do not propagate exceptions to their parent and are
> excluded from this rule. A further [Supervision](#supervision) section of this document gives more details.
>
{style="note"}

```kotlin
@OptIn(DelicateCoroutinesApi::class)
fun main() = runBlocking {
    
    val handler = CoroutineExceptionHandler { _, exception -> 
        println("CoroutineExceptionHandler got $exception") 
    }
    
    val job = GlobalScope.launch(handler) { // root coroutine, running in GlobalScope
        throw AssertionError()
    }
    
    val deferred = GlobalScope.async(handler) { // also root, but async instead of launch
        throw ArithmeticException() // Nothing will be printed, relying on user to call deferred.await()
    }
    
    joinAll(job, deferred)
}
```

The output of this code is:

```text
CoroutineExceptionHandler got java.lang.AssertionError
```

## Cancellation and exceptions

Cancellation is closely related to exceptions. Coroutines internally use `CancellationException` for cancellation, these
exceptions are ignored by all handlers, so they should be used only as the source of additional debug information, which can
be obtained by `catch` block.

When a coroutine is cancelled using [Job.cancel], it terminates, but it does not cancel its parent.

```kotlin
fun main() = runBlocking {
    
    val job = launch {
        val child = launch {
            try {
                delay(Long.MAX_VALUE)
            } finally {
                println("Child is cancelled")
            }
        }
        yield()
        println("Cancelling child")
        child.cancel()
        child.join()
        yield()
        println("Parent is not cancelled")
    }
    job.join()
    
}
```

The output of this code is:

```text
Cancelling child
Child is cancelled
Parent is not cancelled
```

If a coroutine encounters an exception other than `CancellationException`, it cancels its parent with that exception.
This behaviour cannot be overridden and is used to provide stable coroutines hierarchies for
[structured concurrency](https://github.com/Kotlin/kotlinx.coroutines/blob/master/docs/composing-suspending-functions.md#structured-concurrency-with-async).
[CoroutineExceptionHandler] implementation is not used for child coroutines.

> In these examples, [CoroutineExceptionHandler] is always installed to a coroutine
> that is created in [GlobalScope]. It does not make sense to install an exception handler to a coroutine that
> is launched in the scope of the main [runBlocking], since the main coroutine is going to be always cancelled
> when its child completes with exception despite the installed handler.
>
{style="note"}

The original exception is handled by the parent only when all its children terminate,
which is demonstrated by the following example.

```kotlin
@OptIn(DelicateCoroutinesApi::class)
fun main() = runBlocking {
    
    val handler = CoroutineExceptionHandler { _, exception -> 
        println("CoroutineExceptionHandler got $exception") 
    }
        
    val job = GlobalScope.launch(handler) {
        
        launch { // the first child
            try {
                delay(Long.MAX_VALUE)
            } finally {
                withContext(NonCancellable) {
                    println("Children are cancelled, but exception is not handled until all children terminate")
                    delay(100)
                    println("The first child finished its non cancellable block")
                }
            }
        }
        
        launch { // the second child
            delay(10)
            println("Second child throws an exception")
            throw ArithmeticException()
        }
        
    }
        
    job.join()
        
}
```

The output of this code is:

```text
Second child throws an exception
Children are cancelled, but exception is not handled until all children terminate
The first child finished its non cancellable block
CoroutineExceptionHandler got java.lang.ArithmeticException
```

## Exceptions aggregation

When multiple children of a coroutine fail with an exception, the
general rule is "the first exception wins", so the first exception gets handled.
All additional exceptions that happen after the first one are attached to the first exception as suppressed ones.

```kotlin
@OptIn(DelicateCoroutinesApi::class)
fun main() = runBlocking {
    
    val handler = CoroutineExceptionHandler { _, exception ->
        println("CoroutineExceptionHandler got $exception with suppressed ${exception.suppressed.contentToString()}")
    }
        
    val job = GlobalScope.launch(handler) {
        
        launch {
            try {
                delay(Long.MAX_VALUE) // it gets cancelled when another sibling fails with IOException
            } finally {
                throw ArithmeticException() // the second exception
            }
        }
        
        launch {
            delay(100)
            throw IOException() // the first exception
        }
        
        delay(Long.MAX_VALUE)
    }
        
    job.join()  
}
```

The output of this code is:

```text
CoroutineExceptionHandler got java.io.IOException with suppressed [java.lang.ArithmeticException]
```

Cancellation exceptions are transparent and are unwrapped by default:

```kotlin
@OptIn(DelicateCoroutinesApi::class)
fun main() = runBlocking {
    
    val handler = CoroutineExceptionHandler { _, exception ->
        println("CoroutineExceptionHandler got $exception")
    }
        
    val job = GlobalScope.launch(handler) {
        
        val innerJob = launch { // all this stack of coroutines will get cancelled
            launch {
                launch {
                    throw IOException() // the original exception
                }
            }
        }
        
        try {
            innerJob.join()
        } catch (e: CancellationException) {
            println("Rethrowing CancellationException with original cause")
            throw e // cancellation exception is rethrown, yet the original IOException gets to the handler  
        }
    }
    job.join()
        
}
```

The output of this code is:

```text
Rethrowing CancellationException with original cause
CoroutineExceptionHandler got java.io.IOException
```

## Bidirectional Relationship

> A bidirectional relationship means cancellation flows in both directions through the coroutine hierarchy:
> 1. Parent → Child
> 2. Child → Parent (with an important caveat)
>
{style="note"}

### Parent cancels Child

```kotlin
coroutineScope {
    launch {
        repeat(10) {
            delay(1000)
            println("Working...")
        }
    }

    delay(2500)
    cancel()
}
```

After 2.5 seconds:

```text
Parent cancelled
     ↓
Child cancelled
```

The child immediately receives the cancellation and stops.

### Child cancels Parent

Now suppose the child throws an exception.

```kotlin
coroutineScope {
    launch {
        delay(1000)
        throw RuntimeException("Boom!")
    }

    launch {
        repeat(5) {
            delay(500)
            println("Second child")
        }
    }
}
```

When Child 1 crashes:

```text
Child 1 fails
     ↑
Parent gets cancelled
     ↓
Child 2 gets cancelled
```

Notice the flow:

```text
Child → Parent → Siblings
```

This is why it is called **bidirectional**.

> A parent becomes _cancelled_ immediately when a child fails.
> 
> But a parent does not complete (terminate) until every child has finished, 
> including any cleanup in [finally blocks or NonCancellable sections](#cancellation-and-exceptions).
> 
{style="note"}

### Why isn't it simply parent-to-child ?

If cancellation only flowed downward, then a failed child would not affect anything else.

```text
Parent
 ├── Download file
 ├── Parse file
 └── Save to database
```

If parsing fails but downloading and saving continue anyway, you could end up saving garbage.

Structured concurrency prevents this by saying: **If one child fails, the whole operation fails.**

### The caveat: Cancellation vs Failure

A child cancelling itself normally does not cancel the parent.

```kotlin
launch {
    cancel()
}
```

This child is simply done. The parent continues happily.

But if the **child fails with an exception**:

```kotlin
launch {
    throw RuntimeException()
}
```

then the parent is cancelled.


**Summary:**
```kotlin
Child A (Exception)
      ↑
Parent (Cancelled)
      ↓
Child B (Cancelled)
```

## Supervision

As we have studied before, cancellation is a **bidirectional** relationship propagating through the whole
hierarchy of coroutines. Let us take a look at the case when unidirectional cancellation is required.

A good example of such a requirement is a UI component with the job defined in its scope. If any of the UI's child tasks
have failed, it is not always necessary to cancel (effectively kill) the whole UI component,
but if the UI component is destroyed (and its job is cancelled), then it is necessary to cancel all child jobs as their results are no longer needed.

Another example is a server process that spawns multiple child jobs and needs to _supervise_
their execution, tracking their failures and only restarting the failed ones.

### Supervision job

The [SupervisorJob][SupervisorJob()] can be used for these purposes.
It is similar to a regular [Job][Job()] with the only exception that cancellation is propagated
only downwards. This can easily be demonstrated using the following example:

```kotlin
fun main() = runBlocking {
    
    val supervisor = SupervisorJob()
    
    with(CoroutineScope(coroutineContext + supervisor)) {
        
        // launch the first child -- its exception is ignored for this example (don't do this in practice!)
        val firstChild = launch(CoroutineExceptionHandler { _, _ ->  }) {
            println("The first child is failing")
            throw AssertionError("The first child is cancelled")
        }
        
        // launch the second child
        val secondChild = launch {
            firstChild.join()
            // Cancellation of the first child is not propagated to the second child
            println("The first child is cancelled: ${firstChild.isCancelled}, but the second one is still active")
            try {
                delay(Long.MAX_VALUE)
            } finally {
                // But cancellation of the supervisor is propagated
                println("The second child is cancelled because the supervisor was cancelled")
            }
        }
        
        // wait until the first child fails & completes
        firstChild.join()
        println("Cancelling the supervisor")
        supervisor.cancel()
        secondChild.join()
    }
}
```

The output of this code is:

```text
The first child is failing
The first child is cancelled: true, but the second one is still active
Cancelling the supervisor
The second child is cancelled because the supervisor was cancelled
```

### Supervision scope

Instead of [coroutineScope][_coroutineScope], we can use [supervisorScope][_supervisorScope] for **scoped** concurrency. 

It propagates the cancellation in one direction only and cancels all its children only if it failed itself. 
It also waits for all children before completion just like [coroutineScope][_coroutineScope] does.

```kotlin
fun main() = runBlocking {
    
    try {
        supervisorScope {
            val child = launch {
                try {
                    println("The child is sleeping")
                    delay(Long.MAX_VALUE)
                } finally {
                    println("The child is cancelled")
                }
            }
            
            // Give our child a chance to execute and print using yield 
            yield()
            println("Throwing an exception from the scope")
            throw AssertionError()
        }
    } catch(e: AssertionError) {
        println("Caught an assertion error")
    }
    
}
```

The output of this code is:

```text
The child is sleeping
Throwing an exception from the scope
The child is cancelled
Caught an assertion error
```

#### Exceptions in supervised coroutines

Another crucial difference between regular and supervisor jobs is exception handling.

Every child should handle its exceptions by itself via the exception handling mechanism.
This difference comes from the fact that child's failure does not propagate to the parent.

It means that coroutines launched directly inside the [supervisorScope][_supervisorScope] **do** use the [CoroutineExceptionHandler]
that is installed in their scope in the same way as root coroutines do
(see the [CoroutineExceptionHandler](#coroutineexceptionhandler) section for details).

```kotlin
fun main() = runBlocking {
    
    val handler = CoroutineExceptionHandler { _, exception -> 
        println("CoroutineExceptionHandler got $exception") 
    }
    
    supervisorScope {
        val child = launch(handler) {
            println("The child throws an exception")
            throw AssertionError()
        }
        println("The scope is completing")
    }
    
    println("The scope is completed")
}
```

The output of this code is:

```text
The scope is completing
The child throws an exception
CoroutineExceptionHandler got java.lang.AssertionError
The scope is completed
```

[CancellationException]: https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-cancellation-exception/index.html
[launch]: https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/launch.html
[async]: https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/async.html
[Deferred.await]: https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-deferred/await.html
[GlobalScope]: https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-global-scope/index.html
[CoroutineExceptionHandler]: https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-exception-handler/index.html
[Job]: https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/index.html
[Deferred]: https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-deferred/index.html
[Job.cancel]: https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/cancel.html
[runBlocking]: https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/run-blocking.html
[SupervisorJob()]: https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-supervisor-job.html
[Job()]: https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job.html
[_coroutineScope]: https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/coroutine-scope.html
[_supervisorScope]: https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/supervisor-scope.html
[produce]: https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/produce.html
[ReceiveChannel.receive]: https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-receive-channel/receive.html
