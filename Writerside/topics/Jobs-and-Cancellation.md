# Jobs and Cancellation

<show-structure depth="2"/>

![job-deferred.png](job-deferred.png)

## Job hierarchy

![job-hierarchy.png](job-hierarchy.png)

```Kotlin
var job = launch(Disapatchers.IO) {
    var fb = async { getFbFollowers() }
    var ig = async { getIgFollowers() }
    println("FB = ${ fb.await() } | IG = ${ ig.await() }")
}
```

Advantage of using job hierarchy is that
- if we cancel the parent job, the child jobs are automatically cancelled
- if we want to wait until the parent job is completed, then parent job will wait until the child jobs are completed

Child jobs inherit the parent coroutine context :

```Kotlin
val parentJob = launch(Dispatchers.Main) {
    println("parentJob scope is $coroutineContext")
    
    val childJob = launch {
        println("childJob scope is $coroutineContext")
    }
    
    // We can also override the parent coroutine context
    val childJob = launch(Dispatchers.IO) {
        println("childJob scope is $coroutineContext")
    }
}
```

## Scenarios

### join parentJob

```Kotlin
suspend fun execute() {
    val parentJob = launch(Dispatchers.Main) {
        println("parentJob started")
    
        val childJob = launch(Dispatchers.IO) {
            println("childJob started")
            delay(5000)
            println("childJob ended")
        }
        
        delay(3000)
        println("parentJob ended")
    }
    
    parentJob.join()
    // This method suspends the calling routine until the parentJob is completed
    
    println("parentJob is completed")
}

// Output:
parentJob started
childJob started
parentJob ended
childJob ended
parentJob is completed      // parentJob will not be in completed state 
                            // until all the childJobs are completed
```

### cancel parentJob

```Kotlin
suspend fun execute() {
    val parentJob = launch(Dispatchers.Main) {
        println("parentJob started")
    
        val childJob = launch(Dispatchers.IO) {
            println("childJob started")
            delay(5000)
            println("childJob ended")
        }
        
        delay(3000)
        println("parentJob ended")
    }
    
    delay(1000)
    parentJob.cancel()
    parentJob.join()    
    println("parentJob is completed")
}

// Output:
parentJob started
childJob started
parentJob is completed
```

We have cancelled the `parentJob` and `childJob` has automatically been cancelled.

### cancel childJob

```Kotlin
suspend fun execute() {
    val parentJob = launch(Dispatchers.Main) {
        println("parentJob started")
    
        val childJob = launch(Dispatchers.IO) {
            println("childJob started")
            delay(5000)
            println("childJob ended")
        }
        
        delay(3000)
        childJob.cancel()
        println("childJob cancelled")
        println("parentJob ended")
    }
    
    parentJob.join()    
    println("parentJob is completed")
}

// Output:
parentJob started
childJob started
childJob cancelled
parentJob ended
parentJob is completed
```

What happens under the hood is, when we cancel the `childJob`, it throws a `CancellationException` which is caught by the parent coroutine and it cancels the `childJob`.


```Kotlin
suspend fun execute() {
    val parentJob = launch(Dispatchers.Main) {
        println("parentJob started")
    
        val childJob = launch(Dispatchers.IO) {
            try {
                println("childJob started")
                delay(5000)
                println("childJob ended")
            } catch (e: CancellationException) {
                println("childJob cancelled")
            }
        }
        
        delay(3000)
        childJob.cancel()
        println("parentJob ended")
    }
    
    parentJob.join()    
    println("parentJob is completed")
}

// Output:
parentJob started
childJob started
parentJob ended
childJob cancelled
parentJob is completed
```

### cancel a running job

```Kotlin
suspend fun execute() {
    val parentJob = CoroutineScope(Dispatchers.IO).launch {
        for (i in 1..1000) {
            longRunningTask()
            println(i)
        }
    }

    delay(100)
    println("Cancelling Job")
    parentJob.cancel()
    parentJob.join()
    println("parentJob is Completed")
}

fun longRunningTask() {
    for (i in 1..10000000) {
    }
}
```

In this case, we are running a long running task in a loop. We are cancelling the job after 100ms.

But actually it will not cancel after 100ms.

What happens is job is in cancelled state, but thread is busy in executing the long running task.

We can solve this issue by checking the `isActive` property of the coroutine.

```Kotlin
suspend fun execute() {
    val parentJob = CoroutineScope(Dispatchers.IO).launch {
        if (isActive) {
            for (i in 1..1000) {
                longRunningTask()
                println(i)
            }
        }
    }

    delay(10)
    println("Cancelling Job")
    parentJob.cancel()
    parentJob.join()
    println("parentJob is Completed")
}
```