# What is Dependency Injection? How does Spring handle it internally?

## Definition
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
