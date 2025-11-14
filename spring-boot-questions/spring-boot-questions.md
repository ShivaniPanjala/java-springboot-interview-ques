# What is Dependency Injection? How does Spring handle it internally?
Dependency Injection is a design pattern in which an object **receives its dependencies from an external source** rather than creating them itself.

## Types of Dependency Injection
1. **Constructor Injection** – Dependencies are provided through the class constructor.  
2. **Setter Injection** – Dependencies are provided through setter methods after object creation.  
3. **Field Injection** – Dependencies are injected directly into fields.

## How Spring Handles DI
- **Component Scanning:** Detects classes annotated with `@Component`, `@Service`, etc.  
- **Bean Registration:** Registers beans in the `ApplicationContext`.  
- **Dependency Resolution:** Identifies required dependencies for each bean.  
- **Injection:** Injects dependencies via constructor, setter, or field.  
- **Lifecycle Management:** Manages bean lifecycle, including `@PostConstruct` and `@PreDestroy`.
Note:  Constructor injection provides dependencies when the object is created (mandatory), while setter injection provides them after creation (optional or changeable).
---

# Difference between @Component, @Service, and @Repository

| Annotation       | Purpose / Use Case                                           | Special Features / Notes |
|-----------------|--------------------------------------------------------------|--------------------------|
| `@Component`    | Generic stereotype for any Spring-managed bean              | Base annotation; can be used for any class that Spring should manage. |
| `@Service`      | Specialized for service layer classes (business logic)     | Semantically indicates a service; no extra behavior, mainly for readability. |
| `@Repository`   | Specialized for DAO / repository classes (data access)     | Translates database exceptions into Spring’s `DataAccessException` automatically. |

---

# How does Spring Boot auto-configuration work?

- **What:** Automatically configures Spring beans based on **classpath dependencies** and application properties, reducing boilerplate code.  
- **How it works:** 
  1. `@SpringBootApplication` → enables `@EnableAutoConfiguration`.  
  2. Spring Boot loads auto-configuration classes from `META-INF/spring.factories`.  
  3. Conditional annotations (`@ConditionalOnClass`, `@ConditionalOnMissingBean`, etc.) decide which beans to create.  
  4. Beans are registered in the `ApplicationContext`.  
- **Key Points:**  
  - Opinionated defaults based on dependencies.  
  - Can be overridden by defining your own beans.  
  - Auto-configurations can be excluded using `exclude` attribute.
---

# How does Hibernate manage transactions?

Hibernate manages transactions using its `Transaction` API. 
    - You begin a transaction, 
    - perform database operations, 
    - then **commit** or **rollback**. 
### Hibernate Transaction Example

Hibernate provides the `org.hibernate.Transaction` interface to control transactions manually.  

**Typical workflow:**

```java
Session session = sessionFactory.openSession();
Transaction tx = null;

try {
    tx = session.beginTransaction();
    
    // Perform CRUD operations
    
    tx.commit(); // commit changes
} catch (Exception e) {
    if (tx != null) tx.rollback(); // rollback in case of error
} finally {
    session.close();
}
```

In Spring, transaction management can be simplified using `@Transactional`, where Spring handles starting, committing, and rolling back transactions automatically.

**Key Points:**
- Ensures **ACID** properties.
- All operations within a transaction are part of the same **Session / persistence context**.
- `commit()` writes changes to the database; `rollback()` discards them.

---

# Hibernate Entity Lifecycle States
Hibernate entities go through **four main states** during their lifecycle:
- **Transient:** Newly created object, not associated with any Hibernate session.  
- **Persistent:** Associated with an active Hibernate session; changes are tracked and saved to the DB.  
- **Detached:** Was persistent, but session is closed; changes are no longer tracked.  
- **Removed:** Marked for deletion; will be deleted from the database on commit/flush.
---

# Exception handling best practices in REST APIs
- In Spring REST APIs, handle exceptions centrally using `@ControllerAdvice` and `@ExceptionHandler` to return meaningful HTTP status codes and messages.  
Always use a consistent response structure, avoid exposing internal details, and log errors for debugging.
---

# Difference between Lazy and Eager loading
  - Lazy Loading:  Data is loaded **only when it’s accessed** for the first time.  
  - Eager Loading: Data is loaded **immediately along with the parent entity.**
---

# What are N+1 query problems? How to solve them?
**N+1 Query Problem:**  
Occurs when Hibernate executes **1 query to fetch parent entities** and then **N additional queries** (one for each parent) to fetch their child entities lazily.

**Example:**  
1 query for all doctors → N queries for each doctor’s appointments.

**Solution:**  
 - Use **`JOIN FETCH`** (JPQL) or **`@EntityGraph`** to fetch related entities in a single query, or adjust the **fetch strategy** wisely.
---

# How to avoid code duplication when multiple entities share common fields?
We can avoid code duplication by using a **Base Entity class** with common fields 
and marking it with `@MappedSuperclass`.

All other entities extend this base class and automatically inherit its fields.

**Example:**
```java
@MappedSuperclass
public abstract class BaseEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;
}

@Entity
public class Doctor extends BaseEntity {
    private String name;
}
```

# Difference between Spring Boot starters vs manual dependency management?
**Spring Boot Starters:**
- Predefined dependency bundles provided by Spring Boot (e.g., `spring-boot-starter-web`, `spring-boot-starter-data-jpa`).
- Automatically include all required libraries with compatible versions.
- Simplify project setup and reduce version conflicts.

**Manual Dependency Management:**
- You add each dependency (and its version) manually in `pom.xml` or `build.gradle`.
- You must manage compatibility and transitive dependencies yourself.
- More flexible but requires more maintenance and version management.
---

# How do you implement idempotency in REST APIs?
**Definition:**  
Idempotency ensures that making the same request multiple times results in the **same outcome** — preventing duplicate operations.

## How It’s Implemented
- The client sends a unique header, e.g., **`Idempotency-Key`**, with each request.
- The server stores this key and the result of the first request.
- If the same key is received again, the server **returns the same response** instead of executing the operation again.

## Example
**Request:**
```http
POST /api/payments
Idempotency-Key: daznId-12345
Content-Type: application/json

{
  "amount": 100,
  "currency": "USD",
  "userId": "U1001"
}
```
---

# How do you handle huge traffic in Spring Boot APIs? (connection pool, caching, async calls)
1. **Connection Pooling (HikariCP):**  
   Use a properly tuned connection pool (e.g., `maximumPoolSize`, `connectionTimeout`) to efficiently reuse database connections instead of opening new ones for every request.
2. **Caching:**  
   Use caching (e.g., `@Cacheable`, Redis, or Caffeine) to store frequently accessed data in memory, reducing database load and improving response time.
3. **Asynchronous Calls:**  
   Use `@Async` or message queues (like RabbitMQ/Kafka) to process heavy or background tasks asynchronously, freeing up threads for incoming requests.

**Summary:**  
Optimize database access (via pooling), reduce repetitive work (via caching), and offload slow operations (via async processing) to efficiently handle **high traffic** and maintain **fast, scalable APIs**.
---

# Explain Spring Boot Starter dependencies and how they simplify configuration.
Starters are like ready-made packages of libraries for a specific feature
## How They Simplify Things
  1. **Automatic setup**: Spring Boot configures most things for you.
  2. **Compatible versions**: You don’t need to worry about version conflicts.
  3. **Less manual work**: No XML or complicated setup—just start coding.
## Examples:
  - spring-boot-starter-web
  - spring-boot-starter-data-jpa
  - spring-boot-starter-security
---

# Difference between @Autowired and @Qualifier.
@Autowired-  finds beans **by type**
@Qualifier- **resolves ambiguity** when multiple beans of the same type exist
---

# How do you handle global exception handling in Spring Boot REST APIs?
handle exceptions globally using `@ControllerAdvice` and `@ExceptionHandler`.
**@ControllerAdvice** allows centralized exception handling across all controllers.
**@ExceptionHandler** maps specific exceptions to custom responses.
You can create custom response objects instead of returning plain strings.
---

# How do you implement Spring Boot profiles for different environments?
Activate the profile using **spring.profiles.active=dev**
Use **@Profile** to load beans only for certain environments
@Profile("dev") - Spring will only load the beans that match the active profile.
---

# How do you secure Spring Boot REST APIs using JWT/OAuth2?


---
