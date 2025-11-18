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
- ThreadLocal is a mechanism in Java that provides thread isolation by giving each thread its own, private copy of a variable. This allows you to store state that is local to a single thread, effectively making a variable "per-thread global."

How It Works Internally (The Map)
The implementation relies on two internal components:

ThreadLocalMap: This is a specialized hash map that resides inside the Thread object itself. Every Thread instance (the object representing the executing thread) has its own ThreadLocalMap.

Weak References: The keys in this map are weak references to the ThreadLocal objects, and the values are the actual data objects you store (e.g., your User object).

When you call:

threadLocal.set(value): Java uses the current Thread as a key to find its internal ThreadLocalMap. The map then stores the association: (Key: weak reference to threadLocal instance, Value: value object).

threadLocal.get(): The current Thread is used to look up its internal map. The map uses the threadLocal instance to retrieve the associated value object.

Crucial Point: The ThreadLocalMap is owned by the Thread, not the ThreadLocal variable


Internally, ThreadLocal stores values using a map specific to each thread called the **thread local map**
Here's how it works:

Each thread has its own ThreadLocalMap, which is essentially a key-value store.
The keys in this map are the ThreadLocal objects themselves.
The values are the thread-specific values associated with these ThreadLocal objects.
When a thread sets a value on a ThreadLocal variable, it stores the value in its own ThreadLocalMap with the ThreadLocal object as the key.
When a thread retrieves a value using get(), the ThreadLocalMap of that thread returns the value associated with that ThreadLocal key.
This design ensures that although multiple threads may share the same ThreadLocal object, each thread can store and access its own independent value.
Since each thread's map is isolated, there's no risk of one thread seeing or modifying another thread's ThreadLocal values, thus providing thread safety.


![Diagram illustrating ThreadLocal in Java with two threads, Thread 1 and Thread 2, each having its own ThreadLocalMap. The map keys are the ThreadLocal objects 'a' and 'b', and the values are the thread-specific copies (A1, B1 for Thread 1 and A2, B2 for Thread 2). Arrows show how each thread accesses its specific value via the ThreadLocal object.](Screenshot 2025-11-18 at 8.11.14 PM.jpg)
