# Domain 3: Applying Object-Oriented Principles in Java Programs (18%)

The largest section of the exam: seven objectives covering objects and nested classes, classes and records, overloading, scope and immutability, inheritance and sealed types, interfaces, and enums. Items are short programs. You trace initialization order, decide which overload or override runs, spot illegal declarations, and count objects eligible for garbage collection. "Compilation fails" is a real answer on many of them. Oracle publishes no domain weights; the percentage here is an estimate.

## 3.1 Objects, nested classes and the object life cycle

- **A variable holds a reference, not an object.** Only `new` (and array creation) creates objects. Count `new` expressions that actually run, not variables.
- `new` sets every field to its default (0, false, `'\u0000'`, null), runs the constructor chain and returns a reference.
- An unassigned reference field is `null`, so calling a method through it compiles and then throws `NullPointerException`. A local variable has no default; reading it unassigned is a compile error.
- **Aliasing:** a field write through either alias changes the shared object. Reassigning one variable moves only that variable.
- **Pass-by-value of the reference:** a method can mutate the caller's object but can never reseat the caller's variable.

| Nested kind          | Enclosing instance?         | Created with                                           |
| -------------------- | --------------------------- | ------------------------------------------------------ |
| Static nested class  | None                        | `new Outer.Nested()`                                   |
| Inner (member) class | Exactly one, fixed for life | `outerRef.new Inner()`; `new Inner()` in instance code |
| Local class          | Yes in an instance context  | `new Local()` inside its block, after the declaration  |
| Anonymous class      | Yes in an instance context  | `new Type(args) { ... }`                               |

- **Implicitly static:** member and local records, enums and interfaces. They cannot read outer instance fields or capture locals.
- **Instantiation near-twins:**
  - `new Outer().new Inner()` and `ref.new Inner()` compile.
  - `new Inner()` in a static method fails ("non-static variable this cannot be referenced from a static context"), and `new Outer.Inner()` fails because it supplies no enclosing instance.
  - `new ref.Inner()` is not valid syntax.
  - `new Outer().new Nested()` for a static nested class fails.
- Inner objects made from the same outer object share its state. Since Java 16 an inner class may declare static members.
- **Shadowing:** the innermost declaration wins. `Outer.this.name` reaches an enclosing instance's field, and it reads the live value, not a snapshot.
- **Capture:** local and anonymous classes (and lambdas) may use only final or effectively final locals.
  - A classic `for` index is not effectively final.
  - An enhanced-for variable, or a local declared in the loop body, is.
- **`this`:** inside an anonymous class it is the anonymous object; inside a lambda it is the enclosing instance.
- An anonymous class can extend an abstract class, passing arguments to its constructor, but cannot declare its own constructor.
- **GC eligibility:** an object is eligible when no live thread can reach it from a root (running methods' locals and parameters, static fields, live threads).
  - An unreachable cycle ("island of isolation") is eligible.
  - Setting a reference to `null` helps only if it was the last path.
  - Arrays are objects too.
- `System.gc()` is only a request; it guarantees nothing. `finalize()` is deprecated for removal.
- A reachable inner object that uses its outer object keeps the outer object alive. Prefer a static nested class when the link is not needed.

📖 Full lesson: [Java Objects, Nested Classes and Garbage Collection (1Z0-831)](https://www.savemycert.com/revision/oracle-java-se-developer/java-objects-nested-classes-garbage-collection/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide)

## 3.2 Classes, records, constructors and initializers

- **Static vs instance:** a static field is one variable per class; an instance field is one per object. Static code cannot use an instance member without naming an object.
- **Constructors:** same name as the class, no return type. Not inherited; cannot be `static`, `final` or `abstract`.
  - The **default constructor** exists only when the class declares none.
  - `int Gauge() { }` is a method, not a constructor.
- **Chaining:** each constructor calls `this(...)` or `super(...)`, or the compiler inserts `super();`.
  - The implicit `super()` needs an accessible no-arg superclass constructor.
  - `this(...)` chains complete inside-out.
  - Only the constructor that calls `super(...)` runs the instance initializers, once per object.
- **Flexible constructor bodies (Java 25, JEP 513):** statements may come before `this(...)`/`super(...)`.

| In the prologue you may                                           | You may not                                                          |
| ----------------------------------------------------------------- | -------------------------------------------------------------------- |
| Use parameters and locals; call static methods; use static fields | Read any instance field                                              |
| Throw exceptions                                                  | Call instance methods or pass `this`                                 |
| Assign this class's own fields **that have no initializer**       | Assign fields with an initializer, or inherited fields; use `return` |

- **Construction order for `new Leaf()`:**
  1. Prologues run from the subclass **up**.
  2. Then, from the top class **down**, each class runs its field initializers and instance blocks in source order, then its constructor body.
  - Example: `leaf-pro mid-pro top-pro top-body mid-init mid-body leaf-init leaf-body`.
- **Static initialization** runs once per class, parent first, on first **active use**: creating an instance, calling a static method, or reading a non-constant static field.
- Reading a **constant variable** (`static final` primitive or String with a constant initializer) is inlined and does **not** initialize the class. A `static final Long` or a value from a method call does.
- A method called from an earlier initializer sees later fields at their **defaults**. A direct forward reference by simple name is a compile error.
- **Overridable call from a superclass constructor:** the subclass override runs before subclass fields are initialized, so it sees null/0, unless the subclass assigned them in its prologue.
- **Blank finals:** assigned exactly once on every constructor path (or in an instance initializer). Never in a method. Errors: "might not have been initialized" and "might already have been assigned".
- **Checked exceptions** in an instance initializer must be caught there or declared by every constructor. Static initializers may not throw checked exceptions.
- **Records** (`record Seat(int row, char letter) {}`) generate:
  - A private final field per component.
  - A canonical constructor with the record's access level.
  - Public accessors named exactly after the components (`row()`, not `getRow()`).
  - `equals`, `hashCode` and `toString` (`Seat[row=4, letter=C]` in current JDKs).
- Records are implicitly final and extend `java.lang.Record`. They may implement interfaces.

| Record constructor    | Rules                                                                                                    |
| --------------------- | -------------------------------------------------------------------------------------------------------- |
| Explicit canonical    | Same parameters as the header; must assign every field                                                   |
| Compact (`Range { }`) | Validates or reassigns the parameters; fields are assigned at the end. `this.x = ...` is a compile error |
| Non-canonical         | Must delegate with `this(...)`; may use a prologue                                                       |

- A record body cannot add instance fields or instance initializers. Static fields, static methods, instance methods and nested types are fine.

📖 Full lesson: [Java Classes, Records, Constructors and Initializers (1Z0-831)](https://www.savemycert.com/revision/oracle-java-se-developer/java-classes-records-constructors-initializers/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide)

## 3.3 Overloaded and var-arg methods

- **A legal overload** differs in parameter types or their order. Return type, parameter names, `static`, access and `throws` alone do not count. Generic parameters are erased (`List<String>` vs `List<Integer>` clash).
- **Resolution runs three phases** and stops at the first with a candidate:

| Phase       | Conversions allowed                                    | Varargs?         |
| ----------- | ------------------------------------------------------ | ---------------- |
| 1 Strict    | Identity, primitive widening, reference widening       | Only as an array |
| 2 Loose     | Phase 1 plus boxing/unboxing (then reference widening) | Only as an array |
| 3 Var-arity | Phase 2 plus packing into an array                     | Yes              |

- So **widening beats boxing beats varargs**. Declaration order never matters.
- **Arguments are never narrowed.** `volume(300)` does not reach `volume(short)`, and `2.5` never reaches a `float` parameter.
- **Nearest widening wins:** int before long before float before double. `w(7L)` with float and double overloads picks float.
- `char` and `short` never convert into each other. A `char` skips a `short` overload; a `byte` takes `short` first.
- A wrapper reaches `Number`/`Object` in phase 1, before unboxing is tried. `tag('B')` with `tag(long)` and `tag(Character)` picks long and sees 66.
- **Box then widen** is legal (`int` → `Integer` → `Number`). **Widen then box** is not (`int` never becomes `Long` or `Double`).
- **null** goes to the most specific reference overload. `save(Integer)` and `save(String)` with `save(null)` is ambiguous; adding `save(Object)` does not help. A cast fixes it.
- **Var-arg rules:**
  - `Type... name` is one parameter, it must be last, and it is an array.
  - `m(char...)` and `m(char[])` cannot coexist.
  - No arguments → an empty array, never `null`.
  - Passing an array passes the caller's own array (no copy).
  - An `Integer[]` passed to `Object...` is the array; a `long[]` (primitive) is wrapped as one element.
  - `(Object[]) null` makes the parameter `null`; `(Object) null` is one null element.
- When two var-arg overloads reach phase 3, compare element types. `int...` beats `long...`; `char...` vs `Character...` is ambiguous.
- **Compile time vs run time:** overloads are chosen from declared types; overrides from the object type. A subclass method with a different parameter type is an overload that a superclass reference never sees.
- **`equals(Money)` trap:** it overloads `equals(Object)`, so collections and `Object`-typed arguments use identity. Use `@Override`.

📖 Full lesson: [Java Method Overloading and Varargs Rules (1Z0-831)](https://www.savemycert.com/revision/oracle-java-se-developer/java-method-overloading-varargs/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide)

## 3.4 Scope, encapsulation, immutability, var and unnamed variables

- **Local scope:** from the declaration to the end of its block. An inner block cannot redeclare an outer local or parameter (Java has no local-shadows-local). A sibling block may reuse the name.
- **Colon switch block = one scope.** A local declared under `case 1:` is in scope under later cases but not initialised there. Arrow cases each have their own block.
- **Shadowing a field is legal:** `this.name` or `ClassName.name` reaches the field. The constructor bug `x = x;` assigns the parameter to itself and leaves the field at its default.
- **Definite assignment:** locals have no default. The compiler trusts only constant expressions and statement structure.

| Before reading `n`                                    | Compiles?               |
| ----------------------------------------------------- | ----------------------- |
| `if (1 < 2) n = 1;`                                   | Yes (constant)          |
| `final boolean D = true; if (D) n = 1;`               | Yes (constant variable) |
| `boolean on = true; if (on) n = 1;`                   | No                      |
| `while (true) { n = 1; break; }`                      | Yes                     |
| `do { n = 1; } while (more);`                         | Yes                     |
| `if (x > 0) ... else if (x <= 0) ...` (no final else) | No                      |
| `for (...; i < 3; ...) n = i;`                        | No (may run zero times) |

- **Pattern variables are flow-scoped:**
  - In scope on the right of `&&`, never `||`.
  - In the then-branch of `if (o instanceof String s)`.
  - With a negated match, in the else branch, and after the `if` when the then-branch cannot complete normally.
  - A pattern variable may shadow a field but not a local.
- **Lambda vs anonymous class:** a lambda parameter or body local cannot reuse an enclosing local's name. An anonymous class method's can shadow it.
- **Access:** `private` (the top-level class, nested classes included) < package < `protected` (package + subclasses) < `public`. Access is checked per **class**, not per object. Two top-level classes in one file cannot see each other's privates.
- **Defensive copies:** copy mutable input (`days.clone()`) and output. `final` is shallow.

| Expression                          | Copies?   | Modifiable? | Sees later source changes? |
| ----------------------------------- | --------- | ----------- | -------------------------- |
| `new ArrayList<>(src)`              | Yes       | Yes         | No                         |
| `Collections.unmodifiableList(src)` | No (view) | No          | Yes                        |
| `List.copyOf(src)`                  | Yes       | No          | No (rejects nulls)         |

- **Immutable class recipe:** private final fields, no setters, a final class (or a record), defensive copies in and out. In a record, reassign the compact constructor's parameter: `names = List.copyOf(names);`.
- **`var`:** local type inference from the initializer, still statically typed. `var` is a reserved type name, not a keyword (`var var = 2;` compiles).
  - Rejected for fields, parameters and return types.
  - Rejected with no initializer, `null`, a bare `{1, 2}`, a lambda initializer or a compound declaration.
  - Lambda parameters may use `var` only if all of them do.
  - `var map = new HashMap<>();` is `HashMap<Object, Object>`.
  - `var a = 1;` is an `int` forever.
  - With an anonymous class, `var` keeps the anonymous type, so its extra methods are callable.
- **Unnamed `_`** (standard since Java 22):
  - Allowed for locals with an initializer, for and enhanced-for variables, resources, catch and lambda parameters, and patterns.
  - **Not** allowed for fields, method parameters or a local without an initializer.
  - It can repeat in one scope and can never be read, but its initializer still runs.

📖 Full lesson: [Java Variable Scope, Encapsulation, Immutability, var and _ (1Z0-831)](https://www.savemycert.com/revision/oracle-java-se-developer/java-variable-scope-encapsulation-immutability-var/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide)

## 3.5 Inheritance, abstract and sealed types, overriding and polymorphism

| Question                                   | Decided by                        | When         |
| ------------------------------------------ | --------------------------------- | ------------ |
| Which members can be named                 | Reference type                    | Compile time |
| Which checked exceptions must be handled   | Reference type's declaration      | Compile time |
| Which overload                             | Reference and argument types      | Compile time |
| Which overridden instance method body runs | Object type                       | Run time     |
| Which field, static or private method      | Reference type / containing class | Compile time |

- **Override rules:**
  - Same signature (or its erasure).
  - Same or wider access.
  - Same return type, or a subtype for references (covariant). Any change to a primitive return fails.
  - No new or broader **checked** exceptions. Unchecked ones are free.
  - Cannot override a `final` method or switch between static and instance.
- `toString`, `equals(Object)` and `hashCode` are public in `Object`, so an override without `public` fails.
- **Not overridden:** `private` methods (a same-named subclass method is unrelated), `static` methods (hidden, bound at compile time), and **fields** (hidden).
  - `s.tag` reads the field of the reference type.
  - An inherited, non-overridden method reads its own class's field.
- **Casts:**
  - Upcasts always succeed.
  - Downcasts compile and may throw `ClassCastException`.
  - A cast between unrelated classes fails to compile.
  - Casting a non-final class to an interface compiles; a `final` class to an interface it lacks does not.
  - Prefer `instanceof` patterns.
- **Constructors in a hierarchy:** the implicit `super()` needs a no-arg parent constructor. `super.method()` reaches the direct superclass version; there is no `super.super`. An overridable call from a parent constructor sees subclass fields at their defaults unless a Java 25 prologue set them.
- **Abstract classes:**
  - Cannot be instantiated.
  - May have constructors, fields and concrete or final methods.
  - Abstract methods end with `;` and cannot be `private`, `static` or `final`.
  - A class cannot be both `abstract` and `final`.
  - A concrete subclass must implement every inherited abstract method.
- **Sealed types:** `sealed ... permits A, B`. Each permitted subtype must directly extend the sealed type and be:

| Modifier     | Meaning                                          |
| ------------ | ------------------------------------------------ |
| `final`      | Closed (records are implicitly final)            |
| `sealed`     | Restricted again; must permit its own subclasses |
| `non-sealed` | Reopened to any subclass                         |
| none         | Compile error, even for an abstract class        |

- Permitted subtypes live in the same module (or the same package in the unnamed module). `permits` may be omitted when they are in the same file.
- **Exhaustive switches over sealed types** need no default when every permitted subtype is covered.
  - Guarded cases do not count.
  - Covering one subclass of a `non-sealed` branch does not cover the branch.
  - A hierarchy change after compilation gives `MatchException`.
- **equals/hashCode contract:** equal objects must return the same hash. Hash collections use `hashCode` to find the bucket, then `equals(Object)`. Override both, with the exact public signatures.

📖 Full lesson: [Java Inheritance, Polymorphism, Overriding and Sealed Classes (1Z0-831)](https://www.savemycert.com/revision/oracle-java-se-developer/java-inheritance-sealed-classes-polymorphism-overriding/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide)

## 3.6 Interfaces, functional interfaces and interface methods

- **Implicit modifiers:** abstract methods are `public abstract`; fields are `public static final`. An implementing method must be **`public`**; without the modifier it is package access and fails.
- An abstract class may implement an interface partially. The first concrete subclass supplies the rest.
- **Constants** must be initialised and can never be assigned (`MAX++` fails). There is one shared copy, so a final list's contents can still change.

| Declaration in an interface                                               | Result                             |
| ------------------------------------------------------------------------- | ---------------------------------- |
| `default void f() { }`, `static`, `private`, `private static` with a body | Compiles                           |
| `protected void f();`                                                     | Fails                              |
| `int size;`                                                               | Fails (`= expected`)               |
| `final`, `synchronized` methods                                           | Fail                               |
| `default void f();` (no body)                                             | Fails                              |
| `default static` or `private default`                                     | Illegal combination                |
| `default boolean equals(Object o) { ... }`                                | Fails (overrides an Object member) |

- **Default methods** are public instance methods with a body. Call the interface version from an override with `InterfaceName.super.method()`.
  - Only a **direct** superinterface works ("not an enclosing class").
  - `super.method()` means the superclass.
  - `Report.render()` is static-call syntax and fails for a default.
- **Conflict rules, in order:**
  1. **Classes win:** a superclass method beats any default, even an **abstract** one (which the class must then implement).
  2. **The more specific interface wins**, even when it re-abstracts the method.
  3. Otherwise the class **must override**. It can call `Walker.super.move()`.
- Casting `this` to one interface does not select its default; the call is still virtual and recurses.
- **Static interface methods are not inherited.** Call them only as `InterfaceName.method()`. A same-signature static in a class is unrelated. `X.super.staticMethod()` is illegal.
- **Private helpers:**
  - Default and private instance methods can call both private and private static helpers.
  - Static methods can call only **private static** ones (and not defaults).
  - Implementing classes can call neither.
- **Functional interface:** exactly one abstract method after inheritance.
  - Defaults, statics and privates do not count.
  - Inherited abstract methods do count.
  - Redeclared `equals`/`hashCode`/`toString` do not count, but `clone()` does (it is protected in Object).
  - Override-equivalent redeclarations count once. Zero abstract methods is not functional.
- **`@FunctionalInterface`** only asks the compiler to check. On a two-method interface it fails at the interface; removing it moves the error to the lambda. It is not inherited.

📖 Full lesson: [Java Interfaces: Functional, Default, Static and Private Methods (1Z0-831)](https://www.savemycert.com/revision/oracle-java-se-developer/java-interfaces-functional-default-static-private-methods/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide)

## 3.7 Enums with fields, methods and constructors

- **Layout:** constants first, then `;`, then fields, constructors and methods.
  - A member before the constants fails.
  - `SLOW = 30` is not Java.
  - Every argument list must match a declared constructor. With no constructor declared, only the implicit no-arg one exists.
- **What an enum is:** it implicitly extends `java.lang.Enum`, so there is no `extends` clause. It may implement interfaces.
  - It is implicitly final, or implicitly sealed when constants have bodies. Never write `final`, `sealed` or `abstract` on it.
  - Nested enums are implicitly static; local enums are legal.
- **Constructors** are private (no modifier means private; `public` fails). They can be overloaded and chained with `this(...)`. `super(...)` and `new Level()` are compile errors.
- **Timing:** all constants are constructed once, in declaration order, at the enum's **first active use**, before any later static field or static block.
- **Static state in constructors:**
  - A direct reference to the enum's own static field fails, unless it is a constant variable. Sibling constants count as static fields.
  - Calling a static method that touches a static field compiles, but a later field initializer can wipe its work.
  - Calling `values()` during construction throws, surfacing as `ExceptionInInitializerError`.
- **Mutable fields** on a constant are shared program-wide, because each constant is a singleton.

| Method         | Behaviour                                                                                                 |
| -------------- | --------------------------------------------------------------------------------------------------------- |
| `values()`     | A **new copy** of the array each call, in declaration order                                               |
| `valueOf("X")` | Exact, case-sensitive name match; `IllegalArgumentException` if unknown, NPE for null; never returns null |
| `ordinal()`    | Zero-based position                                                                                       |
| `name()`       | Declared identifier; **final**                                                                            |
| `toString()`   | Same as `name()` by default; can be overridden                                                            |
| `compareTo()`  | Declaration order (final)                                                                                 |

- `==` and `equals` give the same answer. A String is never equal to a constant.
- Overriding `toString` breaks `valueOf(x.toString())`. Use `name()` to round-trip.
- **Constant-specific bodies** create an anonymous subclass:
  - An abstract enum method must be implemented in **every** constant.
  - `getClass()` returns the anonymous subclass; `getDeclaringClass()` returns the enum.
  - A method declared only in a body cannot be called from outside, even with `var`.
- **Switch on enums:**
  - Plain or (since Java 21) qualified labels.
  - A switch expression must list every constant or have `default`.
  - A `null` selector throws unless there is `case null`.
  - A switch statement need not be exhaustive, but a local assigned only in its arms is not definitely assigned without a `default`.

📖 Full lesson: [Java Enums: Fields, Methods and Constructors (1Z0-831)](https://www.savemycert.com/revision/oracle-java-se-developer/java-enum-fields-methods-constructors/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide)

[← Back to the study guide](../README.md)
