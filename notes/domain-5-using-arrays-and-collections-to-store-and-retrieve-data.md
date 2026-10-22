# Domain 5: Using Arrays and Collections to Store and Retrieve Data (8%)

One objective covering arrays, lists, sets, maps, deques and the Java 21 sequenced-collection methods. Questions build a collection, change it and print it, so you need exact return values, which calls throw, and which objects are copies and which are live views. Oracle publishes no domain weights; the percentage here is an estimate.

## 5.1 Arrays, List, Set, Map, Deque and sequenced collections

**Arrays**

- Create with a size **or** an initializer, never both. `new int[3] {1, 2, 3}` fails. A bare `{1, 2, 3}` is allowed only in a declaration.
- Elements default to 0, 0.0, `false`, `'\u0000'` or `null`, even in a method.
- `length` is a field. `a[a.length]` compiles and throws `ArrayIndexOutOfBoundsException`.
- **Brackets:** `int[] p, q[];` makes `p` an `int[]` and `q` an `int[][]`. `int[] g[]` is an `int[][]`.
- **Ragged arrays:** `new int[3][]` leaves its rows `null`, so `grid[1].length` throws `NullPointerException`. `new int[][2]` does not compile.
- **Arrays do not override `equals`/`toString`.** `a.equals(b)` is identity. Printing shows `[I@...`.
- `clone()` is shallow: a cloned 2D array shares its rows.

| `java.util.Arrays` method      | Result                                                                  |
| ------------------------------ | ----------------------------------------------------------------------- |
| `toString` / `deepToString`    | Element text                                                            |
| `equals` / `deepEquals`        | Same length and equal elements in order                                 |
| `copyOf(a, n)` / `copyOfRange` | New array; pads with defaults; range is half-open                       |
| `sort(a)`                      | In place, returns `void`                                                |
| `binarySearch(a, key)`         | Index, or `-(insertion point) - 1`; undefined if unsorted               |
| `mismatch(a, b)`               | First differing index; -1 if identical; the shorter length for a prefix |
| `compare(a, b)`                | Lexicographic sign; a proper prefix is smaller                          |

- Strings sort by UTF-16 value: digits, then uppercase, then lowercase.

**Lists**

| Created with                      | `set` | `add`/`remove`      | Nulls                          | Linked to the source? |
| --------------------------------- | ----- | ------------------- | ------------------------------ | --------------------- |
| `new ArrayList<>(...)`            | Yes   | Yes                 | Yes                            | No                    |
| `Arrays.asList(arr)`              | Yes   | **No** (fixed size) | Yes                            | Yes, both ways        |
| `List.of(...)` / `List.copyOf`    | No    | No                  | **NPE**, even `contains(null)` | No                    |
| `Collections.unmodifiableList(l)` | No    | No                  | As source                      | Yes (read-only view)  |

- Sorting follows the `set` column: it works on `Arrays.asList` (and reorders the array) and throws on `List.of`.
- `Arrays.asList(intArray)` is a one-element `List<int[]>`.
- `subList(from, to)` is a half-open live view. `list.subList(0, 2).clear()` removes from the original.
- **List methods:**
  - `add(i, e)` allows `i == size()`.
  - `set` returns the replaced element.
  - `get`/`set` with a bad index throw `IndexOutOfBoundsException`.
  - `remove(Object)` returns `false` if nothing matches.
- **`List<Integer>.remove(1)`** removes **index** 1. Use `remove(Integer.valueOf(1))` for the value. Declared as a `Collection<Integer>`, only `remove(Object)` exists.
- `removeAll`/`retainAll` match whole elements with `equals`; `removeIf` takes a predicate.
- **Removing in a for-each** usually throws `ConcurrentModificationException`. Removing the second-to-last element ends the loop quietly instead. Use `removeIf` or `Iterator.remove()`.
- **Sorting:** `list.sort(null)` and `Collections.sort` use natural order.
  - `thenComparing` only breaks ties.
  - **`reversed()` reverses the whole chain to its left.**
  - A non-Comparable element makes `sort(null)` throw `ClassCastException`.

**Sets**

| Set             | Order                    | Duplicate test                    | Nulls                       |
| --------------- | ------------------------ | --------------------------------- | --------------------------- |
| `HashSet`       | None guaranteed          | `hashCode`, then `equals(Object)` | One                         |
| `LinkedHashSet` | Insertion (SequencedSet) | `hashCode` + `equals`             | One                         |
| `TreeSet`       | Sorted (NavigableSet)    | `compareTo`/`compare` == 0        | Rejected with natural order |

- Override `equals(Object)` **and** `hashCode` together. An `equals(Point p)` overload is never called. Mutating a hashed field strands the element.
- **TreeSet** needs Comparable elements or a Comparator. Even the first `add` of a non-comparable element throws `ClassCastException`.
  - Comparator result 0 means duplicate, and the first element added is kept. A length comparator keeps one word per length.
  - `contains` and `remove` use the comparator too.
- `Set.equals` compares elements only, across implementations.
- **Navigation:** `ceiling` (>=), `floor` (<=), `higher` (>), `lower` (<), each returning `null` if none. `headSet(to)` excludes `to`, `tailSet(from)` includes `from`, `subSet` is half-open. Ranges are live views.

**Maps**

| Method                                | Returns / does                                                |
| ------------------------------------- | ------------------------------------------------------------- |
| `put(k, v)`                           | Previous value or `null`                                      |
| `get(k)`                              | `null` for missing **or** null-valued keys; use `containsKey` |
| `getOrDefault(k, d)`                  | `d` only when the key is absent (a null value returns null)   |
| `putIfAbsent(k, v)`                   | Stores when absent **or null**; returns the old value         |
| `merge(k, v, fn)`                     | Puts `v` if absent, else `fn(old, v)`                         |
| `compute*`                            | Stores the function's result                                  |
| Any of merge/compute returning `null` | Removes the mapping                                           |

- `computeIfAbsent(k, x -> new ArrayList<>()).add(v)` is the grouping idiom.
- **`Map.of`, `Map.ofEntries` and `Map.copyOf`:**
  - Unmodifiable.
  - Null key or value → NPE.
  - A duplicate key in `of` → `IllegalArgumentException`.
  - Iteration order is unspecified.
- `HashMap` allows one null key. `TreeMap` with natural order rejects one.
- **Views:** `entrySet`, `keySet` and `values` are live. `setValue` updates the map. Removing through a view deletes the entry. `keySet().add` throws.
  - Reassigning a loop variable or a lambda parameter changes nothing.
  - `values().replaceAll` does not compile; use `map.replaceAll`.
- **TreeMap:**
  - `floorKey`, `ceilingKey`, `lowerKey` and `higherKey` return `null` when nothing qualifies.
  - `firstKey` on an empty map throws.
  - `headMap(to)` is exclusive, `tailMap(from)` inclusive.
  - Range views are live; putting an out-of-range key through a view throws `IllegalArgumentException`.

**Deque (`ArrayDeque`, `LinkedList`)**

| Operation    | Throws on failure                | Returns a special value      |
| ------------ | -------------------------------- | ---------------------------- |
| Insert head  | `addFirst`, `push`               | `offerFirst`                 |
| Insert tail  | `addLast`, `add`                 | `offerLast`, `offer`         |
| Remove head  | `removeFirst`, `remove()`, `pop` | `pollFirst`, `poll` → `null` |
| Remove tail  | `removeLast`                     | `pollLast` → `null`          |
| Examine head | `getFirst`, `element`            | `peekFirst`, `peek` → `null` |
| Examine tail | `getLast`                        | `peekLast` → `null`          |

- **Stack:** `push`, `pop` and `peek` work at the **head**. **Queue:** `offer`/`add` at the **tail**, `poll`/`peek` at the head.
- Empty-deque failures throw `NoSuchElementException`. `remove(Object)` just returns `false`.
- `ArrayDeque` rejects `null`; `LinkedList` accepts it. `toString` lists head to tail.

**Sequenced collections (Java 21)**

- **Implemented by:** `List`, `Deque`, `LinkedHashSet`, `SortedSet`/`TreeSet`, `LinkedHashMap` and `SortedMap`/`TreeMap`. **Not** by `HashSet` or `HashMap`.
- The methods are callable only through a sequenced **declared** type (`Set<String> s = new LinkedHashSet<>(); s.addFirst(...)` fails to compile).
- **Methods:** `addFirst`, `addLast`, `getFirst`, `getLast`, `removeFirst`, `removeLast` and `reversed()`. The get/remove methods throw `NoSuchElementException` on an empty collection.
- **LinkedHashSet:** a plain `add` keeps an existing element's position; `addFirst`/`addLast` **move** it.
- **SequencedMap:**
  - `putFirst` and `putLast` (moving an existing key).
  - `firstEntry`, `lastEntry`, `pollFirstEntry` and `pollLastEntry`, whose entries are unmodifiable snapshots.
  - `sequencedKeySet`, `sequencedValues`, `sequencedEntrySet` and `reversed`.
- **`reversed()` is a live view** that writes through: `rev.addLast(x)` prepends to the original.
- **`UnsupportedOperationException`:**
  - Positional inserts on a sorted set or map (`TreeSet.addLast`).
  - Any mutator on an unmodifiable collection or a view of one.
  - `addFirst` on `sequencedKeySet()`.
  - `removeFirst` on an `Arrays.asList` list.

📖 Full lesson: [Java Arrays and Collections: List, Set, Map, Deque and Sequenced Collections (1Z0-831)](https://www.savemycert.com/revision/oracle-java-se-developer/java-arrays-collections-list-set-map-deque/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide)

[← Back to the study guide](../README.md)
