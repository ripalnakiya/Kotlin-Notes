# Scope functions

<show-structure depth="2"/>

The Kotlin standard library contains several functions whose sole purpose is to execute a block of code within the context of an object. When you call such a function on an object with a lambda expression provided, it forms a temporary scope. 

In this scope, you can access the object without its name. Such functions are called scope functions. There are five of them: `let`, `run`, `with`, `apply`, and `also`.

Basically, these functions all perform the same action: execute a block of code on an object. What's different is how this object becomes available inside the block and what the result of the whole expression is.

## Example

```Kotlin
Person("Alice", 20, "Amsterdam").let {
    println(it)
    it.moveTo("London")
    it.incrementAge()
    println(it)
}

// Output:
Person(name=Alice, age=20, city=Amsterdam)
Person(name=Alice, age=21, city=London)
```

If you write the same without `let`, you'll have to introduce a new variable and repeat its name whenever you use it.

```Kotlin
val alice = Person("Alice", 20, "Amsterdam")
println(alice)
alice.moveTo("London")
alice.incrementAge()
println(alice)

// Output:
Person(name=Alice, age=20, city=Amsterdam)
Person(name=Alice, age=21, city=London)
```

Scope functions don't introduce any new technical capabilities, but they can make your code more concise and readable.

## Function selection

| Function | Object Reference | Return Value   | Is Extension Function             |
|----------|------------------|----------------|-----------------------------------|
| `let`    | `it`             | Lambda result  | Yes                               |
| `run`    | `this`           | Lambda result  | Yes                               |
| `run`    | -                | Lambda result  | No: called without the context object |
| `with`   | `this`           | Lambda result  | No: takes the context object as an argument |
| `apply`  | `this`           | Context object | Yes                               |
| `also`   | `it`             | Context object | Yes                               |

Here is a short guide for choosing scope functions depending on the intended purpose:

- Executing a lambda on non-nullable objects: `let`
- Introducing an expression as a variable in local scope: `let`
- Object configuration: `apply`
- Object configuration and computing the result: `run`
- Running statements where an expression is required: non-extension `run`
- Additional effects: `also`
- Grouping function calls on an object: `with`

Although scope functions can make your code more concise, avoid overusing them: it can make your code hard to read and lead to errors.

## Distinctions

Because scope functions are similar in nature, it's important to understand the differences between them. There are two main differences between each scope function:

- The way they refer to the context object.

- Their return value.

### Context object: this or it

Inside the lambda passed to a scope function, the context object is available by a short reference instead of its actual name. 

Each scope function uses one of two ways to reference the context object: as a lambda receiver (`this`) or as a lambda argument (`it`). 

Both provide the same capabilities, so we describe the pros and cons of each for different use cases and provide recommendations for their use.

```Kotlin
fun main() {
    val str = "Hello"
    // this
    str.run {
        println("The string's length: $length")
        //println("The string's length: ${this.length}") // does the same
    }

    // it
    str.let {
        println("The string's length is ${it.length}")
    }
}
```

#### this

`run`, `with`, and `apply` reference the context object as a lambda receiver - by keyword `this`. Hence, in their lambdas, the object is available as it would be in ordinary class functions.

In most cases, you can omit `this` when accessing the members of the receiver object, making the code shorter. On the other hand, if `this` is omitted, it can be hard to distinguish between the receiver members and external objects or functions. 

So having the context object as a receiver (`this`) is recommended for lambdas that mainly operate on the object's members by calling its functions or assigning values to properties.

```Kotlin
val adam = Person("Adam").apply { 
    age = 20                       // same as this.age = 20
    city = "London"
}
println(adam)           // Person(name=Adam, age=20, city=London)
```

#### it

In turn, `let` and `also` reference the context object as a lambda argument. If the argument name is not specified, the object is accessed by the implicit default name `it`. 

`it` is shorter than `this` and expressions with `it` are usually easier to read.

However, when calling the object's functions or properties, you don't have the object available implicitly like `this`. 

Hence, accessing the context object via `it` is better when the object is mostly used as an argument in function calls. `it` is also better if you use multiple variables in the code block.

```Kotlin
fun getRandomInt(): Int {
    return Random.nextInt(100).also {
        writeToLog("getRandomInt() generated value $it")
    }
}

val i = getRandomInt()
println(i)

// Output:
INFO: getRandomInt() generated value 22
22
```

The example below demonstrates referencing the context object as a lambda argument with argument name: `value`.

```Kotlin
fun getRandomInt(): Int {
    return Random.nextInt(100).also { value ->
        println("getRandomInt() generated value $value")
    }
}

val i = getRandomInt()
println(i)

// Output:
getRandomInt() generated value 58
58
```

- Use `it` when the object is primarily passed as an argument to other functions or when you have multiple variables in the lambda block, making it more concise and less ambiguous.
- `this` when you need to call the object's own properties or functions directly, making the code more natural and readable.

### Return value

Scope functions differ by the result they return:

- `apply` and `also` return the context object.

- `let`, `run`, and `with` return the lambda result.

You should consider carefully what return value you want based on what you want to do next in your code. This helps you to choose the best scope function to use.

#### Context object

The return value of `apply` and `also` is the context object itself. 

Hence, they can be included into call chains as side steps: you can continue chaining function calls on the same object, one after another.

```Kotlin
val numberList = mutableListOf<Double>()
numberList.also { println("Populating the list") }
    .apply {
        add(2.71)
        add(3.14)
        add(1.0)
    }
    .also { println("Sorting the list") }
    .sort()
    
// Output:
Populating the list
Sorting the list
[1.0, 2.71, 3.14]
```

They also can be used in return statements of functions returning the context object.

```Kotlin
fun getRandomInt(): Int {
    return Random.nextInt(100).also {
        println("getRandomInt() generated value $it")
    }
}

val i = getRandomInt()

// Output:
getRandomInt() generated value 85
```

#### Lambda result

`let`, `run`, and `with` return the lambda result. So you can use them when assigning the result to a variable, chaining operations on the result, and so on.

```Kotlin
val numbers = mutableListOf("one", "two", "three")
val countEndsWithE = numbers.run { 
    add("four")
    add("five")
    count { it.endsWith("e") }
}
println("There are $countEndsWithE elements that end with e.")
// There are 3 elements that end with e.
```

Additionally, you can ignore the return value and use a scope function to create a temporary scope for local variables.

```Kotlin
val numbers = mutableListOf("one", "two", "three")
with(numbers) {
    val firstItem = first()
    val lastItem = last()        
    println("First item: $firstItem, last item: $lastItem")
}

// First item: one, last item: three
```

## Functions

### let

- The **context object** is available as an argument (`it`).

- The **return value** is the lambda result.

`let` can be used to invoke one or more functions on results of call chains. 

For example, the following code prints the results of two operations on a collection:

```Kotlin
val numbers = mutableListOf("one", "two", "three", "four", "five")
val resultList = numbers.map { it.length }.filter { it > 3 }
println(resultList)             // [5, 4, 4]
```

<note>

The `map` function takes a lambda expression and applies it to each element of the collection, returning a new list with the transformed elements.

Here, The result of `map` is a new list: [3, 3, 5, 4, 4].
</note>

With `let`, you can rewrite the above example so that you're not assigning the result of the list operations to a variable:

```Kotlin
val numbers = mutableListOf("one", "two", "three", "four", "five")
numbers.map { it.length }.filter { it > 3 }.let { 
    println(it)
    // and more function calls if needed
} 
// [5, 4, 4]
```

If the code block passed to `let` contains a single function with `it` as an argument, you can use the method reference (`::`) instead of the lambda argument:

```Kotlin
val numbers = mutableListOf("one", "two", "three", "four", "five")
numbers.map { it.length }.filter { it > 3 }.let(::println)
// [5, 4, 4]
```

`let` is often used to execute a code block containing non-null values. To perform actions on a non-null object, use the safe call operator ?. on it and call let with the actions in its lambda.

```Kotlin
val str: String? = "Hello"   
//processNonNullString(str)       // compilation error: str can be null
val length = str?.let { 
    println("let() called on $it")        
    processNonNullString(it)      // OK: 'it' is not null inside '?.let { }'
    it.length
}

// Output: let() called on Hello
```

You can also use `let` to introduce local variables with a limited scope to make your code easier to read. To define a new variable for the context object, provide its name as the lambda argument so that it can be used instead of the default `it`.

```Kotlin
val numbers = listOf("one", "two", "three", "four")
val modifiedFirstItem = numbers.first().let { firstItem ->
    println("The first item of the list is '$firstItem'")
    if (firstItem.length >= 5) firstItem else "!" + firstItem + "!"
}.uppercase()
println("First item after modifications: '$modifiedFirstItem'")

// Output:
The first item of the list is 'one'
First item after modifications: '!ONE!'
```

### with

- The **context object** is available as a receiver (`this`).

- The **return value** is the lambda result.

As `with` is not an extension function: the context object is passed as an argument, but inside the lambda, it's available as a receiver (`this`).

We recommend using `with` for calling functions on the context object when you don't need to use the returned result. In code, `with` can be read as "**with this object, do the following.**"

```Kotlin
val numbers = mutableListOf("one", "two", "three")
with(numbers) {
    println("'with' is called with argument $this")
    println("It contains $size elements")
}

// Output:
'with' is called with argument [one, two, three]
It contains 3 elements
```

You can also use `with` to introduce a helper object whose properties or functions are used for calculating a value.

```Kotlin
val numbers = mutableListOf("one", "two", "three")
val firstAndLast = with(numbers) {
    "The first element is ${first()}," +
    " the last element is ${last()}"
}
println(firstAndLast)
// The first element is one, the last element is three
```

### run

- The **context object** is available as a receiver (`this`).

- The **return value** is the lambda result.

`run` does the same as `with` but it is implemented as an extension function. So like `let`, you can call it on the context object using dot notation.

`run` is useful when your lambda both initializes objects and computes the return value.

```Kotlin
val service = MultiportService("https://example.kotlinlang.org", 80)

val result = service.run {
    port = 8080
    query(prepareRequest() + " to port $port")
}

// the same code written with let() function:
val letResult = service.let {
    it.port = 8080
    it.query(it.prepareRequest() + " to port ${it.port}")
}
```

You can also invoke `run` as a non-extension function. The non-extension variant of `run` has no context object, but it still returns the lambda result. Non-extension `run` lets you execute a block of several statements where an expression is required. 

In code, non-extension run can be read as "**run the code block and compute the result.**"

```Kotlin
val hexNumberRegex = run {
    val digits = "0-9"
    val hexDigits = "A-Fa-f"
    val sign = "+-"

    Regex("[$sign]?[$digits$hexDigits]+")
}

for (match in hexNumberRegex.findAll("+123 -FFFF !%*& 88 XYZ")) {
    println(match.value)
}

// Output:
+123
-FFFF
88
```

### apply

- The **context object** is available as a receiver (`this`).

- The **return value** is the object itself.

As `apply` returns the context object itself, we recommend that you use it for code blocks that don't return a value and that mainly operate on the members of the receiver object. The most common use case for apply is for object configuration. 

Such calls can be read as "**apply the following assignments to the object.**"

```Kotlin
val adam = Person("Adam").apply {
    age = 32
    city = "London"        
}
println(adam)
```

Another use case for `apply` is to include `apply` in multiple call chains for more complex processing.

### also

- The **context object** is available as an argument (`it`).

- The **return value** is the object itself.

`also` is useful for performing some actions that take the context object as an argument. 

Use `also` for actions that need a reference to the object rather than its properties and functions, or when you don't want to shadow the `this` reference from an outer scope.

When you see also in code, you can read it as "**and also do the following with the object.**"

```Kotlin
val numbers = mutableListOf("one", "two", "three")
numbers
    .also { println("The list elements before adding new one: $it") }
    .add("four")
    
// Output: The list elements before adding new one: [one, two, three]
```

## takeIf and takeUnless

These functions let you embed checks of an object's state in call chains.

When called on an object along with a predicate, `takeIf` returns this object if it satisfies the given predicate. Otherwise, it returns `null`. So, `takeIf` is a filtering function for a single object.

`takeUnless` has the opposite logic of `takeIf`. When called on an object along with a predicate, `takeUnless` returns `null` if it satisfies the given predicate. Otherwise, it returns the object.

When using `takeIf` or `takeUnless`, the object is available as a lambda argument (`it`).

```Kotlin
val number = Random.nextInt(100)

val evenOrNull = number.takeIf { it % 2 == 0 }
val oddOrNull = number.takeUnless { it % 2 == 0 }
println("even: $evenOrNull, odd: $oddOrNull")
// even: 42, odd: null
```

<note>

When chaining other functions after takeIf and takeUnless, don't forget to perform a null check or use a safe call (`?.`) because their return value is nullable.
</note>

```Kotlin
val str = "Hello"
val caps = str.takeIf { it.isNotEmpty() }?.uppercase()
val caps = str.takeIf { it.isNotEmpty() }.uppercase()   //compilation error
println(caps)
```

`takeIf` and `takeUnless` are especially useful in combination with scope functions. 

For example, you can chain `takeIf` and `takeUnless` with `let` to run a code block on objects that match the given predicate. To do this, call `takeIf` on the object and then call `let` with a safe call (`?`). 

For objects that don't match the predicate, `takeIf` returns `null` and `let` isn't invoked.

```Kotlin
fun displaySubstringPosition(input: String, sub: String) {
    input.indexOf(sub).takeIf { it >= 0 }?.let {
        println("The substring $sub is found in $input.")
        println("Its start position is $it.")
    }
}

displaySubstringPosition("010000011", "11")
displaySubstringPosition("010000011", "12")

// Output:
The substring 11 is found in 010000011.
Its start position is 7.
```

For comparison, below is an example of how the same function can be written without using `takeIf` or scope functions:

```Kotlin
fun displaySubstringPosition(input: String, sub: String) {
    val index = input.indexOf(sub)
    if (index >= 0) {
        println("The substring $sub is found in $input.")
        println("Its start position is $index.")
    }
}

displaySubstringPosition("010000011", "11")
displaySubstringPosition("010000011", "12")
```