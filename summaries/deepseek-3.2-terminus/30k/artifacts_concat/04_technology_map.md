```markdown
| Component Name | Language | Frameworks | Database | Communication | Patterns |
|---------------|----------|------------|----------|---------------|----------|
| **Account Management** | Java 17 | Spring 5 (DI, Transactions), Stripes MVC | HSQLDB | In-process method calls, HTTP sessions | MVC, Dependency Injection, Transactional, Mapper |
| **Product Catalog** | Java 17 | Spring 5, Stripes MVC, MyBatis 3 | HSQLDB | In-process method calls | Layered Architecture, Mapper, Dependency Injection |
| **Shopping Cart** | Java 17 | Spring 5, Stripes MVC | HSQLDB (for item data), HTTP Session | Session-based state, In-process calls | Session State, Domain Model, Thread-safe collections |
| **Order Processing** | Java 17 | Spring 5 (Transactions), Stripes MVC, MyBatis 3 | HSQLDB | In-process method calls | Transactional, Mapper, Sequence Generation |
| **Inventory Management** | Java 17 | Spring 5, MyBatis 3 | HSQLDB | In-process method calls | Mapper, Transactional updates |
| **Web Interface** | Java 17, JSP, CSS | Stripes MVC, JSTL, Spring 5 | HSQLDB | HTTP requests/responses, Session management | MVC, Action Beans, Template includes |
| **Data Access Layer** | Java 17 | MyBatis 3, Spring 5 | HSQLDB | JDBC, Spring-managed transactions | Mapper interfaces, ORM, Declarative transactions |
| **Testing Framework** | Java 17 | JUnit 5, Mockito, AssertJ, Spring Test, Selenide | HSQLDB (embedded) | In-process test calls | Unit Testing, Integration Testing, E2E Testing |
| **CI/CD Pipeline** | YAML, Java | GitHub Actions, Maven, Docker | HSQLDB | HTTP, Container orchestration | Matrix builds, Multi-container deployment |
```