# Domain 10: Developing Applications with Localization Support (7%)

One objective: locales, resource bundles, and parsing and formatting messages, numbers, currency, percentages, dates and times. Questions list several `.properties` files (or a `ListResourceBundle` class) and a default locale, then ask which bundle loads and what each key returns. Others ask for the exact output of a formatter, where locale symbols and rounding mode decide the digits. Oracle publishes no domain weights; the percentage here is an estimate.

## 10.1 Locales, resource bundles and formatting

**Creating locales**

| API                                                        | Bad input                                                                     |
| ---------------------------------------------------------- | ----------------------------------------------------------------------------- |
| `Locale.of("de", "CH")` (Java 19+)                         | No validation; case normalized; `Locale.of("de-CH")` becomes language `de-ch` |
| `Locale.forLanguageTag("de-CH")`                           | Never throws for an ill-formed tag; `"de_CH"` gives `Locale.ROOT`             |
| `new Locale.Builder()...build()`                           | Validates; throws the unchecked `IllformedLocaleException`                    |
| Constants (`Locale.UK`, `Locale.GERMANY`, `Locale.FRENCH`) | n/a                                                                           |

- `toString()` uses underscores (`ja_JP`, the bundle-file suffix). `toLanguageTag()` uses hyphens (`ja-JP`).
- `Locale.ROOT` prints as an empty string; its tag is `und`.

**Default locales**

| Call                                       | Effect                                           |
| ------------------------------------------ | ------------------------------------------------ |
| `Locale.setDefault(loc)`                   | Sets the general default **and** both categories |
| `Locale.setDefault(Category.FORMAT, loc)`  | FORMAT only                                      |
| `Locale.setDefault(Category.DISPLAY, loc)` | DISPLAY only                                     |

- **FORMAT** is read by number, date and message formatters, `String.format`, `formatted` and `printf` when no locale is passed.
- **DISPLAY** is read by `getDisplayCountry()` and the other display names.
- **The general default** is read by `Locale.getDefault()` and the `getBundle` fallback.
- Order matters: a later one-argument `setDefault` wipes earlier category settings. An explicit `Locale` argument always wins.

**ResourceBundle.getBundle(base, locale)**

- The search order for `es_MX` with default `de_DE`:
  1. `Menu_es_MX`, then `Menu_es`.
  2. Only if neither exists: `Menu_de_DE`, then `Menu_de`.
  3. Then the base `Menu`.
  4. Otherwise `MissingResourceException`.
- **A partial match on the requested locale beats the default.** `es_AR` with only `Menu_es` returns `Menu_es`.
- A `Locale.ROOT` request returns the base bundle when it exists. The default locale is tried only when there is no base bundle.
- `getLocale()` reports the bundle actually loaded, not the one requested.
- **Parent chain for keys:** `es_MX` → `es` → base. The default locale's files and sibling locales are **never** parents. `keySet()` and `containsKey()` cover the whole chain.
- **`MissingResourceException`** is unchecked. `getBundle` throws it when no bundle exists (it never returns null), and `getString` when the key is in no bundle of the chain. `getKey()` names the missing key.
- **Properties vs class:**
  - A `ListResourceBundle` class beats a same-named `.properties` file.
  - Property values are always Strings.
  - `getObject` returns the stored object.
  - `getString` casts, so an `Integer` value throws `ClassCastException`.
  - `getStringArray` returns a `String[]`.

**MessageFormat**

- `MessageFormat.format(pattern, args...)` uses FORMAT; `new MessageFormat(p, locale)` fixes the locale.
- `{n}` refers to the argument **position** and can repeat. An unmatched `{2}` prints literally.
- **A single quote starts literal text**: `Don't stop {0}` prints `Dont stop {0}`. Write `''` for an apostrophe.
- A plain `{0}` formats Numbers with grouping and uses `toString()` for other objects.
- Format types:
  - `{0,number,integer}` rounds HALF_EVEN.
  - `{0,number,percent}` multiplies by 100.
  - `{0,number,currency}` uses the locale's currency.
  - `{0,date}` accepts a `Date` or `Number` and throws for a `LocalDate`.
  - `{0,choice,0#none|1#one|1<{0} many}` picks by limit.
- `parse()` returns an `Object[]` and throws the checked `ParseException`.

**NumberFormat**

| Factory                                     | Fractions             | UK example                            |
| ------------------------------------------- | --------------------- | ------------------------------------- |
| `getInstance` / `getNumberInstance`         | 0 to 3                | 4321.98765 → `4,321.988`              |
| `getIntegerInstance`                        | 0                     | 4.5 → `4`                             |
| `getCurrencyInstance`                       | The currency's digits | -7.5 → `-£7.50`                       |
| `getPercentInstance`                        | 0                     | 0.738 → `74%`                         |
| `getCompactNumberInstance(loc, SHORT/LONG)` | 0                     | 3,500 → `4K`; 7,000,000 → `7 million` |

- **HALF_EVEN rounding** in NumberFormat and DecimalFormat: 4.5 → 4, 5.5 → 6, 0.625 → 0.62. Only exact binary ties count.
- Percent expects a ratio, so 40 prints `4,000%`.
- `setCurrency` swaps the symbol but keeps the locale's layout.
- Dutch and German swap the separators: `5.432,1`.
- **`parse()`** reads the longest leading number and ignores the rest.
  - It returns a `Long` when the value is integral, otherwise a `Double`.
  - An integer instance stops at the decimal separator; parsing never rounds.
  - A currency instance requires the symbol.
  - It throws the **checked** `ParseException`.

| Exception                | Thrown by                                   | Checked? |
| ------------------------ | ------------------------------------------- | -------- |
| `ParseException`         | `NumberFormat.parse`, `MessageFormat.parse` | Yes      |
| `NumberFormatException`  | `Integer.parseInt`, `Double.parseDouble`    | No       |
| `DateTimeParseException` | java.time parsing                           | No       |

**DecimalFormat and String.format**

- **DecimalFormat patterns:**
  - `0` is a required digit, `#` an optional one, `,` sets grouping and `%` multiplies by 100.
  - `;` starts a negative subpattern: `#,##0.0;[#,##0.0]` prints `[98,765.2]`.
  - The locale supplies only the symbols (`DecimalFormatSymbols`).
- **`String.format`, `formatted` and `printf`** use FORMAT unless given a Locale; `formatted` has no locale parameter.
  - `%,.2f` means grouping with 2 decimals.
  - **Formatter rounds HALF_UP** (`%.1f` of 6.25 is 6.3), unlike DecimalFormat.
- String concatenation of a `double` is never localized.

**DateTimeFormatter**

| Letter                | Meaning                                              |
| --------------------- | ---------------------------------------------------- |
| `M`/`MM`/`MMM`/`MMMM` | Month: number, padded, short name, full name         |
| `m`/`mm`              | **Minute**                                           |
| `d`/`dd`              | Day of month (`dd` requires two digits when parsing) |
| `EEE`/`EEEE`          | Day of week                                          |
| `H` / `h` / `K`       | Hour 0–23 / 1–12 / 0–11                              |
| `a`                   | AM/PM (text depends on the locale)                   |
| `uuuu` / `yyyy`       | Proleptic year / year of era                         |
| `'text'`              | Literal                                              |

- Pattern mistakes compile. `mm` prints minutes. Formatting a `LocalDate` with `HH` throws `UnsupportedTemporalTypeException`.
- **Localized styles** (`ofLocalizedDate(FormatStyle.SHORT)`) differ per locale: SHORT is `6/4/27` in the US and `04/06/2027` in the UK.
  - LONG and FULL **time** styles need a zone, so on a `LocalDateTime` they throw `DateTimeException` ("Unable to extract ZoneId").
- **Resolver styles** for `"2027-04-31"`:
  - SMART (the pattern default) gives April 30.
  - STRICT (`LocalDate.parse` default) throws.
  - LENIENT gives May 1.
  - STRICT needs `uuuu`; `yyyy` fails without an era.
- Parsing month names needs the right locale and width (`MMMM` for full names).

📖 Full lesson: [Java Localization: Locales, Resource Bundles and Formatting Numbers, Dates and Messages (1Z0-831)](https://www.savemycert.com/revision/oracle-java-se-developer/java-localization-resource-bundles-number-date-formatting/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide)

[← Back to the study guide](../README.md)
