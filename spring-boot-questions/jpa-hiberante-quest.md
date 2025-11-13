# How would you optimize large batch inserts/updates in Hibernate?
  1. Enable JDBC batching **hibernate.jdbc.batch_size = 50**
  2. Manage session memory to avoid memory issues
      - session.flush() (to execute the batch)
      - session.clear() (to free the cache)
  3. Do not use **GenerationType.IDENTITY** for IDs, as it disables batching. Use GenerationType.SEQUENCE instead.

  ```
  Transaction tx = session.beginTransaction();
  int batchSize = 50;
  for (int i = 0; i < items.size(); i++) {
  session.save(items.get(i));
  
      if (i % batchSize == 0) {
          session.flush();
          session.clear(); // Detach objects to free memory
      }
  }
  
  tx.commit();
  session.close();
  ```
---

# Explain difference between entity graphs and fetch joins for optimizing queries
1. Fetch Join
    - Defined directly in JPQL/HQL using JOIN FETCH.
    - Eagerly loads related entities in the same query.
    - Simple and explicit, but hardcoded in the query.
    - Best for one-off queries where you know exactly which associations to fetch.
    Example:
    ```
      @Query("SELECT d FROM Department d JOIN FETCH d.employees WHERE d.id = :id")
      Department findDepartmentWithEmployees(@Param("id") Long id)
    ```
2. Entity Graph
    - Defined via annotations or dynamically at runtime.
    - Specifies which associations to fetch without modifying the JPQL query.
    - Reusable across multiple queries; more flexible for APIs or dynamic fetch plans.
---

# How does 2nd level caching work in Hibernate and which providers have you used?
- Second-level cache stores entities across sessions (shared at SessionFactory level).
- Reduces database hits by reusing data from memory.

## How It Works
   - Hibernate first checks the first-level cache (per session).
   - If the entity is not found, it checks the second-level cache.
   - If still not found, it queries the database.
   - The fetched entity is then stored in the second-level cache for future sessions.

## Common Cache Providers
  1. Ehcache
  2. Infinispan
  3. Hazelcast
  4. JCache

## configuration
```
    hibernate.cache.use_second_level_cache=true
    hibernate.cache.region.factory_class=org.hibernate.cache.ehcache.EhCacheRegionFactory
```
## example
```
    @Entity
    @Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
    public class Employee {
        @Id
        private Long id;
        private String name;
    }
```

Second-level cache allows Hibernate to reuse entities across sessions, improving performance by avoiding repeated database queries.
---

