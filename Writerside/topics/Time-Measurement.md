# Time Measurement

<show-structure depth="2"/>

The Kotlin standard library gives you the tools to calculate and measure time in different units. Accurate time measurement is important for activities like:

- Managing threads or processes

- Collecting statistics

- Detecting timeouts

- Debugging

By default, time is measured using a monotonic time source, but other time sources can be configured.

## Calculate duration

To represent an amount of time, the standard library has the `Duration` class. A Duration can be expressed in the following units from the `DurationUnit` enum class:

- `NANOSECONDS`
- `MICROSECONDS`
- `MILLISECONDS`
- `SECONDS`
- `MINUTES`
- `HOURS`
- `DAYS`

A `Duration` can be positive, negative, zero, positive infinity, or negative infinity.

### Create duration

To create a `Duration`, use the extension properties available for `Int`, `Long`, and `Double` types: `nanoseconds`, `microseconds`, `milliseconds`, `seconds`, `minutes`, `hours`, and `days`.

```Kotlin
val fiveHundredMilliseconds: Duration = 500.milliseconds
val zeroSeconds: Duration = 0.seconds
val tenMinutes: Duration = 10.minutes
val negativeNanosecond: Duration = (-1).nanoseconds
val infiniteDays: Duration = Double.POSITIVE_INFINITY.days
val negativeInfiniteDays: Duration = Double.NEGATIVE_INFINITY.days

println(fiveHundredMilliseconds)        // 500ms
println(zeroSeconds)                    // 0s
println(tenMinutes)                     // 10m
println(negativeNanosecond)             // -1ns
println(infiniteDays)                   // Infinity
println(negativeInfiniteDays)           // -Infinity
```

You can also perform basic arithmetic with `Duration` objects:

```Kotlin
val fiveSeconds: Duration = 5.seconds
val thirtySeconds: Duration = 30.seconds

println(fiveSeconds + thirtySeconds)
// 35s
println(thirtySeconds - fiveSeconds)
// 25s
println(fiveSeconds * 2)
// 10s
println(thirtySeconds / 2)
// 15s
println(thirtySeconds / fiveSeconds)
// 6.0
println(-thirtySeconds)
// -30s
println((-thirtySeconds).absoluteValue)
// 30s
```

### Get string representation

It can be useful to have a string representation of a `Duration` so that you can print, serialize, transfer, or store it.

To get a string representation, use the `.toString()` function. By default, the time is reported using each unit that is present. For example: `1h 0m 45.677s` or `-(6d 5h 5m 28.284s)`

To configure the output, use the `.toString()` function with your desired `DurationUnit` and number of decimal places as function parameters:

```Kotlin
// Print in seconds with 2 decimal places
println(5887.milliseconds.toString(DurationUnit.SECONDS, 2))
// 5.89s
```

### Convert duration

To convert your `Duration` into a different DurationUnit, use the following properties:

- `inWholeNanoseconds`
- `inWholeMicroseconds`
- `inWholeMilliseconds`
- `inWholeSeconds`
- `inWholeMinutes`
- `inWholeHours`
- `inWholeDays`

```Kotlin
val thirtyMinutes: Duration = 30.minutes
println(thirtyMinutes.inWholeSeconds)
// 1800
```

Alternatively, you can use your desired `DurationUnit` as a function parameter in the following extension functions:

- `.toInt()`

- `.toDouble()`

- `.toLong()`

```Kotlin
println(270.seconds.toDouble(DurationUnit.MINUTES))
// 4.5
```

### Compare duration

To check if `Duration` objects are equal, use the equality operator (`==`):

```Kotlin
val thirtyMinutes: Duration = 30.minutes
val halfHour: Duration = 0.5.hours
println(thirtyMinutes == halfHour)              // true
```

To compare Duration objects, use the comparison operators (`<`, `>`):

```Kotlin
println(3000.microseconds < 25000.nanoseconds)
// false
```

### Break duration into components

To break down a Duration into its time components and perform a further action, use the overload of the `toComponents()` function. 

Add your desired action as a function or lambda expression as a function parameter.

```Kotlin
val thirtyMinutes: Duration = 30.minutes
println(thirtyMinutes.toComponents { hours, minutes, _, _ -> "${hours}h:${minutes}m" })
// 0h:30m
```

In this example, the lambda expression has `hours` and `minutes` as function parameters with underscores (`_`) for the unused `seconds` and `nanoseconds` parameters. 

The expression returns a concatenated string using string templates to get the desired output format of `hours` and `minutes`.

## Measure time

To track the passage of time, the standard library provides tools so that you can easily:

- Measure the time taken to execute some code with your desired time unit.

- Mark a moment in time.

- Compare and subtract two moments in time.

- Check how much time has passed since a specific moment in time.

- Check whether the current time has passed a specific moment in time.

### Measure code execution time

To measure the time taken to execute a block of code, use the `measureTime` inline function:

```Kotlin
val timeTaken = measureTime {
    Thread.sleep(100)
}
println(timeTaken) // e.g. 103 ms
```

To measure the time taken to execute a block of code **and** return the value of the block of code, use inline function `measureTimedValue`.

```Kotlin
val (value, timeTaken) = measureTimedValue {
    Thread.sleep(100)
    42
}
println(value)     // 42
println(timeTaken) // e.g. 103 ms
```

By default, both functions use a monotonic time source.

### Mark moments in time

To mark a specific moment in time, use the `TimeSource` interface and the `markNow()` function to create a `TimeMark`:

```Kotlin
import kotlin.time.*

fun main() {
   val timeSource = TimeSource.Monotonic
   val mark = timeSource.markNow()
}
```

### Measure differences in time

To measure differences between `TimeMark` objects from the same time source, use the subtraction operator (`-`).

To compare `TimeMark` objects from the same time source, use the comparison operators (`<`, `>`).

```Kotlin
val timeSource = TimeSource.Monotonic
val mark1 = timeSource.markNow()
Thread.sleep(500) // Sleep 0.5 seconds.
val mark2 = timeSource.markNow()

repeat(4) { n ->
    val mark3 = timeSource.markNow()
    val elapsed1 = mark3 - mark1
    val elapsed2 = mark3 - mark2

    println("Measurement 1.${n + 1}: elapsed1=$elapsed1, elapsed2=$elapsed2, diff=${elapsed1 - elapsed2}")
}

println(mark2 > mark1) // This is true, as mark2 was captured later than mark1.
// true

// Output:
Measurement 1.1: elapsed1=500.568199ms, elapsed2=2.719us, diff=500.565480ms
Measurement 1.2: elapsed1=517.542268ms, elapsed2=16.976788ms, diff=500.565480ms
Measurement 1.3: elapsed1=517.639927ms, elapsed2=17.074447ms, diff=500.565480ms
Measurement 1.4: elapsed1=517.715181ms, elapsed2=17.149701ms, diff=500.565480ms
true
```

To check if a deadline has passed or a timeout has been reached, use the `hasPassedNow()` and `hasNotPassedNow()` extension functions:

```Kotlin
val timeSource = TimeSource.Monotonic
val mark1 = timeSource.markNow()
val fiveSeconds: Duration = 5.seconds
val mark2 = mark1 + fiveSeconds

// It hasn't been 5 seconds yet
println(mark2.hasPassedNow())
// false

// Wait six seconds
Thread.sleep(6000)
println(mark2.hasPassedNow())
// true
```
