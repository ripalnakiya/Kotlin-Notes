# Reflection

Reflection is a set of language and library features that allows you to introspect the structure of your program at runtime.

Functions and properties are first-class citizens in Kotlin, and the ability to introspect them (for example, learning the name or the type of a property or function at runtime) is essential when using a functional or reactive style.

## JVM dependency

Gradle:

```Kotlin
dependencies {
    implementation(kotlin("reflect"))
}
```

TODO:

[Kotlin Docs](https://kotlinlang.org/docs/reflection.html)