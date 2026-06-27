# Strings
<show-structure depth="2"/>

> On the JVM, an object of `String` type uses approximately 2 bytes per character.

Generally, a string value is a sequence of characters in double quotes (`"`):

```Kotlin
val str = "abcd 123"
```

Elements of a string are characters that you can access via the indexing operation: `s[i]`. 

You can iterate over these characters with a `for` loop:
```Kotlin
for (c in str) {
    println(c)
}
```

**Strings are immutable.** 

Once you initialize a string, you can't change its value or assign a new value to it. 

All operations that transform strings return their results in a new String object, 
leaving the original string unchanged:

```Kotlin
val str = "abcd"

// Creates and prints a new String object
println(str.uppercase())        
// ABCD

// The original string remains the same
println(str)                    
// abcd
```

To concatenate strings, use the `+` operator. 

This also works for concatenating strings with values of other types, 
as long as the first element in the expression is a string:

```Kotlin
val s = "abc" + 1
println(s + "def")              // abc1def    
```

> In most cases using string templates or multiline strings 
> is preferable to string concatenation.
{style="note"}


## String literals

Kotlin has two types of string literals:

### 1. Escaped strings

Escaped strings can contain escaped characters.

```Kotlin
val s = "Hello, world!\n"
```

Escaping is done in the conventional way, with a backslash (`\`).

### 2. Multiline strings

Multiline strings can contain newlines and arbitrary text, delimited by a triple quote (`"""`).

It contains **no escaping characters** and **can contain newlines and any other characters**:

```Kotlin
val text = """
    for (c in "Android")
        print(c)
"""
```

It doesn't support backslash escaping.

To remove leading whitespace from multiline strings, use the `trimMargin()` function:

```Kotlin
val text = """
    |Tell me and I forget.
    |Teach me and I remember.
    |Involve me and I learn.
    |(Benjamin Franklin)
    """.trimMargin()
```

By default, a pipe symbol `|` is used as margin prefix, 
but you can choose another character and pass it as a parameter, 
like `trimMargin(">")`.

## String templates

String literals may contain template expressions – 
pieces of code that are evaluated and whose results are concatenated into a string.

When a template expression is processed, Kotlin automatically calls 
the `.toString()` function on the expression's result to convert it into a string.

A template expression starts with a dollar sign (`$`) and 
consists of either a variable name:

```Kotlin
val i = 10
println("i = $i") 
// i = 10

val letters = listOf("a","b","c","d","e")
println("Letters: $letters") 
// Letters: [a, b, c, d, e]
```

or an expression in curly braces:

```Kotlin
val s = "abc"
println("$s.length is ${s.length}") 
// abc.length is 3
```

## String formatting

To format a string to your specific requirements, use the `String.format()` function.

The String.format() function accepts **a format string** and **one or more arguments**.

The **format string** contains one placeholder (indicated by `%`) for a given argument, 
followed by format specifiers. 

Format specifiers are formatting instructions for the respective argument, 
consisting of flags, width, precision, and conversion type.
Collectively, format specifiers shape the output's formatting. 

Common format specifiers include `%d` for integers, `%f` for floating-point numbers, 
and `%s` for strings. 

You can also use the `argument_index$` syntax to reference 
the same argument multiple times within the format string in different formats.

```Kotlin
// Formats an integer, adding leading zeroes to reach a length of seven characters
val integerNumber = String.format("%07d", 31416)
println(integerNumber)
// 0031416

// Formats a floating-point number to display with a + sign and four decimal places
val floatNumber = String.format("%+.4f", 3.141592)
println(floatNumber)
// +3.1416

// Formats two strings to uppercase, each taking one placeholder
val helloString = String.format("%S %S", "hello", "world")
println(helloString)
// HELLO WORLD

// Formats a negative number to be enclosed in parentheses, then repeats the same number in a different format (without parentheses) using `argument_index$`.
val negativeNumberInParentheses = String.format("%(d means %1\$d", -31416)
println(negativeNumberInParentheses)
//(31416) means -31416
```

The `String.format()` function provides similar functionality to string templates. 

However, the `String.format()` function is more versatile 
because there are more formatting options available.
