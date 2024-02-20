# Conditions and Loops

## if expression

In Kotlin, `if` is an expression: it returns a value.

```Kotlin
var max = a
if (a < b) max = b

// With else
if (a > b) {
    max = a
} else {
    max = b
}

// As expression
max = if (a > b) a else b

// You can also use `else if` in expressions:
val maxLimit = 1
val maxOrLimit = if (maxLimit > a) maxLimit else if (a > b) a else b
```

Branches of an if expression can be blocks. In this case, the last expression is the value of a block:

```Kotlin
val max = if (a > b) {
    print("Choose a")
    a
} else {
    print("Choose b")
    b
}
```

<note>
If you're using if as an expression, for example, for returning its value or assigning it to a variable, the else branch is mandatory.
</note>

## when expression

`when` defines a conditional expression with multiple branches. 

It is similar to the switch statement in C-like languages.

```Kotlin
when (x) {
    1 -> print("x == 1")
    2 -> print("x == 2")
    else -> {
        print("x is neither 1 nor 2")
    }
}
```

`when` matches its argument against all branches sequentially until some branch condition is satisfied.

`when` can be used either as an expression or as a statement.
- If it is used as an expression, the value of the first matching branch becomes the value of the overall expression.
- If it is used as a statement, the values of individual branches are ignored.

The `else` branch is evaluated if none of the other branch conditions are satisfied.

<br></br>

In `when` statements, the `else` branch is mandatory in the given condition:
- `when` has a subject of a `Boolean`, `enum`, or `sealed` type, or their nullable counterparts 
- and branches of `when` don't cover all possible cases for this subject.

```Kotlin
enum class Color {
    RED, GREEN, BLUE
}

when (getColor()) {
    Color.RED -> println("red")
    Color.GREEN -> println("green")
    Color.BLUE -> println("blue")
    // 'else' is not required because all cases are covered
}

when (getColor()) {
    Color.RED -> println("red") // no branches for GREEN and BLUE
    else -> println("not red") // 'else' is required
}
```

To define a common behavior for multiple cases, combine their conditions in a single line with a comma:

```Kotlin
when (x) {
    0, 1 -> print("x == 0 or x == 1")
    else -> print("otherwise")
}
```

You can use arbitrary expressions (not only constants) as branch conditions

```Kotlin
when (x) {
    s.toInt() -> print("s encodes x")
    else -> print("s does not encode x")
}
```

You can also check a value for being `in` or `!in` a range or a collection:

```Kotlin
when (x) {
    in 1..10 -> print("x is in the range")
    in validNumbers -> print("x is valid")
    !in 10..20 -> print("x is outside the range")
    else -> print("none of the above")
}
```

Another option is checking that a value `is` or `!is` of a particular type.

```Kotlin
fun hasPrefix(x: Any) = when(x) {
    is String -> x.startsWith("prefix")
    else -> false
}
```

`when` can also be used as a replacement for an `if`-`else` `if` chain. 

If no argument is supplied, the branch conditions are simply boolean expressions, and a branch is executed when its condition is true:

```Kotlin
when {
    x.isOdd() -> print("x is odd")
    y.isEven() -> print("y is even")
    else -> print("x+y is odd")
}
```

You can capture when subject in a variable using following syntax:

```Kotlin
fun Request.getBody() =
    when (val response = executeRequest()) {
        is Success -> response.body
        is HttpError -> throw HttpException(response.status)
    }
```

The scope of variable introduced in `when` subject is restricted to the body of this `when`.

## for Loops

The for loop iterates through anything that provides an iterator.

```Kotlin
for (item in collection) print(item)

for (item: Int in ints) {
    // ...
}
```

```Kotlin
for (i in 1..3) {
    print(i)                    \\ 1 2 3
}
for (i in 6 downTo 0 step 2) {
    print(i)                    \\ 6 4 2 0
}
```

A `for` loop over a range or an array is compiled to an index-based loop that does not create an iterator object.

If you want to iterate through an array or a list with an index, you can do it this way:

```Kotlin
for (i in array.indices) {
    println(array[i])
}
```

Alternatively, you can use the withIndex library function:

```Kotlin
for ((index, value) in array.withIndex()) {
    println("the element at $index is $value")
}
```

## while loops

```Kotlin
while (x > 0) {
    x--
}

do {
    val y = retrieveData()
} while (y != null) // y is visible here!
```