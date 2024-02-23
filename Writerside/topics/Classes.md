# Classes

```Kotlin
class Person { /*...*/ }
```

The class declaration consists of the class name, the class header (specifying its type parameters, the primary constructor, and some other things), and the class body surrounded by curly braces.

```Kotlin
class Box<T>(var item: T) {
    fun getItem(): T {
        return item
    }
}
```

> `Box` is a generic class with a type parameter `T` specified in the class header. The class also has a primary constructor that takes an initial value of type `T` and assigns it to a property `item`.
> 
{style="note"}

Both the header and the body are optional; if the class has no body, the curly braces can be omitted.

```Kotlin
class Empty
```

## Constructors

A class in Kotlin has a primary constructor and possibly one or more secondary constructors. 

The primary constructor is declared in the class header, and it goes after the class name and optional type parameters.

```Kotlin
class Person constructor(firstName: String) { /*...*/ }
```

If the primary constructor does not have any annotations or visibility modifiers, the constructor keyword can be omitted:

```Kotlin
class Person(firstName: String) { /*...*/ }
```

The primary constructor initializes a class instance and its properties in the class header. 

The class header can't contain any runnable code. If you want to run some code during object creation, use initializer blocks inside the class body. 

Initializer blocks are declared with the `init` keyword followed by curly braces. Write any code that you want to run within the curly braces.

During the initialization of an instance, the initializer blocks are executed in the same order as they appear in the class body, interleaved with the property initializers:

```Kotlin
class InitOrderDemo(name: String) {
    val firstProperty = "First property: $name".also(::println)
    
    init {
        println("First initializer block that prints $name")
    }
    
    val secondProperty = "Second property: ${name.length}".also(::println)
    
    init {
        println("Second initializer block that prints ${name.length}")
    }
}

// Output:
First property: hello
First initializer block that prints hello
Second property: 5
Second initializer block that prints 5
```

Primary constructor parameters can be used in the initializer blocks. They can also be used in property initializers declared in the class body:

```Kotlin
class Customer(name: String) {
    val customerKey = name.uppercase()
}
```

Kotlin has a concise syntax for declaring properties and initializing them from the primary constructor:

```Kotlin
class Person(
    val firstName: String, 
    val lastName: String, 
    var age: Int,       // trailing comma
)
```

Such declarations can also include default values of the class properties:

```Kotlin
class Person(
    val firstName: String, 
    val lastName: String, 
    var isEmployed: Boolean = true,     // trailing comma
)
```

Much like regular properties, properties declared in the primary constructor can be mutable (var) or read-only (val).

If the constructor has annotations or visibility modifiers, the constructor keyword is required and the modifiers go before it:

```Kotlin
class Customer public @Inject constructor(name: String) { /*...*/ }
```

## Secondary Constructors

A class can also declare secondary constructors, which are prefixed with constructor:

```Kotlin
class Person(val pets: MutableList<Pet> = mutableListOf())

class Pet {
    constructor(owner: Person) {
        owner.pets.add(this) 
        // adds this pet to the list of its owner's pets
    }
}
```

If the class has a primary constructor, each secondary constructor needs to delegate to the primary constructor, either directly or indirectly through another secondary constructor(s). 

Delegation to another constructor of the same class is done using the `this` keyword:

```Kotlin
class Person(val name: String) {
    val children: MutableList<Person> = mutableListOf()
    constructor(name: String, parent: Person) : this(name) {
        parent.children.add(this)
    }
}
```

Code in **initializer blocks** effectively becomes part of the primary constructor. 

Delegation to the primary constructor happens at the moment of access to the first statement of a secondary constructor, so the code in **all initializer blocks** and **property initializers** is executed before the body of the secondary constructor.

Even if the class has no primary constructor, the delegation still happens implicitly, and the initializer blocks are still executed:

```Kotlin
class Constructors {
    init {
        println("Init block")
    }

    constructor(i: Int) {
        println("Constructor $i")
    }
}

// Output:
Init block
Constructor 1
```

If a non-abstract class does not declare any constructors (primary or secondary), it will have a generated primary constructor with no arguments. The visibility of the constructor will be public.

If you don't want your class to have a public constructor, declare an empty primary constructor with non-default visibility:

```Kotlin
class DontCreateMe private constructor() { /*...*/ }
```

<note>
On the JVM, if all the primary constructor parameters have default values, the compiler will generate an additional parameterless constructor which will use the default values.
</note>

## Creating instances of classes

To create an instance of a class, call the constructor as if it were a regular function. You can assign the created instance to a variable:

```Kotlin
val invoice = Invoice()

val customer = Customer("Joe Smith")
```

## Class members

Classes can contain:
- [Constructors and initializer blocks](#constructors)
- Functions
- Properties
- Nested and inner classes
- Object declarations

## Abstract classes

A class may be declared `abstract`, along with some or all of its members. 

An abstract member does not have an implementation in its class. 

You don't need to annotate abstract classes or functions with `open`.

```Kotlin
abstract class Polygon {
    abstract fun draw()
}

class Rectangle : Polygon() {
    override fun draw() {
        // draw the rectangle
    }
}
```

You can override a non-abstract `open` member with an abstract one.

```Kotlin
open class Polygon {
    open fun draw() {
        // some default polygon drawing method
    }
}

abstract class WildShape : Polygon() {
    // Classes that inherit WildShape need to provide their own
    // draw method instead of using the default on Polygon
    abstract override fun draw()
}
```
