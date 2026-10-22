# Domain 2: Implementing Program Flow Control Using Decision and Looping Constructs (8%)

One objective, but it shows up inside questions from every other domain. Expect programs that test the dangling else, colon-switch fall-through, switch expressions and their exhaustiveness, pattern labels and their order, loop conditions with increments, labelled `break`/`continue`, and unreachable code. Before computing any output, check whether the code compiles at all. Oracle publishes no domain weights; the percentage here is an estimate.

## 2.1 if/else, switch statements and expressions, and loops

**if/else**

- Without braces, an `else` binds to the **nearest unmatched `if`**. Indentation means nothing.
- Each branch is one statement. A second indented line after `else` always runs.
- An `else if` chain runs only the **first** true branch.
- Conditions of `if`, `while`, `do` and `for` must be `boolean` or `Boolean`. Java never treats a number or `null` as true or false.
  - `if (qty = 0)` with an `int` fails. `if (open = qty < 5)` with a `boolean` compiles: it assigns, then tests the assigned value.
  - Unboxing a `null` `Boolean` throws `NullPointerException`. `ok != null && ok` is safe; a separate later test of the same variable is not.

**Switch labels**

| Feature                | `case X:` (colon)              | `case X ->` (arrow)                                 |
| ---------------------- | ------------------------------ | --------------------------------------------------- |
| Fall-through           | Yes, until `break`             | Never                                               |
| Several constants      | `case 1, 2:` or stacked labels | `case 1, 2 ->`                                      |
| Body                   | Any statements                 | One expression statement, one block, or one `throw` |
| Value in an expression | `yield`                        | The expression, or `yield` inside a block           |
| Variable scope         | Rest of the switch block       | That arm's block only                               |

- **Fall-through:** find the entry label, then read straight down, ignoring labels, until `break`, `return`, `continue`, a throw or the closing brace.
- **`default` position** never changes which label matches (every constant is tried first). It only changes which statements follow it.
- A local variable declaration straight after an arrow fails; wrap it in braces.
- Two statements after an arrow without braces: the second does not compile.
- Mixing colon and arrow labels in one switch fails ("different case kinds").
- A variable declared in an earlier colon group is in scope later but only initialised on the path that ran its declaration.

**Selector types and constants**

- Constant labels allow selectors of type `char`, `byte`, `short`, `int`, their wrappers, `String`, or an enum.
- `long`, `float`, `double` and `boolean` selectors **do not compile** in Java 25. Primitive patterns are a preview feature, and preview features are outside the exam.
- Any other reference type (Object, a sealed interface) needs pattern labels or qualified enum constants.
- Each label must be a compile-time constant that fits the selector type, and no constant may repeat.
  - `case 40000` on a `short` fails.
  - A `final` local initialised in its declaration with a constant works. One assigned later does not.
  - A `static final String` built with a method call is not a constant.
  - A `final` wrapper (`Character`) is never a constant variable.
- **Enum labels:** `case LOW` or (since Java 21) `case Level.LOW`. On an `Object` selector only the qualified form resolves.
- A String switch uses `equals()`, so case matters.

**Switch expressions**

- They produce a value, so the enclosing statement ends with `;`. Every path must yield a value or throw.
- Colon groups still fall through inside an expression, and `yield` ends it.
- `return`, `break` or `continue` out of a switch expression is a compile error.
- **Exhaustive** if it has any of:
  - `default` (or `case null, default`).
  - An **unguarded** type pattern of the selector's type or a supertype (`case String other`).
  - Every enum constant.
  - Every permitted subtype of a sealed type.
- A **guarded** pattern never counts towards exhaustiveness.
- A new enum constant or sealed subtype that the switch was not recompiled for throws `MatchException`.
- Switch **statements** with only constant labels need not be exhaustive. A statement using a pattern label or `case null` must be.

| Arms (inferred type)            | Result type                                           |
| ------------------------------- | ----------------------------------------------------- |
| `7` and `1.5f`                  | `float`: the `7` arm prints `7.0`                     |
| `'a'` and the constant `48`     | `char`: 48 prints as `0`                              |
| `'a'` and the constant `70000`  | `int`: `'a'` prints as `97`                           |
| `Character` var and `short` var | `int` (unboxed, promoted)                             |
| `"n/a"` and `4L`                | Common reference supertype; `4L` is boxed to a `Long` |

**null and patterns**

- A `null` selector throws `NullPointerException` unless there is a `case null`. **`default` does not catch null.**
- `case null, default ->` is the only list that includes `default`, and `null` must come first.
- Type patterns (`case Double d`) bind a variable; `when` adds a guard. Labels are tried top to bottom and the first match wins.
- **Dominance** (compile error, "dominated by a preceding case label"):
  - An unguarded pattern before a subtype pattern of the same type (`Number n` before `Double d`).
  - An unguarded pattern before a constant label of its type (`String t` before `"yes"`).
  - `default` before a pattern label.
  - A **guarded** pattern dominates nothing, so it may come first.
- **Record patterns:** `case Range(int lo, int hi)` deconstructs in declaration order.
  - Each component can be a type pattern, `var x`, `_` or a nested record pattern.
  - The number of component patterns must match exactly.
  - `Integer n` on an `Object` component is a run-time test. `int n` on an `Object` or `Integer` component is a primitive pattern (preview), so it fails.
  - Binding names cannot repeat in one label.

**Loops**

- `while` tests first (zero or more runs); `do-while` runs at least once and ends with `;`.
- A variable declared inside a `do` block is out of scope in its `while` condition.
- **Increments in conditions:** `--t > 2` compares the new value, `t-- > 2` the old. The final failing test still decrements, so trace it too.
- **Basic for:** the initializer runs once, then test, body, update, test. `for (;;)` loops forever.
  - The initializer can declare several variables of **one** type only.
  - The update can list several expression statements.
- **Loop-variable scope:** an initializer variable exists only in the loop. Redeclaring a name that is still in scope fails.
- **Enhanced for:** works on an array or any `Iterable`, but **not** on an `Iterator` or a `Stream`.
  - The variable is a copy. Reassigning it changes nothing.
  - Mutating the object it points to changes the shared element.

**break, continue and labels**

- `break` leaves the innermost loop or switch. `continue` goes to the next test (after the update in a `for`).
- Inside a switch inside a loop, `break` leaves only the switch, and `continue` moves to the next loop iteration.
- `break label` works on any labelled statement, a plain block included. `continue label` needs a **loop** label ("not a loop label").
- A label is visible only inside the statement it labels ("undefined label").

**Unreachable code**

| Code                                             | Compiles?             |
| ------------------------------------------------ | --------------------- |
| `while (1 > 2) { ... }` or `for (; 3 < 2; )`     | No: constant false    |
| `final int LIMIT = 0; while (LIMIT > 0)`         | No: constant variable |
| `boolean off = false; while (off)`               | Yes: not constant     |
| `if (1 > 2) { ... }`                             | Yes: `if` is exempt   |
| `while (true)` with no `break`, then a statement | No: unreachable       |
| `while (true)` with a `break`, then a statement  | Yes                   |

- A `break` that only leaves a switch inside `while (true)` does not make the loop completable.
- A method ending in an infinite loop needs no final `return`. A loop with a non-constant condition does ("missing return statement").
- **Traps:**
  - `default` placed mid-switch.
  - A dangling `else` hidden by indentation.
  - Guarded vs unguarded pattern order.
  - `default` assumed to catch `null`.
  - `continue` inside a switch expression.

📖 Full lesson: [Java Flow Control: if/else, switch and Loops (1Z0-831)](https://www.savemycert.com/revision/oracle-java-se-developer/java-flow-control-if-switch-loops-break-continue/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide)

[← Back to the study guide](../README.md)
