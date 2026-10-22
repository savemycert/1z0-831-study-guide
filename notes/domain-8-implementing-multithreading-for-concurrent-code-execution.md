# Domain 8: Implementing Multithreading for Concurrent Code Execution (11%)

Three objectives: creating and running threads (platform and virtual), executors, `CompletableFuture` and scoped values; making code thread-safe with `synchronized`, `volatile`, atomics, locks and synchronizers; and concurrent collections and parallel streams. The test programs force ordering with `join`, `Future.get`, latches, single-thread executors or `close()`, so the questions turn on each method's blocking and exception behaviour, not on luck. Oracle publishes no domain weights; the percentage here is an estimate.

## 8.1 Platform and virtual threads, executors and scoped values

| Code                                          | Kind                  | Started?           |
| --------------------------------------------- | --------------------- | ------------------ |
| `new Thread(task)`                            | Platform              | No, call `start()` |
| `Thread.ofPlatform().start(task)`             | Platform              | Yes                |
| `Thread.ofVirtual().unstarted(task)`          | Virtual               | No                 |
| `Thread.startVirtualThread(task)`             | Virtual               | Yes                |
| `Executors.newVirtualThreadPerTaskExecutor()` | Virtual, one per task | Per task           |

- **Builders:**
  - `name("io-", 3)` numbers threads from 3 upward.
  - `factory()` gives a `ThreadFactory`.
  - Only the platform builder has `daemon(...)` and `priority(...)`.

| Property     | Platform thread                   | Virtual thread                                                    |
| ------------ | --------------------------------- | ----------------------------------------------------------------- |
| Backed by    | One OS thread for life            | A carrier platform thread, only while running                     |
| Daemon       | Inherited or set before `start()` | Always true; `setDaemon(false)` throws `IllegalArgumentException` |
| Priority     | Settable                          | Fixed at 5; `setPriority` silently ignored                        |
| Default name | `Thread-N`                        | Empty string                                                      |
| Pooling      | Usually pooled                    | Never pool; one per task                                          |
| Best for     | Long-running or CPU-bound work    | Many tasks that mostly wait (I/O)                                 |

- Virtual threads make blocking cheap, not computation faster. The `main` thread is a platform thread.
- `setDaemon` on a **live platform** thread throws `IllegalThreadStateException`.
- The JVM does not wait for daemon (and so virtual) threads. Wait with `join`, a `Future` or `close()`.
- **`start()` vs `run()`:** `run()` is a plain call on the current thread. A second `start()` throws `IllegalThreadStateException`, even after the thread terminated.

| State           | Typical cause                              |
| --------------- | ------------------------------------------ |
| `NEW`           | Before `start()`                           |
| `RUNNABLE`      | Running or ready                           |
| `BLOCKED`       | Waiting for a **monitor** (`synchronized`) |
| `WAITING`       | `join()`, `wait()`, `park`                 |
| `TIMED_WAITING` | `sleep(ms)`, `join(ms)`, `wait(ms)`        |
| `TERMINATED`    | `run()` returned or threw                  |

- `join(200)` just returns when time runs out; check `isAlive()` afterwards. `join` and `sleep` throw the checked `InterruptedException`.
- **Interrupts** are cooperative:
  - Throwing `InterruptedException` **clears** the flag.
  - A pending interrupt makes `sleep` throw at once.
  - A busy loop that never checks the flag ignores it.
  - `t.isInterrupted()` reads without clearing; the static `Thread.interrupted()` reads **and clears**.

|                        | `Runnable`     | `Callable<V>`               |
| ---------------------- | -------------- | --------------------------- |
| Method                 | `void run()`   | `V call() throws Exception` |
| Checked exceptions     | Must be caught | May propagate               |
| `Future.get()` returns | `null`         | The value                   |

- **Which `submit` overload:** a value-returning body is a `Callable`; a void body is a `Runnable`. An expression valid as both (`count++`) picks `Callable`. A body that only throws also fits both, so `Callable` wins. `execute` accepts only `Runnable`.
- **Future:**
  - `get()` wraps a task failure in `ExecutionException` (cause = the original).
  - `get(timeout, unit)` throws `TimeoutException` without cancelling the task.
  - `cancel(true)` interrupts the task.
- **Executors:**
  - `newSingleThreadExecutor` runs one task at a time in submission order.
  - `newFixedThreadPool(n)` gives no order between workers.
  - `newCachedThreadPool` grows as needed.
  - `newScheduledThreadPool` handles delayed and periodic tasks.
- **`invokeAll`** returns futures in **task-list order**. With a timeout, unfinished tasks are cancelled.
- **`invokeAny`** returns one successful result and throws `ExecutionException` only if **all** tasks fail.

| Method               | Running tasks | Queued tasks                   | Blocks? |
| -------------------- | ------------- | ------------------------------ | ------- |
| `shutdown()`         | Finish        | Still run                      | No      |
| `shutdownNow()`      | Interrupted   | Removed and returned as a list | No      |
| `awaitTermination()` | Waited for    | Waited for; `false` on timeout | Yes     |
| `close()` (Java 19+) | Finish        | Still run                      | Yes     |

- After shutdown, a new `submit` throws `RejectedExecutionException`.
- A pool that is never shut down keeps the JVM alive. `ExecutorService` is `AutoCloseable`, so try-with-resources waits for all tasks.
- **Scheduling:**
  - `schedule` returns a `ScheduledFuture` immediately.
  - `scheduleAtFixedRate` measures **start to start**; `scheduleWithFixedDelay` measures **end to start**.
  - A periodic task never overlaps itself.
  - If one run throws, later runs stop and `get()` throws `ExecutionException`.
- **CompletableFuture:**
  - `thenApply` is like map, `thenCompose` like flatMap, and `thenCombine` joins two futures.
  - A failure skips the dependent stages. Only **`exceptionally`** and **`handle`** recover; `whenComplete` only observes.
  - `get()` throws `ExecutionException`; `join()` throws the unchecked `CompletionException`.
  - Handlers usually receive a `CompletionException`.
- **Scoped values** (`ScopedValue`, final in Java SE 25):
  - Declare with `static final ScopedValue<String> USER = ScopedValue.newInstance();`.
  - Bind with `ScopedValue.where(USER, "ana").run(...)`, or `.call(...)` to return a value. Chain `.where` calls to bind several.
  - `get()` when unbound throws `NoSuchElementException`. Use `isBound()`, `orElse(x)` (not `null`) or `orElseThrow`.
  - The value is immutable, with no `set`. A nested `where(...).run(...)` rebinds, and the outer value returns afterwards.
  - The binding lasts only for the `run`/`call` on that thread. New threads and plain executor tasks do **not** inherit it.

📖 Full lesson: [Java Threads, Virtual Threads, Executors and Scoped Values (1Z0-831)](https://www.savemycert.com/revision/oracle-java-se-developer/java-virtual-threads-executors-callable-scoped-values/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide)

## 8.2 Thread-safe code with locks and the concurrent API

- **Race condition:** `count++` (and `count = count + 1`) is read-modify-write, so concurrent updates are lost and the result varies between runs. Check-then-act (`if (!map.containsKey(k)) map.put(k, v)`) races the same way.
- **Three fixes:** don't share (confine to one thread), don't mutate (immutable objects), or coordinate (locks, atomics, concurrent collections).

| Form                           | Monitor locked                 |
| ------------------------------ | ------------------------------ |
| `synchronized void m()`        | `this`                         |
| `static synchronized void m()` | The `Class` object             |
| `synchronized (expr) { }`      | The object `expr` evaluates to |

- Two calls exclude each other **only on the same object**.
  - Instance and static synchronized methods never block each other.
  - A synchronized instance method does not protect a static field across instances.
- **Lock a private final dedicated object.** A reassigned field, a boxed `Integer`, `new Object()` or `Thread.currentThread()` protects nothing. A `null` lock expression throws NPE.
- `synchronized` is not allowed on fields, constructors or classes. An override can add or drop it.
- A monitor is released however the block exits. A thread waiting for a monitor is `BLOCKED` and cannot be interrupted.
- **Reentrancy:** an owner can re-enter, tracked by a hold count. `wait()` releases all holds and restores them on wake.
- **`volatile`:** visibility and happens-before for single reads and writes. **No atomicity**, so `v++` still loses updates. It is a field modifier only.
- **Happens-before edges:**
  - Unlock happens-before a later lock of the same monitor.
  - A volatile write happens-before a later read of that field.
  - `start()` happens-before the started thread's actions.
  - A thread's actions happen-before `join()` returns.
  - Edges are transitive.

| Tool                        | Visibility | Atomic compound update | Mutual exclusion |
| --------------------------- | ---------- | ---------------------- | ---------------- |
| `volatile`                  | Yes        | No                     | No               |
| `AtomicInteger` and friends | Yes        | Yes, one variable      | No               |
| `synchronized` / `Lock`     | Yes        | Yes, any number        | Yes              |

- **Atomics:**
  - `getAndX` returns the **old** value; `xAndGet` returns the **new** one.
  - `compareAndSet(expected, v)` stores only on a match and returns whether it did.
  - `updateAndGet`/`getAndUpdate` and `accumulateAndGet`/`getAndAccumulate` take side-effect-free functions.
  - `LongAdder` suits hot counters.
- **Lock idiom:** `lock.lock(); try { ... } finally { lock.unlock(); }`.
  - Call `lock()` before the `try`.
  - Unlock only what you acquired; a non-owner `unlock()` throws `IllegalMonitorStateException`.
  - A Lock is never released automatically, not even when its thread dies.

| Method                    | If held by another thread | Interrupt while waiting | Checked `InterruptedException` |
| ------------------------- | ------------------------- | ----------------------- | ------------------------------ |
| `lock()`                  | Waits forever             | Ignored                 | No                             |
| `tryLock()`               | Returns `false` at once   | n/a                     | No                             |
| `tryLock(long, TimeUnit)` | Waits up to the timeout   | Throws                  | Yes                            |
| `lockInterruptibly()`     | Waits                     | Throws                  | Yes                            |

- There is no `tryLock(long)` and no `tryLock(Duration)`.
- `ReentrantLock` methods: `getHoldCount()`, `isHeldByCurrentThread()`, `isLocked()`. A fair lock (`new ReentrantLock(true)`) reduces starvation but does not prevent deadlock.
- **ReentrantReadWriteLock:** readers share and the writer is exclusive. A writer may **downgrade** (take read, then release write). A reader cannot **upgrade**: `writeLock().tryLock()` returns `false`.
- **wait/notify:**
  - The caller must own the monitor, or the call throws `IllegalMonitorStateException` at run time (it compiles).
  - `wait()` releases the lock.
  - Always wait in a **`while`** loop: wakeups can be spurious, and a notify sent before anyone waits is lost.
- **Condition:** `lock.newCondition()`, then `await`/`signal`/`signalAll` while holding the lock. One lock can have several conditions. Calling `wait()` on a Condition uses the wrong monitor.

|               | `CountDownLatch`                                             | `CyclicBarrier`                                                                           | `Semaphore`                                                   |
| ------------- | ------------------------------------------------------------ | ----------------------------------------------------------------------------------------- | ------------------------------------------------------------- |
| Reusable      | No                                                           | Yes, resets each trip                                                                     | Yes                                                           |
| Key behaviour | `countDown` at 0 does nothing; timed `await` returns `false` | Last arrival runs the action; a timeout or interrupt breaks it (`BrokenBarrierException`) | Permits have no owner; `release` can exceed the initial count |

- **Liveness failures:**
  - **Deadlock:** threads blocked forever by locks taken in different orders. Fix with one global lock order, or `tryLock` and back off.
  - **Livelock:** threads busy retrying in lockstep. Fix with randomized back-off.
  - **Starvation:** one thread rarely gets the lock. Fix with fair locks and shorter critical sections.
  - Changing timing or priorities never fixes a deadlock.

📖 Full lesson: [Java Thread Safety: synchronized, Locks, volatile and Atomic Variables (1Z0-831)](https://www.savemycert.com/revision/oracle-java-se-developer/java-thread-safety-synchronized-locks-atomic-variables/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide)

## 8.3 Concurrent collections and parallel streams

| Family       | Examples                                                                                       | Iterating while it changes                                |
| ------------ | ---------------------------------------------------------------------------------------------- | --------------------------------------------------------- |
| Plain        | `ArrayList`, `HashMap`, `TreeMap`                                                              | Fail-fast `ConcurrentModificationException` (best effort) |
| Synchronized | `Vector`, `Hashtable`, `Collections.synchronizedX`                                             | **Still fail-fast**                                       |
| Concurrent   | `ConcurrentHashMap`, `ConcurrentSkipListMap/Set`, `CopyOnWriteArrayList/Set`, `BlockingQueue`s | Never throws: weakly consistent or snapshot               |

- **Synchronized wrappers:**
  - Single calls are safe.
  - Iteration (and streams) need `synchronized (wrapper)`. Lock the map, not its view.
  - The wrapper's own `forEach`, `removeIf` and `replaceAll` are already locked.
  - Check-then-act needs one lock around both steps.
- **ConcurrentHashMap:**
  - Rejects null keys and values (NPE).
  - A function returning `null` from `compute`, `computeIfPresent` or `merge` removes the entry; from `computeIfAbsent` it adds nothing.
  - Use **`merge`/`compute`/`computeIfAbsent`** for read-modify-write. Get-then-put loses updates.
  - `computeIfAbsent(k, x -> ConcurrentHashMap.newKeySet()).add(v)` is the safe grouping idiom.
  - `ConcurrentHashMap.newKeySet()` is the thread-safe hash set; there is no `ConcurrentHashSet` class.
  - Bulk `forEach`/`search`/`reduce(threshold, ...)`: the threshold affects only splitting, `search` returns any non-null match, and a `null` transformer result skips the element.
- **ConcurrentSkipListMap/Set:** sorted, navigable, thread-safe versions of TreeMap/TreeSet. Nulls are rejected; elements must be comparable; iteration is lock-free.
- **CopyOnWriteArrayList/Set:** copy the array on every write, so they suit read-mostly data.
  - Iterators are snapshots, so changes during a loop are allowed and invisible to it.
  - `Iterator.remove` throws `UnsupportedOperationException`.
  - `addIfAbsent` checks and adds atomically.

| BlockingQueue | Throws                                    | Special value     | Blocks | Times out                  |
| ------------- | ----------------------------------------- | ----------------- | ------ | -------------------------- |
| Insert        | `add` (`IllegalStateException` when full) | `offer` → `false` | `put`  | `offer(e, t, u)` → `false` |
| Remove        | `remove()` (`NoSuchElementException`)     | `poll` → `null`   | `take` | `poll(t, u)` → `null`      |
| Examine       | `element()`                               | `peek` → `null`   | n/a    | n/a                        |

- `put`, `take` and the timed forms throw the checked `InterruptedException`. Nulls are rejected.
- **Capacity:**
  - `ArrayBlockingQueue` is always bounded.
  - `LinkedBlockingQueue` defaults to `Integer.MAX_VALUE`.
  - `SynchronousQueue` has no capacity.
  - `ConcurrentLinkedQueue` is not a BlockingQueue (no `put` or `take`).
- **Parallel streams:**
  - Create with `collection.parallelStream()` or `.parallel()`.
  - The **last** `parallel()`/`sequential()` call sets the mode for the whole pipeline.
  - `unordered()` does not parallelize.
  - `Stream.of(list)` is a stream of one list.

| On an ordered parallel stream                         | Order guaranteed? |
| ----------------------------------------------------- | ----------------- |
| `forEach`                                             | No                |
| `forEachOrdered`                                      | Yes               |
| `toList()`, `collect(toList())`, `joining`, `toArray` | Yes               |
| `findFirst()`                                         | Yes               |
| `findAny()`                                           | No                |
| `skip`, `limit`, `distinct`, `sorted`                 | Yes (costlier)    |
| Side effects in `peek`/`map`/`filter`                 | No                |

- An unordered source (`HashSet`, `Set.of`) has no order to keep. `peek` may be skipped entirely.
- **`forEach(list::add)` into a plain list** from a parallel stream loses elements or throws. Use a collector.
- `toConcurrentMap` needs a merge function for duplicate keys. `groupingByConcurrent` is unordered within groups.

📖 Full lesson: [Java Concurrent Collections and Parallel Streams (1Z0-831)](https://www.savemycert.com/revision/oracle-java-se-developer/java-concurrent-collections-parallel-streams/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide)

[← Back to the study guide](../README.md)
