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
```
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
    ```
    ExecutorService pool = Executors.newFixedThreadPool(10);
    ```
---

# ThreadPoolExecutor
- The actual, configurable class that implements a thread pool in Java.
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
- **Executors.newFixedThreadPool()** internally creates a ThreadPoolExecutor.
```
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



