````markdown
## How does Spring Boot autoconfiguration pick the right DataSource bean?
```
┌────────────────────────────────────────────────────┐
│ 1️⃣ @SpringBootApplication                           │
│ └─ Combines:                                        │
│     @Configuration + @EnableAutoConfiguration +     │
│     @ComponentScan                                  │
└────────────────────────────────────────────────────┘
                          │
                          ▼
┌────────────────────────────────────────────────────┐
│ 2️⃣ @EnableAutoConfiguration                         │
│ └─ Imports AutoConfigurationImportSelector          │
└────────────────────────────────────────────────────┘
                          │
                          ▼
┌────────────────────────────────────────────────────┐
│ 3️⃣ AutoConfigurationImportSelector                  │
│ └─ Calls getCandidateConfigurations()               │
│     → ImportCandidates.load(AutoConfiguration.class)│
│     → Reads META-INF/spring/org...AutoConfiguration.imports │
│     → Loads list of auto-config classes             │
│        e.g., DataSourceAutoConfiguration,           │
│        WebMvcAutoConfiguration, etc.                │
└────────────────────────────────────────────────────┘
                          │
                          ▼
┌────────────────────────────────────────────────────┐
│ 4️⃣ Spring evaluates conditions on each class       │
│    @ConditionalOnClass(DataSource.class) ✅          │
│    @ConditionalOnMissingBean(DataSource.class) ✅     │
│    @ConditionalOnProperty(...) ✅                    │
│  ✅ → DataSourceAutoConfiguration is active          │
└────────────────────────────────────────────────────┘
                          │
                          ▼
┌────────────────────────────────────────────────────┐
│ 5️⃣ DataSourceAutoConfiguration loads               │
│ └─ @EnableConfigurationProperties(DataSourceProperties.class) │
│ └─ @Import(DataSourceConfiguration.Hikari.class, ...)         │
│  → Registers DataSourceProperties bean             │
│  → Imports nested Pooled/Embedded configurations   │
└────────────────────────────────────────────────────┘
                          │
                          ▼
┌────────────────────────────────────────────────────┐
│ 6️⃣ ConfigurationPropertiesBindingPostProcessor     │
│  → Binds application.properties → DataSourceProperties │
│     spring.datasource.url, username, password, etc.│
│  → Populates fields in DataSourceProperties bean   │
└────────────────────────────────────────────────────┘
                          │
                          ▼
┌────────────────────────────────────────────────────┐
│ 7️⃣ DataSourceConfiguration.<Type> (e.g. Hikari)    │
│  → @ConditionalOnClass(HikariDataSource.class) ✅    │
│  → @ConditionalOnProperty(type=...)                 │
│  ✅ → Creates HikariDataSource bean using           │
│     DataSourceBuilder.create()                      │
│       .url(properties.getUrl())                     │
│       .username(properties.getUsername())           │
│       .password(properties.getPassword())           │
│       .build()                                      │
└────────────────────────────────────────────────────┘
                          │
                          ▼
┌────────────────────────────────────────────────────┐
│ 8️⃣ DataSource bean is registered in ApplicationContext│
│  → Now available for JdbcTemplate, JPA, etc.       │
└────────────────────────────────────────────────────┘
                          │
                          ▼
┌────────────────────────────────────────────────────┐
│ 9️⃣ Additional auto-configs wrap around DataSource │
│   • DataSourceHealthContributorAutoConfiguration   │
│   • SqlInitializationAutoConfiguration             │
│   • DataSourcePoolMetadataProvidersConfiguration   │
│  → Adds health checks, metrics, initialization     │
└────────────────────────────────────────────────────┘
```
---
````

````markdown
## Explain Spring Boot starter parent and dependency management?
Spring Boot Starter Parent = a ready-made parent POM with defaults + dependency management.  
Dependency Management = Spring Boot’s curated list of dependency versions, ensuring compatibility and reducing boilerplate.

## 1. Spring Boot Starter Parent

The **Spring Boot Starter Parent** is a special parent POM (`pom.xml`) provided by Spring Boot that offers:
- Default configurations for common Maven plugins.
- Dependency management for Spring Boot dependencies.
- Common build settings like Java version, encoding, and test configurations.

You include it in your `pom.xml` like this:

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.3.5</version> <!-- example version -->
</parent>
```

### What It Does

* Sets default plugin versions (no need to specify them manually).  
* Manages dependency versions through a built-in dependency management section.  
* Provides default configuration values, such as:

```xml
<properties>
    <java.version>17</java.version>
    <maven.compiler.source>${java.version}</maven.compiler.source>
    <maven.compiler.target>${java.version}</maven.compiler.target>
</properties>
```

* Simplifies project setup — you don’t need to define everything from scratch.

---

## 2. Dependency Management

Dependency management in Spring Boot (through the parent POM) means:

* You **don’t have to specify versions** for most Spring dependencies.  
* Spring Boot manages versions for you via its curated **dependency management section**.

Example:

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
</dependencies>
```

✅ You don’t need to specify versions for these dependencies because the parent POM already knows which versions are compatible with your chosen Spring Boot version.

---

## 3. Without the Starter Parent

If you **don’t** want to use the `spring-boot-starter-parent`, you can still get dependency management using the **Spring Boot dependency management BOM** (Bill of Materials):

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>3.3.5</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

This is useful when:

* You already have a custom parent POM.  
* You just want the dependency management benefits without inheriting plugin configurations or default properties.
````

````markdown
## How to Debug LazyInitializationException in JPA/Hibernate

A `LazyInitializationException` occurs when you try to access a **lazy-loaded association** (like a `@OneToMany` or `@ManyToOne`) **after the Hibernate session has been closed** — meaning Hibernate can’t fetch the data it deferred.

---
Example

```java
Doctor doctor = doctorRepository.findById(1L).orElseThrow();
System.out.println(doctor.getAppointments().size()); // ❌ LazyInitializationException
```

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

````

````markdown
## Explain how @Transactional isolation levels work in banking systems.

````
