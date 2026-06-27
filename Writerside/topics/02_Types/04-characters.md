# Characters
<show-structure depth="2"/>

Characters are represented by the type `Char`. 

Character literals go in single quotes: `'1'`.

> On the JVM, a character stored as primitive type: `char`, 
> represents a 16-bit Unicode character.

Special characters start from an escaping backslash `\`.

- The following escape sequences are supported:
  - `\t` – tab
  - `\b` – backspace 
  - `\n` – new line (LF)
  - `\r` – carriage return (CR)
  - `\'` – single quotation mark 
  - `\"` – double quotation mark 
  - `\\` – backslash 
  - `\$` – dollar sign


If a value of character variable is a digit, 
you can explicitly convert it to an `Int` number using the `digitToInt()` function.

> On the JVM, characters are boxed in Java classes when a nullable reference is needed, 
> just like with numbers.
