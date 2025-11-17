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
   - EX: caching Configuration lists, event listeners

**Not to use**
   - Write-heavy workloads (adds/removes frequently).
   - Large lists (copying the whole array on each write is costly).
   - Memory-sensitive systems (each write allocates a full new array).
---

# Explain ThreadLocal – how it works internally, pitfalls.
