# Coroutine Builders
<show-structure depth="2"/>

## runBlocking

```Kotlin
fun main() {
    GlobalScope.launch {
        delay(1000)
        println("World")
    }
    println("Hello")
}

// Output:
Hello
```

We know that coroutines does not block the thread, therefore program continues and ends because it does not find any thread which is blocked/waiting.

```Kotlin
fun main() {
    GlobalScope.launch {
        delay(1000)
        println("World")
    }
    println("Hello")
    Thread.sleep(2000)
}

// Output:
Hello
World
```

But this approach uses `Thread.sleep` which is not a good practice. We can use `runBlocking` which blocks the thread until the coroutine finishes.

It also gives coroutine scope. It is a coroutine builder which blocks the thread until the coroutine finishes.

```Kotlin
fun main() {
    runBlocking {
        launch {
            delay(1000)
            println("World")
        }
        println("Hello")
    }
}

// Output:
Hello
World
```

It is generally used by defining the main() as function expression.

```Kotlin
fun main() = runBlocking {
    launch {
        delay(1000)
        println("World")
    }
    println("Hello")
}

// Output:
Hello
World
```

## launch vs async

| `launch`                                                           | `aysnc`                                                       |
|--------------------------------------------------------------------|---------------------------------------------------------------|
| it is used when you don't want to return any value from coroutine. | it is used when you want to return some value from coroutine. |
| it returns a job object                                            | it returns a deferred object                                  |

```Kotlin
suspend fun myCoroutine(name: String, time: Long) : String {
    println("$name started")
    delay(time)
    return "$name completed"
}

fun main() = runBlocking {
    var response = async {
        myCoroutine("Worker", 2000)
    }.await()
    println(response)
}
// Worker completed
```
We cannot get a `return` value with `launch`.

But similar functionality can be achieved using `let`.

```Kotlin
fun main() = runBlocking {
    launch {
        myCoroutine("Worker", 2000).let { println(it) }
    }.join()
}
// Worker completed
```

### Scenario 1

```Kotlin
suspend fun getFbFollowers(): Int {
    delay(1000)
    return 54
}

suspend fun printFollowers() {
    var fb = getFbFollowers()
    println(fb)
}

fun main() = runBlocking {
    launch {
        printFollowers()
    }
    println("main finished")
}

// Output: 
main finished
54
```

### Scenario 2

```Kotlin
suspend fun getFbFollowers(): Int {
    delay(1000)
    return 54
}

suspend fun printFollowers() = runBlocking {
    var fb = 0
    launch {
        fb = getFbFollowers()
    }
    println(fb)
}

fun main() = runBlocking {
    launch {
        printFollowers()
    }
    println("main finished")
}

// Output: 
main finished
0
```

This happened because when we launched the **coroutine** in `printFollowers`,
execution went on to next line and printed the value of fb
before the **coroutine** could finish.

This issue can be solved by joining the **coroutine**.

```Kotlin
suspend fun getFbFollowers(): Int {
    delay(1000)
    return 54
}

suspend fun printFollowers() = runBlocking {
    var fb = 0
    val job = launch {
        fb = getFbFollowers()
    }
    job.join()          // wait for job to complete then move ahead
    println(fb)
}

fun main() = runBlocking {
    launch {
        printFollowers()
    }
    println("main finished")
}

// Output: 
main finished
54
```

But this is not the best way to do this. We can use `async` to get the result.

```Kotlin
suspend fun getFbFollowers(): Int {
    delay(1000)
    return 54
}

suspend fun printFollowers() = runBlocking {
    val deferred = async {
        getFbFollowers()
    }
    println(deferred.await())
}

fun main() = runBlocking {
    launch {
        printFollowers()
    }
    println("main finished")
}
```


### Scenario 3

```Kotlin

suspend fun getFbFollowers(): Int {
    delay(1000)
    return 54
}

suspend fun getIgFollowers(): Int {
    delay(1000)
    return 102
}

suspend fun printFollowers() = runBlocking {
    // This approach will take 2000 milliseconds to execute
    launch {
        // This method will be called, which is a suspending funtion
        // So it will suspend the corouting until is completed (1000 milliseconds)
        var fb = getFbFollowers()

        // Now this method will be called, which is also a suspending funtion
        // So it will suspend the corouting until is completed (1000 milliseconds)
        var ig = getIgFollowers()
        
        println("FB: $fb, IG: $ig")
    }
}

fun main() = runBlocking {
    launch {
        printFollowers()
    }
    println("main finished")
}
```

This problem can be solved by launching the coroutines in parallel.

```Kotlin
suspend fun getFbFollowers(): Int {
    delay(1000)
    return 54
}

suspend fun getIgFollowers(): Int {
    delay(1000)
    return 102
}

suspend fun printFollowers() = runBlocking {
    // This approach will take 1000 milliseconds to execute
    // because we have two different coroutines launched at a time
    // And then both response are in await
    
    val deferredFb = async { getFbFollowers() }
    val deferredIg = async { getIgFollowers() }
    
    println("FB: ${deferredFb.await()}, IG: ${deferredIg.await()}")
}

fun main() = runBlocking {
    launch {
        printFollowers()
    }
    println("main finished")
}
```

### Scenario 4

```Kotlin
suspend fun getFbFollowers(): Int {
    delay(1000)
    return 54
}

suspend fun getIgFollowers(): Int {
    delay(1000)
    return 102
}

suspend fun printFollowers() = runBlocking {
    // This approach will take 2000 milliseconds to execute
    // because first coroutine is launched and then it awaits for result
    // and then the second coroutine is launched, which later on awaits for result
    val deferredFb = async {
        getFbFollowers()
    }
    println("FB: ${deferredFb.await()}")
    val deferredIg = async {
        getIgFollowers()
    }
    println("IG: ${deferredIg.await()}")
}

fun main() = runBlocking {
    launch {
        printFollowers()
    }
    println("main finished")
}
```
