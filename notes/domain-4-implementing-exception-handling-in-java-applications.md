# Domain 4: Implementing Exception Handling in Java Applications (8%)

One objective, tested two ways. Flow questions put print statements in `try`, `catch`, `finally` and calling methods, and ask for the exact output. Compile questions offer catch, multi-catch, `throws` and resource-header variants, and ask which are legal. Try-with-resources questions ask which exception is primary and what `getSuppressed()` holds. Oracle publishes no domain weights; the percentage here is an estimate.

## 4.1 try/catch/finally, try-with-resources, multi-catch and custom exceptions

**Checked vs unchecked**

| Branch                            | Checked? | Examples                                                                                                                                                 |
| --------------------------------- | -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Error` and subclasses            | No       | `StackOverflowError`, `OutOfMemoryError`, `ExceptionInInitializerError`                                                                                  |
| `RuntimeException` and subclasses | No       | `NullPointerException`, `ArithmeticException`, `IllegalArgumentException` (and `NumberFormatException`), `IllegalStateException`, `UncheckedIOException` |
| Every other `Throwable`           | Yes      | `IOException`, `FileNotFoundException`, `InterruptedException`, `Exception`, `Throwable`                                                                 |

- **Handle or declare:** a checked exception that _can_ be thrown must be caught or listed in `throws`. Sources include a `throw`, a call, a constructor and the hidden `close()`. The check is about what could happen, so a `throw` on a branch that never runs still counts.
- **Lambdas** may throw only what their functional method declares. `Supplier.get()` and `Runnable.run()` declare nothing; `Callable.call()` declares `Exception`.
- **`catch (Exception e)` never catches an `Error`.** A failing static initializer surfaces as `ExceptionInInitializerError`, with the original exception as its cause.

**Custom exceptions**

- Extend `Exception` (or any checked subclass) for checked; extend `RuntimeException` for unchecked. Extending `Throwable` directly gives a checked exception.
- `getMessage()` is `null` when no message was given.
- `toString()` is `ClassName: message`, or just the class name when the message is null. JDK classes print fully qualified.
- `super(cause)` alone sets the message to `cause.toString()`.
- A cause can be set only once. `initCause()` after the cause was set throws `IllegalStateException`.

**Execution order**

- A throw skips the rest of `try`; the **first** matching catch runs, then `finally`, then the code after the statement.
- A `return` in `try` runs `finally` before leaving, and the code after the statement is skipped.
- **Shape:** a plain `try` needs a `catch` or a `finally`; try-with-resources may stand alone. Order is try, catches, then at most one `finally`.
- If the try block and every catch end in `return`/`throw`, a statement after the try statement is unreachable.
- An inner `try/finally` with no matching catch runs its finally and passes the exception outward.

**Catch rules**

| Situation                                                                       | Result                                    |
| ------------------------------------------------------------------------------- | ----------------------------------------- |
| Superclass catch before subclass catch                                          | Compile error ("has already been caught") |
| Subclass before superclass                                                      | Fine                                      |
| Catching a **checked** type the try block cannot throw                          | Compile error ("never thrown in body")    |
| Catching `Exception`, `Throwable` or any unchecked type                         | Always fine                               |
| Checked subclass of a declared type (`FileNotFoundException` for `IOException`) | Fine                                      |

- **Compile time vs run time:** what the try block can throw is judged from **declarations**. Which catch runs is decided by the **actual object**.
- An exception thrown **inside a catch block** skips sibling catches. It runs `finally` and goes outward.
- A single-type catch parameter can be reassigned. Since Java 22 it can be `_`, but the type is still required.

**Multi-catch: `catch (A | B e)`**

- Alternatives must not be related by subclassing (`EOFException | IOException` fails). `UncheckedIOException` is a `RuntimeException`, not an `IOException`.
- The parameter is **implicitly final**.
- Its static type is the **least upper bound**: for `ArrayIndexOutOfBoundsException | ClassCastException` it is `RuntimeException`. Only shared members are callable, and shared interfaces are included.
- A later single catch for a subtype of an alternative is unreachable.

**Precise rethrow**

- `throw e;` inside `catch (Exception e)` throws only the checked types the try block can actually throw, if `e` is effectively final. The method can declare a narrow `throws`.
- Assigning to `e` turns this off.
- A new call in the try block that throws a wider checked type breaks the narrow clause.

**finally and return values**

- The return value is computed **before** `finally` runs. A primitive copy is unaffected. A returned object can still be mutated by `finally`; reassigning the local does nothing.
- **Abrupt completion in `finally` wins:** `return`, `break`, `continue` or `throw` there replaces any pending value or exception. A `return` in finally silently swallows an exception.
- Plain `try/finally` records nothing as suppressed.

**Propagation**

- An uncaught exception runs every `finally` from the inside out until a matching catch stops it. Statements between the throw and the handler are skipped.
- With no handler at all, the thread ends with "Exception in thread "main"".

**try-with-resources**

- Resources must implement `AutoCloseable` (`Closeable` extends it).
- They open left to right and **close right to left**, immediately after the body and **before** the statement's own `catch` and `finally`.
- Resource variables are implicitly final and scoped to the try block.
- Since Java 9 the header may name an existing **effectively final** local, parameter or final field. It cannot hold an arbitrary expression.
- A `null` resource is skipped. `var _ = ...` resources (Java 22) are still opened and closed.
- **Checked exceptions from the implicit `close()`** must be handled. `AutoCloseable.close()` declares `Exception` and `Closeable.close()` declares `IOException`.
  - `StringReader`, `CharArrayReader`, `CharArrayWriter`, `Scanner` and `Formatter` declare none.
  - `StringWriter` and the `ByteArray` streams still declare `IOException`.

| What fails                     | Primary exception                                    | Suppressed                                     |
| ------------------------------ | ---------------------------------------------------- | ---------------------------------------------- |
| Body throws, then closes throw | The body's exception                                 | Each close failure, in closing order           |
| Body succeeds, closes throw    | The first `close()` failure (last-declared resource) | Later close failures                           |
| A resource initializer throws  | The initializer's exception; body never runs         | Close failures of the resources already opened |

- A failing `close()` beats a `return` in the body.
- **Traps:**
  - Expecting `catch (Exception e)` to catch an `Error`.
  - Expecting a sibling catch to handle an exception thrown in a catch.
  - Assuming the catch block can still use the resource (it is already closed).
  - A `finally` return hiding an exception.

📖 Full lesson: [Java Exception Handling: try/catch/finally, try-with-resources and Custom Exceptions (1Z0-831)](https://www.savemycert.com/revision/oracle-java-se-developer/java-exceptions-try-catch-finally-try-with-resources/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide)

[← Back to the study guide](../README.md)
