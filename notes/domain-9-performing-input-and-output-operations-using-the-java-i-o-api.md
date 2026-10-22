# Domain 9: Performing Input and Output Operations Using the Java I/O API (11%)

Three objectives: classic `java.io` streams for console and file data, object serialization, and `Path` with the `java.nio.file.Files` API. Path questions assume a Unix-style file system and depend on exact output. Files and stream questions ask what a file contains after a sequence of calls, how many entries a walk returns, or which exception is thrown. Oracle publishes no domain weights; the percentage here is an estimate.

## 9.1 Console and file I/O streams

|                  | Byte streams                                         | Character streams          |
| ---------------- | ---------------------------------------------------- | -------------------------- |
| Base classes     | `InputStream`, `OutputStream`                        | `Reader`, `Writer`         |
| `read()` returns | A byte 0–255, or -1                                  | A char 0–65535, or -1      |
| File classes     | `FileInputStream`, `FileOutputStream`                | `FileReader`, `FileWriter` |
| Buffering        | `BufferedInputStream`/`OutputStream`                 | `BufferedReader`/`Writer`  |
| Printing         | `PrintStream` (`System.out`)                         | `PrintWriter`              |
| Typed            | `DataInput/OutputStream`, `ObjectInput/OutputStream` | n/a                        |

- **The superclass decides the family, not the name.** `PrintStream` prints text but is a byte stream. Copy binary files with byte streams; a Reader/Writer pair corrupts them.
- **Reading:**
  - `read()` returns -1 only at the end. A stored 0xFF comes back as 255.
  - `read(byte[])` returns a **count** and leaves the rest of the array stale, so write `buf, 0, n`.
  - `readAllBytes()` returns an empty array at the end, and `readNBytes(n)` may return fewer bytes.
  - `transferTo` closes neither stream.
  - Java 24 added `Reader.of(CharSequence)`; Java 25 added `Reader.readAllAsString()` and `readAllLines()`.
- **`mark`/`reset`:** supported by `BufferedReader`, `BufferedInputStream`, `ByteArrayInputStream` and `StringReader`.
  - `reset()` returns to the first unread item after the mark.
  - `FileReader.mark` throws `IOException`.
  - `FileInputStream.mark` does nothing, and its `reset` throws.
- **Wrapping:** decorators accept only their own family.
  - `new BufferedReader(new InputStreamReader(System.in))` is the bridge pattern.
  - `BufferedReader` over an `InputStream` does not compile.
  - `PrintWriter` builds the bridge itself when given an `OutputStream`.
- **Closing a wrapper closes the inner stream.**
  - Using a closed file stream afterwards throws `IOException` ("Stream closed").
  - Closing `ByteArrayInputStream`, `ByteArrayOutputStream`, `StringWriter` or `CharArrayWriter` has no effect, so they keep working after their wrapper is closed.
  - A second `close()` is harmless.
  - Data written straight to the inner stream arrives before the wrapper's buffered data.

| Expression                                                     | Existing file                  |
| -------------------------------------------------------------- | ------------------------------ |
| `new FileWriter("f")`, `new PrintWriter("f")`                  | Truncated                      |
| `new FileWriter("f", true)`, `new FileOutputStream("f", true)` | Appended                       |
| `new PrintWriter(new FileWriter("f", true))`                   | Appended                       |
| `new PrintWriter(new FileWriter("f"), true)`                   | Truncated (`true` = autoflush) |

- Reading constructors on a missing file throw `FileNotFoundException`.
- **Buffering:** output arrives only on `flush()`, `close()` or a full buffer. **PrintWriter autoflush covers only `println`, `printf` and `format`**, never `print`. The JVM does not flush your buffers at exit; `System.out` itself autoflushes.
- **`readLine()`:**
  - Drops the terminator (`\n`, `\r` or `\r\n`).
  - Returns `""` for a blank line and `null` at the end.
  - A trailing terminator adds no empty line.
- **IOException:**
  - Almost every `java.io` method declares it, `close()` included.
  - `PrintStream` and `PrintWriter` swallow it and set `checkError()` instead. The exception is `PrintStream.write(byte[])`; use `writeBytes`.
  - `System.in.read()` throws it.
- **DataOutputStream** writes big-endian bytes:
  - `boolean` and `byte` take 1 byte; `short` and `char` 2; `int` and `float` 4; `long` and `double` 8.
  - `writeUTF` writes a 2-byte length prefix plus modified UTF-8.
  - Read back in the same order. Reading another type reinterprets the bytes.
  - Typed reads throw `EOFException` at the end.
- **Charsets:**
  - The default has been UTF-8 since Java 18. Pass a `Charset` explicitly for anything else.
  - UTF-8 sizes: `a` is 1 byte, `é` 2, `€` 3.
  - UTF-16 adds a 2-byte byte-order mark.
  - A writer replaces unmappable characters with `?`, and decoding with the wrong charset fails silently.
- **Console:**
  - `System.in`, `System.out` and `System.err` are separate; `> log.txt` redirects only `out`.
  - `System.console()` can return `null`, so null-check it.
  - `readPassword()` returns a `char[]`.
  - Console methods throw the unchecked `IOError`, not `IOException`.
- **`java.lang.IO`** (Java 25) has only `print`, `println` and `readln` (no `printf`). `readln` returns `null` at the end of input and throws `IOError`.

📖 Full lesson: [Java I/O Streams: Console and File Readers, Writers and Streams (1Z0-831)](https://www.savemycert.com/revision/oracle-java-se-developer/java-io-streams-console-file-readers-writers/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide)

## 9.2 Serialization and deserialization

- **Serializable** is a marker interface. The whole reachable graph must be serializable, and this is checked **at run time**: `writeObject` compiles and then throws `NotSerializableException`.
  - A `null` field of a non-serializable type is fine.
  - One bad element in a collection fails the whole write.
  - `Object`, `Thread` and `Optional` are not serializable.
- **API:** wrap streams in `ObjectOutputStream`/`ObjectInputStream`.
  - `readObject()` returns `Object`, so cast it.
  - It declares `IOException` **and** `ClassNotFoundException` (which is not an `IOException`).
  - The end of the stream is signalled by `EOFException`, never `null`.
  - Read in write order. Calling `readObject` where primitive data comes next throws `OptionalDataException`.

| Exception                  | Cause                                                     |
| -------------------------- | --------------------------------------------------------- |
| `NotSerializableException` | Writing a non-serializable object                         |
| `InvalidClassException`    | UID mismatch, no valid constructor, or a filter rejection |
| `ClassNotFoundException`   | The reader lacks the class                                |
| `EOFException`             | Reading past the end                                      |
| `OptionalDataException`    | `readObject` meets primitive data                         |

- **Not saved:**
  - `transient` fields return as 0, false or null; their initializers do not run.
  - `static` fields show the reading JVM's current value.
  - A `transient final` primitive or String with a constant initializer is inlined, so it still reads as the constant.
- **Constructors on read:** none of the serializable classes' constructors or initializers run. Only the **first non-serializable superclass's no-arg constructor** runs (with its field initializers). If it is missing or inaccessible, reading throws `InvalidClassException` ("no valid constructor").

| Kind             | Runs on read                                               |
| ---------------- | ---------------------------------------------------------- |
| Ordinary class   | The first non-serializable superclass's no-arg constructor |
| Record           | The canonical constructor (validation runs)                |
| `Externalizable` | The public no-arg constructor, then `readExternal`         |
| Enum             | Nothing; the constant is looked up by name                 |

- **`serialVersionUID`** must be named exactly that and be `static final long` (any access). Otherwise the JVM computes one that changes with most edits.
  - With matching UIDs, added fields read as defaults and removed fields are ignored.
  - Changing a primitive field's type fails.
  - Annotate with `@Serial`.
- **Custom hooks:** `private void writeObject(ObjectOutputStream)` and `private void readObject(ObjectInputStream)`.
  - They must be private and non-static; otherwise they are silently ignored.
  - Call `defaultWriteObject()`/`defaultReadObject()` first, then read extras in write order. Omitting the default calls is not detected, but the fields are then not saved.
  - Calling the default methods outside a hook throws `NotActiveException`.
- **`readResolve()`** (any access, returns `Object`, no parameters) keeps singletons single. `writeReplace` plus a proxy's `readResolve` is the serialization-proxy pattern.
- **Records** ignore `readObject`/`writeObject` but honour `writeReplace`/`readResolve`.
- **Enums** serialize by name, stay `==`, ignore every hook, and have a UID of 0L.
- **Externalizable** saves only what `writeExternal` writes, and needs a **public** no-arg constructor.
- **Graphs:** each object is written once per stream; later appearances are back-references. Cycles and sharing are preserved in brand-new objects.
  - Writing a changed object again sends only a reference, so the reader sees the old state.
  - Fix it on the writing side with `reset()` or `writeUnshared()`.
- **Filters** (`ObjectInputFilter.Config.createFilter("com.x.*;!*")`):
  - Patterns are checked left to right, and the first match wins.
  - `!` rejects. Unmatched classes are undecided (not rejected), so end an allow-list with `!*`.
  - Limits: `maxdepth`, `maxrefs`, `maxbytes`, `maxarray`.
  - A rejection throws `InvalidClassException`.
  - Set the filter once, before the first read.

📖 Full lesson: [Java Serialization and Deserialization: Serializable, transient and serialVersionUID (1Z0-831)](https://www.savemycert.com/revision/oracle-java-se-developer/java-serialization-deserialization-serializable-transient/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide)

## 9.3 Path objects and the java.nio.file API

- **A Path never touches the disk** when built or manipulated. `Path.of`, `Paths.get` and `FileSystems.getDefault().getPath` are equivalent.
  - Empty strings and redundant separators collapse.
  - The empty path has one (empty) name element.
  - `Path.of(URI)` needs a scheme.
- **Anatomy** of `/opt/shop/static/img/logo.png`: the root `/` is **not** a name element, and `getNameCount()` is 5.

| Method          | Returns                                   | Missing or bad                                       |
| --------------- | ----------------------------------------- | ---------------------------------------------------- |
| `getRoot()`     | `/`                                       | `null` for a relative path                           |
| `getFileName()` | The last element                          | `null` for a root-only path                          |
| `getParent()`   | All but the last element                  | `null` (`Path.of("a").getParent()`)                  |
| `getName(i)`    | Element i (zero-based, excludes the root) | `IllegalArgumentException`                           |
| `subpath(b, e)` | Elements b to e−1, always relative        | `IllegalArgumentException`, including an empty range |

- **`resolve`:**
  - Appends a relative argument and returns an absolute argument unchanged.
  - An empty argument returns the base.
  - It never normalizes.
- **`resolveSibling(x)`** = `getParent().resolve(x)`.
- **`relativize`** climbs with `..` to the common ancestor. Mixing an absolute and a relative path throws `IllegalArgumentException`. The same path relativizes to the empty path.
- **Clean-up methods:**
  - `normalize()` is textual: a leading `..` survives in a relative path but is removed after a root.
  - `toAbsolutePath()` does not touch the disk.
  - **`toRealPath()` does**, and declares `IOException` (`NoSuchFileException`).
- **Comparison is element-wise:**
  - `startsWith("/home/ki")` is false for `/home/kim/...`.
  - `endsWith("/music")` is false for a longer path.
  - `x/y` does not equal `x/./y`.
- **Immutable:** a discarded `q.toAbsolutePath();` changes nothing.

| Create call         | Parent must exist?          | Target exists                                      |
| ------------------- | --------------------------- | -------------------------------------------------- |
| `createFile`        | Yes (`NoSuchFileException`) | `FileAlreadyExistsException`                       |
| `createDirectory`   | Yes                         | `FileAlreadyExistsException`, even for a directory |
| `createDirectories` | No                          | Fine for a directory; throws for a regular file    |

- **`copy`/`move`:**
  - The target is the full result path, never a directory to drop into.
  - An existing target throws unless `REPLACE_EXISTING` is passed, and replacing a non-empty directory throws `DirectoryNotEmptyException`.
  - Copying a directory creates an empty directory; nothing is recursive.
  - `ATOMIC_MOVE` either works atomically or throws.
- **Delete:** `delete` on a missing file throws `NoSuchFileException`; `deleteIfExists` returns `false`. Both throw `DirectoryNotEmptyException` on a non-empty directory.
- **Content methods:**
  - `readString`, `readAllLines`, `readAllBytes`, `writeString` and `write` read or write everything at once, with UTF-8 by default for text.
  - `readAllLines` adds no empty element for a trailing terminator.
  - **`Files.lines`, `list`, `walk` and `find` are lazy and must be closed** (try-with-resources). Use `lines` for huge files.
- **Open options:**
  - The default is `CREATE`, `TRUNCATE_EXISTING` and `WRITE`.
  - Passing **any** option drops the implied `CREATE`/`TRUNCATE_EXISTING` (`WRITE` stays). Writing `"xy"` with `WRITE` over `"1234567"` gives `xy34567`.
  - `APPEND` on a missing file throws; `CREATE_NEW` on an existing file throws.

| Traversal                       | Includes start? | Depth                                             |
| ------------------------------- | --------------- | ------------------------------------------------- |
| `Files.list(dir)`               | No              | 1 (a regular file throws `NotDirectoryException`) |
| `Files.walk(start)`             | Yes             | Unlimited                                         |
| `Files.walk(start, n)`          | Yes             | n levels (0 = start only)                         |
| `Files.find(start, n, matcher)` | If it matches   | n (required)                                      |

- **`walkFileTree`** calls `preVisitDirectory`, `visitFile`, `visitFileFailed` and `postVisitDirectory`.
  - `SKIP_SUBTREE` from `preVisitDirectory` skips that directory and its `postVisitDirectory`.
  - `SKIP_SIBLINGS` skips the rest of the current directory.
  - `TERMINATE` stops the walk.
  - A directory found exactly at `maxDepth` goes to `visitFile`.
  - Links are not followed without `FOLLOW_LINKS`.
- **Attributes:**
  - `exists`, `isDirectory`, `isRegularFile` and `isReadable` never throw. `exists` and `notExists` can both be false.
  - `size`, `getLastModifiedTime` and `readAttributes` throw `NoSuchFileException` for a missing file.
  - Size is in bytes.
  - `isSameFile` on two equal paths is true without checking the disk.
  - `NOFOLLOW_LINKS` asks about the link itself.

📖 Full lesson: [Java NIO.2 Path and Files API: Build, Traverse, Read and Write (1Z0-831)](https://www.savemycert.com/revision/oracle-java-se-developer/java-nio-file-path-files-api/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide)

[← Back to the study guide](../README.md)
