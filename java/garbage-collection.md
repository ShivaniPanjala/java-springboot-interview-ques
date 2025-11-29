- 2 Types of Memory
-------------------
1. Stack
2. Heap
- Both stack and heap are created by `JVM` and `stored in RAM`

1. **Stack Memory:**
------------------
- Stores temporary variables and seperate memory block for methods
- Primitive types (int, double, boolean, char, etc.) declared inside methods are stored directly in the stack frame
- Java objects are always created on the heap, but the reference variables live in the stack
- Types of references
    1. Strong Reference - As long as a strong reference exists → object is not garbage collected.
    2. Weak Reference - Object becomes eligible for GC if only weak refs remain.
        1. Soft Reference - GC collects these only if memory is low
        2. Phanthom Reference - 
- Each thread maintains its own separate stack.
- Variables within a `SCOPE` is only visible and as soon as any variable goes out of the scope, it get deleted from the stack(in `LIFO` order)
- when stack memory goes full, it throws `java.lang.StackOverflowError`

2. **Heap Memory:**
---------------- 
- Store Objects
- There is no order of allocating the memory
- `Garbage Collector` is used to delete the unrefernced objects from heap.
- Heap Memory is shared with all the threads
- HEap also contains the string pool
- When heap memory goes full, it throws `java.lang.OutOfMemoryError`

# Heap Structure in Java (Generational GC)
Most JVMs divide the heap into generations:
- **Young Generation**
    - New objects are allocated here.
    - Consists of **Eden** + **Survivor S0** + **Survivor S1**.
    - Frequent GC called Minor GC occurs here.

- **Old / Tenured Generation**
    - holding long-lived objects
    - Less frequent GC, called Major GC or Full GC.

- **Permanent generation** - old java version before java 7, its a part of heap, not expandable, when memory is full we get OutOfMemoryError

- **Metaspace**
    - Stores class metadata (replaced PermGen after Java 8).
    - out of heap, it is called as non heap, it is expandable as required
---


# Garbage Collection
- Java programs create objects on the heap. 
- When an object is no longer reachable (i.e., no reference points to it), it becomes eligible for garbage collection. 
- The Java Garbage Collector automatically finds such objects and deletes them to reclaim memory.

# GC Algorithms
- **Mark and Sweep Algorithm**(100MB)
   - Mark Phase - the object is `reachable`(there is a reference path from stack/static/active threads. to the object) → it is `marked alive`
   - Sweep Phase - object is `unreachable`(no references → eligible for GC) → it stays `unmarked`-> All unmarked (unreachable) objects are cleaned up from the heap.

- **Mark and Compact**(2GB) 
   - Mark all reachable objects, then shift them together to remove gaps and eliminate fragmentation.

- **G1 GC (Garbage First)** (>2GB) -> focusing on its region-based approach and its goal of predictable pauses.
    - split large heap into hundreds or thousands of these equal-sized `regions` (typically between 1MB and 32MB),
    - each region can be anything
        - Young(Eden, Survivor{S0, S1}) -> holding new objects
        - Old -> holding long-lived objects
        - Humongous -> holding objects larger than half a region size
        - Free -> ready for allocation
    - collects priority regions first.
    - prioritize cleaning the regions(old regions) with the most garbage first to achieve predictable, short pause times
    - -XX:MaxGCPauseMillis=200(ms) -> Sets a target value for desired maximum pause time
    - don't stop the application for more than 200ms
    
    - **Generational**
        - Still classifies objects as Young or Old, but assigns these roles to flexible heap regions
    - **"Garbage First"**
    	- Prioritizes collecting Old regions with the most garbage to maximize efficiency per pause.
    - **Incremental** 
        - Cleans in small batches (a few regions at a time) to prevent long, unpredictable pauses.
    - **Parallel**
        - Uses multiple CPU cores during "Stop-The-World" pauses for fast execution.
    - **Most Concurrent**
        - Does most complex analysis work in the background while the application runs.
    - **Stop-the-world(STW)**
        - All application threads are paused so the JVM can perform garbage collection or other internal work.
        - During an STW event, your program temporarily stops doing any real work until the JVM finishes the GC task.
    - **Evacuating**
        - Copies live objects from old regions to new ones, which automatically compacts the heap.


---

- **Common problems and solutions**
    - Long or growing pauses : increase heap, raise MaxGCPauseMillis
    - Frequent Full GCs : increase heap size

- **Garbage collectors in java**
    1. `G1 Garbage Collector (G1GC)` 
        - Default from java 9+,  
        - Efficient for multi-threaded,
        - high throughput and low pause times with predictable, low latency GC
        - small, predictable pauses

    2. Serial GC
        - Uses a single thread for both Young and Old generation GC. 
        - Pause times grow significantly as the heap grows because all `GC work is done by one thread`, 
        - it can only handle about 100 MB heap before running into pause time issues
        - long pauses (single-threaded)

    3. Parallel GC 
        - Uses multiple threads for Young and Old generation GC.
        - Designed to `reduce overall GC time by parallelizing the work`, but still uses `stop-the-world (STW)` pauses.
        - Performs well up to multi-GB heaps, but pause times increase when heap sizes get very large.
        - pause time started to become an issue around 2GB
        - shorter but still noticeable pauses (multi-threaded)

    4. CMS (Concurrent Mark-Sweep) (deprecated) -> could perform work concurrently, but utilizing only a single thread

    5. ZGC (low latency) - extremely low pauses (sub-millisecond)
    
    6. Shenandoah GC (ultra low pause) - extremely low pauses (sub-millisecond)

- **When Does an Object Become Eligible for GC?**
    - Its reference is set to null
    - It goes out of scope
    - It forms an island of isolation (cyclic garbage)

- **Can You Force Garbage Collection?**
    - Not exactly.
    - You can request it, but JVM may ignore it.

- **Benefits of Garbage Collection**
    - Reduces memory leaks
    - Simplifies development (no manual free() like C)
    - Prevents many memory-related crashes

- **Limitations**
    - GC pauses (stop-the-world events)
    - Overhead of running GC cycles
    - Not deterministic → can't predict exactly when GC will run 