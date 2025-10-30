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

