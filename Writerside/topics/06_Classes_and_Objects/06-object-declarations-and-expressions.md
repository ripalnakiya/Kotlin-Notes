# Object declarations and expressions
<show-structure depth="2"/>

In Kotlin, objects allow you to define a class and create an instance of it in a single step.

This is useful when you need either a reusable singleton instance or a one-time object.

To handle these scenarios, Kotlin provides two key approaches: 
**object declarations** for creating singletons 
and **object expressions** for creating anonymous, one-time objects.

> A singleton ensures that a class has only one instance and provides a global point of access to it.
>
{style="tip"}

Object declarations and object expressions are best used for scenarios when:

* **Using singletons for shared resources:** You need to ensure that 
  only one instance of a class exists throughout the application.
  For example, managing a database connection pool.
* **Creating factory methods:** You need a convenient way to create instances efficiently.
  Companion objects allow you to define class-level functions and properties tied to a class, 
  simplifying the creation and management of these instances.
* **Modifying existing class behavior temporarily:** You want to modify the behavior of an existing class 
  without the need to create a new subclass.
  For example, adding temporary functionality to an object for a specific operation.
* **Type-safe design is required:** You require one-time implementations of 
  interfaces or abstract classes using object expressions.
  This can be useful for scenarios like a button click handler.

## Object declarations

You can create single instances of objects in Kotlin using object declarations, 
which always have a name following the `object` keyword.

This allows you to define a class and create an instance of it in a single step, 
which is useful for implementing singletons:

```Kotlin
// Declares a Singleton object to manage data providers
object DataProviderManager {
    private val providers = mutableListOf<DataProvider>()

    // Registers a new data provider
    fun registerDataProvider(provider: DataProvider) {
        providers.add(provider)
    }

    // Retrieves all registered data providers
    val allDataProviders: Collection<DataProvider> 
        get() = providers
}

// Example data provider interface
interface DataProvider {
    fun provideData(): String
}

// Example data provider implementation
class ExampleDataProvider : DataProvider {
    override fun provideData(): String {
        return "Example data"
    }
}

fun main() {
    // Creates an instance of ExampleDataProvider
    val exampleProvider = ExampleDataProvider()

    // To refer to the object, use its name directly
    DataProviderManager.registerDataProvider(exampleProvider)

    // Retrieves and prints all data providers
    println(DataProviderManager.allDataProviders.map { it.provideData() })
    // [Example data]
}
```

> The initialization of an object declaration is thread-safe and done on first access.
>
{style="tip"}

To refer to the `object`, use its name directly:

```kotlin
DataProviderManager.registerDataProvider(exampleProvider)
```

Object declarations can also have supertypes,
similar to how [anonymous objects can inherit from existing classes or implement interfaces](#inherit-anonymous-objects-from-supertypes):

```kotlin
object DefaultListener : MouseAdapter() {
    override fun mouseClicked(e: MouseEvent) { ... }

    override fun mouseEntered(e: MouseEvent) { ... }
}
```

Like variable declarations, object declarations are not expressions, 
so they cannot be used on the right-hand side of an assignment statement:

```kotlin
// Syntax error: An object expression cannot bind a name.
val myObject = object MySingleton {
    val name = "Singleton"
}
```
Object declarations cannot be local, which means they cannot be nested directly inside a function.
However, they can be nested within other object declarations or non-inner classes.

### Data objects

When printing a plain object declaration in Kotlin, 
the string representation contains both its name and the hash of the `object`:

```Kotlin
object MyObject

fun main() {
    println(MyObject) 
    // MyObject@1f32e575
}
```

However, by marking an object declaration with the `data` modifier,
you can instruct the compiler to return the actual name of the object when calling `toString()`, 
the same way it works for data classes:

```kotlin
data object MyDataObject {
    val number: Int = 3
}

fun main() {
    println(MyDataObject) 
    // MyDataObject
}
```

Additionally, the compiler generates several functions for your `data object`:

* `toString()` returns the name of the data object
* `equals()`/`hashCode()` enables equality checks and hash-based collections

> You can't provide a custom `equals` or `hashCode` implementation for a `data object`.
>
{style="note"}

#### Differences between data objects and data classes

While `data object` and `data class` declarations 
are often used together and have some similarities, 
there are some functions that are not generated for a `data object`:

- No `copy()` function. Because a data object declaration is 
  intended to be used as singleton objects, no `copy()` function is generated.
  - The singleton pattern restricts the instantiation of a class to a single instance, 
    which would be violated by allowing copies of the instance to be created.

- No `componentN()` function. Unlike a data class, a data object does not have any data properties. 
  Since attempting to destructure such an object without data properties would not make sense, 
  no `componentN()` functions are generated.

#### Using data objects with sealed hierarchies

Data object declarations are particularly useful for sealed hierarchies 
like sealed classes or sealed interfaces, since they allow you 
to maintain symmetry with any data classes you may have defined alongside the object.

In this example, declaring `EndOfFile` as a `data object` instead of a plain `object` means 
that it will get the `toString()` function without the need to override it manually:

```Kotlin
sealed interface ReadResult
data class Number(val number: Int) : ReadResult
data class Text(val text: String) : ReadResult
data object EndOfFile : ReadResult

fun main() {
    println(Number(7)) // Number(number=7)
    println(EndOfFile) // EndOfFile
}
```

### Companion objects

**Companion objects** allow you to define class-level functions and properties. 
This makes it easy to create factory methods, hold constants, and access shared utilities.

An object declaration inside a class can be marked with the `companion` keyword:

```Kotlin
class MyClass {
    companion object Factory {
        fun create(): MyClass = MyClass()
    }
}
```

Members of the `companion object` can be called simply by using the class name as the qualifier:

```Kotlin
class User(val name: String) {
    // Defines a companion object that acts as a factory for creating User instances
    companion object Factory {
        fun create(name: String): User = User(name)
    }
}

fun main(){
    // Calls the companion object's factory method using the class name as the qualifier. 
    // Creates a new User instance
    val userInstance = User.create("John Doe")
    println(userInstance.name)
    // John Doe
}
```

The name of the `companion object` can be omitted, in which case the name `Companion` is used:

```Kotlin
class User(val name: String) {
    // Defines a companion object without a name
    companion object { }
}

// Accesses the companion object
val companionUser = User.Companion
```

Class members can access `private` members of their corresponding `companion object`:

```kotlin
class User(val name: String) {
    companion object {
        private val defaultGreeting = "Hello"
    }

    fun sayHi() {
        println(defaultGreeting)
    }
}
User("Nick").sayHi()
// Hello
```

When a class name is used by itself, it acts as a reference to the companion object of the class,
regardless of whether the companion object is named or not:

```kotlin
//sampleStart
class User1 {
    // Defines a named companion object
    companion object Named {
        fun show(): String = "User1's Named Companion Object"
    }
}

// References the companion object of User1 using the class name
val reference1 = User1

class User2 {
    // Defines an unnamed companion object
    companion object {
        fun show(): String = "User2's Companion Object"
    }
}

// References the companion object of User2 using the class name
val reference2 = User2
//sampleEnd

fun main() {
    // Calls the show() function from the companion object of User1
    println(reference1.show()) 
    // User1's Named Companion Object

    // Calls the show() function from the companion object of User2
    println(reference2.show()) 
    // User2's Companion Object
}
```

Although members of companion objects in Kotlin look like static members from other languages,
they are actually instance members of the companion object, meaning they belong to the object itself.
This allows companion objects to implement interfaces:

```kotlin
interface Factory<T> {
    fun create(name: String): T
}

class User(val name: String) {
    // Defines a companion object that implements the Factory interface
    companion object : Factory<User> {
        override fun create(name: String): User = User(name)
    }
}

fun main() {
    // Uses the companion object as a Factory
    val userFactory: Factory<User> = User
    val newUser = userFactory.create("Example User")
    println(newUser.name)
    // Example User
}
```

## Object Expressions

Object expressions declare a class and create an instance of that class, but without naming either of them.

These classes are useful for one-time use. 
They can either be created from scratch, inherit from existing classes, or implement interfaces. 

Instances of these classes are also called **anonymous objects** because 
they are defined by an expression, not a name.

### Create anonymous objects from scratch

Object expressions start with the `object` keyword.

If the object doesn't extend any classes or implement interfaces, 
you can define an object's members directly inside curly braces after the `object` keyword:

```kotlin
fun main() {
    val helloWorld = object {
        val hello = "Hello"
        val world = "World"
        // Object expressions extend the Any class, which already has a toString() function,
        // so it must be overridden
        override fun toString() = "$hello $world"
    }

    print(helloWorld)
    // Hello World
}
```

### Inherit anonymous objects from supertypes

To create an anonymous object that inherits from some type (or types), 
specify this type after `object` and a 

colon `:`. Then implement or override the members of this class 
as if you were inheriting from it:

```kotlin
window.addMouseListener(object : MouseAdapter() {
    override fun mouseClicked(e: MouseEvent) { /*...*/ }

    override fun mouseEntered(e: MouseEvent) { /*...*/ }
})
```

If a supertype has a constructor, pass the appropriate constructor parameters to it.
Multiple supertypes can be specified, separated by commas, after the colon:

```kotlin
// Creates an open class BankAccount with a balance property
open class BankAccount(initialBalance: Int) {
    open val balance: Int = initialBalance
}

// Defines an interface Transaction with an execute() function
interface Transaction {
    fun execute()
}

// A function to perform a special transaction on a BankAccount
fun specialTransaction(account: BankAccount) {
    // Creates an anonymous object that inherits from the BankAccount class and implements the Transaction interface
    // The balance of the provided account is passed to the BankAccount superclass constructor
    val temporaryAccount = object : BankAccount(account.balance), Transaction {

        override val balance = account.balance + 500  // Temporary bonus

        // Implements the execute() function from the Transaction interface
        override fun execute() {
            println("Executing special transaction. New balance is $balance.")
        }
    }
    // Executes the transaction
    temporaryAccount.execute()
}

fun main() {
    // Creates a BankAccount with an initial balance of 1000
    val myAccount = BankAccount(1000)
    // Performs a special transaction on the created account
    specialTransaction(myAccount)
    // Executing special transaction. New balance is 1500.
}
```

### Use anonymous objects as return and value types

When you return an anonymous object from a local or `private` function or property,
all the members of that anonymous object are accessible through that function or property:

```kotlin
class UserPreferences {
    private fun getPreferences() = object {
        val theme: String = "Dark"
        val fontSize: Int = 14
    }

    fun printPreferences() {
        val preferences = getPreferences()
        println("Theme: ${preferences.theme}, Font Size: ${preferences.fontSize}")
    }
}

fun main() {
    val userPreferences = UserPreferences()
    userPreferences.printPreferences()
    // Theme: Dark, Font Size: 14
}
```

This allows you to return an anonymous object with specific properties,
offering a simple way to encapsulate data or behavior without creating a separate class.

If a function or property that returns an anonymous object 
has `public`, `protected`, or `internal` visibility, its actual type is:

* `Any` if the anonymous object doesn't have a declared supertype.
* The declared supertype of the anonymous object, if there is exactly one such type.
* The explicitly declared type if there is more than one declared supertype.

In all these cases, members added in the anonymous object are not accessible. Overridden members are accessible if they
are declared in the actual type of the function or property. For example:

```kotlin
interface Notification {
    // Declares notifyUser() in the Notification interface
    fun notifyUser()
}

interface DetailedNotification

class NotificationManager {
    // The return type is Any. The message property is not accessible.
    // When the return type is Any, only members of the Any class are accessible.
    fun getNotification() = object {
        val message: String = "General notification"
    }

    // The return type is Notification because the anonymous object implements only one interface
    // The notifyUser() function is accessible because it is part of the Notification interface
    // The message property is not accessible because it is not declared in the Notification interface
    fun getEmailNotification() = object : Notification {
        override fun notifyUser() {
            println("Sending email notification")
        }
        val message: String = "You've got mail!"
    }

    // The return type is DetailedNotification. The notifyUser() function and the message property are not accessible
    // Only members declared in the DetailedNotification interface are accessible
    fun getDetailedNotification(): DetailedNotification = object : Notification, DetailedNotification {
        override fun notifyUser() {
            println("Sending detailed notification")
        }
        val message: String = "Detailed message content"
    }
}

fun main() {
    // This produces no output
    val notificationManager = NotificationManager()

    // The message property is not accessible here because the return type is Any
    // This produces no output
    val notification = notificationManager.getNotification()

    // The notifyUser() function is accessible
    // The message property is not accessible here because the return type is Notification
    val emailNotification = notificationManager.getEmailNotification()
    emailNotification.notifyUser()
    // Sending email notification

    // The notifyUser() function and message property are not accessible here because the return type is DetailedNotification
    // This produces no output
    val detailedNotification = notificationManager.getDetailedNotification()
}
```

### Access variables from anonymous objects

Code within the body of object expressions can access variables from the enclosing scope:

```kotlin
import java.awt.event.MouseAdapter
import java.awt.event.MouseEvent

fun countClicks(window: JComponent) {
    var clickCount = 0
    var enterCount = 0

    // MouseAdapter provides default implementations for mouse event functions
    // Simulates MouseAdapter handling mouse events
    window.addMouseListener(object : MouseAdapter() {
        override fun mouseClicked(e: MouseEvent) {
            clickCount++
        }

        override fun mouseEntered(e: MouseEvent) {
            enterCount++
        }
    })
    // The clickCount and enterCount variables are accessible within the object expression
}
```

## Behavior difference between object declarations and expressions

There are differences in the initialization behavior between object declarations and object expressions:

* Object expressions are executed (and initialized) **immediately**, where they are used.
* Object declarations are initialized **lazily**, when accessed for the first time.
* A companion object is initialized when the corresponding class is loaded (resolved) 
  that matches the semantics of a Java static initializer.
