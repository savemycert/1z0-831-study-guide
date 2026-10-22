# Domain 6: Processing Data Using Streams and Lambda Expressions (9%)

Two objectives. The first covers how a pipeline runs, primitive streams, sorting, the `java.util.function` interfaces, lambda syntax and method references. The second covers reduction, collectors (grouping, partitioning, `toMap`, `teeing`), `flatMap`/`mapMulti`/`concat`, and gatherers, which are standard since Java SE 24. Expect exact-output questions, "which declaration compiles" questions about types, and runtime exceptions such as stream reuse. Oracle publishes no domain weights; the percentage here is an estimate.

## 6.1 Object and primitive streams with lambda expressions

- **Pipeline** = source + intermediate operations + **one** terminal operation.
  - Nothing runs without a terminal operation, so prints inside lambdas do not appear.
  - Elements travel one at a time through every stage.
  - Short-circuiting operations (`limit`, `findFirst`, `anyMatch`) stop pulling early.
- **Single use:** a second operation on the same stream object throws `IllegalStateException` at that call. It is never a compile error.

| Factory                               | Produces                                                                            |
| ------------------------------------- | ----------------------------------------------------------------------------------- |
| `IntStream.range(a, b)`               | a to b−1                                                                            |
| `IntStream.rangeClosed(a, b)`         | a to b                                                                              |
| `Stream.iterate(seed, next)`          | Infinite                                                                            |
| `Stream.iterate(seed, hasNext, next)` | Like a `for` loop; the predicate is checked before every element, the seed included |
| `Stream.generate(supplier)`           | Infinite; the supplier is called per element                                        |

- **Infinite streams** need a bound the pipeline actually reaches. A filter that never passes anything before `limit` hangs silently. `sorted()` and `count()` on an infinite stream never return.
- **Slicing** applies in the order written:
  - `distinct` keeps first occurrences.
  - `skip(n)` discards; `limit(n)` short-circuits.
  - **`takeWhile`** ends at the first false.
  - **`dropWhile`** stops _testing_ at the first false and passes everything after.
  - `filter` tests every element.
- `map` changes the element type for every later stage. `sorted()` after `limit` sorts only the survivors.

| Terminal on an empty stream | Result           |
| --------------------------- | ---------------- |
| `allMatch`, `noneMatch`     | `true`           |
| `anyMatch`                  | `false`          |
| `findFirst`, `findAny`      | `Optional.empty` |

- An `Optional` prints as `Optional[a]`. Read it with `get`, `orElse` or `ifPresent`.
- **Primitive streams** (`IntStream`, `LongStream`, `DoubleStream`) add `sum`, `average`, `max`, `min` and `summaryStatistics`. **`Stream<Integer>` has no `sum()`**, which is a compile error.
- **Conversions:** `mapToInt`/`mapToLong`/`mapToDouble`, `mapToObj`, `boxed()`, `asLongStream`/`asDoubleStream`.
  - `IntStream.map` must return an `int`, so `map(i -> i * 1.5)` fails.
  - Arithmetic follows the element type at that stage.

| IntStream terminal | Returns          | Empty                  |
| ------------------ | ---------------- | ---------------------- |
| `sum()`            | `int`            | 0                      |
| `count()`          | `long`           | 0                      |
| `max()`/`min()`    | `OptionalInt`    | `OptionalInt.empty`    |
| `average()`        | `OptionalDouble` | `OptionalDouble.empty` |

- `int m = IntStream.of(1, 2).max();` does not compile. Use `getAsInt()` or `orElse`.
- **Sorting:** `sorted()` needs `Comparable` elements. Otherwise it **compiles and throws `ClassCastException`** when the terminal runs.
  - Natural String order puts uppercase before lowercase.
  - `reversed()` reverses every key before it.
  - To reverse one key, pass `Comparator.reverseOrder()` as the second argument of `comparing`/`thenComparing`.
  - A lambda in `comparing(...)` that has another method chained on loses its target type, so its parameter becomes `Object`. A method reference, an explicit parameter type or a type witness fixes it.
  - Stream sorting of an ordered stream is stable.

| Interface                                | Method                      | Shape            |
| ---------------------------------------- | --------------------------- | ---------------- |
| `Supplier<T>`                            | `get`                       | () → T           |
| `Consumer<T>` / `BiConsumer`             | `accept`                    | T → void         |
| `Predicate<T>` / `BiPredicate`           | `test`                      | T → boolean      |
| `Function<T,R>` / `BiFunction`           | `apply`                     | T → R            |
| `UnaryOperator<T>` / `BinaryOperator<T>` | `apply`                     | T → T, (T,T) → T |
| `IntFunction<R>`                         | `apply(int)`                | int → R          |
| `ToIntFunction<T>`                       | `applyAsInt`                | T → int          |
| `IntUnaryOperator`                       | `applyAsInt`                | int → int        |
| `IntSupplier` / `BooleanSupplier`        | `getAsInt` / `getAsBoolean` | () → value       |

- **Naming rule:** `IntFunction` _takes_ an int; `ToIntFunction` _returns_ one. Methods returning a primitive are `applyAsX`/`getAsX`. Every predicate uses `test`.
- `IntStream.filter` takes an `IntPredicate`, so the parameter is a primitive `int`.
- A `Supplier` does not cache; each `get()` runs the body again.
- **Lambda syntax:**
  - One implicit parameter can drop its parentheses.
  - Parameters are all implicit, all typed, or all `var`, never mixed.
  - A block body needs `return` for a value.
  - A void target accepts only statement expressions (`() -> sb.append("!")` is fine; `() -> 1 + 2` is not).
  - A lambda parameter cannot reuse an in-scope local's name.
- **Effectively final:** any reassignment anywhere, even after the lambda, breaks every capture. Assigning once on each path of an if/else is fine. Mutating a captured object is fine. Fields are exempt.

| Method reference kind | Example              | Lambda                      |
| --------------------- | -------------------- | --------------------------- |
| Static                | `Integer::parseInt`  | `s -> Integer.parseInt(s)`  |
| Bound                 | `greeting::contains` | `s -> greeting.contains(s)` |
| Unbound               | `String::endsWith`   | `(s, t) -> s.endsWith(t)`   |
| Constructor           | `TreeSet::new`       | `c -> new TreeSet<>(c)`     |
| Array constructor     | `int[]::new`         | `n -> new int[n]`           |

- For an unbound reference the first argument is the receiver, so `String::isBlank` cannot be a `Supplier`.
- A constructor reference picks the constructor that matches the functional shape, or fails to compile.
- **Composition:**
  - `f.andThen(g)` = g(f(x)); `f.compose(g)` = f(g(x)).
  - `p.negate().or(q)` is (not p) or q.
  - `Predicate.not(p)` is the static form of negate.
  - Composing a `UnaryOperator` returns a `Function`.

📖 Full lesson: [Java Streams, Lambdas and Primitive Streams (1Z0-831)](https://www.savemycert.com/revision/oracle-java-se-developer/java-streams-lambda-expressions-primitive-streams/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide)

## 6.2 Reduction, grouping, partitioning and gatherers on streams

| `reduce` overload                         | Returns       | Empty stream     |
| ----------------------------------------- | ------------- | ---------------- |
| `reduce(identity, BinaryOperator<T>)`     | `T`           | The identity     |
| `reduce(BinaryOperator<T>)`               | `Optional<T>` | `Optional.empty` |
| `reduce(identity, accumulator, combiner)` | `U`           | The identity     |

- Only the three-argument form can change type.
- A `null` final result from `reduce(op)` throws `NullPointerException`.
- `opt.stream().toList()` gives a zero- or one-element list.
- **Parallel-safe reduction** needs a true identity, an associative operation, and a combiner compatible with the accumulator.
  - Identity examples: 0 for sum, 1 for product, `Integer.MAX_VALUE` for min, `Integer.MIN_VALUE` for max, `""` for concatenation.
  - A wrong identity is applied once per chunk. `reduce(0, Integer::min)` returns 0 even sequentially.
- **`collect(supplier, accumulator, combiner)`:** each chunk gets its own container; the combiner merges the **second** container into the **first**. A broken combiner only shows on split streams. `Collectors.toCollection(LinkedList::new)` is the Collector form.
- Primitive streams have only the three-function `collect`. Use `boxed()` before `Collectors.joining()`.
- **Empty-stream results:** `count` is 0, `sum` is 0, `average` is an empty `OptionalDouble`, `max`/`min` are empty optionals. `summaryStatistics()` never throws: count 0, min `Integer.MAX_VALUE`, max `Integer.MIN_VALUE`, average 0.0.
- `Stream.min(byLen.reversed())` returns the longest element.
- **`joining(delim, prefix, suffix)`:**
  - The delimiter goes only between elements.
  - The prefix and suffix are always added, even when empty.
  - Elements must be `CharSequence`.
- **`toMap`:**
  - A duplicate key with no merge function throws `IllegalStateException`.
  - The merge function is `(existing, incoming)`.
  - The four-argument form adds a map supplier.
- **`teeing(c1, c2, merger)`:** the result is whatever the merger returns. `counting()` is `Long`, so `Integer / Long` is integer division.
- `collectingAndThen(collector, finisher)` transforms a result. `reducing(identity, op)` is reduce as a collector.

| Downstream collector       | Value type        |
| -------------------------- | ----------------- |
| default (`toList`)         | `List<T>`         |
| `counting()`               | `Long`            |
| `summingInt`               | `Integer`         |
| `averagingInt/Long/Double` | `Double`          |
| `mapping(f, downstream)`   | Downstream's type |
| `minBy` / `maxBy`          | `Optional<T>`     |

- A mismatched declaration (for example `Map<K, Integer>` for `counting()`) does not compile.
- **`groupingBy`:**
  - Keys exist only for values that occur.
  - A `null` key throws `NullPointerException`.
  - The map type is unspecified unless you pass a factory such as `TreeMap::new`.
  - Nest a second `groupingBy` as the downstream for multilevel maps.
- **`partitioningBy`** always has **both** `false` and `true` keys. An empty side holds the downstream's zero result (`[]`, `0`, `""`, `Optional.empty`), never null. It takes no map factory.
- **`flatMap(Function<T, Stream<R>>)`** flattens in order; a `null` stream counts as empty.
- **`mapMulti(BiConsumer<T, Consumer<R>>)`** emits once per `accept` and often needs a type witness: `.<Integer>mapMulti(...)`.
- **`Stream.concat(a, b)`:**
  - Lazy, in argument order.
  - Both arguments must be the same stream kind.
  - A consumed or already-linked stream throws `IllegalStateException`.

| Gatherer (`Gatherers.*`) | Emits                                | Edge case                                                                  |
| ------------------------ | ------------------------------------ | -------------------------------------------------------------------------- |
| `windowFixed(n)`         | Lists of n                           | Short last window                                                          |
| `windowSliding(n)`       | Overlapping lists, k − n + 1 of them | A stream shorter than n gives one short window; an empty stream gives none |
| `fold(init, f)`          | One final state                      | Stays in the stream                                                        |
| `scan(init, f)`          | Each running state                   | The initial value is not emitted                                           |
| `mapConcurrent(max, f)`  | Mapped values                        | Runs on virtual threads but keeps encounter order                          |

- `fold` and `scan` are sequential, even in a parallel pipeline. Gatherers compose with `andThen`.
- **Custom gatherers:** `Gatherer.ofSequential(initializer, integrator, finisher)` or `Gatherer.of(...)` (with a combiner for parallel use).
  - The integrator `(state, element, downstream) -> boolean` pushes zero or more elements. Returning `false` short-circuits.
  - Push before testing and the triggering element is already emitted.
  - The finisher runs once at the end, even after a `false`, and pushes any leftovers.
  - Type parameters are `Gatherer<T, A, R>`: input, state, output.

📖 Full lesson: [Java Stream Reduction, Grouping, Partitioning and Gatherers (1Z0-831)](https://www.savemycert.com/revision/oracle-java-se-developer/java-stream-reduction-grouping-partitioning-gatherers/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide)

[← Back to the study guide](../README.md)
