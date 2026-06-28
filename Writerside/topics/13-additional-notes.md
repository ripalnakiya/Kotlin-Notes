# Additional Notes

## Lists

| `List`                           | `MutableList`                       |
|----------------------------------|-------------------------------------|
| Read-only interface              | Read and write interface            |
| Can only access elements         | Can add, remove, or modify elements |
| Safer to share between functions | Can be changed accidentally         |

```kotlin
val fruits: List<String> = listOf("Apple", "Banana", "Orange")

println(fruits[0])      // Apple
println(fruits.size)    // 3

fruits.contains("Apple")
fruits.first()
fruits.last()
fruits.filter { it.startsWith("A") }

fruits.add("Mango")      // ❌ Error
fruits.remove("Apple")   // ❌ Error
fruits[0] = "Grapes"     // ❌ Error
```

```kotlin
val fruits = mutableListOf("Apple", "Banana", "Orange")

fruits.add("Mango")
fruits.remove("Banana")
fruits[0] = "Grapes"

println(fruits)
```

### Common Misunderstanding

Many people think `List` is immutable. It isn't necessarily.

`List` is only a read-only view.

```kotlin
val mutable = mutableListOf("A", "B")
val readOnly: List<String> = mutable

println(readOnly)   // [A, B]

mutable.add("C")

println(readOnly)   // [A, B, C]
```

Even though `readOnly` is a `List`, 
it still sees the change because both variables point to the same underlying object.

- `List` = "You have permission to look."
- `MutableList` = "You have permission to edit."
