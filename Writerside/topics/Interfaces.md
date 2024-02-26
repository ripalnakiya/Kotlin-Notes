# Interfaces

<show-structure depth="2"/>

Interfaces in Kotlin can contain declarations of abstract methods, as well as method implementations. 

What makes them different from abstract classes is that interfaces cannot store state. 

They can have properties, but these need to be abstract or provide accessor implementations.

```Kotlin
interface MyInterface {
    fun bar()
    fun foo() {
      // optional body
    }
}
```

## Implementing interfaces

```Kotlin
class Child : MyInterface {
    override fun bar() {
        // body
    }
}
```

## Properties in interfaces

You can declare properties in interfaces. 

A property declared in an interface can either be abstract or provide implementations for accessors. 

Properties declared in interfaces can't have backing fields, and therefore accessors declared in interfaces can't reference them:

```Kotlin
interface MyInterface {
    val prop: Int // abstract

    val propertyWithImplementation: String
        get() = "foo"

    fun foo() {
        print(prop)
    }
}

class Child : MyInterface {
    override val prop: Int = 29
}
```

## Interfaces Inheritance

An interface can derive from other interfaces, meaning it can both provide implementations for their members and declare new functions and properties. 

Quite naturally, classes implementing such an interface are only required to define the missing implementations:

```Kotlin
interface Named {
    val name: String
}

interface Person : Named {
    val firstName: String
    val lastName: String

    override val name: String get() = "$firstName $lastName"
}

data class Employee(
    // implementing 'name' is not required
    override val firstName: String,
    override val lastName: String,
    val position: Position
) : Person
```

## Resolving overriding conflicts

When you declare many types in your supertype list, you may inherit more than one implementation of the same method:

### Scenario 1

We **don't need** to override the methods which have a default implementation in the interface

```Kotlin
interface First {
    fun foo() { println("First") }
    fun bar() { println("A") }
}

class Test : First {
    fun test() {
        println("test")
    }
}
```

### Scenario 2

We **need** to override the methods which doesn't have a default implementation in the interface

```Kotlin
interface Second {
    fun foo() { println("Second") }
    fun bar()
}

class Test : Second {
    override fun bar() {
        println("T")
    }
}
```

### Scenario 3

We **can** override the methods which have a default implementation in the interface

```Kotlin
interface First {
    fun foo() { println("First") }
    fun bar() { println("A") }
}

class Test : First {
    override fun foo() {
        println("Test")
    }
    
    override fun bar() {
        println("T")
    }
}
```

### Scenario 4

When a class inherits from more than one interface and those interfaces have the same method (with different implementations)

Then we need to provide the implementation of those methods(methods that are present in different interfaces) in our class

```Kotlin
interface First {
    fun foo() { println("First") }
    fun bar() { println("A") }
}

interface Second {
    fun foo() { println("Second") }
}

class Test : First, Second {
    override fun foo() {
        println("Test")
    }
}
```

```Kotlin
interface First {
    fun foo() { println("First") }
    fun bar() { println("A") }
}

interface Second {
    fun foo() { println("Second") }
    fun bar()
}

class Test : First, Second {
    override fun foo() {
        println("Test")
    }

    override fun bar() {
        println("T")
    }
}
```

### Scenario 5

```Kotlin
interface First {
    fun foo() { println("First") }
    fun bar() { println("A") }
}

interface Second {
    fun foo() { println("Second") }
    fun bar()
}

class Test : First, Second {
    override fun foo() {
        super<First>.foo()
        super<Second>.foo()
    }

    override fun bar() {
        super<First>.bar()
    }
}
```