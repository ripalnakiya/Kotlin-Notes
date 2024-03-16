# This Expressions

To denote the current **receiver**, you use `this` expressions:
- In a member of a class, `this` refers to the current object of that class.
- In an [extension function](Extensions.md) or a [function literal with receiver](Higher-order-functions-and-lambdas.md#function-literals-with-receiver) `this` denotes the **receiver** parameter that is passed on the left-hand side of a dot.

If `this` has no qualifiers, it refers to the **innermost enclosing scope**. To refer to this in other scopes, **label qualifiers** are used:

## Qualified this

To access `this` from an outer scope (a class, extension function, or labeled function literal with receiver) you write `this@label`, where `@label` is a label on the scope this is meant to be from:

```Kotlin
class A { // implicit label @A
    inner class B { // implicit label @B
        fun Int.foo() { // implicit label @foo
            val a = this@A // A's this
            val b = this@B // B's this

            val c = this // foo()'s receiver, an Int
            val c1 = this@foo // foo()'s receiver, an Int

            val funLit = lambda@ fun String.() {
                val d = this // funLit's receiver, a String
                println(d) // Prints the receiver String
            }

            val funLit2 = { s: String ->
                // foo()'s receiver, since enclosing lambda expression
                // doesn't have any receiver
                val d1 = this
            }
        }
    }
}

fun main(){
    fun main() {
        val instanceA = A()
        val instanceB = instanceA.B()
        instanceB.run {
            42.foo() // '42' is the Int receiver
        }
    }
}
```

## Implicit this

When you call a member function on `this`, you can skip the `this.` part. 

If you have a non-member function with the same name, use this with caution because in some cases it can be called instead:

```Kotlin
fun printLine() { println("Top-level function") }

class A {
    fun printLine() { println("Member function") }

    fun invokePrintLine(omitThis: Boolean = false)  { 
        if (omitThis) printLine()
        else this.printLine()
    }
}

A().invokePrintLine() // Member function
A().invokePrintLine(omitThis = true) // Top-level function
```