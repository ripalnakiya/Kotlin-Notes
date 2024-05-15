# Extensions

Kotlin provides the ability to extend a class or an interface with new functionality without having to inherit from the class or use design patterns such as **Decorator**. 

This is done via special declarations called **extensions**.

For example, you can write new functions for a class or an interface from a third-party library that you can't modify. Such functions can be called in the usual way, as if they were methods of the original class. 

This mechanism is called an **extension function**. There are also **extension properties** that let you define new properties for existing classes.

## Extension functions

To declare an extension function, prefix its name with a **receiver type**, which refers to the type being extended.

The following adds a `swap` function to `MutableList<Int>`:

```Kotlin
fun MutableList<Int>.swap(index1: Int, index2: Int) {
    val tmp = this[index1] // 'this' corresponds to the list
    this[index1] = this[index2]
    this[index2] = tmp
}
```

The `this` keyword inside an extension function corresponds to the **receiver object** (the one that is passed before the dot). 

Now, you can call such a function on any `MutableList<Int>`:

```Kotlin
val list = mutableListOf(1, 2, 3)
list.swap(0, 2) // 'this' inside 'swap()' will hold the value of 'list'
```

This function makes sense for any `MutableList<T>`, and you can make it generic:

```Kotlin
fun <T> MutableList<T>.swap(index1: Int, index2: Int) {
    val tmp = this[index1] // 'this' corresponds to the list
    this[index1] = this[index2]
    this[index2] = tmp
}
```

You need to declare the generic type parameter before the function name to make it available in the receiver type expression.

## Extensions are resolved statically

Extensions do not actually modify the classes they extend. By defining an extension, you are not inserting new members into a class, only making new functions callable with the dot-notation on variables of this type.

Extension functions are **dispatched statically**. 

So which extension function is to be called is already known at compile time based on the receiver type.

```Kotlin
open class Shape
class Rectangle: Shape()

fun Shape.getName() = "Shape"
fun Rectangle.getName() = "Rectangle"

fun printClassName(s: Shape) {
    println(s.getName())
}

printClassName(Rectangle())

// Output:
Shape
```

This example prints **Shape**, because the extension function called depends only on the declared type of the parameter `s`, which is the `Shape` class.

```Kotlin
open class Shape
class Rectangle: Shape()

fun Shape.getName() = "Shape"
fun Rectangle.getName() = "Rectangle"

fun printClassName(s: Shape) {
    println(s.getName())
}

fun printClassName(r: Rectangle) {
    println(r.getName())
}

printClassName(Rectangle())

// Output:
Rectangle
```

If a class has a member function, and an extension function is defined which has the same receiver type, the same name, and is applicable to given arguments, the **member always wins**.

```Kotlin
class Example {
    fun printFunctionType() { println("Class method") }
}

fun Example.printFunctionType() { println("Extension function") }

Example().printFunctionType()

// Output:
Class method
```

However, it's perfectly OK for extension functions to overload member functions that have the same name but a different signature:

```Kotlin
class Example {
    fun printFunctionType() { println("Class method") }
}

fun Example.printFunctionType(i: Int) { println("Extension function #$i") }

Example().printFunctionType(1)

// Output:
Extension function #1
```

## Nullable receiver

Note that extensions can be defined with a nullable receiver type. 

These extensions can be called on an object variable even if its value is null. 

If the receiver is `null`, then `this` is also `null`. 

So when defining an extension with a nullable receiver type, we recommend performing a `this == null` check inside the function body to avoid compiler errors.

You can call `toString()` in Kotlin without checking for null, as the check already happens inside the extension function:

```Kotlin
fun Any?.toString(): String {
    if (this == null) return "null"         // this can be removed
    // After the null check, 'this' is autocast to a non-nullable type
    // so the toString() below resolves to the
    // member function of the `Any` class
    return toString()
}
```

## Extension Properties

```Kotlin
val <T> List<T>.lastIndex: Int
    get() = size - 1
```

<note>
Since extensions do not actually insert members into classes, there's no efficient way for an extension property to have a backing field.

This is why **initializers are not allowed** for extension properties. 

Their behavior can only be defined by explicitly providing getters/setters.
</note>

```Kotlin
val House.number = 1 
// error: initializers are not allowed for extension properties
```

## Companion object extensions

Just like regular members of the companion object, they can be called using only the class name as the qualifier:

```Kotlin
class MyClass {
    companion object { }  // will be called "Companion"
}

fun MyClass.Companion.printCompanion() { println("companion") }

fun main() {
    MyClass.printCompanion()
}
```

## Scope of extensions

In most cases, you define extensions on the top level, directly under packages:

```Kotlin
package org.example.declarations

fun List<String>.getLongestString() { /*...*/}
```

To use an extension outside its declaring package, import it at the call site:


```Kotlin
package org.example.usage

import org.example.declarations.getLongestString

fun main() {
    val list = listOf("red", "green", "blue")
    list.getLongestString()
}
```

## Declaring extensions as members

When you define an extension function inside another class, the extension function can access members of both the class, it's extending class and the class in which it's defined. 

These two classes are referred to as **implicit receivers**.

- The class in which the extension function is defined is called the **dispatch receiver**.
- The class being extended is called the **extension receiver**.

```Kotlin
class Host(val hostname: String) {
    fun printHostname() { print(hostname) }
}

class Connection(val host: Host, val port: Int) {
    fun printPort() { print(port) }

    fun Host.printConnectionString() {
        printHostname()   // calls Host.printHostname()
        print(":")
        printPort()   // calls Connection.printPort()
    }

    fun connect() {
        host.printConnectionString()   // calls the extension function
    }
}

fun main() {
    Connection(Host("example.com"), 443).connect()
    
    Host("example.com").printConnectionString()  
    // error, the extension function is unavailable outside Connection
    // because printConnectionString() needs methods of Connection
}

// Output:
example.com:443
```

In the event of a name conflict between the members of a dispatch receiver and an extension receiver, the extension receiver takes precedence. 

To refer to the member of the dispatch receiver, you can use the qualified this syntax.

```Kotlin
class Connection {
    fun Host.getConnectionString() {
        toString()         // calls Host.toString()
        this@Connection.toString()  // calls Connection.toString()
    }
}
```

When you declare an extension function inside a class and mark it as `open` (meaning it can be overridden) and then override it in a subclass, the behavior of the function is determined by the type of the object it's called on, not the type of the class where the function is declared.

- The dispatch receiver type determines whether the function can be overridden.
- The extension receiver type determines the behavior of the function when it's called.

```Kotlin
open class Base { }

class Derived : Base() { }

open class BaseCaller {
    open fun Base.printFunctionInfo() {
        println("Base extension function in BaseCaller")
    }

    open fun Derived.printFunctionInfo() {
        println("Derived extension function in BaseCaller")
    }

    fun call(b: Base) {
        b.printFunctionInfo()   // call the extension function
    }
}

class DerivedCaller: BaseCaller() {
    override fun Base.printFunctionInfo() {
        println("Base extension function in DerivedCaller")
    }

    override fun Derived.printFunctionInfo() {
        println("Derived extension function in DerivedCaller")
    }
}

fun main() {
    BaseCaller().call(Base())   
    // "Base extension function in BaseCaller"
    
    DerivedCaller().call(Base())  
    // "Base extension function in DerivedCaller"
    // dispatch receiver is resolved virtually
    
    DerivedCaller().call(Derived())  
    // "Base extension function in DerivedCaller"
    // extension receiver is resolved statically
}
```

This means that the dispatch of such functions is virtual with regard to the dispatch receiver type (runtime polymorphism) , but static with regard to the extension receiver type (compile-time polymorphism) .

## Visibility

Extensions utilize the same visibility modifiers as regular functions declared in the same scope would.

- An extension declared at the top level of a file has access to the other `private` top-level declarations in the same file.
- If an extension is declared outside its receiver type, it cannot access the receiver's `private` or `protected` members.
