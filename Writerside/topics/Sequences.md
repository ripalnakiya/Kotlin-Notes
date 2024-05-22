# Sequences

Along with collections, the Kotlin standard library contains another type – sequences (`Sequence<T>`). 

Unlike collections, sequences don't contain elements, they produce them while iterating. Sequences offer the same functions as `Iterable` but implement another approach to multi-step collection processing.

When the processing of an `Iterable` includes multiple steps, they are executed eagerly: each processing step completes and returns its result – an intermediate collection. The following step executes on this collection. In turn, multi-step processing of sequences is executed lazily when possible: actual computing happens only when the result of the whole processing chain is requested.

