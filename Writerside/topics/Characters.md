# Characters

Characters are represented by the type `Char`. 

Character literals go in single quotes: `'1'`.

<note>
On the JVM, a character stored as primitive type: `char`, represents a 16-bit Unicode character.
</note>

Special characters start from an escaping backslash `\`.

If a value of character variable is a digit, you can explicitly convert it to an `Int` number using the `digitToInt()` function.

<note>
On the JVM, characters are boxed in Java classes when a nullable reference is needed, just like with numbers.
</note>