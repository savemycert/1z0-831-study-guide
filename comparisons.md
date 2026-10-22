# 1Z0-831 Commonly Confused Services and Concepts

Side-by-side comparisons of the 1Z0-831 services and ideas that exam questions most often set against each other. Each table ends with the one thing to remember.

- [Domain 1: Handling Date, Time, Text, Numeric and Boolean Values](#domain-1-handling-date-time-text-numeric-and-boolean-values)
- [Domain 2: Implementing Program Flow Control Using Decision and Looping Constructs](#domain-2-implementing-program-flow-control-using-decision-and-looping-constructs)
- [Domain 3: Applying Object-Oriented Principles in Java Programs](#domain-3-applying-object-oriented-principles-in-java-programs)
- [Domain 4: Implementing Exception Handling in Java Applications](#domain-4-implementing-exception-handling-in-java-applications)
- [Domain 5: Using Arrays and Collections to Store and Retrieve Data](#domain-5-using-arrays-and-collections-to-store-and-retrieve-data)
- [Domain 6: Processing Data Using Streams and Lambda Expressions](#domain-6-processing-data-using-streams-and-lambda-expressions)
- [Domain 7: Packaging and Deploying Java Code](#domain-7-packaging-and-deploying-java-code)
- [Domain 8: Implementing Multithreading for Concurrent Code Execution](#domain-8-implementing-multithreading-for-concurrent-code-execution)
- [Domain 9: Performing Input and Output Operations Using the Java I/O API](#domain-9-performing-input-and-output-operations-using-the-java-io-api)
- [Domain 10: Developing Applications with Localization Support](#domain-10-developing-applications-with-localization-support)

## Domain 1: Handling Date, Time, Text, Numeric and Boolean Values

### == vs equals() (Strings and wrappers)

|                      | `==`                                                         | `equals()`                                                                     |
| -------------------- | ------------------------------------------------------------ | ------------------------------------------------------------------------------ |
| Two Strings          | Same object? True for literals, folded constants, `intern()` | Same characters                                                                |
| Two wrappers         | Same object? Only reliable in the `Integer` cache range      | Same wrapper type **and** same value                                           |
| Wrapper vs primitive | Unboxes, then compares numbers                               | Primitive is boxed to its own type first: `Long.valueOf(9).equals(9)` is false |
| Two StringBuilders   | Same object                                                  | Also same object (not overridden)                                              |

**Remember:** `==` on objects is identity. For content, use `equals()` on String and wrappers, and `compareTo()` or `contentEquals()` for StringBuilder.

### String vs StringBuilder

|                  | String                                 | StringBuilder                                        |
| ---------------- | -------------------------------------- | ---------------------------------------------------- |
| Mutability       | Immutable; methods return a new String | Mutable; methods change it and return `this`         |
| Unassigned call  | Result is lost                         | Change is kept                                       |
| `replace`        | `replace(old, new)` by content         | `replace(start, end, str)` by range                  |
| Out-of-range end | `substring` throws                     | `delete`/`replace` clamp the end; `substring` throws |
| `equals()`       | Content                                | Identity                                             |

**Remember:** a String call you don't assign does nothing; a StringBuilder call always counts, and every alias sees it.

### trim() vs strip(), isEmpty() vs isBlank()

|             | `trim()`               | `strip()`                            | `isEmpty()`     | `isBlank()`               |
| ----------- | ---------------------- | ------------------------------------ | --------------- | ------------------------- |
| White space | Chars `<= U+0020` only | Unicode white space (`isWhitespace`) | n/a             | Unicode white space       |
| Result      | New String             | New String                           | `length() == 0` | Empty or only white space |

**Remember:** `strip` and `isBlank` are the Unicode-aware pair; `trim` stops at U+0020, and `isEmpty` only checks length.

### Duration vs Period

|                 | Duration                                    | Period                                       |
| --------------- | ------------------------------------------- | -------------------------------------------- |
| Stores          | Seconds + nanos                             | Years, months, days                          |
| Prints as       | `PTnHnMnS` (no days: `ofDays(3)` → `PT72H`) | `PnYnMnD` (no weeks: `ofWeeks(2)` → `P14D`)  |
| "One day" means | Exactly 24 hours                            | One calendar day (23 to 25 hours across DST) |
| Works on        | Types with time units                       | Types with date units                        |

**Remember:** Duration is elapsed time, Period is calendar time. Adding the wrong one to a type compiles and throws at run time.

### plusDays(1) vs plusHours(24) on a ZonedDateTime

|                  | `plusDays(1)`, `Period.ofDays(1)`, `ChronoUnit.DAYS` | `plusHours(24)`, `Duration.ofDays(1)` |
| ---------------- | ---------------------------------------------------- | ------------------------------------- |
| Works on         | The local date-time                                  | The instant                           |
| Keeps            | Wall-clock time                                      | Exact elapsed time                    |
| Across fall back | Same clock time, 25 real hours later                 | One hour earlier on the clock         |

**Remember:** date-based amounts keep the clock; time-based amounts keep the stopwatch.

### withZoneSameInstant vs withZoneSameLocal

|         | `withZoneSameInstant(zone)` | `withZoneSameLocal(zone)`    |
| ------- | --------------------------- | ---------------------------- |
| Keeps   | The moment                  | The wall-clock time          |
| Changes | The wall-clock time         | The moment                   |
| Use for | "What time is it there?"    | "Same time, different place" |

**Remember:** same instant means converting a time; same local means moving a meeting.

### Math.round vs Math.rint vs floor/ceil

|          | `round`                            | `rint`              | `floor` / `ceil`       |
| -------- | ---------------------------------- | ------------------- | ---------------------- |
| Returns  | `long` (double arg), `int` (float) | `double`            | `double`               |
| Ties     | Toward positive infinity           | To the even integer | n/a (always down / up) |
| `-3.5` → | -3                                 | -4.0                | -4.0 / -3.0            |

**Remember:** only `round` returns an integer type, and its ties always go up.

## Domain 2: Implementing Program Flow Control Using Decision and Looping Constructs

### Colon labels vs arrow labels

|                  | `case X:`                                  | `case X ->`                                |
| ---------------- | ------------------------------------------ | ------------------------------------------ |
| Fall-through     | Yes, until `break`                         | No                                         |
| Allowed body     | Any statements                             | One expression statement, block or `throw` |
| In an expression | Needs `yield`                              | Expression value, or `yield` in a block    |
| Mixing           | Not allowed with arrows in the same switch | Not allowed with colons in the same switch |

**Remember:** colon groups keep running until something stops them; arrow arms always stop by themselves.

### Switch statement vs switch expression

|                             | Switch statement                        | Switch expression                                                                          |
| --------------------------- | --------------------------------------- | ------------------------------------------------------------------------------------------ |
| Produces a value            | No                                      | Yes; the enclosing statement ends with `;`                                                 |
| Must be exhaustive          | Only with pattern labels or `case null` | Always                                                                                     |
| `return`/`break`/`continue` | Allowed                                 | Compile error; leave only with `yield` or `throw`                                          |
| No matching label           | Nothing runs                            | Cannot happen at compile time; `MatchException` if a new constant or subtype appears later |

**Remember:** an expression must cover every input and can only be left by `yield` or `throw`.

### default vs case null

|            | `default`                              | `case null`                                     |
| ---------- | -------------------------------------- | ----------------------------------------------- |
| Matches    | Any non-null value with no other match | Only `null`                                     |
| Without it | No match runs nothing (statement)      | A `null` selector throws `NullPointerException` |
| Combined   | `case null, default ->` (null first)   | Same                                            |

**Remember:** `default` never catches `null`.

### Guarded vs unguarded pattern

|                           | Unguarded (`case String s`)                   | Guarded (`case String s when ...`) |
| ------------------------- | --------------------------------------------- | ---------------------------------- |
| Counts for exhaustiveness | Yes                                           | No                                 |
| Dominates later labels    | Same type, subtypes and constants of its type | Nothing                            |
| Safe position             | After the more specific labels                | Before the unguarded label         |

**Remember:** specific and guarded first, broad and unguarded last, `default` at the end.

### while vs do-while

|                                 | `while`         | `do-while`           |
| ------------------------------- | --------------- | -------------------- |
| Test                            | Before the body | After the body       |
| Minimum runs                    | Zero            | One                  |
| Syntax                          | No trailing `;` | Ends `while (cond);` |
| Body variables in the condition | n/a             | Out of scope         |

**Remember:** a do-while body runs once even when the condition starts false.

### break vs continue (inside a switch inside a loop)

|                        | `break`                | `continue`                               |
| ---------------------- | ---------------------- | ---------------------------------------- |
| Unlabelled             | Leaves only the switch | Skips to the loop's next iteration       |
| Labelled               | Any labelled statement | Labelled loops only ("not a loop label") |
| In a switch expression | Compile error          | Compile error                            |

**Remember:** `break` stops at the nearest switch; `continue` goes straight past it to the loop.

## Domain 3: Applying Object-Oriented Principles in Java Programs

### Overloading vs overriding

|                           | Overloading                                   | Overriding                                                 |
| ------------------------- | --------------------------------------------- | ---------------------------------------------------------- |
| Signature                 | Same name, different parameter types or order | Same name and parameter types                              |
| Chosen                    | At compile time, from declared types          | At run time, from the object's class                       |
| Return type               | Free                                          | Same, or covariant for references                          |
| Access, exceptions        | Free                                          | Same or wider access; no new or broader checked exceptions |
| Applies to static methods | Yes                                           | No (static methods are hidden)                             |

**Remember:** a subclass method with a different parameter type is a new overload, so a superclass reference never sees it. Put `@Override` on every intended override.

### Static nested class vs inner class

|                        | Static nested class                | Inner (member) class                               |
| ---------------------- | ---------------------------------- | -------------------------------------------------- |
| Enclosing instance     | None                               | Exactly one, fixed for life                        |
| Creation               | `new Outer.Nested()`               | `outerRef.new Inner()`                             |
| Outer instance members | Only through an explicit reference | Directly, or as `Outer.this.x`                     |
| Keeps outer alive      | No                                 | Yes, while reachable (if it uses the outer object) |

**Remember:** `new Outer.Inner()` names the type but supplies no enclosing object, so it fails from static code.

### Anonymous class vs lambda

|                              | Anonymous class            | Lambda                     |
| ---------------------------- | -------------------------- | -------------------------- |
| `this`                       | The anonymous object       | The enclosing instance     |
| Parameter named like a local | Allowed (shadows)          | Compile error              |
| Captured locals              | Final or effectively final | Final or effectively final |
| Target                       | Any class or interface     | Functional interfaces only |

**Remember:** a lambda opens no new scope for names; an anonymous class does.

### Collections.unmodifiableList vs List.copyOf vs new ArrayList

|                        | `unmodifiableList(src)` | `List.copyOf(src)` | `new ArrayList<>(src)` |
| ---------------------- | ----------------------- | ------------------ | ---------------------- |
| Copies elements        | No (a view)             | Yes                | Yes                    |
| Callers can modify     | No                      | No                 | Yes                    |
| Follows changes to src | Yes                     | No                 | No                     |
| Null elements          | Allowed                 | NPE                | Allowed                |

**Remember:** only `List.copyOf` closes both leaks in one call.

### Abstract class vs interface

|                   | Abstract class                            | Interface                                             |
| ----------------- | ----------------------------------------- | ----------------------------------------------------- |
| Fields            | Any instance or static fields             | `public static final` constants only                  |
| Constructors      | Yes, run through `super(...)`             | None                                                  |
| Method modifiers  | Any, including `protected` and `final`    | Public (abstract, default, static) or private helpers |
| Inheritance       | One superclass                            | Any number, via `implements` or `extends`             |
| Conflict priority | A class method beats an interface default | Defaults lose to class methods                        |

**Remember:** an interface has no instance state and no constructors; a superclass method, even an abstract one, wins over a default.

### final vs sealed vs non-sealed

|                 | `final`                      | `sealed`                                         | `non-sealed`            |
| --------------- | ---------------------------- | ------------------------------------------------ | ----------------------- |
| Subclasses      | None                         | Only those listed in `permits`                   | Anyone                  |
| Used on         | Any class                    | Classes and interfaces                           | Permitted subtypes only |
| Records / enums | Records are implicitly final | Enums with constant bodies are implicitly sealed | n/a                     |

**Remember:** every permitted subtype must pick one of the three; leaving it off is a compile error.

### Explicit canonical vs compact record constructor

|                  | Explicit canonical               | Compact                                                |
| ---------------- | -------------------------------- | ------------------------------------------------------ |
| Parameter list   | Written out, matching the header | Omitted                                                |
| Field assignment | You must assign every field      | Automatic at the end; `this.x = ...` fails             |
| Typical use      | Full control                     | Validation and normalisation by reassigning parameters |

**Remember:** in a compact constructor, change the parameter, not the field.

### name() vs toString() on an enum

|               | `name()`                | `toString()`               |
| ------------- | ----------------------- | -------------------------- |
| Default value | The declared identifier | Same as `name()`           |
| Overridable   | No (final)              | Yes                        |
| Used by       | `valueOf`               | Printing and concatenation |

**Remember:** `valueOf` matches `name()`, so an overridden `toString` does not round-trip.

### Reference type vs object type

|         | Reference type                                                | Object type                       |
| ------- | ------------------------------------------------------------- | --------------------------------- |
| Known   | At compile time                                               | At run time                       |
| Decides | What compiles, overloads, fields, statics, checked exceptions | Which overriding method body runs |

**Remember:** the compiler checks the variable's declared type; the JVM dispatches on the actual object.

## Domain 4: Implementing Exception Handling in Java Applications

### Checked vs unchecked exceptions

|                                   | Checked                                                                    | Unchecked                                        |
| --------------------------------- | -------------------------------------------------------------------------- | ------------------------------------------------ |
| Types                             | `Exception`, `Throwable` and every subclass outside the unchecked branches | `RuntimeException`, `Error` and their subclasses |
| Handle or declare                 | Required                                                                   | Optional                                         |
| Catching one the try cannot throw | Compile error (except `Exception` and `Throwable`)                         | Always allowed                                   |
| Custom version                    | `extends Exception`                                                        | `extends RuntimeException`                       |

**Remember:** the superclass alone decides. `Exception` itself is checked, and `Error` is not an `Exception`.

### finally vs try-with-resources cleanup

|                   | `try/finally`                           | try-with-resources                              |
| ----------------- | --------------------------------------- | ----------------------------------------------- |
| Who closes        | Your code                               | The compiler-generated `close()`, right to left |
| When              | After the catch blocks                  | Before the statement's own catch and finally    |
| Second exception  | Replaces the first; nothing is recorded | Attached to the first as suppressed             |
| Checked `close()` | Your code handles it                    | Must be caught or declared like any other call  |

**Remember:** only try-with-resources produces suppressed exceptions.

### Multi-catch vs separate catch blocks

|                | `catch (A \| B e)`                    | Separate catches        |
| -------------- | ------------------------------------- | ----------------------- |
| Related types  | Not allowed                           | Allowed, subclass first |
| Parameter      | Implicitly final                      | Can be reassigned       |
| Parameter type | Least upper bound of the alternatives | The declared type       |

**Remember:** multi-catch alternatives must be unrelated, and `e` cannot be reassigned.

### return in try vs return in finally

|                          | `return` in try or catch                                    | `return` in finally        |
| ------------------------ | ----------------------------------------------------------- | -------------------------- |
| When the value is fixed  | Before finally runs                                         | When finally runs          |
| Effect of a later change | Primitive value unchanged; a returned object can be mutated | Replaces any pending value |
| Pending exception        | Propagates after finally                                    | Discarded silently         |

**Remember:** abrupt completion in `finally` always wins.

### getMessage() vs toString() on an exception

|                  | `getMessage()`     | `toString()`                                |
| ---------------- | ------------------ | ------------------------------------------- |
| No message given | `null`             | Class name only                             |
| Message given    | The message        | `ClassName: message`                        |
| `super(cause)`   | `cause.toString()` | `ClassName: <cause class>: <cause message>` |

**Remember:** `println(e)` uses `toString()`, which includes the class name.

## Domain 5: Using Arrays and Collections to Store and Retrieve Data

### Arrays.asList vs List.of vs new ArrayList

|                  | `Arrays.asList(arr)`            | `List.of(...)`                  | `new ArrayList<>(c)` |
| ---------------- | ------------------------------- | ------------------------------- | -------------------- |
| `set`, `sort`    | Yes                             | `UnsupportedOperationException` | Yes                  |
| `add`, `remove`  | `UnsupportedOperationException` | `UnsupportedOperationException` | Yes                  |
| Nulls            | Allowed                         | `NullPointerException`          | Allowed              |
| Linked to source | Writes through to the array     | No                              | No                   |

**Remember:** `Arrays.asList` is fixed-size but writable; `List.of` is not writable at all.

### HashSet vs LinkedHashSet vs TreeSet

|                | `HashSet`             | `LinkedHashSet`       | `TreeSet`                           |
| -------------- | --------------------- | --------------------- | ----------------------------------- |
| Order          | None guaranteed       | Insertion             | Sorted                              |
| Duplicate test | `hashCode` + `equals` | `hashCode` + `equals` | Comparator or `compareTo` returns 0 |
| Null           | One                   | One                   | Rejected (natural order)            |
| Sequenced      | No                    | Yes                   | Yes, but positional adds throw      |

**Remember:** a TreeSet never calls `equals`; whatever its comparator calls equal is a duplicate.

### remove(int) vs remove(Object) on a List<Integer>

|            | `remove(int index)`            | `remove(Object o)`                |
| ---------- | ------------------------------ | --------------------------------- |
| Chosen for | `list.remove(1)` (exact match) | `list.remove(Integer.valueOf(1))` |
| Returns    | The removed element            | `true` or `false`                 |
| Not found  | `IndexOutOfBoundsException`    | `false`, no exception             |

**Remember:** an `int` argument means a position. Box it to remove a value.

### push/pop vs offer/poll on a Deque

|               | Stack style: `push`, `pop`, `peek`    | Queue style: `offer`, `poll`, `peek` |
| ------------- | ------------------------------------- | ------------------------------------ |
| Inserts at    | Head                                  | Tail                                 |
| Removes from  | Head                                  | Head                                 |
| Empty removal | `pop` throws `NoSuchElementException` | `poll` returns `null`                |

**Remember:** `push` = `addFirst`, `offer` = `offerLast`, `pop` = `removeFirst`, `poll` = `pollFirst`.

### get vs getOrDefault vs putIfAbsent with a null value

|                    | `get(k)` | `getOrDefault(k, d)`      | `putIfAbsent(k, v)`        |
| ------------------ | -------- | ------------------------- | -------------------------- |
| Key absent         | `null`   | `d`                       | Stores `v`, returns `null` |
| Key mapped to null | `null`   | `null` (default not used) | Stores `v`, returns `null` |

**Remember:** `getOrDefault` treats a null value as present; `putIfAbsent` treats it as absent.

### Collections.unmodifiableList vs reversed() vs subList

|                    | `unmodifiableList(l)` | `l.reversed()`                | `l.subList(a, b)`                                                              |
| ------------------ | --------------------- | ----------------------------- | ------------------------------------------------------------------------------ |
| Copy or view       | View                  | View                          | View                                                                           |
| Writes             | Throw                 | Write through (if `l` allows) | Write through (if `l` allows)                                                  |
| Sees later changes | Yes                   | Yes                           | Yes, until a structural change to `l` (then `ConcurrentModificationException`) |

**Remember:** all three are views. Only `List.copyOf` and `new ArrayList<>(l)` take a copy.

### ceiling vs higher, floor vs lower

|          | Inclusive         | Strict          |
| -------- | ----------------- | --------------- |
| Upward   | `ceiling(e)` (>=) | `higher(e)` (>) |
| Downward | `floor(e)` (<=)   | `lower(e)` (<)  |

**Remember:** all four return `null` when nothing qualifies, never throw.

## Domain 6: Processing Data Using Streams and Lambda Expressions

### takeWhile vs dropWhile vs filter

|                                  | `takeWhile(p)`                  | `dropWhile(p)`                       | `filter(p)`               |
| -------------------------------- | ------------------------------- | ------------------------------------ | ------------------------- |
| Keeps                            | Elements before the first false | Elements from the first false onward | Every element that passes |
| Tests after first false          | No, the stream ends             | No, everything passes                | Yes, every element        |
| On `1, 3, 4, 5, 7` with "is odd" | `1 3`                           | `4 5 7`                              | `1 3 5 7`                 |

**Remember:** both `While` operations stop testing at the first failure; only `filter` keeps looking.

### map vs flatMap vs mapMulti

|                     | `map`            | `flatMap`                        | `mapMulti`                      |
| ------------------- | ---------------- | -------------------------------- | ------------------------------- |
| Outputs per element | Exactly one      | Zero or more (a returned stream) | Zero or more (`accept` calls)   |
| Mapper shape        | `Function<T, R>` | `Function<T, Stream<R>>`         | `BiConsumer<T, Consumer<R>>`    |
| Typing              | Inferred         | Inferred                         | Often needs `.<R>mapMulti(...)` |

**Remember:** a stream of lists stays a stream of lists until `flatMap(List::stream)`.

### reduce vs collect

|                 | `reduce`                                           | `collect`                                           |
| --------------- | -------------------------------------------------- | --------------------------------------------------- |
| Model           | Immutable combination into a new value             | Mutable accumulation into a container               |
| Parallel safety | Needs identity, associativity, compatible combiner | Each chunk gets its own container from the supplier |
| Typical use     | Sums, max, string length totals                    | Lists, maps, StringBuilders                         |

**Remember:** use `reduce` for values and `collect` for containers.

### groupingBy(boolean) vs partitioningBy

|             | `groupingBy(s -> boolean)`   | `partitioningBy(predicate)`        |
| ----------- | ---------------------------- | ---------------------------------- |
| Keys        | Only values that occurred    | Always `false` and `true`          |
| Empty side  | Absent; `get` returns `null` | Holds the downstream's zero result |
| Map factory | Allowed                      | Not available                      |

**Remember:** a partition map always has two entries.

### andThen vs compose

|                   | `f.andThen(g)`  | `f.compose(g)`  |
| ----------------- | --------------- | --------------- |
| Order             | f first, then g | g first, then f |
| Equivalent        | `g(f(x))`       | `f(g(x))`       |
| `inc`, `dbl` on 5 | 12              | 11              |

**Remember:** `compose` runs its argument first.

### IntFunction vs ToIntFunction vs IntUnaryOperator

|         | `IntFunction<R>`     | `ToIntFunction<T>` | `IntUnaryOperator` |
| ------- | -------------------- | ------------------ | ------------------ |
| Takes   | `int`                | `T`                | `int`              |
| Returns | `R`                  | `int`              | `int`              |
| Method  | `apply`              | `applyAsInt`       | `applyAsInt`       |
| Used by | `IntStream.mapToObj` | `Stream.mapToInt`  | `IntStream.map`    |

**Remember:** "To" names the return type; a plain prefix names the parameter.

### reduce(identity, op) vs reduce(op)

|              | With identity                            | Without identity                              |
| ------------ | ---------------------------------------- | --------------------------------------------- |
| Returns      | `T`                                      | `Optional<T>`                                 |
| Empty stream | The identity                             | `Optional.empty`                              |
| Risk         | A wrong identity skews results per chunk | None from identity; still needs associativity |

**Remember:** if no neutral starting value exists, drop the identity and handle the Optional.

### fold vs scan

|                                 | `Gatherers.fold`             | `Gatherers.scan`    |
| ------------------------------- | ---------------------------- | ------------------- |
| Emits                           | One element: the final state | Every running state |
| Initial                         | Not emitted                  | Not emitted         |
| `x, y, z` with concat from `""` | `[xyz]`                      | `[x, xy, xyz]`      |

**Remember:** fold is a reduction that stays in the stream; scan shows every step.

## Domain 7: Packaging and Deploying Java Code

### exports vs opens

|                                  | `exports p`                        | `opens p`                  |
| -------------------------------- | ---------------------------------- | -------------------------- |
| Compile against `p`              | Yes                                | No                         |
| Reflect on public members        | Yes                                | Yes                        |
| `setAccessible(true)` on private | No (`InaccessibleObjectException`) | Yes                        |
| Missing package                  | Compile error                      | Warning, then launch fails |

**Remember:** `exports` is for code that calls you; `opens` is for frameworks that reflect into you.

### requires vs requires transitive vs requires static

|                     | `requires m` | `requires transitive m` | `requires static m` |
| ------------------- | ------------ | ----------------------- | ------------------- |
| Compile time        | Required     | Required                | Required            |
| Run time            | Required     | Required                | Optional            |
| Readers also read m | No           | Yes                     | No                  |

**Remember:** use `transitive` when your exported API exposes `m`'s types, and `static` for optional dependencies.

### uses vs provides

|             | `uses S;`                                             | `provides S with Impl;`              |
| ----------- | ----------------------------------------------------- | ------------------------------------ |
| Declared by | The consumer                                          | The provider                         |
| If missing  | `ServiceConfigurationError` from `ServiceLoader.load` | The provider is never found          |
| Readability | None; still `requires` the service's module           | None; must read the service's module |

**Remember:** neither directive creates a read edge, and the consumer never requires the provider.

### Named vs automatic vs unnamed module

|               | Named               | Automatic         | Unnamed           |
| ------------- | ------------------- | ----------------- | ----------------- |
| Source        | Modular JAR on `-p` | Plain JAR on `-p` | Anything on `-cp` |
| Exports       | Declared only       | Everything        | Everything        |
| Reads unnamed | No                  | Yes               | n/a               |
| jlink         | Yes                 | No                | No                |

**Remember:** automatic modules are the bridge: named modules can require them, and they can read the class path.

### java -jar vs java -cp vs java -p -m

|                   | `java -jar app.jar`                   | `java -cp ... pkg.Main`           | `java -p mods -m mod[/cls]`  |
| ----------------- | ------------------------------------- | --------------------------------- | ---------------------------- |
| Main class from   | Manifest `Main-Class`                 | The fully qualified name you give | `ModuleMainClass`, or `/cls` |
| Dependencies from | Manifest `Class-Path` (`-cp` ignored) | The class path                    | The module path              |
| JAR becomes       | Unnamed module                        | Unnamed module                    | Named module                 |

**Remember:** `-jar` ignores `-cp`, and without `-m` the module path never supplies the main class.

### Source-file mode vs javac + java

|                          | `java Main.java`                  | `javac` then `java Main`           |
| ------------------------ | --------------------------------- | ---------------------------------- |
| Class files written      | None                              | Yes                                |
| Class launched           | First top-level class in the file | The named class                    |
| File-name rules          | Relaxed                           | A public class must match its file |
| Stale `.class` beside it | Source wins (JDK 22+)             | The class file runs                |

**Remember:** in source-file mode, options go before the file and arguments after it.

### Bottom-up vs top-down migration

|                  | Bottom-up                                 | Top-down                                    |
| ---------------- | ----------------------------------------- | ------------------------------------------- |
| Converted first  | Libraries with no dependencies            | The application                             |
| Unconverted code | Stays on the class path                   | Libraries move to `-p` as automatic modules |
| jlink            | Possible when the graph is fully explicit | Not until automatic modules are replaced    |

**Remember:** a new named module can only require named or automatic modules, never the class path.

### Single-type import vs on-demand import vs module import

|                 | `import a.b.C;`  | `import a.b.*;`  | `import module M;` |
| --------------- | ---------------- | ---------------- | ------------------ |
| Priority        | Highest          | Middle           | Lowest             |
| Same-kind clash | Error at the use | Error at the use | Error at the use   |

**Remember:** a more specific import always shadows a broader one, and compact source files carry an implicit `import module java.base;`.

## Domain 8: Implementing Multithreading for Concurrent Code Execution

### Platform thread vs virtual thread

|            | Platform thread               | Virtual thread                    |
| ---------- | ----------------------------- | --------------------------------- |
| Backed by  | One OS thread                 | A carrier thread while running    |
| Daemon     | Configurable before `start()` | Always; `setDaemon(false)` throws |
| Priority   | Configurable                  | Fixed at 5                        |
| Pooling    | Common                        | Never; one per task               |
| Helps with | CPU-bound work                | Many blocking tasks               |

**Remember:** virtual threads make waiting cheap, not computing faster.

### start() vs run()

|             | `start()`                     | `run()`             |
| ----------- | ----------------------------- | ------------------- |
| Thread used | A new thread                  | The caller's thread |
| Call twice  | `IllegalThreadStateException` | Runs again          |

**Remember:** only `start()` creates concurrency.

### Runnable vs Callable

|                    | `Runnable`     | `Callable<V>`                 |
| ------------------ | -------------- | ----------------------------- |
| Returns            | Nothing        | `V`                           |
| Checked exceptions | Must be caught | Declared (`throws Exception`) |
| `execute`          | Yes            | No                            |
| `Future.get()`     | `null`         | The value                     |

**Remember:** a lambda that returns a value is a Callable; a void body is a Runnable.

### shutdown() vs shutdownNow() vs close()

|               | `shutdown()` | `shutdownNow()`     | `close()` |
| ------------- | ------------ | ------------------- | --------- |
| Running tasks | Finish       | Interrupted         | Finish    |
| Queued tasks  | Run          | Returned, never run | Run       |
| Waits         | No           | No                  | Yes       |

**Remember:** only `close()` (or `awaitTermination`) waits; all three reject new tasks.

### scheduleAtFixedRate vs scheduleWithFixedDelay

|                        | Fixed rate             | Fixed delay            |
| ---------------------- | ---------------------- | ---------------------- |
| Interval measured      | Start to start         | End to start           |
| 1 s task, 4 s interval | Starts about every 4 s | Starts about every 5 s |
| Overlap                | Never                  | Never                  |

**Remember:** rate counts from each start; delay counts from each finish.

### get() vs join() on CompletableFuture

|                    | `get()`                                      | `join()`                          |
| ------------------ | -------------------------------------------- | --------------------------------- |
| Failure wrapped in | `ExecutionException` (checked)               | `CompletionException` (unchecked) |
| Declares           | `InterruptedException`, `ExecutionException` | Nothing                           |

**Remember:** both put the original exception in `getCause()`.

### ScopedValue vs ThreadLocal

|                     | `ThreadLocal`                       | `ScopedValue`             |
| ------------------- | ----------------------------------- | ------------------------- |
| Mutable             | Yes, `set` any time                 | No; nested rebinding only |
| Lifetime            | Until `remove()` or the thread ends | The `run`/`call` extent   |
| Unset read          | `null` or `initialValue`            | `NoSuchElementException`  |
| New threads inherit | Only `InheritableThreadLocal`       | No                        |

**Remember:** a scoped value is bound for a call, not for a thread.

### volatile vs AtomicInteger vs synchronized

|                              | `volatile` | `AtomicInteger`     | `synchronized` |
| ---------------------------- | ---------- | ------------------- | -------------- |
| Visibility                   | Yes        | Yes                 | Yes            |
| `count++` safe               | No         | `incrementAndGet()` | Yes            |
| Several variables consistent | No         | No                  | Yes            |

**Remember:** `volatile` is for flags; atomics for one counter; locks for invariants across fields.

### lock() vs tryLock() vs lockInterruptibly()

|                   | `lock()` | `tryLock()` | `tryLock(t, unit)` | `lockInterruptibly()` |
| ----------------- | -------- | ----------- | ------------------ | --------------------- |
| Waits             | Forever  | No          | Up to t            | Until free            |
| Interruptible     | No       | n/a         | Yes                | Yes                   |
| Checked exception | No       | No          | Yes                | Yes                   |

**Remember:** only the timed `tryLock` and `lockInterruptibly` can be cancelled by an interrupt.

### Synchronized collection vs concurrent collection

|              | `Collections.synchronizedList`, `Vector` | `CopyOnWriteArrayList`, `ConcurrentHashMap` |
| ------------ | ---------------------------------------- | ------------------------------------------- |
| Single calls | Thread-safe                              | Thread-safe                                 |
| Iteration    | Fail-fast; lock the wrapper              | Snapshot or weakly consistent; no lock      |
| Nulls        | As the backing collection                | `ConcurrentHashMap` rejects them            |

**Remember:** synchronized is not concurrent.

### CountDownLatch vs CyclicBarrier vs Semaphore

|           | `CountDownLatch`       | `CyclicBarrier`            | `Semaphore`                    |
| --------- | ---------------------- | -------------------------- | ------------------------------ |
| Purpose   | Wait for N events once | N threads meet, repeatedly | Limit concurrent access        |
| Reuse     | No                     | Yes                        | Yes                            |
| Ownership | n/a                    | n/a                        | None; any thread may `release` |

**Remember:** latch counts down once, barrier trips every round, semaphore counts permits.

### forEach vs forEachOrdered on a parallel stream

|                | `forEach` | `forEachOrdered`    |
| -------------- | --------- | ------------------- |
| Order          | Any       | Encounter order     |
| Parallel speed | Full      | Reduced by ordering |

**Remember:** `sorted()` earlier in the pipeline does not make `forEach` ordered.

## Domain 9: Performing Input and Output Operations Using the Java I/O API

### Byte streams vs character streams

|                | Byte streams                  | Character streams                   |
| -------------- | ----------------------------- | ----------------------------------- |
| Superclasses   | `InputStream`, `OutputStream` | `Reader`, `Writer`                  |
| Unit           | 8-bit byte                    | 16-bit char, decoded with a charset |
| Printing class | `PrintStream`                 | `PrintWriter`                       |
| Use for        | Binary data and exact copies  | Text                                |

**Remember:** check the superclass, not the name; `PrintStream` is a byte stream.

### PrintWriter vs BufferedWriter

|                      | `PrintWriter`                                   | `BufferedWriter`         |
| -------------------- | ----------------------------------------------- | ------------------------ |
| Throws `IOException` | No; check `checkError()`                        | Yes, including `close()` |
| Line break           | `println`                                       | `newLine()`              |
| Autoflush            | Optional; only on `println`, `printf`, `format` | None                     |
| Accepts              | `Writer`, `OutputStream`, file name, `File`     | `Writer` only            |

**Remember:** PrintWriter swallows I/O errors; BufferedWriter reports them.

### transient vs static fields under serialization

|                     | `transient` instance field | `static` field                          |
| ------------------- | -------------------------- | --------------------------------------- |
| Written             | No                         | No                                      |
| Value after reading | Default (0, false, null)   | The class's current value in the reader |
| Initializer runs    | No                         | Only at class initialization            |

**Remember:** neither travels; transient comes back as a default, static as whatever the class holds now.

### Serializable vs Externalizable

|                     | `Serializable`                                             | `Externalizable`                          |
| ------------------- | ---------------------------------------------------------- | ----------------------------------------- |
| Methods             | None required                                              | Public `writeExternal` and `readExternal` |
| Saved               | All non-transient instance fields, automatically           | Only what `writeExternal` writes          |
| Constructor on read | The first non-serializable superclass's no-arg constructor | The class's own public no-arg constructor |

**Remember:** Externalizable saves nothing automatically and requires a public no-arg constructor.

### readObject hook vs readResolve

|         | `private void readObject(ObjectInputStream)` | `Object readResolve()`              |
| ------- | -------------------------------------------- | ----------------------------------- |
| Purpose | Customize how fields are read                | Replace the object that is returned |
| Access  | Must be private                              | Any                                 |
| Records | Ignored                                      | Honoured                            |
| Enums   | Ignored                                      | Ignored                             |

**Remember:** `readResolve` swaps the result (singletons); `readObject` fills in the fields.

### resolve vs resolveSibling vs relativize

|              | `a.resolve(b)`      | `a.resolveSibling(b)`        | `a.relativize(b)`        |
| ------------ | ------------------- | ---------------------------- | ------------------------ |
| Result       | `b` appended to `a` | `b` appended to `a`'s parent | The path from `a` to `b` |
| Absolute `b` | Returns `b`         | Returns `b`                  | Needs `a` absolute too   |
| Normalizes   | No                  | No                           | No                       |

**Remember:** resolve builds a path forward; relativize finds the way between two.

### normalize vs toAbsolutePath vs toRealPath

|                   | `normalize()` | `toAbsolutePath()` | `toRealPath()`               |
| ----------------- | ------------- | ------------------ | ---------------------------- |
| Touches the disk  | No            | No                 | Yes                          |
| Checked exception | No            | No                 | `IOException`                |
| Follows links     | No            | No                 | Yes, unless `NOFOLLOW_LINKS` |

**Remember:** only `toRealPath` can fail because the file is missing.

### Files.list vs Files.walk vs Files.find

|                | `list(dir)` | `walk(start[, n])` | `find(start, n, matcher)` |
| -------------- | ----------- | ------------------ | ------------------------- |
| Start included | No          | Yes                | If it matches             |
| Depth          | 1           | Unlimited or n     | n (required)              |
| Must close     | Yes         | Yes                | Yes                       |

**Remember:** `walk(start, 1)` is the start plus what `list` returns.

### Files.delete vs Files.deleteIfExists

|                     | `delete`                     | `deleteIfExists`             |
| ------------------- | ---------------------------- | ---------------------------- |
| Missing file        | `NoSuchFileException`        | Returns `false`              |
| Non-empty directory | `DirectoryNotEmptyException` | `DirectoryNotEmptyException` |

**Remember:** neither deletes recursively.

### Files.readAllLines vs Files.lines

|            | `readAllLines(p)` | `lines(p)`                   |
| ---------- | ----------------- | ---------------------------- |
| Returns    | `List<String>`    | Lazy `Stream<String>`        |
| Memory     | Whole file        | Line by line                 |
| Must close | No                | Yes, with try-with-resources |

**Remember:** for a huge file, use `lines` inside try-with-resources.

## Domain 10: Developing Applications with Localization Support

### Locale.of vs Locale.forLanguageTag vs Locale.Builder

|           | `Locale.of(lang, country)` | `Locale.forLanguageTag(tag)`           | `Locale.Builder`           |
| --------- | -------------------------- | -------------------------------------- | -------------------------- |
| Input     | Separate strings           | One hyphenated BCP 47 tag              | Field by field             |
| Validates | No                         | Stops at the first bad subtag          | Yes                        |
| Bad input | Accepted as given          | Partial or `Locale.ROOT`, no exception | `IllformedLocaleException` |

**Remember:** only the Builder throws; `"de_CH"` as a tag silently becomes the root locale.

### FORMAT vs DISPLAY vs the general default

|                                        | FORMAT                                               | DISPLAY                               | General default                             |
| -------------------------------------- | ---------------------------------------------------- | ------------------------------------- | ------------------------------------------- |
| Read by                                | Number, date and message formatters; `String.format` | `getDisplayCountry()` and other names | `Locale.getDefault()`, `getBundle` fallback |
| Changed by `setDefault(loc)`           | Yes                                                  | Yes                                   | Yes                                         |
| Changed by `setDefault(Category, loc)` | Its own category only                                | Its own category only                 | No                                          |

**Remember:** one-argument `setDefault` resets everything; category calls change one thing.

### Bundle search vs key lookup

|                | Bundle search (`getBundle`)                      | Key lookup (`getString`)                    |
| -------------- | ------------------------------------------------ | ------------------------------------------- |
| Path           | Requested locale, then default locale, then base | The chosen bundle's parent chain only       |
| Default locale | Tried as a fallback                              | Never a parent                              |
| Failure        | `MissingResourceException` from `getBundle`      | `MissingResourceException` from `getString` |

**Remember:** the default locale can be chosen as the bundle but never supplies a missing key.

### Properties file vs ListResourceBundle

|                             | `.properties` file | `ListResourceBundle` class |
| --------------------------- | ------------------ | -------------------------- |
| Value types                 | Strings only       | Any object                 |
| Same name exists            | Loses              | Wins                       |
| `getString` on a non-String | n/a                | `ClassCastException`       |

**Remember:** the class format is tried first.

### NumberFormat/DecimalFormat rounding vs Formatter rounding

|                   | `NumberFormat`, `DecimalFormat`, `MessageFormat` | `String.format`, `printf`, `formatted` |
| ----------------- | ------------------------------------------------ | -------------------------------------- |
| Rounding          | HALF_EVEN                                        | HALF_UP                                |
| 6.25 to one place | 6.2                                              | 6.3                                    |

**Remember:** the java.text formatters round ties to even; `java.util.Formatter` rounds ties up.

### NumberFormat.parse vs Integer.parseInt vs LocalDate.parse

|               | `NumberFormat.parse`       | `Integer.parseInt`      | `LocalDate.parse`        |
| ------------- | -------------------------- | ----------------------- | ------------------------ |
| Trailing text | Ignored                    | Error                   | Error                    |
| Returns       | `Long` or `Double`         | `int`                   | `LocalDate`              |
| Exception     | `ParseException` (checked) | `NumberFormatException` | `DateTimeParseException` |

**Remember:** only the java.text parsers throw a checked exception.

### M vs m, h vs H in date patterns

|       | Uppercase | Lowercase                 |
| ----- | --------- | ------------------------- |
| M / m | Month     | Minute                    |
| H / h | Hour 0–23 | Hour 1–12 (pair with `a`) |

**Remember:** `dd/mm/uuuu` prints minutes in the month position, and it compiles.

### SMART vs STRICT vs LENIENT resolving

|              | SMART            | STRICT                   | LENIENT    |
| ------------ | ---------------- | ------------------------ | ---------- |
| Default for  | `ofPattern`      | `LocalDate.parse` (ISO)  | Opt-in     |
| `2027-04-31` | 2027-04-30       | `DateTimeParseException` | 2027-05-01 |
| Year letter  | `uuuu` or `yyyy` | Needs `uuuu`             | Either     |

**Remember:** SMART clamps, STRICT rejects, LENIENT rolls over.

[← Back to the study guide](README.md)
