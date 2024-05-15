# Functional Interfaces

An interface with only one abstract method is called a **functional interface**, or a **Single Abstract Method (SAM) interface**. 

The functional interface can have **several non-abstract members** but only one abstract member.

To declare a functional interface in Kotlin, use the `fun` modifier.

```Kotlin
fun interface KRunnable {
   fun invoke()
}
```

## SAM Conversions

For functional interfaces, you can use SAM conversions that help make your code more concise and readable by using lambda expressions.

Instead of creating a class that implements a functional interface manually, you can use a lambda expression. 

With a SAM conversion, Kotlin can convert any lambda expression whose signature matches the signature of the interface's single method into the code, which dynamically instantiates the interface implementation.

```Kotlin
fun interface IntPredicate {
   fun accept(i: Int): Boolean
}

// Create an instance of the interface using an anonymous object
val isEven = object : IntPredicate {
   override fun accept(i: Int): Boolean {
       return i % 2 == 0
   }
}

// Creating an instance using lambda
val isEven = IntPredicate { it % 2 == 0 }

fun main() {
   println("Is 7 even? - ${isEven.accept(7)}")
}
```

## Migration from an interface with constructor function to a functional interface

Interface with constructor function:

```Kotlin
interface Printer {
    fun print()
}

fun Printer(block: () -> Unit): Printer = object : Printer { override fun print() = block() }

// OR

fun Printer(block: () -> Unit): Printer {
    return object : Printer {
        override fun print() {
            block()
        }
    }
}

fun main() {
    Printer { println("Hello World") }.print()
}
```

With callable references to functional interface constructors enabled, this code can be replaced with just a functional interface declaration:

```Kotlin
fun interface Printer {
    fun print()
}

fun main() {
    Printer { println("Hello World") }.print()
}
```

When you declare a functional interface (an interface with a single abstract method), the constructor for the interface is created implicitly. 

This means you can create instances of the functional interface without explicitly defining a constructor function.

With the implicit constructor, you can use callable references (`::Printer`) to refer to the constructor of the functional interface. 

This allows you to pass the constructor reference around as a first-class citizen in your code.

For example, if `documentsStorage` has a method `addPrinter` that takes a `Printer` constructor reference as an argument, you can now pass `::Printer` to it directly:

```Kotlin
documentsStorage.addPrinter(::Printer)
```

## Functional interfaces vs. type aliases

You can also simply rewrite the above using a type alias for a functional type:

```Kotlin
typealias IntPredicate = (i: Int) -> Boolean

val isEven: IntPredicate = { it % 2 == 0 }

fun main() {
   println("Is 7 even? - ${isEven(7)}")
}
```

However, functional interfaces and type aliases serve different purposes. 

Type aliases are just names for existing types – they don't create a new type, while functional interfaces do. 

Type aliases can have only one member, while functional interfaces can have multiple non-abstract members and one abstract member. Functional interfaces can also implement and extend other interfaces.

Functional interfaces are more flexible and provide more capabilities than type aliases, but they can be more costly both syntactically and at runtime because they can require conversions to a specific interface. 

When you choose which one to use in your code, consider your needs:

- If your API needs to accept a function (any function) with some specific parameter and return types – use a simple functional type or define a type alias to give a shorter name to the corresponding functional type.

- If your API accepts a more complex entity than a function – for example, it has non-trivial contracts and/or operations on it that can't be expressed in a functional type's signature – declare a separate functional interface for it.

## Examples

```Kotlin
fun interface Printer {
    fun print(first: Int, second: Int) : Unit
}

fun main() {
    val first = 5
    val second = 10
    
    val printer = Printer { first, second ->  println("Hello World ${first*second} ") }
    printer.print(first, second)
    
    // OR
    
    Printer { first, second ->  println("Hello World ${first*second} ") }.print(first, second)
}
```