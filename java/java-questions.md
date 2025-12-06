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
```Java
public class ThreadLocalExample {
    private static final ThreadLocal<String> threadSpecificData = new ThreadLocal<>();

    public static void main(String[] args) {
        Runnable task1 = () -> {
            threadSpecificData.set("Data for Thread 1");
            System.out.println(Thread.currentThread().getName() + ": " + threadSpecificData.get());
            threadSpecificData.remove(); // Clean up
        };

        Runnable task2 = () -> {
            threadSpecificData.set("Data for Thread 2");
            System.out.println(Thread.currentThread().getName() + ": " + threadSpecificData.get());
            threadSpecificData.remove(); // Clean up
        };

        Thread thread1 = new Thread(task1, "Thread-1");
        Thread thread2 = new Thread(task2, "Thread-2");

        thread1.start();
        thread2.start();
    }
}
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

# Difference between volatile keyword and synchronized
- volatile
    - Ensures visibility of changes to variables across threads
    - No locking, doesn’t block threads
- synchronized
    - Acquires a lock, only one thread can enter

---
# Deep copy vs Shallow copy
There are two ways to duplicate objects
1. **Deep copy**  copy everything (full duplication).
    - A mechanism to copy an object and all its related objects
    - Deep copy visits all the graph of the refernced objects and copies them, which can be costly because it is hard to know the cost in advance

2. **Shallow Copy**(surface-level copy (doesn’t go inside).)
    - just copies the current object, If this object is referencing more objects they are not copied
    - it is the default behaviour
---

# How does the Java memory model work (Heap, Stack, Metaspace)?
- Java divides memory into different areas so the JVM can effectively manage objects, method calls, class info and runtime data
1. **Heap**  
    - Objects created with `new`, Arrays, Instance variables are stored in heap
    - Shared accross `all threads`
    - Managed by the `GC`
    - Divided into:
        - Young Generation(Eden + Survivor space)
        - Old Generation
    
2. **Stack**
    - Method call frames, Local variables (Primitives + object references), Return addresses are stored in stack
    - Each thread has its `own stack`
    - Very fast allocation/deallocation
    - No GC needed

3. **MetaSpace**
    - `Class definitions`, Method metadata, Runtime constant pool(literals, string references, method references) are stored 
    - `Replaced premGen` in java 8+
    - uses native memory(outside the heap)
    - Automatically grows unless capped
    - Cleaner and less error-prone than PremGen

# Explain the internal working of ConcurrentHashMap in Java 8.
```java 
ConcurrentHashMap<string, Integer> map = new ConcurrentHashMap()
```

## In java 7
- Used `Segments` (an array of smaller hashmaps).
- `segment based locking` -> 16 segments by default-> entier ConcurrentHashMap is divide into 16 hashmaps
- 16 segments have `independent lock (ReentrantLock)`, i.e., every segment has its own lock, so threads could operate on different segments in parallel
- only the segment being written to or read from is locked
    - suppose we need to read from a map or update/put in a map, we will do that in a particular segment,  only that particular segment will be locked, other segments will not be blocked
- **read:** do not require locking unless there is a write operation happening on the same segment
- **write:** (put/update/remove) require locking the entire segment.
- **Drawbacks of Segmentation**
    - Scalability is limited to a fixed number of segments (default 16).
    - If the HashMap grows large, each segment also becomes large.
    - Many keys may cluster into a single segment → more waiting → reduced concurrency.


## In java 8
- `No Segmentation`
    - Java 8 completely removed segments.
    - Why?
        - 16 segments limit scalability.
        - Large maps mean large segments → more contention.
        - Finer-grained locking was needed.

- `Each bucket` stores either:
    - A `linked list of nodes` (few collisions)
    - A `tree (Red-Black Tree)` if too many collisions happen

- **Lock-Free Reads**: - completely lock-free -> This greatly improves concurrency.

- **write:** uses `Compare and Swap approach(CAS)` -> `no locking except resizing or collision`
    - Ex: 
        - Thread A last saw x = 42.
        - Thread A wants to update it to 50.
        - CAS: if x is still 42 → change to 50;
            - otherwise retry.
    
    -  In HashMap Terms:
        - When putting a value, CAS checks whether the bucket (index) is unchanged.
        - If it is unchanged → insert/update.
        - If changed (collision/resizing) → fallback to synchronization. 

- **resizing**
    - when 0.75% of capacity is reached then resizing is done
    - the resizing of concurrent hashmap and conevnetional hashmap is different
        - the interenal array of the hashmap is different
        - the internal array in the map is doubled imediately as soon as it exceeds the threshold
        - things don't double in the internal hashmap, it happens step by step, incremental resizing happens
        - suppose there are 4 buckets and resizing is happend and added 1 more bucket then it possible two threads are comming and try to hold the same bucket it will be necessary to apply a lock so taht is why locking is done in case of resizing and collision

----

# Difference between synchronized, ReentrantLock and ReadWriteLock
| Feature / Aspect              | `synchronized`                                  | `ReentrantLock`                                | `ReadWriteLock`                             |
|-------------------------------|-------------------------------------------------|-----------------------------------------------|--------------------------------------------|
| **Type**                      | Built-in Java keyword                           | Class in `java.util.concurrent.locks`        | Interface in `java.util.concurrent.locks`  |
| **Lock Type**                  | Intrinsic (monitor)                             | Explicit lock                                 | Read and write locks                        |
| **Reentrancy**                 | Yes                                             | Yes                                           | Yes (for both read and write locks)        |
| **Fairness Policy**            | No                                              | Optional (true/false in constructor)         | Optional (depends on implementation)       |
| **Lock Acquisition / Release** | Implicit (via entering/exiting synchronized block) | Explicit (`lock()` / `unlock()`)             | Explicit (`readLock()/writeLock().lock()` / `unlock()`) |
| **Interruptible Lock**         | No                                              | Yes (`lockInterruptibly()`)                   | Yes (via `lockInterruptibly()` on read/write locks) |
| **Try Lock**                   | No                                              | Yes (`tryLock()` with optional timeout)      | Yes (for both read and write locks)        |
| **Condition Support**          | Implicit via `wait()` / `notify()` / `notifyAll()` | Yes (`newCondition()`)                        | Yes (via read/write lock's condition)      |
| **Use Case**                   | Simple mutual exclusion                        | Advanced locking with flexibility             | High concurrency scenarios (many readers, few writers) |
| **Performance**                | Slightly lower under high contention           | Higher under high contention                  | Highest for read-heavy workloads           |
---