# How does Spring Boot autoconfiguration pick the right DataSource bean?
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
