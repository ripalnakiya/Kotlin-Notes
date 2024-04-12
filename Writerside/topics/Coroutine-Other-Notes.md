# Other Notes

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

## suspending functions and suspending points

```Kotlin
suspend fun fun1() {
    println("fun1 started")
    yield()
    println("fun1 finished")
}

suspend fun fun2() {
    println("fun2 started")
    yield()
    println("fun2 finished")
}

fun main() = runBlocking {
    launch {
        fun1()
    }
    launch {
        fun2()
    }
    println("main finished")
}
```

`yield` is also a suspending function. It is used to pause the execution of the coroutine and give chance to other coroutines to execute.

It is similar to `Thread.yield()` in Java.

Another suspending function is `delay()`. It is used to pause the execution of the coroutine for a specific time.

## kotlin for android

```Kotlin
// Coroutine Scope can be viewed as lifecycle of the coroutine
// Coroutine Contex can be viewed as thread for the coroutine

fun main() {
    // It gives a scope for a coroutine, and gives context (thread) to execute on
    CoroutineScope(Dispatchers.IO).launch {
        println("1 ${Thread.currentThread().name}")
    }

    // It gives global scope for coroutine to work,
    // Coroutine execute as long as Appliction is running
    GlobalScope.launch(Dispatchers.Main) {
        println("2 ${Thread.currentThread().name}")
    }

    // We can also have a scope specific to MainAcitivity
    MainScope().launch(Dispatchers.Default) {
        println("3 ${Thread.currentThread().name}")
    }
}
```