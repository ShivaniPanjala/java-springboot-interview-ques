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
    - **Thread T1's ThreadLocalMap:**
       - Key: ThreadLocal object A → Value: A1 (thread-specific)
       - Key: ThreadLocal object B → Value: B1
    - **Thread T2's ThreadLocalMap:**
       - Key: ThreadLocal object A → Value: A2 (thread-specific)
       - Key: ThreadLocal object B → Value: B2
    - Each thread maintains its own map with keys as ThreadLocal objects and values as thread-specific data.
- **Weak References:** The keys in this map are weak references to the ThreadLocal objects, and the values are the actual data objects you store (e.g., your User object).

- When you call:
    - **threadLocal.set(value):** Java uses the current Thread as a key to find its internal ThreadLocalMap. The map then stores the association: (Key: weak reference to threadLocal instance, Value: value object).

    - **threadLocal.get():** The current Thread is used to look up its internal map. The map uses the threadLocal instance to retrieve the associated value object.

    - **Crucial Point:** The ThreadLocalMap is owned by the Thread, not the ThreadLocal variable


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

# How does ForkJoinPool differ from a regular ThreadPoolExecutor?
- **ForkJoinPool**
    - tasks that can be recursively split into smaller **subtasks (forked)** and then **combined (joined)** to form a final result.
    - Each thread in a ForkJoinPool has its own **double-ended queue (deque)** to store subtasks it creates
    - It uses a **work-stealing** algorithm where idle threads can steal tasks from busier threads’ queues to balance the workload and improve performance.
    - To use ForkJoinPool effectively, tasks should **avoid synchronization**, shared variables, or blocking operations to keep execution pure and isolated.
    - divide-and-conquer problems, recursive, fine-grained tasks (like parallel algorithms)

    ```
    public Result solve(Task t) {
        split t into smaller tasks

        for each of these tasks:
        solve(ti)

        wait for all tasks to complete

        join all individual results

        return result
    }
    ```
- **ThreadPoolExecutor**
    - It manages a pool of worker threads and a task queue to execute Runnable or Callable tasks efficiently.
    - helps to create a customizable threadpool

    ```
    public ThreadPoolExecutor(
        int corePoolSize, - Minimum number of threads to keep alive
        int maximumPoolSize, - Max number of threads allowed (used when the queue is full).
        long keepAliveTime, - Idle threads beyond corePoolSize are terminated after this time (allowCoreThreadTimeOut: true)
        TimeUnit unit, - time unit for keep alive time
        BlockingQueue<Runnable> workQueue, - Where tasks wait -> bounded Queue(ArrayBlockingQueue), unbounded Queue(LinkedBlockingQueue)
        ThreadFactory threadFactory, - factory for creating new thread by giving custom name, thread priority, and set deamon flag
        RejectedExecutionHandler handler - handler for tasks that can not be accepted by thread pool. Rejection Policies -> AbortPolicy, AbortPolicy, DiscardPolicy, DiscardOldestPolicy
        )

    ```
    - It improves performance by **reusing threads** and helps control concurrency under load.
    - general-purpose asynchronous task execution


| Feature / Aspect        | ThreadPoolExecutor                                   | ForkJoinPool                                              |
|-------------------------|--------------------------------------------------------|-----------------------------------------------------------|
| **Purpose**             | General-purpose task execution                         | Parallelism and divide-and-conquer algorithms             |
| **Task Types**          | `Runnable`, `Callable`                                 | `RecursiveTask`, `RecursiveAction`                        |
| **Work Queue Model**    | Single shared blocking queue                           | Per-thread deque + work-stealing                          |
| **Best For**            | Long-running, I/O-bound, independent tasks             | Many small CPU-bound subtasks that recursively split      |
| **Thread Count**        | Fully configurable; can exceed CPU cores               | Usually equals number of CPU cores                        |
| **Blocking Behavior**   | Blocking is fine                                       | Blocking discouraged; use `ManagedBlocker` if needed      |

