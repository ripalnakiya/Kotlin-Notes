# Strings

<show-structure depth="2"/>

<note>
On the JVM, an object of `String` type uses approximately 2 bytes per character.
</note>

```Kotlin
val str = "abcd 123"
```

Elements of a string are characters that you can access via the indexing operation: s[i]. You can iterate over these characters with a for loop:

```Kotlin
for (c in str) {
    println(c)
}
```

**Strings are immutable.** 

Once you initialize a string, you can't change its value or assign a new value to it. 

All operations that transform strings return their results in a new String object, leaving the original string unchanged:

```Kotlin
val str = "abcd"

// Creates and prints a new String object
println(str.uppercase())        // ABCD

// The original string remains the same
println(str)                    // abcd
```

To concatenate strings, use the + operator. 

This also works for concatenating strings with values of other types, as long as the first element in the expression is a string:

```Kotlin
val s = "abc" + 1
println(s + "def")              // abc1def    
```

## String literals

Kotlin has two types of string literals:
- Escaped strings
- Multiline strings

### Escaped strings

Escaped strings can contain escaped characters.

```Kotlin
val s = "Hello, world!\n"
```

Escaping is done in the conventional way, with a backslash (`\`).

### Multiline strings

Multiline strings can contain newlines and arbitrary text, delimited by a triple quote (`"""`).

It contains **no escaping characters** and **can contain newlines and any other characters**:

```Kotlin
val text = """
    for (c in "Android")
        print(c)
"""
```

It doesn't support backslash escaping.

## String templates

String literals may contain template expressions – pieces of code that are evaluated and whose results are concatenated into a string.

When a template expression is processed, Kotlin automatically calls the .toString() function on the expression's result to convert it into a string.

```Kotlin
val i = 10
println("i = $i")               // i = 10

val letters = listOf("a","b","c","d","e")
println("Letters: $letters")    // Letters: [a, b, c, d, e]

val str = "abc"
println("$str.length is ${str.length}")     // abc.length is 3
```
