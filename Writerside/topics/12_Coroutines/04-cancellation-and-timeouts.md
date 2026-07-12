# Cancellation and Timeouts
<show-structure depth="3"/>

Cancellation lets you stop a coroutine before it completes.

It stops work that's no longer needed, such as when a user closes a window 
or navigates away in a user interface while a coroutine is still running.
You can also use it to release resources early 
and to stop a coroutine from accessing objects past their disposal.

> You can use cancellation to stop long-running coroutines 
> that keep producing values even after other coroutines no longer need them, for example, in `pipelines`.
>
{style="tip"}

Cancellation works through the [`Job`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/) handle, 
which represents the lifecycle of a coroutine and its parent-child relationships.
`Job` allows you to check whether the coroutine is active and allows you to cancel it, 
along with its children, as defined by [structured concurrency](01-coroutines-basics.md#coroutine-scope-and-structured-concurrency).

## Cancel coroutines

A coroutine is canceled when the [`cancel()`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/cancel.html) function is invoked on its `Job` handle.

[Coroutine builder functions](01-coroutines-basics.md#coroutine-builder-functions) such as
[`.launch()`](01-coroutines-basics.md#coroutinescope-launch) return a `Job`. The [`.async()`](01-coroutines-basics.md#coroutinescope-async)
function returns a [`Deferred`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-deferred/), which
implements `Job` and supports the same cancellation behavior.

You can call the `cancel()` function manually, or it can be invoked automatically through cancellation propagation when a parent coroutine is canceled.

When a coroutine is canceled, it throws a [`CancellationException`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-cancellation-exception/) **the next time it checks for cancellation**.
For more information about how and when this happens, see [Suspension points and cancellation](#suspension-points-and-cancellation).

> You can use the [`awaitCancellation()`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/await-cancellation.html) function to suspend a coroutine until it's canceled.
>
{style="tip"}

Here's an example on how to manually cancel coroutines:

```kotlin
suspend fun main() {

    withContext(Dispatchers.Default) {
        // Used as a signal that the coroutine has started running
        val job1Started = CompletableDeferred<Unit>()
        
        val job1: Job = launch {
            println("The coroutine has started")

            // Completes the CompletableDeferred, signaling that the coroutine has started running
            job1Started.complete(Unit)

            try {
                // Suspends indefinitely
                delay(Duration.INFINITE) // Without cancellation, this call would never return
            } catch (e: CancellationException) {
                println("The coroutine was canceled: $e")
                // Always rethrow cancellation exceptions!
                throw e
            }
            println("This line will never be executed")
        }
      
        // Waits for job1 to start before canceling it
        job1Started.await()

        // Cancels the coroutine, so delay() throws a CancellationException
        job1.cancel()

        // async returns a Deferred handle, which inherits from Job
        val job2 = async {
            // If the coroutine is canceled before its body starts executing, this line may not be printed
            println("The second coroutine has started")
            try {
                // Equivalent to delay(Duration.INFINITE)
                awaitCancellation() // Suspends until this coroutine is canceled
            } catch (e: CancellationException) {
                println("The second coroutine was canceled")
                throw e
            }
        }
        job2.cancel()
    }

    // Coroutine builders such as withContext() or coroutineScope() wait for all child coroutines to complete, even when the children are canceled
    println("All coroutines have completed")
}
```

```text
The coroutine has started
The coroutine was canceled: kotlinx.coroutines.JobCancellationException: StandaloneCoroutine was cancelled; job="coroutine#1":StandaloneCoroutine{Cancelling}@5e1bf8c
All coroutines have completed
```

In this example, [`CompletableDeferred`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-completable-deferred/) is used as a signal that the coroutine has started running.
The coroutine calls `complete()` when it starts executing, 
and `await()` only returns once that `CompletableDeferred` is completed. 

This way, cancellation happens only after the coroutine has started running.

The coroutine created by `.async()` doesn't have this check, 
so it may be canceled before it can run the code inside its block.

> Catching `CancellationException` can break the cancellation propagation.
> If you must catch it, rethrow it to let the cancellation propagate correctly through the coroutine hierarchy.
>
> For more information, see [Coroutine exceptions handling](07-coroutine-exception-handling.md#cancellation-and-exceptions).
> 
{style="warning"}

### Cancellation propagation

[Structured concurrency](01-coroutines-basics.md#coroutine-scope-and-structured-concurrency) ensures that canceling a coroutine also cancels all of its children.
This prevents child coroutines from working after the parent has already stopped.

Here's an example:

```kotlin
suspend fun main() {

    withContext(Dispatchers.Default) {
        // Used as a signal that the child coroutines have been launched
        val childrenLaunched = CompletableDeferred<Unit>()

        // Launches two child coroutines
        val parentJob = launch {
            launch {
                println("Child coroutine 1 has started running")
                try {
                    awaitCancellation()
                } finally {
                    println("Child coroutine 1 has been canceled")
                }
            }
            launch {
                println("Child coroutine 2 has started running")
                try {
                    awaitCancellation()
                } finally {
                    println("Child coroutine 2 has been canceled")
                }
            }
            // Completes the CompletableDeferred, signaling that the child coroutines have been launched
            childrenLaunched.complete(Unit)
        }

        // Waits for the parent coroutine to signal that it has launched all of its children
        childrenLaunched.await()

        // Cancels the parent coroutine, which cancels all its children
        parentJob.cancel()
    }

}
```

```text
Child coroutine 1 has started running
Child coroutine 2 has started running
Child coroutine 1 has been canceled
Child coroutine 2 has been canceled
```

In this example, each child coroutine uses a `finally` block, 
so the code inside it runs when the coroutine is canceled.

Here, `CompletableDeferred` signals that the child coroutines are launched before they are canceled, 
but it doesn't guarantee that they start running. If they are canceled first, nothing is printed.

## Make coroutines react to cancellation {id="cancellation-is-cooperative"}

In Kotlin, coroutine cancellation is **cooperative**.
This means that coroutines only react to cancellation 
when they cooperate by [suspending](#suspension-points-and-cancellation) 
or [checking for cancellation explicitly](#check-for-cancellation-explicitly).

In this section, you can learn how to create cancelable coroutines.

### Suspension points and cancellation

When a coroutine is canceled, 
it continues running until it reaches a point in the code where it may suspend, 
also known as a **suspension point**.

If the coroutine suspends there, the suspending function checks whether it has been canceled.
If it has, the coroutine stops and throws `CancellationException`.

A call to a `suspend` function is a **suspension point**, but it **doesn't always suspend**.
For example, when awaiting a `Deferred` result, 
the coroutine only suspends if that `Deferred` isn't completed yet.

Here's an example using common suspending functions that suspend, enabling the coroutine to check and stop when it's canceled:

```kotlin
suspend fun main() {

    withContext(Dispatchers.Default) {

        val childJobs = listOf(
            launch {
                awaitCancellation() // Suspends until canceled
            },
            launch {
                delay(Duration.INFINITE) // Suspends until canceled
            },
            launch {
                val channel = Channel<Int>()
                channel.receive() // Suspends while waiting for a value that is never sent
            },
            launch {
                val deferred = CompletableDeferred<Int>()
                deferred.await() // Suspends while waiting for a value that is never completed
            },
            launch {
                val mutex = Mutex(locked = true)
                mutex.lock() // Suspends while waiting for a mutex that remains locked indefinitely
            }
        )

        // Gives the child coroutines time to start and suspend
        delay(100.milliseconds)

        // Cancels all child coroutines
        childJobs.forEach { it.cancel() }
    }

    println("All child jobs completed!")
}
```

```text
All child jobs completed!
```

> All suspending functions in the `kotlinx.coroutines` library cooperate with cancellation because they use [`suspendCancellableCoroutine()`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/suspend-cancellable-coroutine.html) internally, which checks for cancellation when the coroutine suspends.
> In contrast, custom suspending functions that use [`suspendCoroutine()`](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin.coroutines/suspend-coroutine.html) don't react to cancellation.
>
{style="tip"}

### Check for cancellation explicitly

If a coroutine doesn't [suspend](#suspension-points-and-cancellation) for a long time, 
it doesn't stop when it's canceled unless it explicitly checks for cancellation.

To check for cancellation, use the following APIs:

* [`isActive`](#isactive) property is `false` when the coroutine is canceled.
* [`ensureActive()`](#ensureactive) function throws `CancellationException` immediately if the coroutine is canceled.
* [`yield()`](#yield) function suspends the coroutine, releasing the thread and giving other coroutines a chance to run on it. Suspending the coroutine lets it check for cancellation and throw `CancellationException` if it's canceled.

These APIs are useful when your coroutines run for a long time between suspension points 
or are unlikely to suspend at suspension points.

#### isActive

Use the [`isActive`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/is-active.html) property in long-running computations to periodically check for cancellation.

This property is `false` when the coroutine is no longer active, 
which you can use to gracefully stop the coroutine when it no longer needs to continue the operation:

Here's an example:

```kotlin
suspend fun main() {

    withContext(Dispatchers.Default) {
        val unsortedList = MutableList(10) { Random.nextInt() }

        val listSortingJob = launch { // Starts a long-running computation
            var i = 0

            while (isActive) { // Repeatedly sorts the list while the coroutine remains active
                unsortedList.sort()
                ++i
            }
            println("Stopped sorting the list after $i iterations")
        }

        // Sorts the list for 100 milliseconds, then considers it sorted enough
        delay(100.milliseconds)

        // Cancels the sorting when the result is good enough        
        listSortingJob.cancel()

        // Waits until the sorting coroutine finishes before accessing the shared list to avoid data races
        listSortingJob.join()

        println("The list is probably sorted: $unsortedList")
    }

}
```

```text
Stopped sorting the list after 460840 iterations
The list is probably sorted: [-2031584277, -1825661993, -1759586376, -1695060751, -1102935320, -767646355, -667958005, -36208129, 1213611900, 2048246356]
```

In this example, the [`join()`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/join.html) function suspends the coroutine until it finishes. This ensures that the list isn't accessed while the sorting coroutine is still running.

> You can use the [`cancelAndJoin()`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/cancel-and-join.html) function to cancel a coroutine and wait for it to finish in a single call.
>
{style="note"}

#### ensureActive()

Use the [`ensureActive()`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/ensure-active.html) function to check for cancellation and stop the current computation by throwing `CancellationException` if the coroutine is canceled:

```kotlin
suspend fun main() {

    withContext(Dispatchers.Default) {
        val childJob = launch {
            var start = 0
            try {
                while (true) {
                    ++start
                    // Checks the Collatz conjecture for the current number
                    var n = start
                    while (n != 1) {
                        ensureActive() // Throws CancellationException if the coroutine is canceled
                        n = if (n % 2 == 0) n / 2 else 3 * n + 1
                    }
                }
            } finally {
                println("Checked the Collatz conjecture for 0..${start-1}")
            }
        }

        // Runs the computation for one second
        delay(100.milliseconds)

        // Cancels the coroutine
        childJob.cancel()
    }

}
```

```text
Checked the Collatz conjecture for 0..69875
```

#### yield()

The [`yield()`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/yield.html) function suspends the coroutine and checks for cancellation before resuming.
Without suspending, coroutines on the same thread run sequentially.

Use `yield` to allow other coroutines to run on the same thread or thread pool before one of them finishes:

```kotlin
fun main() {

    runBlocking { // runBlocking uses the current thread for running all coroutines
        val coroutineCount = 5
        repeat(coroutineCount) { coroutineIndex ->
            launch {
                val id = coroutineIndex + 1

                repeat(3) { iterationIndex ->
                    val iteration = iterationIndex + 1
                    // Temporarily suspends to give other coroutines a chance to run
                    yield() // Without this, the coroutines run sequentially
                    println("$id * $iteration = ${id * iteration}") // Prints the coroutine index and iteration index
                }
            }
        }
    }

}
```

```text
1 * 1 = 1
2 * 1 = 2
3 * 1 = 3
4 * 1 = 4
5 * 1 = 5
1 * 2 = 2
2 * 2 = 4
3 * 2 = 6
4 * 2 = 8
5 * 2 = 10
1 * 3 = 3
2 * 3 = 6
3 * 3 = 9
4 * 3 = 12
5 * 3 = 15
```

In this example, each coroutine uses `yield()` to let other coroutines run between iterations.

```kotlin
fun main() {

    runBlocking { // runBlocking uses the current thread for running all coroutines
        val coroutineCount = 5
        repeat(coroutineCount) { coroutineIndex ->
            launch {
                val id = coroutineIndex + 1

                repeat(3) { iterationIndex ->
                    val iteration = iterationIndex + 1
                    println("$id * $iteration = ${id * iteration}") // Prints the coroutine index and iteration index
                }
            }
        }
    }

}
```

```text
1 * 1 = 1
1 * 2 = 2
1 * 3 = 3
2 * 1 = 2
2 * 2 = 4
2 * 3 = 6
3 * 1 = 3
3 * 2 = 6
3 * 3 = 9
4 * 1 = 4
4 * 2 = 8
4 * 3 = 12
5 * 1 = 5
5 * 2 = 10
5 * 3 = 15
```

### Interrupt blocking code when coroutines are canceled

On the JVM, some functions, such as `Thread.sleep()` or `BlockingQueue.take()`, can block the current thread.
These blocking functions can be interrupted, which stops them prematurely.

However, when you call them from a coroutine, cancellation doesn't interrupt the thread.

To interrupt the thread when canceling a coroutine, use the [`runInterruptible()`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/run-interruptible.html) function:

```kotlin
suspend fun main() {

    withContext(Dispatchers.Default) {
        val childStarted = CompletableDeferred<Unit>()
        val childJob = launch {
            try {
                // Cancellation triggers a thread interruption
                runInterruptible {
                    childStarted.complete(Unit)
                    try {
                        // Blocks the current thread for a very long time
                        Thread.sleep(Long.MAX_VALUE)
                    } catch (e: InterruptedException) {
                        println("Thread interrupted (Java): $e")
                        throw e
                    }
                }
            } catch (e: CancellationException) {
                println("Coroutine canceled (Kotlin): $e")
                throw e
            }
        }
        childStarted.await()

        // Cancels the coroutine and interrupts the thread by running Thread.sleep()
        childJob.cancel()
    }

}
```

```text
Thread interrupted (Java): java.lang.InterruptedException: sleep interrupted
Coroutine canceled (Kotlin): kotlinx.coroutines.JobCancellationException: StandaloneCoroutine was cancelled; job="coroutine#1":StandaloneCoroutine{Cancelling}@78f8764b
```

## Handle values safely when canceling coroutines

When a suspended coroutine is canceled, 
it resumes with a `CancellationException` instead of returning any values, 
even if those values are already available.
This behavior is called **prompt cancellation**.

It prevents your code from continuing in a canceled coroutine's scope, 
such as updating a screen that's already closed.

Here's an example:

```kotlin
// Defines a coroutine scope that uses the UI thread
class ScreenWithFileContents(private val scope: CoroutineScope) {

    fun displayFile(path: Path) {
        scope.launch {
            val contents = withContext(Dispatchers.IO) {
                Files.newBufferedReader(path, Charset.forName("US-ASCII")).use {
                    it.readLines()
                }
            }
            // It's safe to call updateUi here,
            // In case of cancellation, withContext() wouldn't return any values
            updateUi(contents)
        }
    }

    // Throws an exception if called after the user left the screen
    private fun updateUi(contents: List<String>) {
      contents.forEach { line -> addOneLineToUi(line) }
    }
  
    private fun addOneLineToUi(line: String) {
        // Placeholder for code that adds one line to the UI
    }

    // Only callable from the UI thread
    fun leaveScreen() {
        // Cancels the scope when leaving the screen
        // You can no longer update the UI
        scope.cancel()
    }
}
```

In this example, `withContext(Dispatchers.IO)` cooperates with cancellation and prevents `updateUI()` from running if the
`leaveScreen()` function cancels the coroutine before it returns the contents of the file.

While prompt cancellation prevents using values after they are no longer valid, 
it can also stop your code while an important value is still in use, which might lead to losing that value.

This can happen when a coroutine receives a value, such as an `AutoCloseable` resource, 
but is canceled before it can reach the part of the code that closes it.
To prevent this, keep cleanup logic in a place that's guaranteed to run 
even when the coroutine receiving the value is canceled.

Here's an example:

```kotlin
// scope is a coroutine scope using the UI thread
class ScreenWithFileContents(private val scope: CoroutineScope) {

    fun displayFile(path: Path) {
        scope.launch {
            // Stores the reader in a variable, so the finally block can close it
            var reader: BufferedReader? = null
            try {
                withContext(Dispatchers.IO) {
                    reader = Files.newBufferedReader(path, Charset.forName("US-ASCII"))
                }
                // Uses the stored reader after withContext() completes
                updateUi(reader!!)
            } finally {
                // Ensures the reader is closed even when the coroutine is canceled
                reader?.close()
            }
        }
    }

    private suspend fun updateUi(reader: BufferedReader) {
        // Shows the file contents
        while (true) {
            val line = withContext(Dispatchers.IO) {
                reader.readLine()
            }
            if (line == null)
                break
            addOneLineToUi(line)
        }
    }

    private fun addOneLineToUi(line: String) {
        // Placeholder for code that adds one line to the UI
    }

    // Only callable from the UI thread
    fun leaveScreen() {
        // Cancels the scope when leaving the screen
        // You can no longer update the UI
        scope.cancel()
    }
}
```

In this example, storing the `BufferedReader` in a variable 
and closing it in the `finally` block ensures the resource is released even if the coroutine is canceled.

### Run non-cancelable blocks

You can prevent cancellation from affecting certain parts of a coroutine.
To do so, pass [`NonCancellable`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-non-cancellable/) as an argument to the `withContext()` coroutine builder function.

> Avoid using `NonCancellable` with other coroutine builders like `.launch()` or `.async()`. Doing so disrupts structured concurrency by breaking the parent-child relationship.
>
{style="warning"}

`NonCancellable` is useful when you need to ensure that certain operations, such as closing resources with a suspending `close()` function,
complete even if the coroutine is canceled before they finish.

Here's an example:

```kotlin
val serviceStarted = CompletableDeferred<Unit>()

fun startService() {
    println("Starting the service...")
    serviceStarted.complete(Unit)
}

suspend fun shutdownServiceAndWait() {
    println("Shutting down...")
    delay(100.milliseconds)
    println("Successfully shut down!")
}

suspend fun main() {

    withContext(Dispatchers.Default) {
        val childJob = launch {
            startService()
            try {
                awaitCancellation()
            } finally {
                withContext(NonCancellable) {
                    // Without withContext(NonCancellable), this function doesn't complete because the coroutine is canceled
                    shutdownServiceAndWait()
                }
            }
        }
        serviceStarted.await()
        childJob.cancel()
    }

    println("Exiting the program")
}
```

```text
Starting the service...
Shutting down...
Successfully shut down!
Exiting the program
```

#### Why the finally block doesn’t run without `NonCancellable` ?

When you call `childJob.cancel()`, the coroutine running inside launch is canceled. 
Cancellation in Kotlin coroutines is cooperative: it propagates through suspension points. 

That means:
- `awaitCancellation()` suspends forever until the coroutine is canceled.
- When cancellation happens, the coroutine resumes with a `CancellationException`.
- The `finally` block is entered — but here’s the catch:
    Inside `finally`, you call `shutdownServiceAndWait()`. 
    If that function itself is suspending, it will check for cancellation before running. 
    Since the coroutine is already canceled, the suspension immediately throws `CancellationException` again, 
    preventing the block from completing.

So the `finally` block does start, but any suspending function inside it is canceled right away.

#### Role of `NonCancellable`

Wrapping the cleanup code in `withContext(NonCancellable)` changes the coroutine’s context 
so that cancellation is temporarily ignored. That ensures:

- `shutdownServiceAndWait()` runs to completion, even though the coroutine is canceled.

- Cleanup logic (like shutting down services, closing resources, flushing buffers) is guaranteed to finish.

## Timeout

Timeouts allow you to automatically cancel a coroutine after a specified duration.
They are useful for stopping operations that take too long, helping to keep your application responsive and avoid blocking threads unnecessarily.

To specify a timeout, use the [`withTimeoutOrNull()`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/with-timeout-or-null.html) function with a `Duration`:

```kotlin
suspend fun slowOperation(): Int {
    try {
        delay(300.milliseconds)
        return 5
    } catch (e: CancellationException) {
        println("The slow operation has been canceled: $e")
        throw e
    }
}

suspend fun fastOperation(): Int {
    try {
        delay(15.milliseconds)
        return 14
    } catch (e: CancellationException) {
        println("The fast operation has been canceled: $e")
        throw e
    }
}

suspend fun main() {
    withContext(Dispatchers.Default) {
        val slow = withTimeoutOrNull(100.milliseconds) {
            slowOperation()
        }
        println("The slow operation finished with $slow")
        val fast = withTimeoutOrNull(100.milliseconds) {
            fastOperation()
        }
        println("The fast operation finished with $fast")
    }
}
```

```text
The slow operation has been canceled: kotlinx.coroutines.TimeoutCancellationException: Timed out waiting for 100 ms
The slow operation finished with null
The fast operation finished with 14
```

If the timeout exceeds the specified `Duration`, `withTimeoutOrNull()` returns `null`.
