| Component Name | Language | Frameworks | Database | Communication | Patterns |
|---|---|---|---|---|---|
| Presentation (Stripes MVC + JSP) | Java, JSP/EL | Stripes MVC, JSP 2.3, JSTL, Servlet 4.0 | None | HTTP/Servlet endpoints (*.action); in-process calls to services via Spring DI; server-side rendering | MVC, Front Controller, Page Controller, Session-scoped controllers, Redirect/Forward navigation |
| Account Service | Java | Spring (@Service, @Transactional, DataSourceTransactionManager), MyBatis-Spring | HSQLDB (SIGNON, ACCOUNT, PROFILE, BANNERDATA) | In-process from ActionBeans; JDBC via MyBatis to DB | Service Layer, Transaction Script, DAO/Data Mapper, Anemic Domain Model |
| Catalog Service | Java | Spring (@Service, @Transactional), MyBatis-Spring | HSQLDB (CATEGORY, PRODUCT, ITEM, INVENTORY, SUPPLIER) | In-process from ActionBeans; JDBC via MyBatis to DB | Service Layer, Transaction Script, DAO/Data Mapper, SQL LIKE search |
| Order Service | Java | Spring (@Service, @Transactional), MyBatis-Spring | HSQLDB (ORDERS, ORDERSTATUS, LINEITEM, SEQUENCE) | In-process from ActionBeans; JDBC via MyBatis to DB | Service Layer, Transaction Script, Unit of Work (Spring Tx), Table-based Sequence, Aggregate assembly |
| Persistence Layer (MyBatis Mappers) | Java + XML | MyBatis, MyBatis-Spring | HSQLDB | JDBC | Data Mapper, Repository/DAO, Per-mapper 2nd-level cache (non-distributed) |
| Domain Model (POJOs) | Java | None (Stripes @Validate on some fields) | N/A | In-process objects shared across layers | Anemic Domain Model, Entities (Account, Order, Item), Aggregates (Order->LineItems), Computed properties (totals) |
| Database | SQL | HSQLDB (embedded) | HSQLDB | JDBC from app | Relational schema with FKs, Table-per-entity, Sequence table for IDs |
| Caching | Java | MyBatis 2nd-level cache | None | In-memory per-JVM (no distribution) | Per-mapper cache, Read-through, Stale-read risk |
| Logging | Java | SLF4J (simple binding) | None | Stdout | Logging Facade |
| Application Container/Runtime | Java | Tomcat 9 (Servlet 4.0); profiles for Jetty/WildFly/TomEE/Liberty/GlassFish/Resin | N/A | HTTP on 8080 | Servlet container, Layered monolith WAR deployment |
| Build/Packaging/Deployment | Maven, Docker | Maven WAR, cargo-maven3-plugin, Dockerfile, docker-compose | N/A | N/A | CI/CD pipeline, Containerized runtime |