# Domain 1: Handling Date, Time, Text, Numeric and Boolean Values (11%)

This domain covers primitives and wrappers, how expressions are typed and evaluated, String and StringBuilder, text blocks, and the java.time API. Nearly every question is a short program: what does it print, does it compile, or which exception is thrown. Track the type of every intermediate value as well as its value, because the type decides whether the code compiles and how the result prints. Oracle publishes no domain weights; the percentage here is an estimate based on the share of objectives.

## 1.1 Primitives, wrapper classes, operators and casting

- **Eight primitives:** `byte` (8-bit), `short` (16), `char` (16, unsigned, 0 to 65,535), `int` (32), `long` (64), `float` (32), `double` (64), `boolean` (`true`/`false` only, never converts to or from a number).
- **Literal rules:**
  - A whole number is an `int` unless suffixed `L`. `long v = 5_000_000_000;` fails ("integer number too large").
  - A decimal is a `double` unless suffixed `f`. `float f = 2.25;` fails.
  - Underscores only **between** digits. `9_999_` and `12_345_.67` fail. `_9999` is an identifier.
  - Prefixes: `0b` binary, leading `0` octal (`012` is 10), `0x` hex.
- **Widening** is automatic: `byte → short → int → long → float → double`, and `char → int`. `int`→`float` and `long`→`float`/`double` can lose precision silently.
- **Binary numeric promotion** (before every arithmetic, comparison and bitwise operator):
  1. Either operand `double` → both `double`.
  2. Else either `float` → both `float`.
  3. Else either `long` → both `long`.
  4. Else **both become `int`**, even two `byte`s or two `char`s.
- So `short r = p + q;` does not compile. Cast the **whole** expression: `(short) (p + q)`. `(short) p + q` still produces an `int`.
- **Constant narrowing:** an `int`/`short`/`char`/`byte` compile-time constant that fits may be assigned to `byte`/`short`/`char` without a cast. `byte b = 100;` compiles, `byte b = 128;` does not. A non-`final` variable is never a constant. The rule never applies to a `long` source.
- **No widen-then-box:** `Long x = 9;` and `Double d = 9;` fail. Narrow-then-box of a small constant is allowed: `Byte bx = 10;`, `Character cx = 65;`.

| Cast                                     | What happens to the value                                                 |
| ---------------------------------------- | ------------------------------------------------------------------------- |
| Integer → smaller integer                | Keeps low-order bits, so it wraps: `(byte) 250` is -6                     |
| `double`/`float` → `int`/`long`          | Truncates toward zero, saturates at MIN/MAX, NaN → 0: `(int) -4.75` is -4 |
| `double`/`float` → `byte`/`short`/`char` | First to `int` (saturating), then keeps low-order bits                    |
| `boolean` ↔ anything                     | Compile error                                                             |

- A cast is unary, so it binds tighter than `*` and `/`: `(int) 9.8 / 4` is `9 / 4` = 2.
- **Compound assignment** `x op= y` means `x = (T)(x op y)`, so it hides a narrowing cast. `b *= 3` compiles for a `byte` where `b = b * 3` does not, and it can overflow silently. `++`/`--` also keep the variable's type.
- **Prefix vs postfix:** `++x` yields the new value, `x++` the old. Operands evaluate left to right and the assignment stores last, so `n = n++;` leaves `n` unchanged.
- **Integer arithmetic:**
  - `/` truncates toward zero; `%` takes the sign of the **dividend** (`-9 % 4` is -1).
  - Truncation happens in the operand type: `double avg = 9 / 4;` stores 2.0.
  - `Math.floorDiv`/`floorMod` round toward negative infinity; `floorMod` takes the sign of the **divisor** (`Math.floorMod(-3, 12)` is 9).
  - Overflow wraps silently in the type the expression is computed in: `long a = 1_000_000 * 3_000;` overflows. Make the first operand `long`, or use `Math.addExact`/`multiplyExact`/`toIntExact`, which throw `ArithmeticException`.

| Expression | Integer operands                    | Floating-point operands   |
| ---------- | ----------------------------------- | ------------------------- |
| `x / 0`    | `ArithmeticException` ("/ by zero") | `Infinity` or `-Infinity` |
| `0 / 0`    | `ArithmeticException`               | `NaN`                     |
| `x % 0`    | `ArithmeticException`               | `NaN`                     |

- **NaN** is not `==` to anything, itself included; use `Double.isNaN`. `Double.equals` treats NaN as equal to NaN. `0.1 + 0.7 == 0.8` is false.
- **Precedence** (high to low): postfix, unary and cast, `* / %`, `+ -`, shifts, relational and `instanceof`, `== !=`, `&`, `^`, `|`, `&&`, `||`, `?:`, assignment. Assignment and `?:` group right to left.
- **`+` with Strings:** left to right, pair by pair. `4 + 5 + "x" + 4 + 5` is `9x45`. `'b' + 2 + "c"` is `100c` (a `char` before the first String is numeric).
- **Boolean operators:** `&&` and `||` short-circuit; `&`, `|`, `^` on booleans always evaluate both sides. `&&` binds tighter than `||`, so `a || b && c` is `a || (b && c)`; if `a` is true, every side effect in the group is skipped.
- **Ternary type:** decided from both branches at compile time. Numeric branches are promoted: `Object o = ready ? 7 : 3.5f;` stores a `Float` 7.0.
- **Wrappers:**
  - Unboxing `null` throws `NullPointerException`. A wrapper field defaults to `null`.
  - Wrappers are immutable. `n++` on an `Integer` boxes a new object.
  - `wrapper.equals(x)` is true only for the **same wrapper type**: `Long.valueOf(9).equals(9)` is false (9 boxes to `Integer`).
  - `wrapper == primitive` unboxes and compares numbers. `wrapper == wrapper` compares references. `Integer == Long` does not compile.
  - `Integer.valueOf` caches -128 to 127, so `==` happens to work there. Never rely on it.

| Method                    | Returns   | Note                                                         |
| ------------------------- | --------- | ------------------------------------------------------------ |
| `Integer.parseInt(s)`     | `int`     | Optional radix; `NumberFormatException` on bad text          |
| `Integer.valueOf(s)`      | `Integer` | May return a cached instance                                 |
| `Integer.decode(s)`       | `Integer` | Understands `0x`, `#` and leading-`0` octal                  |
| `Integer.getInteger(s)`   | `Integer` | Reads a **system property**; does not parse its argument     |
| `Boolean.parseBoolean(s)` | `boolean` | True only for "true" in any case; "yes" and `null` are false |

- **Math return types** are part of the answer:
  - `Math.round(double)` → `long`, `round(float)` → `int`; ties go toward positive infinity (`round(-3.5)` is -3).
  - `floor`, `ceil`, `rint`, `pow`, `sqrt` → `double`. One `floor` call turns a whole sum into a `double`.
  - `rint` breaks ties to even (`rint(4.5)` is 4.0).
  - `Math.clamp` (Java 21) has four overloads; there is no `clamp(int, int, int)`, so an `int` value uses `clamp(long, int, int)` and returns `int`. Throws `IllegalArgumentException` if min > max.
- **Traps:**
  - Casting one operand instead of the whole expression.
  - `Math.abs(Long.MIN_VALUE)` is still negative.
  - `(int) -2.6` is -2, not -3.

📖 Full lesson: [Java Primitives, Wrapper Classes, Operators and Casting (1Z0-831)](https://www.savemycert.com/revision/oracle-java-se-developer/ocp-java-primitives-wrappers-operators-casting/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide)

## 1.2 Strings, StringBuilder and text blocks

- **String is immutable.** `toUpperCase()`, `concat()`, `replace()`, `strip()`, `substring()` return a new String. An unassigned result is lost.
- **StringBuilder is mutable.** Its methods change the builder and return `this`, so chained calls each see the previous result and `q = p.append("2")` makes `p == q` true.
- **Arguments:** Java passes a copy of the reference. Reassigning a String or StringBuilder parameter is invisible to the caller. Mutating a StringBuilder argument is visible.
- **Identity and the pool:**
  - Literals are interned and shared. Text blocks are literals too.
  - Compile-time constant expressions are folded: `"Du" + "ke"` and `P + "ke"` (where `P` is a `final` constant) are the pooled `"Duke"`.
  - Concatenation with a non-final variable, and `new String(...)`, create new objects.
  - `intern()` returns the pooled instance.
  - `==` asks about identity; compare content with `equals()`.
- **Indexes:** zero-based; ranges are start-inclusive, end-exclusive. `substring(b, e)` needs `0 <= b <= e <= length()`. `substring(length())` is `""`. Violations throw `StringIndexOutOfBoundsException` at run time, never at compile time.
- `indexOf`/`lastIndexOf` return -1 when not found and never throw for an out-of-range `fromIndex`.
- **`+` mixing:** `'x' + 'y' + "!"` is `241!`; `"!" + 'x' + 'y'` is `!xy`. `null` concatenates as `"null"`. With `t += m + 2`, the right side is evaluated first as its own expression.

| Method                 | Behaviour                                                      |
| ---------------------- | -------------------------------------------------------------- |
| `trim()`               | Removes leading/trailing chars `<= U+0020` only                |
| `strip()` and variants | Unicode-aware (`Character.isWhitespace`)                       |
| `isEmpty()`            | `length() == 0`                                                |
| `isBlank()`            | Empty or only white space (Unicode-aware)                      |
| `compareTo()`          | First char difference, or length difference if one is a prefix |
| `split(regex)`         | Regex; drops **trailing** empty strings; limit `-1` keeps them |
| `repeat(n)`            | n copies total; `repeat(0)` is `""`                            |
| `String.join(d, ...)`  | Delimiter only between elements                                |

- Uppercase letters sort before lowercase: `"Mango".compareTo("kiwi")` is negative.
- `"1.2.3.4".split(".")` is an empty array, because `.` is a regex metacharacter. Escape it as `"\\."`.
- **Format strings** are checked at run time: `%d` with a `double` throws `IllegalFormatConversionException`. `%b` is true for any non-null non-Boolean. `%c` accepts an `int` code point. The `0` flag is numeric only.
- **StringBuilder methods:** `append`, `insert(offset, x)` (offset may equal `length()`), `delete(start, end)`, `deleteCharAt`, `replace(start, end, str)` (a range, unlike `String.replace`), `reverse`, `setCharAt`, `setLength` (truncates or pads with `'\u0000'`).
- **Index checks use `length()`, not capacity.** `new StringBuilder(16)` has length 0. `delete`/`replace` clamp an end past the length; `charAt`/`setCharAt`/`deleteCharAt`/`substring` throw.
- **StringBuilder equality:** `equals()` is inherited from Object (identity). Compare content with `compareTo() == 0`, `str.contentEquals(sb)` or `sb.toString().equals(str)`. `"node".equals(sb)` is always false.
- **Text blocks:**
  - The opening `"""` must be followed by a line terminator. Content on the same line is a compile error.
  - The compiler normalizes line endings to `\n`, strips incidental indentation and trailing spaces, then processes escapes. An invalid escape such as `\w` is a compile error.
  - The closing delimiter's position controls indentation: level with the content strips it all; further left keeps extra spaces. On its own line it adds a trailing `\n`; at the end of the last line it does not.
  - Two text-block-only escapes: `\<newline>` joins lines, `\s` is one space that survives trailing-space stripping.
- **Multi-line helpers:**
  - `lines()` gives a `Stream<String>`, and a final terminator adds no empty line.
  - `indent(n)` adds or removes leading spaces and **always ends with `\n`**.
  - `stripIndent()` removes common indentation and adds no terminator.
  - `translateEscapes()` interprets literal backslash sequences at run time.
  - `chars()` gives an `IntStream` of numbers.
- **Traps:** a `fromIndex` taken from the original string applied to a stripped copy; `sb1.equals(sb2)` with the same content is false.

📖 Full lesson: [Java Strings, StringBuilder and Text Blocks (OCP Java SE 25)](https://www.savemycert.com/revision/oracle-java-se-developer/java-string-stringbuilder-text-blocks/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide)

## 1.3 Date-Time API: dates, times, durations, periods and time zones

| Type             | Holds                                     | `toString()` example                    |
| ---------------- | ----------------------------------------- | --------------------------------------- |
| `LocalDate`      | Date, no time, no zone                    | `2026-08-21`                            |
| `LocalTime`      | Time of day to the nanosecond             | `14:20` (seconds omitted when zero)     |
| `LocalDateTime`  | Date + time, no zone                      | `2026-08-21T14:20`                      |
| `Instant`        | A moment on the UTC timeline              | `2026-08-21T12:20:00Z` (always seconds) |
| `OffsetDateTime` | Local date-time + fixed `ZoneOffset`      | `2026-08-21T14:20+02:00`                |
| `ZonedDateTime`  | Local date-time + offset + `ZoneId` rules | `...+02:00[Europe/Madrid]`              |
| `Duration`       | Seconds + nanos                           | `PT1H35M`                               |
| `Period`         | Years, months, days                       | `P2Y4M9D`                               |

- **No public constructors.** Use `of()`, `now()`, `parse()`, `ofEpochSecond()`, `atTime()`, `atZone()`, `toInstant()`.
- **`of()` validates at run time** and throws `DateTimeException`: months 1 to 12, hours 0 to 23 (there is no 24:00), February 29 only in leap years.
- **Immutable:** `plusX`/`minusX`/`withX` return new objects. `d.minusWeeks(2);` on its own line does nothing.
- **Month ends:**
  - `plusMonths`, `withMonth` and `withYear` **clamp** to the last valid day.
  - `withDayOfMonth(31)` on a 30-day month **throws**.
  - Clamping happens per step: Aug 31 + 1 month + 1 month is Oct 30, but Aug 31 + 2 months is Oct 31.
- **LocalTime wraps** past midnight silently (`21:30` plus 4 hours is `01:30`). `MIDNIGHT.minusSeconds(1)` is `23:59:59`, not `LocalTime.MAX`.
- **Duration vs Period printing:**
  - Duration has no day part: `Duration.ofDays(3)` is `PT72H`; negatives sign each part (`PT-2H-15M`).
  - Period keeps what you gave it: `ofWeeks(2)` is `P14D`. `normalized()` folds months into years only.
- **Static factory trap:** `Period.ofDays(10).ofWeeks(1)` compiles and is `P7D`. Build mixed periods with `Period.of(y, m, d)` or `withMonths()`/`plusDays()`.
- **Duration accessors:** `toMinutes()` and `getSeconds()` return totals; `toMinutesPart()`/`toSecondsPart()` return the displayed component.
- **Measuring between values** counts complete units only, and the result is negative when the end is earlier:
  - `Period.between(a, b)` / `a.until(b)` → a Period.
  - `ChronoUnit.X.between(a, b)` → a `long` count.
  - `Duration.between(a, b)` needs time units, so it fails on two `LocalDate`s.
  - 31 March to 30 April is **0 months**, because the end day has not reached the start day.
  - `getMonths()` is the component; `toTotalMonths()` is years × 12 + months.
  - LocalTime differences never wrap: 21:00 to 03:00 is `PT-18H`.

| Type                           | Time units / Duration | Date units / Period |
| ------------------------------ | --------------------- | ------------------- |
| `LocalDate`                    | No                    | Yes                 |
| `LocalTime`                    | Yes                   | No                  |
| `LocalDateTime`, zoned, offset | Yes                   | Yes                 |
| `Instant`                      | Yes                   | `DAYS` only         |

- Unsupported combinations **compile** and throw `UnsupportedTemporalTypeException` at run time. Adding a Duration to a LocalDate complains about Seconds. A zero Period touches no field, so it never fails.
- **Zones:**
  - `ZoneOffset` is fixed. A region `ZoneId` carries rules, DST included. Asia/Kolkata has one offset all year but `isFixedOffset()` is still false.
  - `withZoneSameInstant` keeps the moment and changes the clock. `withZoneSameLocal` keeps the clock and changes the moment.
  - `equals()` compares local date-time, offset and zone. `isEqual`/`isBefore`/`isAfter` compare the instant.
  - Converting to a local type drops the zone, so comparisons can flip.
- **DST:**
  - **Gap** (spring forward): the time moves forward by the gap length with the summer offset. `ofStrict` throws instead.
  - **Overlap** (fall back): the earlier offset wins by default; `withLaterOffsetAtOverlap()` picks the second occurrence.
- **Date-based vs time-based on ZonedDateTime:** `plusDays`, `Period` and `ChronoUnit.DAYS` keep the wall-clock time. `plusHours` and **any** Duration (including `Duration.ofDays(1)`) add elapsed time. On a fall-back day, `plusHours(24)` lands one hour earlier on the clock than `plusDays(1)`.
- **TemporalAdjusters:** `next` always moves at least one day; `nextOrSame` returns the date if it already qualifies. `lastDayOfMonth()` knows leap years. Also `firstDayOfNextMonth`, `lastInMonth`, `dayOfWeekInMonth`.
- **Traps:** `Duration.ofDays(1)` vs `ChronoUnit.DAYS` across DST; `getMonths()` vs `toTotalMonths()`; adding a Duration to a LocalDate.

📖 Full lesson: [Java Date-Time API: Dates, Durations, Periods and Time Zones (OCP Java SE 25)](https://www.savemycert.com/revision/oracle-java-se-developer/java-date-time-api-period-duration-time-zones/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide)

[← Back to the study guide](../README.md)
