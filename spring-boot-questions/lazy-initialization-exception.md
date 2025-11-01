## How to Debug LazyInitializationException in JPA/Hibernate

A `LazyInitializationException` occurs when you try to access a **lazy-loaded association** (like a `@OneToMany` or `@ManyToOne`) **after the Hibernate session has been closed** — meaning Hibernate can’t fetch the data it deferred.

Think of a Hibernate lazy association as a **“promise”**:

> “I’ll load this data later **if you ask**, as long as the session is still open.”

If you ask **after** the session is closed → it can’t keep that promise → 💥 `LazyInitializationException`

---
Example
```java
Doctor doctor = doctorRepository.findById(1L).orElseThrow();
System.out.println(doctor.getAppointments().size()); // ❌ LazyInitializationException
```
LazyInitializationException occurs because:
- The `appointments` collection in Doctor is mapped as `@OneToMany(fetch = LAZY)`.
- doctorRepository.findById(1L) loads the Doctor entity **within a short-lived session**.
- After the repository method returns, the Hibernate session is **closed**.
- Calling doctor.getAppointments().size() **needs to fetch the lazy collection** from the database.
- Since the session is closed, Hibernate cannot load it → throws LazyInitializationException.


`@OneToMany` defaults to `FetchType.LAZY`.
When the session is closed (e.g., outside a `@Transactional` boundary), the collection can’t be loaded anymore.

How to Debug Step-by-Step

1. Identify which entity and field caused the exception
→ from the stack trace (e.g., `Doctor.appointments`)

2. Check whether you’re accessing that field inside a `@Transactional` method
→ If not, the session is likely closed.

3. Decide how to fix it:
    - Use a `JOIN FETCH` query if you need related data eagerly.
    - Or ensure access happens within a `@Transactional` scope.
    - Avoid using `FetchType.EAGER` on mappings, as it can cause:
        N+1 query problems
        Performance degradation
        Unnecessary data loading

Fix — Using JOIN FETCH

```java
@Query("SELECT d FROM Doctor d JOIN FETCH d.appointments WHERE d.id = :id")
Doctor findDoctorWithAppointments(@Param("id") Long id);
```

Now both Doctor and Appointments are fetched in one query —
no lazy load, no N+1, no exception.

Always keep relationships `LAZY` by default.
Access them inside a transaction** or load them explicitly using JOIN FETCH when you actually need the data.
Avoid `FetchType.EAGER` — it can lead to N+1 queries and heavy performance costs.


