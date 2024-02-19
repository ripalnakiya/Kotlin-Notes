# Type checks and casts

## is and !is operators

These operators are used to perform a runtime check that identifies whether an object conforms to a given type:

```Kotlin
if (obj is String) {
    print(obj.length)
}

if (obj !is String) { // Same as !(obj is String)
    print("Not a String")
} else {
    print(obj.length)
}
```

## Smart Casts

```Kotlin
fun demo(x: Any) {
    if (x is String) {
        print(x.length) // x is automatically cast to String
    }
}
```

```Kotlin
if (x !is String) return
print(x.length) // x is automatically cast to String
```

```Kotlin
// x is automatically cast to String on the right-hand side of `||`
if (x !is String || x.length == 0) return

// x is automatically cast to String on the right-hand side of `&&`
if (x is String && x.length > 0) {
    print(x.length) // x is automatically cast to String
}
```

## "Unsafe" cast operator

Usually, the cast operator throws an exception if the cast isn't possible. Thus, it's called unsafe.

```Kotlin
val x: String = y as String
```

Note that `null` cannot be cast to `String`, as this type is not nullable. 

If `y` is `null`, the code above throws an exception.

To make code like this correct for `null` values, use the nullable type on the right-hand side of the cast:

```Kotlin
val x: String? = y as String?
```

## "Safe" cast operator

To avoid exceptions, use the safe cast operator `as?`, which returns `null` on failure.

```Kotlin
val x: String? = y as? String
```

<note>
Note that despite the fact that the right-hand side of `as?` is a non-nullable type String, the result of the cast is nullable.
</note>