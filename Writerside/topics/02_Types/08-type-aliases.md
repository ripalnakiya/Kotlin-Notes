# Type aliases

Type aliases provide alternative names for existing types. 

If the type name is too long you can introduce a different shorter name 
and use the new one instead.

It's useful to shorten long generic types. 
For instance, it's often tempting to shrink collection types:

```Kotlin
typealias NodeSet = Set<Network.Node>

typealias FileTable<K> = MutableMap<K, MutableList<File>>
```

You can provide different aliases for function types:

```Kotlin
typealias MyHandler = (Int, String, Any) -> Unit

typealias Predicate<T> = (T) -> Boolean
```
