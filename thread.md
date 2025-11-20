# Thread
- A Thread is:
    - A lightweight unit of execution
    - **Runs code independently**
    - Has its **own stack** and execution path
- When you create a thread, the **OS schedules it separately**.

**internal components of thread**
1. Stack Memory
    - Stores method calls, local variables, and execution context
2. Program COunter
    - Keeps track of the current instruction being executed in the thread
3. Register
    - CPU registers hold intermediate calculations and data for the thread
4. ThreadID
    - Undique identifier for each thread
5. Heap access
    - Threads share the heap with other threads in the process, allowing shared object access

**Thread lifecycle**
1. New: Thread object created but not started
2. Runnable: Ready to run; waiting for CPU
3. Running: CPU executes the thread
4. Blocked/Waiting: Thread waiting for a resource or condition
5. Terminated: Thread execution completes

**Thread execution flow**
```Java
Thread.start() → JVM schedules thread → Thread enters Runnable state → OS assigns CPU → Thread runs → completes → Thread terminates
```
- Scheduling: OS decides which thread runs based on priority, time-slicing, and available cores
- Context Switching: CPU saves thread state (stack, registers, PC) and loads the next thread’s state

---


# Thread Pool
- A ThreadPool manages a fixed number of worker threads and reuses them.
- Internally: 
    - A queue holds incoming tasks
    - Worker threads keep pulling a task from the queue
    - Threads do not die after finishing a task
    - They are reused for the next task → avoids overhead of creating threads repeatedly
- why thread pool exists?
    - Creating and destroying threads is expensive.
    - ThreadPool:
        - Reduces CPU overhead
        - Ensures max thread concurrency
        - Prevents too many threads from being created
- In Java, you usually get it via factory methods in Executors class:
    ```Java
    ExecutorService pool = Executors.newFixedThreadPool(10);
    ```
---

# ThreadPoolExecutor
- The actual, configurable class that implements a thread pool in Java.
- It manages a pool of worker threads and a task queue to execute Runnable or Callable tasks efficiently.
- helps to create a customizable threadpool

    ```Java
    public ThreadPoolExecutor(
        int corePoolSize,  // Minimum number of threads to keep alive
        int maximumPoolSize,// Max number of threads allowed (used when the queue is full).
        long keepAliveTime, // Idle threads beyond corePoolSize are terminated after this time (allowCoreThreadTimeOut: true)
        TimeUnit unit, // time unit for keep alive time
        BlockingQueue<Runnable> workQueue, // Where tasks wait -> bounded Queue(ArrayBlockingQueue), unbounded Queue(LinkedBlockingQueue)
        ThreadFactory threadFactory, // factory for creating new thread by giving custom name, thread priority, and set deamon flag
        RejectedExecutionHandler handler // handler for tasks that can not be accepted by thread pool. Rejection Policies -> AbortPolicy, AbortPolicy, DiscardPolicy, DiscardOldestPolicy
        )

    ```
- **Executors.newFixedThreadPool()** internally creates a ThreadPoolExecutor.
```Java
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    5, 10, 60, TimeUnit.SECONDS,
    new LinkedBlockingQueue<Runnable>()
);
```
- It improves performance by **reusing threads** and helps control concurrency under load.
- general-purpose asynchronous task execution
---


# ForkJoinPool
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
---

# **Deadlock**(threads wait on each other forever)
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
---

# **LiveLock**(Thread is Active, repeatedly trying)
- They keep actively trying to acquire resources or change state, but never make progress

- EX:   
    - Thread 1 locks lock1 but fails to get lock2, so it releases lock1 and retries
    - Thread 2 locks lock2 but fails to get lock1, so it releases lock2 and retries
    - Both threads keep releasing and retrying without making progress → livelock

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
---

# **starvation**(a thread is ready but never gets CPU or locks)
- Starvation happens when a thread is ready to run, but **never gets CPU time** or access to a resource because other threads keep taking priority.
- The thread is not blocked (like deadlock) but is ignored indefinitely.
- EX: 
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

---