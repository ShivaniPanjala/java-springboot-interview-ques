- `static variable` -> All instances (objects) of the class share the same single copy of a static variable. Changes made to a static variable by one object are visible to all other objects of that class.

- `static method` -> Static methods can directly access only other static members (variables and methods) of the same class.


# singleton
- Its a creational pattern
- used when you want to create a `single object/instance` of a class in the entire application.
- why singleton? 
    - some objects are heavy (DB connection, Logger, cache)
    - When a class has no state or only constant values, multiple objects are unnecessary.
    - So we reuse the same object everywhere → Saves memory + improves performance.

## Lazy Intialization(Not thread safe)
- “Lazy” means → create the object only when needed, not before.
- Key Rules
1. Private constructor
    - Prevents creating objects using new Singleton()

2. Static instance variable
    - Stores the only object of the class.

3. Static getInstance() method
    - Because:
        - You cannot create an object (constructor is private)
        - You still need a way to access the object
        - Static method can be called without creating an object
    - So this works:
        ```Singleton.getInstance();```
    - even when no object exists.

```java
public class Siingleton{
    private static Singleton instance;  // Stores the one shared object
    private Singleton() {} // Prevents creating objects from outside bcz the constructor is private

    public static Singleton getInstance() { 
        if(instance == null) { // Create object only once
            instance = new Singleton();
        }
        return instance
    }
}
```
---
# Eager Initialization
- the object is created even if no one is using
- less heavy
- Because the JVM ensures static blocks are executed once when the class is loaded, this implementation is thread-safe without synchronization
- cons:
    - If the instance creation is resource-heavy and may never be used, it wastes memory (since it is eagerly initialized).

- using static keyword
    ```java
    public class Singleton {
        private static Singleton instance = new Singleton() //When the class loads, Java creates 1 object immediately
        private Singleton() {}; // nobody outside the class can create the obj as constructor is private

        public static Singleton getInstance() { // returns the same obj every time
            return instance; 
        }
    }
    ```

- using static block 
```java
public class Singleton {
    private static Singleton instance 
    private Singleton() {} // nobody outside the class can create the obj as constructor is private

    /*
    The static block runs only once, when the class is first used.

    Inside the block, it creates the single instance of the Singleton class.

    Because Java ensures a class is loaded only once, this is a safe way to make sure only one instance exists, even if multiple threads try to use it at the same time
    */
    static {
        instance = new Singleton();
    }

    public static Singleton getInstance() { // returns the same obj every time
        return instance; 
    }
    }
```
---

# thread safe singleton(using synchronized method)
- synchronized means only `one thread` can enter this method at a time.
- So if two threads try to create the object at the same time, only one is allowed inside.
- This prevents multiple objects from being created.

- `synchronized method`
    ```java
    public class Singleton {
        private static Singleton instance;
        private Singleton() {}

        public static synchronized Singleton getInstance() {
            if(instance == null) {
                instance = new Singleton();
            }
            return instance
        }
    }
    ```
# Double checking Mechanism
- Double-checked locking is used to `reduce the slowdown caused by synchronized`.
- Why? 
    - Making the whole getInstance() method synchronized slows things down.
    - So double-checked locking synchronizes only during the first creation, not every time.
- How it works (simple):
    1. Check 1 (outside synchronized)
        - If instance already exists f-> return it without locking (fast).

    2. Check 2 (inside synchronized)
        - If instance is still null, then create it.
    
    ```java
    public class Singleton{
        private static volatile Singleton instance; // Without volatile, instruction reordering can cause another thread to get a partially constructed Singleton instance, breaking thread safety.
        private Singleton() {}

        public static Singleton getInstance() {
            if(instance == null) { //double checking mechanism because here we are checking double time instance is null or not
                synchronized(Singleton.class) {
                    if(instance == null) {
                        instance = new Singleton();
                    }
                }
            }
            return instance;
        }
    }
    ```
---

# Break Singleton with Serialization 
- **problem**
    - When a Singleton class is serialized and deserialized, Java creates a `new object`, which `breaks the Singleton pattern`
- **Solution:**
    - Add a `readResolve()` method that `returns the existing instance` instead of creating a new one during deserialization.

```java
public class Singleton implements Serializable {
    private static Singleton instance;
    private Singleton() {}

    public static Singleton getInstance() {
        if(instance == null) { 
            synchronized(Singleton.class) {
                if(instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }

    protected Object readResolve() { 
        // Prevents creation of a new instance during deserialization
        return instance; // prevention
    }
}
```
---

# Break Singleton with Cloning
- Cloning -> it is the process of creating same copy of that object
- **problem** - If a Singleton class implements `Cloneable`, calling `.clone()` `creates a new object`, which breaks the Singleton pattern

```java
Singleton s1 = Singleton.getInstance();
Singleton s2 = (Singleton) s1.clone();

System.out.println(s1 == s2);  // false → Singleton broken
// Cloning creates a completely new copy.
```

- Singleton with Clonning Protection
    ```java
    public class Singleton implements Cloneable {
        private static Singleton instance;
        private Singleton() {}

        public static synchronized Singleton getInstance() {
            if (instance == null) {
                instance = new Singleton();
            }
            return instance;
        }

        @override
        protected Object clone() throws CloneNotSupportedException { 
            throw new CloneNotSupportedException(); // to block cloning // prevention
            // OR: return instance; // return same instance
        }
    }
    ```
---

# Break Singleton with Reflection
- **Reflection** -> can access private constructors, so it can create new objects, even if the constructor is private.


- problem 
```java
Constructor<Singleton> c = Singleton.class.getDeclaredConstructor();
c.setAccessible(true); // makes private constructor accessable
Singleton s1 = Singleton.getInstance();
Singleton s2 = c.newInstance(); // This creates a second object, breaking Singleton.

System.out.println(s1 == s2);  // false → Singleton broken
```

- **solution:**
    - Inside the constructor, check if an instance already exists.
    - If it does → throw an exception.

- Prevention with reflection 
    ```java
    public class Singleton implements Cloneable {
        private static Singleton instance;
        private Singleton() {
            if(instance != null ) {
                throw new RunTimeException("Use getInstance() to create object"); // prevention
            }
        }

        public static synchronized Singleton getInstance() {
            if (instance == null) {
                instance = new Singleton();
            }
            return instance;
        }

    }
    ```
---
