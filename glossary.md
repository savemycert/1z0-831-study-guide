# 1Z0-831 Glossary

315 1Z0-831 terms and services, each in one plain-English sentence.

[#](#) · [A](#a) · [B](#b) · [C](#c) · [D](#d) · [E](#e) · [F](#f) · [G](#g) · [H](#h) · [I](#i) · [J](#j) · [L](#l) · [M](#m) · [N](#n) · [O](#o) · [P](#p) · [Q](#q) · [R](#r) · [S](#s) · [T](#t) · [U](#u) · [V](#v) · [W](#w) · [Y](#y) · [Z](#z)

## #

- **@FunctionalInterface**: An annotation that makes the compiler check that an interface has exactly one abstract method.

## A

- **Abstract class**: A class that cannot be instantiated and may declare abstract methods for subclasses to implement.
- **Active use**: An action that triggers class initialization, such as creating an instance, calling a static method or reading a non-constant static field.
- **Aliasing**: Two variables holding a reference to the same object, so a change through either is visible through both.
- **Anonymous class**: A class declared and instantiated in one new expression, with no name and no constructor of its own.
- **Array**: A fixed-length, indexed container whose elements start at their type's default value.
- **ArrayDeque**: A resizable Deque implementation that rejects null elements.
- **Arrays.asList**: A method that returns a fixed-size list backed by an array, where writes go through in both directions.
- **Arrow label**: A case X -> label that runs exactly one expression statement, block or throw and never falls through.
- **Associativity**: The property that grouping does not change a result, required for correct parallel reductions.
- **Atomic variable**: A class such as AtomicInteger that updates one variable atomically without a lock.
- **Autoboxing**: The automatic conversion of a primitive to its wrapper class.
- **AutoCloseable**: The interface a resource must implement to be used in try-with-resources; its close() declares throws Exception.
- **Autoflush**: A PrintWriter setting that flushes after println, printf and format calls (but never print).
- **Automatic module**: A plain JAR on the module path, named from its manifest or file name, that exports and opens everything.
- **Automatic-Module-Name**: A manifest attribute that fixes an automatic module's name regardless of the JAR file name.

## B

- **Back-reference**: The handle serialization writes when an object already written to the same stream appears again.
- **Base bundle**: The resource bundle with no locale suffix, used as the last fallback and the root of every parent chain.
- **BasicFileAttributes**: The portable set of file attributes, such as size and last-modified time, read in one operation.
- **BCP 47 language tag**: A hyphen-separated locale identifier such as de-CH, read by Locale.forLanguageTag.
- **Binary numeric promotion**: The rule that converts both operands of an operator to a common type, at least int.
- **BinaryOperator**: A BiFunction whose two arguments and result all have the same type.
- **Blank final**: A final field declared without an initializer, which must be assigned exactly once in an initializer or every constructor.
- **BlockingQueue**: A thread-safe queue whose put and take methods wait for space or elements.
- **Boot layer**: The layer of modules the JVM builds at launch by resolving the root module's dependencies.
- **Bottom-up migration**: Converting the libraries with no dependencies to modules first and the application last.
- **Bound method reference**: A method reference whose receiver is a specific object captured when the reference is created.
- **BufferedReader**: A buffered Reader whose readLine method returns one line without its terminator, or null at the end.
- **Byte stream**: An I/O stream that moves raw bytes and extends InputStream or OutputStream.

## C

- **Callable**: A task interface whose call method returns a value and may throw checked exceptions.
- **Canonical constructor**: The record constructor whose parameters match the record components in order.
- **Capacity**: The size of a StringBuilder's internal storage, which is never used for index checks.
- **Carrier thread**: A platform thread that runs virtual threads while they are mounted.
- **Case constant**: A compile-time constant label value that must fit the selector type and appear only once.
- **Character stream**: An I/O stream that moves chars and extends Reader or Writer, applying a charset at the byte boundary.
- **Charset**: A mapping between characters and bytes, such as UTF-8 or ISO-8859-1.
- **Checked exception**: Any Throwable outside the RuntimeException and Error branches, which must be caught or declared.
- **Choice format**: A MessageFormat element that selects text based on numeric limits, such as plural forms.
- **ChronoUnit**: The enum of standard time units, used to count the units between two temporal values.
- **Class path**: The list of directories and JARs searched for classes, whose contents form the unnamed module.
- **ClassCastException**: The run-time exception thrown when a reference is cast to a type the object is not an instance of.
- **Closeable**: An AutoCloseable subinterface for I/O resources whose close() declares throws IOException.
- **Collector**: An object passed to collect that describes how to accumulate elements into a result.
- **Combiner**: The function that merges partial results from different chunks of a parallel stream.
- **Compact constructor**: A record constructor with no parameter list that validates or reassigns parameters before the fields are assigned automatically.
- **Compact number format**: A NumberFormat that abbreviates numbers, such as 4K or 7 million.
- **Compact source file**: A .java file with top-level fields and methods whose class is declared implicitly by the compiler.
- **Comparable**: The interface that defines an object's natural ordering through compareTo.
- **Comparator**: An object that defines an ordering between two values, often built with comparing, thenComparing and reversed.
- **compareAndSet**: An atomic operation that stores a new value only if the current value equals an expected one.
- **Compile-time constant expression**: An expression built only from literals and final variables initialised with constants, evaluated by the compiler.
- **CompletableFuture**: A Future that can be completed explicitly and chained into asynchronous pipelines.
- **CompletionException**: The unchecked exception join() throws when a CompletableFuture failed.
- **Compound assignment operator**: An operator such as += or *= that includes a hidden cast back to the variable's type.
- **computeIfAbsent**: A Map method that computes and stores a value only when the key is absent, returning the current value.
- **ConcurrentHashMap**: A thread-safe hash map with non-blocking reads, atomic compound methods and no null keys or values.
- **ConcurrentModificationException**: The exception thrown when a collection is structurally changed during iteration other than through the iterator.
- **Condition**: An object from Lock.newCondition() that lets threads await and signal a state change under that lock.
- **Console**: The java.io class returned by System.console(), which may be null, with readLine and readPassword methods.
- **Constant variable**: A final primitive or String variable initialised with a constant expression in its declaration.
- **Constant-specific class body**: A body attached to one enum constant, which creates an anonymous subclass for that constant.
- **Constructor chaining**: One constructor calling another with this(...) or a superclass constructor with super(...).
- **Constructor prologue**: The statements before an explicit this(...) or super(...) call, allowed since Java 25 with strict limits on using the object.
- **Constructor reference**: A method reference such as ArrayList::new that creates an object with the matching constructor.
- **Consumer**: A functional interface whose accept method takes a value and returns nothing.
- **CopyOnWriteArrayList**: A thread-safe list that copies its array on every write and iterates over snapshots.
- **CountDownLatch**: A one-time synchronizer that lets threads wait until a count reaches zero.
- **Covariant return type**: An overriding method's return type that is a subtype of the overridden method's return type.
- **Custom exception**: A class that extends an exception type to represent an application-specific condition.
- **CyclicBarrier**: A reusable synchronizer that makes a fixed number of threads wait for each other.

## D

- **Daemon thread**: A thread the JVM does not wait for when the program ends; virtual threads are always daemons.
- **Dangling else**: An else without braces, which binds to the nearest preceding if that has no else, regardless of indentation.
- **DataOutputStream**: A byte stream that writes Java primitives and strings in fixed big-endian binary form.
- **DateTimeFormatter**: The immutable, thread-safe java.time class that formats and parses dates and times.
- **DateTimeParseException**: The unchecked exception thrown when java.time parsing fails.
- **Deadlock**: A state where threads wait forever for locks held by each other.
- **DecimalFormat**: A NumberFormat that formats numbers with an explicit pattern of 0, # and separator symbols.
- **DecimalFormatSymbols**: The locale-specific symbols, such as the decimal and grouping separators, a DecimalFormat uses.
- **Decorator stream**: A stream that wraps another stream of the same family to add buffering, formatting or typed data.
- **Default constructor**: The no-argument constructor the compiler adds only when a class declares no constructors.
- **Default locale**: The locale an API uses when none is passed, set with Locale.setDefault.
- **Default method**: A public instance method with a body declared in an interface and inherited by implementing classes.
- **Defensive copy**: A copy of a mutable object taken when storing or returning it, so outside code cannot change internal state.
- **Definite assignment**: The compiler's check that a local variable has a value on every path before it is read.
- **Deque**: A double-ended queue that supports insertion and removal at both ends, usable as a stack or a queue.
- **Deserialization**: Rebuilding an object graph from bytes with ObjectInputStream.readObject.
- **Dominance**: The compile error raised when an earlier case label would always match before a later one could run.
- **Downstream collector**: A collector applied to the elements of each group produced by groupingBy or partitioningBy.
- **DST gap**: A period of local time that does not exist because clocks jump forward.
- **DST overlap**: A period of local time that occurs twice because clocks go back.
- **Duration**: An exact amount of time stored as seconds and nanoseconds.
- **Dynamic dispatch**: The run-time selection of an overriding method body based on the object's actual class.

## E

- **Effectively final**: A local variable that is assigned once and never reassigned, which makes it capturable by lambdas and local classes.
- **Encapsulation**: Keeping state in private fields and changing it only through methods that enforce rules.
- **Encounter order**: The order in which an ordered stream source presents its elements.
- **Enhanced for loop**: A for (Type v : source) loop over an array or Iterable, where v holds a copy of each element.
- **Enum**: A special class with a fixed set of constant instances that implicitly extends java.lang.Enum.
- **EOFException**: The IOException thrown when a typed read or readObject runs past the end of the data.
- **Error**: A Throwable subclass for serious JVM problems, which is not an Exception and is not caught by catch (Exception e).
- **Exception chaining**: Wrapping a low-level exception as the cause of a higher-level one, read back with getCause().
- **ExceptionInInitializerError**: The error thrown when a static initializer fails, with the original exception as its cause.
- **ExecutionException**: The checked exception Future.get() throws when the task failed, with the task's exception as its cause.
- **ExecutorService**: An object that runs submitted tasks on threads it manages and can be shut down or closed.
- **Exhaustiveness**: The requirement that a switch covers every possible input value, through a default, an unguarded total pattern, every enum constant or every permitted subtype.
- **Externalizable**: A Serializable subinterface whose writeExternal and readExternal methods control the whole format.

## F

- **Fail-fast iterator**: An iterator that throws ConcurrentModificationException when its collection changes during iteration.
- **Fall-through**: The behaviour of a colon switch that keeps running the statements of later labels until a break or the end of the switch.
- **Field hiding**: A subclass field with the same name as an inherited one, chosen by the reference type rather than the object.
- **Files**: The java.nio.file utility class of static methods that create, copy, move, delete, read and inspect files.
- **FileVisitor**: The callback interface walkFileTree uses to visit directories and files.
- **FileVisitResult**: The value a FileVisitor callback returns: CONTINUE, SKIP_SUBTREE, SKIP_SIBLINGS or TERMINATE.
- **finally block**: A block that runs after a try statement's try and catch blocks, whether or not an exception was thrown.
- **Finisher**: The optional gatherer function run once at the end of input to push any remaining output.
- **flatMap**: An intermediate operation that replaces each element with the elements of a returned stream.
- **Flow scoping**: The rule that a pattern variable is in scope only where the compiler can prove the match succeeded.
- **FormatStyle**: The SHORT, MEDIUM, LONG and FULL localized date and time styles.
- **Function**: A functional interface whose apply method maps one value to another.
- **Functional interface**: An interface with exactly one abstract method after inheritance, usable as a lambda target.
- **Future**: A handle to a submitted task's result, read with get().

## G

- **Garbage collection eligibility**: The state of an object that no live thread can reach through any chain of references from a root.
- **Gatherer**: A custom intermediate operation applied with Stream.gather, standard since Java SE 24.
- **getSuppressed()**: The Throwable method that returns the exceptions suppressed onto an exception, in closing order.
- **groupingBy**: A collector that groups elements by a classifier key into a Map.
- **Guard**: A when clause that adds a boolean condition to a pattern label.

## H

- **HALF_EVEN rounding**: A rounding mode that sends exact ties to the even neighbour, used by NumberFormat and DecimalFormat.
- **Handle or declare rule**: The requirement that a checked exception be caught by an enclosing try or listed in the method's throws clause.
- **Happens-before**: The Java Memory Model ordering that guarantees one action's effects are visible to another.
- **HashMap**: A map with no guaranteed order that allows one null key and null values.
- **HashSet**: A set with no guaranteed order that uses hashCode and equals to detect duplicates.

## I

- **Identity value**: A starting value for reduce that leaves every element unchanged when combined with it.
- **IllegalMonitorStateException**: The exception thrown when a thread calls wait, notify or unlock without owning the lock.
- **IllformedLocaleException**: The unchecked exception Locale.Builder throws for an invalid locale field.
- **Immutable object**: An object whose observable state cannot change after construction.
- **Implied readability**: The readability that requires transitive passes on to every module that reads the declaring module.
- **InaccessibleObjectException**: The unchecked exception thrown when setAccessible(true) targets a member in a package that is not opened.
- **Incidental white space**: The indentation shared by all lines of a text block, which the compiler strips.
- **Infinite stream**: A stream from iterate or generate with no natural end, which must be bounded to terminate.
- **Inner class**: A non-static member class whose objects are each bound to one enclosing instance.
- **InputStreamReader**: The bridge that decodes bytes from an InputStream into characters.
- **Insertion point**: The index where a missing key would go in a sorted array, encoded by binarySearch as -(insertion point) - 1.
- **Instance initializer**: A block of code in a class body that runs with the field initializers during every object construction.
- **Instance main method**: A non-static main method, launched after the launcher creates an object with the no-argument constructor.
- **Instant**: A java.time type holding a moment on the UTC timeline.
- **Integer cache**: The guaranteed reuse of Integer objects from -128 to 127 by Integer.valueOf.
- **Integrator**: The gatherer function called once per element that pushes output and returns whether it wants more input.
- **Intermediate operation**: A lazy stream operation such as filter or map that returns a new stream and does nothing until a terminal operation runs.
- **intern()**: A String method that returns the pooled instance with the same content.
- **Interrupt**: A cooperative request to stop, delivered by setting a flag and waking blocking methods with InterruptedException.
- **InvalidClassException**: The exception thrown on reading for a version mismatch, a missing valid constructor or a filter rejection.
- **Island of isolation**: A group of objects that reference each other but are unreachable from any root, and so are eligible for collection.

## J

- **java.lang.IO**: A console helper class with print, println and readln, available everywhere without an import.
- **jdeps**: A JDK tool that lists the dependencies of classes and JARs to help plan a migration.
- **jlink**: The tool that links named modules and their dependencies into a custom runtime image.

## L

- **Labelled statement**: A statement named with label: so that break label or continue label can target it.
- **Lambda expression**: An anonymous function written as parameters, an arrow and a body, implementing a functional interface.
- **Lazy evaluation**: The behaviour of streams that defers all work until a terminal operation is called.
- **LinkedHashSet**: A hash set that keeps insertion order and is a SequencedSet.
- **ListResourceBundle**: A resource bundle written as a class whose getContents method returns key-value pairs of any type.
- **Livelock**: A state where threads keep running but keep undoing each other's progress.
- **Local class**: A class declared inside a method, constructor or block, usable only within that block.
- **LocalDate**: A java.time type holding a date with no time and no zone.
- **Locale**: An object naming a language, optionally with a region, script and variant, used by locale-sensitive APIs.
- **Locale.Category**: The FORMAT and DISPLAY categories, each with its own default locale.
- **Locale.ROOT**: The empty locale with no language or region, whose language tag is und.
- **Lost update**: A race where two threads read the same value and one thread's write overwrites the other's.

## M

- **Manifest**: The META-INF/MANIFEST.MF file in a JAR, which can hold Main-Class, Class-Path and Automatic-Module-Name.
- **Map view**: The live keySet, values or entrySet collection of a map, through which removals and setValue reach the map.
- **mapMulti**: An intermediate operation that emits zero or more elements per input through a consumer.
- **mark and reset**: Methods that remember a stream position and return to it, supported only when markSupported() is true.
- **MatchException**: The run-time exception thrown when an exhaustive switch meets an enum constant or subtype added after it was compiled.
- **merge**: A Map method that stores a value for an absent key or combines it with the existing value, removing the entry if the result is null.
- **MessageFormat**: A java.text class that fills numbered placeholders in a pattern with locale-formatted arguments.
- **Method hiding**: A subclass static method with the same signature as a superclass static method, bound at compile time.
- **Method reference**: Shorthand for a lambda that calls one existing method, written Type::method or object::method.
- **Method signature**: A method's name and parameter types in order, which decide whether two methods overload or clash.
- **MissingResourceException**: The unchecked exception thrown when no bundle is found or a key is missing from the whole chain.
- **Modular JAR**: A JAR with module-info.class at its root, treated as a named module on the module path.
- **Module**: A named, self-describing set of packages whose module-info.java declares its dependencies and what it exposes.
- **Module import declaration**: An import module M; statement that imports every public type in the packages M exports to the current module.
- **Module path**: The list of directories and JARs searched for modules, given with -p or --module-path.
- **module-info.java**: The module declaration file at the root of a module's source tree, compiled to module-info.class.
- **Monitor**: The intrinsic lock every Java object has, acquired by synchronized.
- **Multi-catch**: A catch clause that handles several unrelated exception types with one implicitly final parameter.
- **Multi-file source-code program**: A source-file mode program whose other source files are compiled on demand from the source root.
- **Mutable reduction**: Accumulating elements into a container with collect instead of creating new values.

## N

- **Name element**: One component of a Path after the root, counted by getNameCount.
- **NaN**: The floating-point "not a number" value, which is not equal to anything under ==, including itself.
- **Narrowing conversion**: A conversion to a smaller type, which needs an explicit cast unless it is a constant that fits.
- **Natural ordering**: The order defined by an element type's own compareTo method.
- **NavigableSet**: A sorted set with nearest-element lookups such as ceiling, floor, higher and lower, plus range views.
- **non-sealed**: A modifier that reopens a permitted subtype of a sealed type to unrestricted extension.
- **Normalization**: Removing . elements and cancelling name/.. pairs in a path without touching the file system.
- **NotSerializableException**: The exception thrown at run time when serialization reaches an object whose class is not Serializable.
- **NumberFormat**: The locale-aware java.text formatter for numbers, currency and percentages.

## O

- **ObjectInputFilter**: A filter that accepts or rejects classes and graph limits while a stream is deserialized.
- **OffsetDateTime**: A date-time tied to a fixed offset from UTC that never changes.
- **Open module**: A module declared with open, which opens every package for deep reflection at run time.
- **opens**: A directive that allows deep reflection on all members of a package at run time.
- **Optional**: A container that holds one value or none, returned by operations that may have no result.
- **OptionalInt**: The primitive Optional returned by IntStream max and min.
- **Ordinal**: An enum constant's zero-based position in its declaration.
- **OutputStreamWriter**: The bridge that encodes characters into bytes for an OutputStream.
- **Overload resolution**: The compile-time process that picks a method from overloads in three phases: widening, boxing and varargs.

## P

- **Parallel stream**: A stream whose pipeline may be split and run on several threads of the common ForkJoinPool.
- **Parent chain**: The sequence of less specific bundles searched for a key missing from the chosen bundle.
- **ParseException**: The checked exception thrown when NumberFormat or MessageFormat cannot parse text.
- **partitioningBy**: A collector that splits elements into a Map with exactly the keys false and true.
- **Pass-by-value**: Java's argument passing, which copies primitives and copies references, so a method can never reseat the caller's variable.
- **Path**: An immutable java.nio.file object describing a location as an optional root and a sequence of name elements.
- **Period**: A calendar amount stored as years, months and days.
- **Platform thread**: A Java thread backed by one operating-system thread for its whole life.
- **Polymorphism**: One call site running different method bodies depending on the object's class, through overriding.
- **Precise rethrow**: The rule that rethrowing an effectively final catch parameter throws only the checked types the try block can actually throw.
- **Predicate**: A functional interface whose test method returns a boolean.
- **Primary exception**: The first exception thrown in a try-with-resources statement, which is the one that propagates.
- **Primitive stream**: An IntStream, LongStream or DoubleStream that holds unboxed numbers and offers sum and average.
- **Primitive type**: One of Java's eight built-in value types: byte, short, char, int, long, float, double and boolean.
- **PrintStream**: A byte stream with print and println methods, the type of System.out and System.err.
- **PrintWriter**: A character writer with print and println methods that never throws IOException and reports errors through checkError.
- **Private interface method**: A helper method with a body inside an interface, callable only by code in that interface.
- **Propagation**: The movement of an uncaught exception up the call stack, running each finally block until a matching catch is found.
- **PropertyResourceBundle**: A resource bundle loaded from a .properties file, whose values are always Strings.
- **Provider method**: A public static no-argument method named provider that ServiceLoader calls instead of a constructor.

## Q

- **Qualified export**: An exports directive with a to clause that makes a package accessible only to the named modules.

## R

- **Race condition**: A bug where the result depends on how threads interleave on shared mutable state.
- **Ragged array**: A multidimensional array whose rows have different lengths, created by leaving later dimensions unspecified.
- **Readability**: The relationship created by requires that lets one module use another module's exported packages.
- **readResolve**: A method that returns the object deserialization should hand back instead of the newly read one.
- **ReadWriteLock**: A lock pair in which many readers may share access while a writer has exclusive access.
- **Real path**: The absolute, normalized path with symbolic links resolved, returned by toRealPath after checking the disk.
- **Record**: A final class whose state is declared in its header and whose fields, accessors, canonical constructor, equals, hashCode and toString are generated.
- **Record pattern**: A pattern such as case Range(int lo, int hi) that matches a record and binds its components.
- **Reduction**: A terminal operation that combines all elements into one result, such as reduce, sum or count.
- **Reentrancy**: The ability of a thread that holds a lock to acquire it again without blocking.
- **ReentrantLock**: The standard explicit Lock implementation, reentrant and supporting timed and interruptible acquisition.
- **Reference**: A value that points to one object or is null; declaring a variable creates a reference, never an object.
- **RejectedExecutionException**: The exception thrown when a task is submitted to an executor that has been shut down.
- **ResolverStyle**: The SMART, STRICT or LENIENT setting that decides how a formatter resolves out-of-range dates.
- **Resource bundle**: A set of locale-specific key-value pairs found by a base name and a locale suffix.
- **reversed()**: A sequenced-collection method that returns a live reverse-ordered view of the original.
- **Root component**: The leading part of an absolute path, such as /, which is not a name element.
- **Runnable**: A task interface whose run method returns nothing and declares no checked exceptions.
- **Runtime image**: A self-contained directory with its own java launcher and only the modules an application needs.

## S

- **ScheduledExecutorService**: An executor that runs tasks after a delay or periodically.
- **Scoped value**: An immutable value bound with ScopedValue.where for the duration of a run or call on the current thread.
- **Sealed class**: A class or interface that lists exactly which types may directly extend or implement it.
- **Selector**: The expression in parentheses after switch whose value picks the label.
- **Semaphore**: A synchronizer that limits concurrent access with a count of permits that have no owner.
- **Sequenced collection**: A Java 21 interface family for collections with a defined encounter order, adding first, last and reversed operations.
- **SequencedMap**: A map with a defined order and methods such as putFirst, firstEntry, pollLastEntry and reversed.
- **Serializable**: A marker interface that permits a class's objects to be serialized.
- **Serialization**: Converting an object graph into bytes with ObjectOutputStream.writeObject.
- **Serialization proxy**: A pattern in which a small proxy object is serialized and rebuilds the real object through its constructor.
- **serialVersionUID**: A static final long that identifies a serializable class's version, which must match when reading.
- **Service**: A type, usually an interface, that consumer modules use through ServiceLoader without knowing which modules implement it.
- **Service provider**: A class named in a provides directive that supplies an implementation of a service.
- **ServiceLoader**: The API that locates and lazily instantiates service providers, caching the instances it creates.
- **Shadowing**: A local or parameter with the same name as a field, which hides the field inside its scope.
- **Shallow copy**: A copy of a container that shares the objects or inner arrays it refers to, as clone() does for arrays.
- **Short-circuit operator**: The && and || operators, which skip the right operand when the left one decides the result.
- **Short-circuiting operation**: An operation such as limit, findFirst or anyMatch that can finish without processing every element.
- **Sliding window**: A gatherer output of overlapping lists of consecutive elements, each shifted by one.
- **Source-file mode**: Running a program with java File.java, which compiles it in memory and writes no class files.
- **Split package**: A package present in more than one module of a layer, which stops resolution or boot-layer creation.
- **StandardCopyOption**: The options REPLACE_EXISTING, COPY_ATTRIBUTES and ATOMIC_MOVE for copy and move.
- **StandardOpenOption**: The options such as CREATE, APPEND and TRUNCATE_EXISTING that control how a file is opened.
- **Starvation**: A state where one thread rarely or never gets a resource that others keep acquiring.
- **Static initializer**: A static block that runs once when the class is initialized on its first active use.
- **Static interface method**: A method declared static in an interface, callable only through the interface name and never inherited.
- **Static nested class**: A class declared static inside another class, with no enclosing instance.
- **Stream pipeline**: A source, zero or more intermediate operations and one terminal operation that runs them.
- **String pool**: The JVM's store of interned Strings, shared by every identical literal and constant expression.
- **StringBuilder**: A mutable character sequence whose methods change it in place and return the same object.
- **Strong encapsulation**: The module system rule that public types in unexported packages are inaccessible to other modules.
- **Supplier**: A functional interface whose get method takes no arguments and returns a value.
- **Suppressed exception**: An exception thrown by close() in try-with-resources and attached to the primary exception.
- **Switch expression**: A switch that produces a value and must cover every possible selector value.
- **Symbolic link**: A file-system entry that points to another file; most Files methods follow it unless NOFOLLOW_LINKS is passed, but walk and walkFileTree follow it only with FOLLOW_LINKS.

## T

- **teeing**: A collector that sends every element to two collectors and merges their results.
- **TemporalAdjuster**: An object passed to with() that computes a related date, such as the next Monday or the last day of the month.
- **Terminal operation**: A stream operation such as forEach, count or collect that pulls elements through the pipeline and produces a result.
- **Text block**: A multi-line String literal delimited by three double quotes, with incidental indentation removed by the compiler.
- **Thread state**: The value from getState(): NEW, RUNNABLE, BLOCKED, WAITING, TIMED_WAITING or TERMINATED.
- **Thread.Builder**: The builder returned by Thread.ofPlatform() or Thread.ofVirtual() for configuring and creating threads.
- **Throwable**: The root class of every exception and error in Java.
- **throws clause**: The part of a method or constructor declaration that lists the checked exceptions it may throw.
- **Top-down migration**: Converting the application to a module first while libraries run as automatic modules.
- **transient**: A field modifier that excludes an instance field from default serialization.
- **TreeMap**: A map that keeps its keys sorted and supports navigation methods such as floorKey and ceilingKey.
- **TreeSet**: A sorted set that uses compareTo or a comparator for ordering and duplicate detection.
- **try-with-resources**: A try statement that declares resources and closes them automatically, right to left, after the body.
- **Type pattern**: A case label such as case Double d that matches by type and binds the value to a new variable.

## U

- **UnaryOperator**: A Function whose argument and result have the same type.
- **Unbound method reference**: A method reference of the form Type::instanceMethod whose first argument becomes the receiver.
- **Unboxing**: The automatic conversion of a wrapper back to its primitive, which throws NullPointerException for null.
- **Unchecked exception**: A RuntimeException or Error, or a subclass of either, which may be thrown without being declared.
- **Unmodifiable collection**: A collection whose mutator methods throw UnsupportedOperationException.
- **Unmodifiable view**: A read-only wrapper that still reflects later changes to its source collection.
- **Unnamed module**: The module that holds everything on the class path, which reads every module but cannot be required.
- **Unnamed pattern**: The underscore _ used in a pattern to match any component without binding it.
- **Unnamed variable**: A local declaration written _ that binds a value which can never be read.
- **Unreachable statement**: A statement the compiler can prove never runs, which is a compile error.
- **UnsupportedTemporalTypeException**: The run-time exception thrown when an amount or unit does not fit the temporal type it is applied to.

## V

- **var**: A reserved type name that lets the compiler infer a local variable's type from its initializer.
- **Varargs parameter**: A last parameter written Type... that receives its arguments as an array.
- **Virtual thread**: A lightweight JVM-managed thread that is mounted on a carrier platform thread only while it runs.
- **volatile**: A field modifier that guarantees visibility and ordering of single reads and writes across threads.

## W

- **Weakly consistent iterator**: A concurrent-collection iterator that never throws and may or may not reflect concurrent changes.
- **Widening conversion**: A conversion to a type that can hold every value of the source type, done without a cast.
- **Wrapper class**: The object form of a primitive, such as Integer or Double, used where an object is required.
- **writeReplace**: A method that nominates a different object to be serialized in place of the current one.

## Y

- **yield**: A contextual keyword that ends a switch expression with a value from a block arm or colon group.

## Z

- **ZonedDateTime**: A date-time with an offset and a ZoneId whose rules can change the offset, for example for daylight saving time.
- **ZoneId**: An identifier for a region's time-zone rules, such as Europe/Paris, or for a fixed offset.
- **ZoneOffset**: A fixed difference from UTC, such as +05:30.

[← Back to the study guide](README.md)
