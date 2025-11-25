- 2 Types of Memory
-------------------
1. Stack
2. Heap
- Both stack and heap are created by `JVM` and `stored in RAM`

1. Stack Memory:
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

2. Heap Memory:
---------------- 
- Store Objects
- There is no order of allocating the memory
- `Garbage Collector` is used to delete the unrefernced objects from heap.
- Types of GC:
    1. Single GC
    2. Parallel GC
    3. Concurrent Mark and Sweep(CMS)
    4. G1(Garbage first)
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
    - Objects that survive multiple Minor GCs are moved here.
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
**Mark and Sweep Algorithm**(100MB)
    - Mark Phase - the object is `reachable`(there is a reference path from stack/static/active threads. to the object) → it is `marked alive`
    - Sweep Phase - object is `unreachable`(no references → eligible for GC) → it stays `unmarked`-> All unmarked (unreachable) objects are cleaned up from the heap.

**Mark and Compact**(2GB) - Mark all reachable objects, then shift them together to remove gaps and eliminate fragmentation.

**G1 GC (Garbage First)** (>2GB)
    - Splits heap into regions, each region can be anything (eden, s1, old etc...)
    - collects priority regions first.
    - -XX:MaxGCPauseMillis=200(ms) -> Sets a target value for desired maximum pause time
---

**Common problems and solutions**
- Long or growing pauses : increase heap, raise MaxGCPauseMillis
- Frequent Full GCs : increase heap size



-**Garbage collectors in java**
1. `G1 Garbage Collector (G1GC)` - Default from java 9+,  Efficient for multi-threaded, low-pause apps, high throughput, low latency
2. Serial GC (single-threaded, small apps)
3. Parallel GC (max throughput)
4. CMS (Concurrent Mark-Sweep) (deprecated)
5. ZGC (low latency)
6. Shenandoah GC (ultra low pause)

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

-**Limitations**
- GC pauses (stop-the-world events)
- Overhead of running GC cycles
- Not deterministic → can't predict exactly when GC will run