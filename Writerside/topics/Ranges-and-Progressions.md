# Ranges and Progressions

Kotlin lets you easily create ranges of values using the `.rangeTo()` and `.rangeUntil()` functions from the `kotlin.ranges` package.

To create:
- a closed-ended range, call the `.rangeTo()` function with the `..` operator.
- an open-ended range, call the `.rangeUntil()` function with the `..<` operator.

```Kotlin
// Closed-ended range
println(4 in 1..4)      // true

// Open-ended range
println(4 in 1..<4)     // false
```

Ranges are particularly useful for iterating over `for` loops:

```Kotlin
for (i in 1..4) print(i)    // 1234
```

To iterate numbers in reverse order, use the `downTo` function instead of `..`.

```Kotlin
for (i in 4 downTo 1) print(i)      // 4321
```

It is also possible to iterate over numbers with an arbitrary step (not necessarily 1). This is done via the `step` function.

```Kotlin
for (i in 0..8 step 2) print(i)             // 02468

for (i in 0..<8 step 2) print(i)            // 0246

for (i in 8 downTo 0 step 2) print(i)       // 86420
```

## Progression

The ranges of integral types, such as `Int`, `Long`, and `Char`, can be treated as arithmetic progressions. 

In Kotlin, these progressions are defined by special types: `IntProgression`, `LongProgression`, and `CharProgression`.

Progressions have three essential properties: the `first` element, the `last` element, and a non-zero `step`. The first element is `first`, subsequent elements are the previous element plus a `step`. 

Iteration over a progression with a positive step is equivalent to an indexed `for` loop in Java/JavaScript.

```Kotlin
for (int i = first; i <= last; i += step) {
  // ...
}
```

When you create a progression implicitly by iterating a range, this progression's first and last elements are the range's endpoints, and the step is 1.

```Kotlin
for (i in 1..10) print(i)
// 12345678910
```

To define a custom progression step, use the step function on a range.

```Kotlin
for (i in 1..8 step 2) print(i)
// 1357
```

The `last` element of the progression is calculated this way:

- For a positive step: the maximum value not greater than the end value such that `(last - first) % step == 0`.
- For a negative step: the minimum value not less than the end value such that `(last - first) % step == 0`.

Thus, the `last` element is not always the same as the specified end value.

```Kotlin
for (i in 1..9 step 3) print(i) // the last element is 7
// 147
```

Progressions implement `Iterable<N>`, where `N` is `Int`, `Long`, or `Char` respectively, so you can use them in various collection functions like map, filter, and other.

```Kotlin
println((1..10).filter { it % 2 == 0 })
// [2, 4, 6, 8, 10]
```