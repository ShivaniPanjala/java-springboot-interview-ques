# How do you configure connection pool (HikariCP) for high-throughput services?
For high-throughput services, configure HikariCP for a **fixed-size pool** that minimizes blocking and maximizes connection reuse.

---

### Key Configuration Settings
 **`minimumIdle` = `maximumPoolSize`**: Set these equal to create a **fixed-size pool**. This eliminates the latency of creating new connections during peak load.
 **`maximumPoolSize`**: Tune this based on your CPU cores and I/O limits (a common start is $\text{Cores} \times 2 + 1$). Do not rely on the default of 10.
 **`connectionTimeout`**: Set this aggressively low (e.g., **500ms to 2000ms**) to enforce a **fail-fast** policy when the pool is saturated.
 **`maxLifetime`**: Set this slightly **shorter than the database/firewall timeout** to proactively recycle and replace old connections, preventing silent errors.



# ⚙️ HikariCP High-Throughput Configuration Guide

| **Property**             | **High-Throughput Setting**                   | **Sequential Scenario and Impact** | **Key Takeaway** |
|--------------------------|-----------------------------------------------|------------------------------------|------------------|
| **1. maximumPoolSize**   | Tuned to `Cores × 2 + 1` (e.g., 40–50)        | **Scenario:** A pool is set to 20. When 25 requests arrive simultaneously, 20 are instantly served, and the remaining 5 are queued. | Sets the **absolute capacity** and prevents the application from overloading the database, forcing excess requests to queue internally. |
| **2. minimumIdle**       | Equal to `maximumPoolSize`                    | **Scenario:** The pool is fixed (minimumIdle=20). After a massive load, all 20 connections are returned and remain idle. | Creates a **Fixed-Size Pool**, ensuring connections are instantly available upon request, eliminating latency of new connection creation. |
| **3. connectionTimeout** | Aggressively Low (e.g., `1500ms`)             | **Scenario:** 3 requests are waiting in the queue because the pool is full. After 1500ms, the timeout is hit. | Enforces the **Fail-Fast Policy** — threads that can’t get a connection are unblocked quickly (instead of waiting 30s), improving responsiveness under load. |
| **4. idleTimeout**       | Irrelevant (keep high or default)             | **Scenario:** In a Fixed-Size Pool (`minimumIdle=20`, `maximumPoolSize=20`), the pool never exceeds the minimum idle threshold. | **Ignored** in fixed pools. Only relevant in dynamic pools where it defines how long idle connections stay open before being closed. |
| **5. maxLifetime**       | Shorter than DB/Firewall Timeout(`5 minutes`) | **Scenario:** A connection reaches 5 min lifetime, is marked for retirement, and replaced when returned. | Acts as a **Proactive Recycler** — prevents usage of dead/stale connections killed by DB or firewall, avoiding runtime errors. |

---

- Use a **fixed-size pool** for high-throughput, low-latency services.  
- Set `maxLifetime` below the DB timeout to avoid stale connections.  
- Keep `connectionTimeout` low to fail fast during overloads.  
- Monitor metrics and tune gradually for optimal throughput.
