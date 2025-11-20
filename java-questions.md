# Difference between HashSet, LinkedHashSet, TreeSet
- **HashSet** no duplicate elements, order is not maintained, elements are stored based on their hash code, Allows one null element  
- **LinkedHashSet** child of HashSet, Insertion order is Preserved, combination of LinkedList and HAshtable, Allows one null element  
- **TreeSet** no duplicate elements, order is maintained, Maintains natural ordering or custom ordering (via a Comparator), Does **not** allow null
---

# Explain CopyOnWriteArrayList → when to use, when not to use.
**What it is:**
- A thread-safe List where all writes create a new copy of the internal array
- Every time you modify the list (add, remove, set), it creates a new copy of the underlying array.
- **Read operations:** Just read the current array reference; no locking needed.
- **Write operations:**
    - Lock the array.
    - Create a new copy of the array with the modification.
    - Replace the old array reference with the new one.
- **Effect:** Iterators are safe and do not throw ConcurrentModificationException, because they work on a snapshot of the array.

**Use it when:**
   - Many threads are reading the list concurrently, but updates are rare.
   - EX: caching, Configuration lists, event listeners

**Not to use**
   - Write-heavy workloads (adds/removes frequently).
   - Large lists (copying the whole array on each write is costly).
   - Memory-sensitive systems (each write allocates a full new array).
---

# Explain ThreadLocal – how it works internally, pitfalls.
- A ThreadLocal gives **each thread its own private variable**, not shared with other threads.
- EX: Think of ThreadLocal like a locker room:
    - Each thread has its own locker
    - Same variable name, but each thread gets its own copy
    - Threads cannot see each other’s values
    ```
        ThreadLocal<String> threadLocal = new ThreadLocal<>();
        threadLocal.set("value for this thread");
        String value = threadLocal.get();
    ```     

**How It Works Internally (ThreadLocalMap)**
- Internally, ThreadLocal stores values using a hash map specific to each thread called the **ThreadLocalMap**
- ThreadLocal<T> provides per-thread storage.
- Every Thread object in the JVM contains a field:
    ```
        ThreadLocalMap threadLocals
    ```
    - This is essentially a Map<ThreadLocal, value> that exists inside the Thread object.
- When you call: 
    - **threadLocal.set(value)**
        - Internally:
            - JVM gets the current thread
            - Looks at its ThreadLocalMap
            - Stores the value in the map using the ThreadLocal instance as the key

        - So each thread gets its own separate value, even though you use the same ThreadLocal object.
        - When the thread ends:
            - ThreadLocalMap is destroyed with the thread → memory is released.
- ThreadLocal does NOT create a new thread; it gives each thread its own private copy of a variable.
- **Crucial Point:** The ThreadLocalMap is owned by the Thread, not the ThreadLocal variable

- **Pitfalls**
   - If you use thread pools (like in ExecutorService) and don’t remove values
   - ThreadLocal values can stick around for the lifetime of the thread, which is potentially forever in a thread pool.
   - This happens because the thread never dies, so the ThreadLocalMap keeps the value

---

# How does ForkJoinPool differ from a regular ThreadPoolExecutor?

| Feature / Aspect        | ThreadPoolExecutor                                   | ForkJoinPool                                              |
|-------------------------|--------------------------------------------------------|-----------------------------------------------------------|
| **Purpose**             | General-purpose task execution                         | Parallelism and divide-and-conquer algorithms             |
| **Task Types**          | `Runnable`, `Callable`                                 | `RecursiveTask`, `RecursiveAction`                        |
| **Work Queue Model**    | Single shared blocking queue                           | Per-thread deque + work-stealing                          |
| **Best For**            | Long-running, I/O-bound, independent tasks             | Many small CPU-bound subtasks that recursively split      |
| **Thread Count**        | Fully configurable; can exceed CPU cores               | Usually equals number of CPU cores                        |
| **Blocking Behavior**   | Blocking is fine                                       | Blocking discouraged; use `ManagedBlocker` if needed      |

---
