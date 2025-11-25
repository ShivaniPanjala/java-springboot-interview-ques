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

# Explain Deadlock, Livelock, Starvation → how to prevent them in Java.

| Issue       | Thread State                | Cause                                     | CPU Usage           | Prevention                                             |
|------------|----------------------------|------------------------------------------|-------------------|------------------------------------------------------|
| Deadlock   | Blocked forever            | Circular wait on locks                    | Low               | Lock ordering, tryLock with timeout, minimize nested locks |
| Livelock   | Active but no progress     | Repeated state changes without progress  | High              | Random back-off, avoid infinite retries, proper coordination |
| Starvation | Ready but never executes   | Low priority or unfair scheduling        | Varies (other threads run) | Fair locks, balanced thread priorities, proper scheduling |

# Explain how CompletableFuture improves async programming.
- **Future**
  - Runs on a separate thread, but when we call `.get()` it **waits**, so it is a **blocking call**.
```

Main Thread
   |
   |---- starts async task ----> [ Worker Thread ]
   |
   -------- waits (BLOCKS) --------|
blocked                                |
           future.get()            |
runnable|<------- result returns --------|
``` 
---
- **CompletableFuture**
  - Also runs on a separate thread, but it **does not block**.  
    Instead, we provide a **callback** that executes when the result is ready.
```
Main Thread
   |
   |---- starts async task ----> [ Worker Thread ]
   |
   |---- continues doing other work ---------->
   |
   |---- callback triggered when done --------> (thenApply / thenAccept)
```
---
## What it improves
- Enables **non-blocking** asynchronous execution.
- Supports **chaining** async operations (`thenApply`, `thenCompose`, `thenApplyAsync`).
- Allows **combining** multiple async tasks (`allOf`, `combineAsync`).
- Provides **built-in error handling** (`exceptionally`, `handle`).
- Allows **callbacks** when tasks complete (`thenRun`, `thenAccept`).
- Lets you use a **custom executor** for fine-grained thread control.
---
## Example
```java
CompletableFuture.supplyAsync(() -> fetchData())
    .thenApply(data -> transform(data))
    .thenAccept(result -> System.out.println("Result: " + result))
    .exceptionally(ex -> {
        System.out.println("Error: " + ex.getMessage());
        return null;
    });
```
---

# What is the difference between WeakReference, SoftReference, PhantomReference?
- **StrongReference** -> StrongReference tells Garbage collector don't delete the object in heap memory as i have refernece of the obj stored in stack 
- **WeakReference** → WeakReference says i have access, I have a obj stored in the Heap, but when JVM runs Garbage collector the GC can delete the obj and when i refer to the obj i get null value, “I want it cached but it’s OK if it disappears.”
    1. **SoftReference** → GC is allowed to free up the obj even though i have soft refernce but only when there is no more space and you have to create a space, then GC can remove that obj from heap. “Keep it as long as memory is healthy.”
    2. PhantomReference → “Tell me exactly when it’s gone so I can clean up.”
---

#  Difference between HashMap, HashTable, and ConcurrentHashMap

| Feature              | HashMap                                            | Hashtable                                  | ConcurrentHashMap                                               |
|----------------------|-----------------------------------------------------|---------------------------------------------|------------------------------------------------------------------|
| **Thread Safety**        | Not thread-safe.                                   | Thread-safe (Legacy).                       | Thread-safe (Optimized).                                        |
| **Synchronization**      | None.                                              | Synchronized (`Method-level lock`).           | Locking only on updated segments/bins (`Lock Striping`/CAS).       |
| **Performance**          | High (fastest for single-threaded).                | Low (synchronization overhead for all ops). | High (minimizes contention).                                     |
| **Null Key/Value**       | Allows one null key and multiple null values.      | Does not allow null keys or values.         | Does not allow null keys or values.                              |
| **Fail-Fast Iterator**   | Yes.                                               | No (Fail-safe iteration not guaranteed).    | Yes (weakly consistent iterators).                               |
| **Legacy**               | No (introduced in Java 1.2).                       | Yes (original Java utility).                | No (introduced in Java 5).                                       |


```Java
Hashtable<Integer,String> ht=new Hashtable<Integer,String>(); 
ht.put(101," ajay"); 
ht.put(101,"Vijay"); 
ht.put(102,"Ravi"); 
ht.put(103,"Rahul"); 
for (Map.Entry<Integer, String> m : ht.entrySet()) { // ht.entrySet()→ returns a set of all entries in the Hashtable. Each entry contains key + value together.each element in that set is a Map.Entry object
    System.out.println(m.getKey() + " " + m.getValue());
}
 o/p: 
103 Rahul
102 Ravi
101 Vijay

```
---

