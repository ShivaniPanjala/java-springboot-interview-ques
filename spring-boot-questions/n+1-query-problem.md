# How to detect and fix N+1 query problem in Spring Data JPA?
This problem occurs when an ORM framework executes one initial query (N=1) to retrieve a collection of parent entities, and then executes an additional query (N) for each child entity accessed, leading to poor performance

## How to Detect It
- The quickest way to detect it is 
    by *enabling SQL logging*
        **spring.jpa.show-sql=true**
    and looking for a pattern of one SELECT followed by many identical SELECT statements inside your application logs.

- There are two primary ways to fix this in Spring Data JPA:

    1. **For Complex Queries** : 
        - Use ****@Query**** with the ****JOIN FETCH****@ clause. 
        - This forces Hibernate to load the parent and children in a single, efficient query 
        - (SELECT p FROM Parent p JOIN FETCH p.children)
        - completely eliminating the N separate lookups.

    2. **For Repository-Level Control** : 
        - Use the ****@EntityGraph****@ annotation on your repository method. 
        - This is a cleaner, more declarative way to specify which relationships must be eagerly loaded,
        - ensuring the data is fetched in one optimized query without writing complex SQL.