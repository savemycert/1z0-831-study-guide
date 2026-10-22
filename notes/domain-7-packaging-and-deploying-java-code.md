# Domain 7: Packaging and Deploying Java Code (9%)

Two objectives. The first covers module declarations, exports, opens and readability, services, and `import module`. The second covers compact source files and instance `main` methods, the `java`/`javac`/`jar`/`jlink` command lines, and migration with unnamed and automatic modules. Items show several `module-info.java` files or a command line and ask what happens. Launcher errors (`ClassNotFoundException`, `FindException`, `ResolutionException`) are genuine answers. Oracle publishes no domain weights; the percentage here is an estimate.

## 7.1 Modules, services and module import declarations

| Directive                             | Meaning                                                        |
| ------------------------------------- | -------------------------------------------------------------- |
| `requires m;`                         | Reads `m` at compile time and run time                         |
| `requires transitive m;`              | Reads `m`, and every reader of this module also reads `m`      |
| `requires static m;`                  | Required at compile time, optional at run time                 |
| `exports p;` / `exports p to m1, m2;` | Public types of `p` accessible to all / only the named modules |
| `opens p;` / `opens p to m;`          | Deep reflection on all members of `p` at run time              |
| `uses S;`                             | Consumes service `S` via `ServiceLoader`                       |
| `provides S with Impl;`               | Supplies `Impl` as a provider of `S`                           |

- `java.base` is read implicitly. Other platform modules (`java.sql`, `java.logging`) must be required.
- **Cross-module access needs all three:** the module reads the other module, the package is exported to it, and the type and member are public.
- **`exports` names exactly one package.**
  - Subpackages are unrelated; there are no wildcards or comma lists of packages.
  - Exporting an empty or missing package is a compile error.
  - A duplicate export of the same package is a compile error.
- A method may return a type from a concealed package. Using the result as that type fails; use an exported supertype.
- **Qualified exports:** one directive per package, with a list of target modules. A missing target only warns. `requires transitive` never widens a qualified export.
- **Readability is not transitive** unless declared.
  - If your exported API exposes another module's types, require that module `transitive`.
  - Implied readability chains along transitive edges only.
  - `var` does not get around missing readability.
- **`requires static`:** at launch the module is resolved only if another module requires it normally or `--add-modules` adds it. If it is absent, the first touch throws `NoClassDefFoundError`. Guard with `ModuleLayer.boot().findModule("m").isPresent()`.
- **Graph rules:**
  - Cyclic `requires` fails to compile.
  - A **split package** (the same package in two modules of a layer, even concealed) fails at boot-layer creation with `LayerInstantiationException`, before `main`.
  - A missing required module also stops the launch.

| For package `p` | Compile against it     | Reflect on public members | `setAccessible(true)` on private  |
| --------------- | ---------------------- | ------------------------- | --------------------------------- |
| Neither         | No                     | No                        | No                                |
| `exports p`     | Yes                    | Yes                       | No: `InaccessibleObjectException` |
| `opens p`       | No                     | Yes                       | Yes                               |
| Both            | Yes                    | Yes                       | Yes                               |
| `open module`   | Only exported packages | Yes                       | Yes                               |

- `trySetAccessible()` returns `false` instead of throwing.
- The module that owns the classes opens them to the framework. Exporting is not enough.
- Opening a missing package compiles with a warning but fails at launch (`FindException`).
- An `open module` cannot contain `opens` directives.
- **Services:**
  - The consumer declares `uses S;`. Without it, `ServiceLoader.load` throws `ServiceConfigurationError` at run time.
  - The provider declares `provides S with A, B;` (service first).
  - The consumer needs **no** `requires` on the provider. Service binding adds observable providers.
  - The provider can stay unexported, even in a concealed package.
  - `uses` and `provides` create no readability, so the consumer still requires the service's module.
  - On the class path, providers register in `META-INF/services/` instead.
- **A valid provider type** is public and top level or static nested, and either:
  - has a `public static` no-arg `provider()` method returning the service or a subtype (this wins, and the type may be abstract or an interface); or
  - is a concrete subtype of the service with a public no-arg constructor.
  - Inner classes, enums, and abstract classes without `provider()` fail.
- **ServiceLoader:**
  - Lazy and caching: iterating twice creates each provider once.
  - `stream()` gives `Provider`s; `type()` does not instantiate, `get()` does.
  - `findFirst()` is an `Optional`.
  - `reload()` clears the cache.
  - Zero providers is not an error; `iterator().next()` throws `NoSuchElementException`.
  - Provider order across modules is undefined.
- **`import module M;`** (standard in Java SE 25, JEP 511):
  - Imports every public top-level type in packages M exports to you, plus those of the modules M requires **transitively**.
  - `import module java.sql;` brings `java.util.logging` but not `java.util`.
  - It grants no readability: a named module must already read M.
- **Import priority:** single-type import > on-demand package import > module import. A clash between imports of the same kind is an error at the **use** site.
- **Compact source files** implicitly `import module java.base;`, so `import module java.desktop;` there makes `List` ambiguous.

📖 Full lesson: [Java Modules, Services and Module Imports (1Z0-831)](https://www.savemycert.com/revision/oracle-java-se-developer/java-modules-services-module-imports/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide)

## 7.2 Compiling and running code, JARs, runtime images and migration to modules

- **Compact source files** (standard in Java 25, JEP 512) have top-level fields and methods and no class declaration. The implicit class:
  - is `final` and in the unnamed package, so a `package` declaration fails;
  - has only the default constructor;
  - cannot be referenced by name;
  - must have a launchable `main`.
  - Fields are instance fields unless marked `static`.
- They automatically import all of `java.base`. `java.lang.IO` (`print`, `println`, `readln`) works everywhere without an import, but you still write `IO.println`. An ordinary class still needs `import java.util.List;`.
- **Launch protocol for `main`:**
  1. A non-private `void main(String[])` (or `String...`), declared or inherited, wins.
  2. Otherwise a non-private `void main()`.
  3. If the chosen method is an instance method, the launcher constructs the object first (field initialisers run) using a non-private no-arg constructor, or fails.
  - Private methods and non-void `main` methods are never candidates.
- **Source-file mode:**
  - `java Main.java args` compiles in memory and writes no class files.
  - It launches the **first** top-level class in the file.
  - Options go **before** the file name.
  - Since JDK 22, source wins over a stale `.class` file.
- **Multi-file source programs** (Java 22): other files are compiled on demand from the root implied by the package, and unreferenced files are never compiled. Libraries go in `--class-path` before the file name.
- **javac and java -cp:**
  - `javac -d out` places classes by package.
  - `--source-path src` finds dependencies and must name the package root.
  - `java -cp out org.zoo.Keeper` needs the root plus the fully qualified name.
  - `java -cp out Keeper` gives `ClassNotFoundException`.
  - `-cp out/org/zoo Keeper` gives `NoClassDefFoundError (wrong name)`.
  - The path separator is `:` on Linux/macOS and `;` on Windows.
- **Non-modular JARs:**
  - `jar --create --file app.jar --main-class org.zoo.Keeper -C out .` writes `Main-Class`.
  - Without it, `java -jar` fails with "no main manifest attribute".
  - **`-jar` ignores `-cp`**; dependencies come from the manifest `Class-Path`.
  - Words after the JAR are arguments to `main`.
- **Modular JARs** (`module-info.class` at the root):
  - Named only on the module path (`-p`).
  - `-m module` needs a `ModuleMainClass` (set by `--main-class`); `-m module/class` always works.
  - `-m` takes a module name, not a class name.
  - Without `-m`, the main class comes from the class path.
  - On the class path the same JAR is in the unnamed module (`getModule().getName()` is `null`).
  - Duplicate module names: the first module-path entry wins. Two versions in one directory is an error.
- **jlink:** `jlink --module-path mods --add-modules app --launcher zoo=app --output img`.
  - Roots plus their transitive `requires` are linked.
  - `--output` must not exist.
  - `--strip-debug`, `--no-header-files` and `--no-man-pages` shrink the image.
  - **Automatic modules and class-path JARs cannot be linked.**

|                 | Named module                   | Automatic module                                       | Unnamed module    |
| --------------- | ------------------------------ | ------------------------------------------------------ | ----------------- |
| Comes from      | JAR with `module-info` on `-p` | Plain JAR on `-p`                                      | Anything on `-cp` |
| Name            | From the descriptor            | `Automatic-Module-Name`, or derived from the file name | None              |
| Exports/opens   | What it declares               | Everything                                             | Everything        |
| Reads           | What it requires               | Every module, unnamed included                         | Every module      |
| Can be required | Yes                            | Yes, by name                                           | No                |
| jlink           | Yes                            | No                                                     | No                |

- **A named module cannot read the unnamed module** ("declared in the unnamed module, but module X does not read it").
- With the main class on the class path, module-path modules are not resolved unless `--add-modules` adds them.
- Class-path code calling a non-exported package of a named module throws `IllegalAccessError`.
- **Automatic module names:**
  1. Drop `.jar`.
  2. Drop a version that starts at a hyphen followed by digits.
  3. Turn non-alphanumeric runs into dots.
  - `jackson-core-2.17.1.jar` → `jackson.core`; `fast_csv-4.0.jar` → `fast.csv`.
  - A segment that starts with a digit is unusable without `Automatic-Module-Name`.
- **Migration:**
  - **Bottom-up:** convert the leaves first; unconverted code stays on the class path.
  - **Top-down:** convert the app first; libraries become automatic modules. jlink only works once nothing is automatic.
  - `jdeps` lists dependencies.
- **Split packages on the module path:** a module that reads two modules containing the same package (for example two plain JARs on `-p`) fails resolution (`ResolutionException`). Merge the JARs, or move one to the class path.

📖 Full lesson: [Compiling and Running Java: Compact Source Files, JARs, jlink and Module Migration (1Z0-831)](https://www.savemycert.com/revision/oracle-java-se-developer/java-compile-run-jars-jlink-module-migration/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide)

[← Back to the study guide](../README.md)
