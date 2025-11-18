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
- ThreadLocal is a mechanism in Java that provides thread isolation by giving each thread its own, **private copy of a variable**. This allows you to store state that is local to a single thread, effectively making a variable **per-thread global**

**How It Works Internally (The Map)**
- Internally, ThreadLocal stores values using a hash map specific to each thread called the **ThreadLocalMap**
    **Thread T1's ThreadLocalMap:**
       - Key: ThreadLocal object A → Value: A1 (thread-specific)
       - Key: ThreadLocal object B → Value: B1
    **Thread T2's ThreadLocalMap:**
       - Key: ThreadLocal object A → Value: A2 (thread-specific)
       - Key: ThreadLocal object B → Value: B2
    - Each thread maintains its own map with keys as ThreadLocal objects and values as thread-specific data.
- **Weak References:** The keys in this map are weak references to the ThreadLocal objects, and the values are the actual data objects you store (e.g., your User object).

- When you call:
    **threadLocal.set(value):** Java uses the current Thread as a key to find its internal ThreadLocalMap. The map then stores the association: (Key: weak reference to threadLocal instance, Value: value object).

    **threadLocal.get():** The current Thread is used to look up its internal map. The map uses the threadLocal instance to retrieve the associated value object.

    **Crucial Point:** The ThreadLocalMap is owned by the Thread, not the ThreadLocal variable


- **Example:** 
```
public class ThreadLocalDemo {
    private static final ThreadLocal<String> session = ThreadLocal.withInitial(() -> null);

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            session.set("T1 user 1");
            try {
                Thread.sleep(2000); // Simulating some delay
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + " session user: " + session.get());
            session.remove();
        }, "Thread T1");

        Thread t2 = new Thread(() -> {
            session.set("T2 user 2");
            System.out.println(Thread.currentThread().getName() + " session user: " + session.get());
            session.remove();
        }, "Thread T2");

        t1.start();
        t2.start();

        t1.join();
        t2.join();

        System.out.println("Main thread exiting");
    }
}
```

- **Pitfalls**
   - If you use thread pools (like in ExecutorService) and don’t remove values
   - ThreadLocal values can stick around for the lifetime of the thread, which is potentially forever in a thread pool.
   - This happens because the thread never dies, so the ThreadLocalMap keeps the value

---

