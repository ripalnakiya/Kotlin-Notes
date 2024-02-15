# Booleans

The type `Boolean` represents boolean objects that can have two values: `true` and `false`. 

`Boolean` has a nullable counterpart declared as `Boolean?`.

> On the JVM, booleans stored as the primitive boolean type typically use 8 bits.
{style="tip"}

Built-in operations on booleans include:
- `||` – disjunction (logical OR)
- `&&` – conjunction (logical AND)
- `!` – negation (logical NOT)

The `||` and `&&` operators work lazily, which means:
- If the first operand is true, the `||` operator does not evaluate the second operand.
- If the first operand is false, the `&&` operator does not evaluate the second operand.

<note>
On the JVM, nullable references to boolean objects are boxed in Java classes, just like with numbers.
</note>