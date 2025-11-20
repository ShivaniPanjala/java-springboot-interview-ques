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

- **Deadlock**(threads wait on each other forever)
    - Thread 1 locks lock1 and waits for lock2
    - Thread 2 locks lock2 and waits for lock1
    - Both wait forever → deadlock
    ``` java
    public class DeadlockExample {

        static final Object lock1 = new Object();
        static final Object lock2 = new Object();

        public static void main(String[] args) {

            Thread t1 = new Thread(() -> {
                synchronized (lock1) {
                    System.out.println("Thread 1: Locked lock1");
                    synchronized (lock2) {
                        System.out.println("Thread 1: Locked lock2");
                    }
                }
            });

            Thread t2 = new Thread(() -> {
                synchronized (lock2) {
                    System.out.println("Thread 2: Locked lock2");
                    synchronized (lock1) {
                        System.out.println("Thread 2: Locked lock1");
                    }
                }
            });

            t1.start();
            t2.start();
        }
    }

    ```


- **LiveLock**(Thread is Active, repeatedly trying)
    - They keep actively trying to acquire resources or change state, but never make progress
    
    - Thread 1 locks lock1 but fails to get lock2, so it releases lock1 and retries
    - Thread 2 locks lock2 but fails to get lock1, so it releases lock2 and retries
    - Both threads keep releasing and retrying without making progress → livelock

    - Ex: 
    ```java
    Thread t1 = new Thread(() -> {
        while (true) {
            if (tryLock(lock1)) {
                if (tryLock(lock2)) {
                    System.out.println("Thread 1: Locked both locks");
                    unlock(lock2);
                    unlock(lock1);
                    break;
                } else {
                    // release lock1 and retry
                    unlock(lock1);
                }
            }
        }
    });


    Thread t2 = new Thread(() -> {
        while (true) {
            if (tryLock(lock2)) {
                if (tryLock(lock1)) {
                    System.out.println("Thread 2: Locked both locks");
                    unlock(lock1);
                    unlock(lock2);
                    break;
                } else {
                    // release lock2 and retry
                    unlock(lock2);
                }
            }
        }
    });

    ```
    
- **starvation**(a thread is ready but never gets CPU or locks)
    - Starvation happens when a thread is ready to run, but **never gets CPU time** or access to a resource because other threads keep taking priority.
    - The thread is not blocked (like deadlock) but is ignored indefinitely.

    ```java
    import java.util.concurrent.locks.ReentrantLock;

    public class StarvationExample {

        private static final ReentrantLock lock = new ReentrantLock();

        public static void main(String[] args) {

            // High-priority thread keeps running
            Thread highPriorityThread = new Thread(() -> {
                while (true) {
                    lock.lock();
                    try {
                        System.out.println("High priority thread running");
                        // simulate work
                        Thread.sleep(50);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    } finally {
                        lock.unlock();
                    }
                }
            });
            highPriorityThread.setPriority(Thread.MAX_PRIORITY);

            // Low-priority thread may starve
            Thread lowPriorityThread = new Thread(() -> {
                while (true) {
                    lock.lock();
                    try {
                        System.out.println("Low priority thread running");
                        Thread.sleep(50);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    } finally {
                        lock.unlock();
                    }
                }
            });
            lowPriorityThread.setPriority(Thread.MIN_PRIORITY);

            highPriorityThread.start();
            lowPriorityThread.start();
        }
    }
    ```


| Issue       | Thread State                | Cause                                     | CPU Usage           | Prevention                                             |
|------------|----------------------------|------------------------------------------|-------------------|------------------------------------------------------|
| Deadlock   | Blocked forever            | Circular wait on locks                    | Low               | Lock ordering, tryLock with timeout, minimize nested locks |
| Livelock   | Active but no progress     | Repeated state changes without progress  | High              | Random back-off, avoid infinite retries, proper coordination |
| Starvation | Ready but never executes   | Low priority or unfair scheduling        | Varies (other threads run) | Fair locks, balanced thread priorities, proper scheduling |
