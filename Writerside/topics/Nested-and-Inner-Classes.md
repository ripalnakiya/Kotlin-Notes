# Nested and Inner Classes

```Kotlin
class Outer {
    private val bar: Int = 1
    class Nested {
        fun foo() = 2
    }
}

val demo = Outer.Nested().foo()

// Output:
2
```

All combinations of classes and interfaces are possible, You can nest
- interfaces in classes
- classes in interfaces
- interfaces in interfaces.

```Kotlin
interface OuterInterface {
    class InnerClass
    interface InnerInterface
}

class OuterClass {
    class InnerClass
    interface InnerInterface
}
```

## Inner classes

A nested class marked as `inner` can access the members of its outer class. 

Inner classes carry a reference to an object of an outer class:

```Kotlin
class Outer {
    private val bar: Int = 1
    inner class Inner {
        fun foo() = bar
    }
}

val demo = Outer().Inner().foo()

// Output:
1
```

## Anonymous inner classes

Anonymous inner class instances are created using an object expression:

```Kotlin
window.addMouseListener(
    // Creates an instance of an anonymous inner class that extends `MouseAdapter`
    object : MouseAdapter() {
        override fun mouseClicked(e: MouseEvent) { ... }
        override fun mouseEntered(e: MouseEvent) { ... }
    }
)
```











