# Type checks and casts
<show-structure depth="2"/>

In Kotlin, you can do two things with types at runtime: 
check whether an object is a specific type, or convert it to another type. 

Type **checks** help you confirm the kind of object you're dealing with, 
while type **casts** attempt to convert the object to another type.

## Checks with `is` and `!is` operators

Use the `is` operator (or `!is` to negate it) 
to check if an object matches a type at runtime:

```Kotlin
fun main() {
    val input: Any = "Hello, Kotlin"

    if (input is String) {
        println("Message length: ${input.length}")
        // Message length: 13
    }

    if (input !is String) { // Same as !(input is String)
        println("Input is not a valid message")
    } else {
        println("Processing message: ${input.length} characters")
        // Processing message: 13 characters
    }
}
```

You can also use `is` and `!is` operators to check if an object matches a subtype:

```Kotlin
fun handleAnimal(animal: Animal) {
    println("Handling animal: ${animal.name}")
    animal.speak()

    // Use is operator to check for subtypes
    if (animal is Dog) {
        println("Special care instructions: This is a dog.")
    } else if (animal is Cat) {
        println("Special care instructions: This is a cat.")
    }
}
```

This example uses the `is` operator to check if the `Animal` class instance 
has subtype `Dog` or `Cat` to print the relevant care instructions.

> To identify the type of an object at runtime, see Reflection.

## Type casts

To convert the type of an object in Kotlin to another type is called **casting**.

In some cases, the compiler automatically casts objects for you. 
This is called smart-casting.

If you need to explicitly cast a type, use `as?` or `as` cast operators.

## Smart casts

The compiler tracks the type checks and explicit casts for immutable values 
and inserts implicit (safe) casts automatically:

```Kotlin
fun logMessage(data: Any) {
    // data is automatically cast to String
    if (data is String) {
        println("Received text: ${data.length} characters")
    }
}

fun main() {
    logMessage("Server started")
    // Received text: 14 characters
    logMessage(404)
    // (Doesn't print anything)
}
```

The compiler is even smart enough to know 
that a cast is safe if a negative check leads to a return:

```Kotlin
fun logMessage(data: Any) {
    // data is automatically cast to String
    if (data !is String) return

    println("Received text: ${data.length} characters")
}

fun main() {
    logMessage("User signed in")
    // Received text: 14 characters
    logMessage(true)
}
```

### Control flow

Smart casts work not only for `if` conditional expressions, but also for `when` expressions:

```Kotlin
fun processInput(data: Any) {
    when (data) {
        // data is automatically cast to Int
        is Int -> println("Log: Assigned new ID ${data + 1}")
        // data is automatically cast to String
        is String -> println("Log: Received message \"$data\"")
        // data is automatically cast to IntArray
        is IntArray -> println("Log: Processed scores, total = ${data.sum()}")
    }
}

fun main() {
    processInput(1001)
    // Log: Assigned new ID 1002
    processInput("System rebooted")
    // Log: Received message "System rebooted"
    processInput(intArrayOf(10, 20, 30))
    // Log: Processed scores, total = 60
}
```

### Logical operators

The compiler can perform smart casts on the right-hand side of `&&` or `||` operators 
if there is a type check (regular or negative) on the left-hand side:

```Kotlin
// x is automatically cast to String on the right-hand side of `||`
if (x !is String || x.length == 0) return

// x is automatically cast to String on the right-hand side of `&&`
if (x is String && x.length > 0) {
    print(x.length) // x is automatically cast to String
}
```

### Exception Handling

Smart cast information is passed on to `catch` and `finally` blocks. 

This makes your code safer as the compiler tracks whether your object has a nullable type.

```Kotlin
fun testString() {
    var stringInput: String? = null
    // stringInput is smart-cast to String type
    stringInput = ""
    try {
        // The compiler knows that stringInput isn't null
        println(stringInput.length)
        // 0

        // The compiler rejects previous smart cast information for stringInput.
        // Now stringInput has the String? type.
        stringInput = null

        // Trigger an exception
        if (2 > 1) throw Exception()
        stringInput = ""
    } catch (exception: Exception) {
        // The compiler knows stringInput can be null
        // so stringInput stays nullable.
        println(stringInput?.length)
        // null
    }
}
```

## `as` and `as?` cast operators

Kotlin has two cast operators: `as` and `as?`. 

You can use both to cast, but they have different behaviors.

If a cast fails with the `as` operator, a `ClassCastException` is thrown at runtime. 
That's why it's also called the **unsafe** operator. 

You can use as when casting to a non-null type:

```Kotlin
fun main() {
    val rawInput: Any = "user-1234"

    // Casts to String successfully
    val userId = rawInput as String
    println("Logging in user with ID: $userId")
    // Logging in user with ID: user-1234

    // Triggers ClassCastException
    val wrongCast = rawInput as Int
    println("wrongCast contains: $wrongCast")
    // Exception in thread "main" java.lang.ClassCastException
}
```

If you use the `as?` operator instead, and the cast fails, the operator returns `null`. 

That's why it's also called the **safe** operator:

```Kotlin
fun main() {
    val rawInput: Any = "user-1234"

    // Casts to String successfully
    val userId = rawInput as? String
    println("Logging in user with ID: $userId")
    // Logging in user with ID: user-1234

    // Assigns a null value to wrongCast
    val wrongCast = rawInput as? Int
    println("wrongCast contains: $wrongCast")
    // wrongCast contains: null
}
```

To cast a nullable type safely, use the `as?` operator 
to prevent triggering a `ClassCastException` if the cast fails.

**You can use `as` with a nullable type.** 
This allows the result to be `null`, 
but it still throws a `ClassCastException` if the cast is unsuccessful. 

For this reason, `as?` is the safer option:

```Kotlin
fun main() {
    val config: Map<String, Any?> = mapOf(
        "username" to "kodee",
        "alias" to null,
        "loginAttempts" to 3
    )

    // Unsafely casts to a nullable String
    val username: String? = config["username"] as String?
    println("Username: $username")
    // Username: kodee

    // Unsafely casts a null value to a nullable String
    val alias: String? = config["alias"] as String?
    println("Alias: $alias")
    // Alias: null

    // Fails to cast to nullable String and throws ClassCastException
    val unsafeAttempts: String? = config["loginAttempts"] as String?
    println("Login attempts (unsafe): $unsafeAttempts")
    // Exception in thread "main" java.lang.ClassCastException

    // Fails to cast to nullable String and returns null
    val safeAttempts: String? = config["loginAttempts"] as? String
    println("Login attempts (safe): $safeAttempts")
    // Login attempts (safe): null
}
```

### Up and downcasting

[Visit Kotlin Docs](https://kotlinlang.org/docs/typecasts.html#up-and-downcasting)
