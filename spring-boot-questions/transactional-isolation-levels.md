## Explain how @Transactional isolation levels work in banking systems.

**Dirty Read**: Reading data that has been written but not yet committed by another transaction.

**Non-Repeatable Read**: Reading the same row twice within a single transaction and seeing different committed values because another transaction modified and committed the change in between.

**Phantom Read**: Executing the same range-based query twice within a single transaction and seeing a different set of rows because another transaction inserted or deleted rows matching the range and committed the change.

1. Read Uncommitted
    This is the lowest level of isolation where a transaction can see uncommitted changes made by other transactions. This can result in dirty reads, non-repeatable reads, and phantom reads.

2. Read Committed
    In this isolation level, a transaction can only see changes made by other committed transactions. This eliminates dirty reads but can still result in non-repeatable reads and phantom reads.

3. Repeatable Read
    This isolation level guarantees that a transaction will see the same data throughout its duration, even if other transactions commit changes to the data. However, phantom reads are still possible.

4. Serializable
    This is the highest isolation level where a transaction is executed as if it were the only transaction in the system. All transactions must be executed sequentially, which ensures that there are no dirty reads, non-repeatable reads, or phantom reads.

# Non-Repeatable Read vs. Phantom Read


| **Anomaly**             | **What is Changing?**                                                             | 
|-------------------------|-----------------------------------------------------------------------------------|
| **Non-Repeatable Read** | The **value of a column** in a single, specific row changes between reads.        |
| **Phantom Read**        | The **number of rows** returned by a **range-based query** changes between reads. | 

Non-Repeatable Read Example: 
```sql
SELECT Balance FROM Accounts WHERE AccountID = 101;  
-- Result: 500  
-- (Transaction-B updates balance to 600 and commits)  
SELECT Balance FROM Accounts WHERE AccountID = 101;  
-- Result: 600  
``` 

Phantom Read EXample:
```sql
SELECT * FROM Employees WHERE Salary > 70000;  
-- Result: 10 rows  
-- (Transaction-B inserts a new employee with Salary = 80000 and commits)  
SELECT * FROM Employees WHERE Salary > 70000;  
-- Result: 11 rows  
``` 

| **Type**                | **What Changed?**                                    | **Example**                             | **Prevented By** |
|-------------------------|------------------------------------------------------|-----------------------------------------|------------------|
| **Non-Repeatable Read** | The **value** of an existing record changed          | Account balance updated mid-transaction | `REPEATABLE_READ`|
| **Phantom Read**        | The **number of rows** in the query result changed   | New employee inserted into query range  | `SERIALIZABLE`   |



## Isolation in Banking
In banking, data integrity is paramount, so the isolation level chosen is extremely important, especially for transactions involving money movement:

# For Money Transfers (Write Operations):

The SERIALIZABLE isolation level is often preferred for critical transactions like transferring money from Account A to Account B. This level ensures that the concurrent execution of transactions is equivalent to some serial (one-at-a-time) execution, preventing all anomalies. This guarantees that the money that leaves Account A arrives intact at Account B with no interference.

Some systems might use REPEATABLE READ combined with explicit locking (like SELECT ... FOR UPDATE) on the specific account records being modified to achieve high consistency for the critical path while potentially allowing better concurrency on other parts of the system.

# For Simple Reads (Reporting/Display):

READ COMMITTED is a very common default for many database systems (like PostgreSQL) and is often used for non-critical reads, such as checking an account balance for display. It prevents reading uncommitted changes, ensuring all data seen is "real" (committed). It tolerates non-repeatable reads, meaning if you check your balance twice in a long transaction, it might change if another transaction commits in between

