```markdown
| Component Name | Language | Frameworks | Database | Communication | Patterns |
|---------------|----------|------------|----------|---------------|----------|
| Web Presentation Layer | Java | Stripes 1.6.0, JSP 2.1, JSTL | None | HTTP Request-Response, Session Management | MVC, Front Controller, Session State |
| Business Service Layer | Java | Spring 5.3.39 (Dependency Injection, Transaction Management) | HSQLDB 2.7.4 | Synchronous Method Calls, Spring @Transactional | Service Layer, Transaction Script, Dependency Injection |
| Data Access Layer | Java | MyBatis 3.5.19, Spring JDBC | HSQLDB 2.7.4 | JDBC, MyBatis SQL Mapping | Data Mapper, Repository, Optimistic Locking |
| Domain Model | Java | None (POJOs) | None | Direct Object References | Domain Model, Value Object, Entity |
| Shopping Cart | Java | None | Session Storage (HSQLDB for inventory) | Session Attributes, Direct Method Calls | Session State, Composite, Business Delegate |
| Order Processing | Java | Spring Transactions | HSQLDB 2.7.4 | Synchronous Method Calls, Database Transactions | Unit of Work, Saga Pattern, Sequence Generation |
| User Authentication | Java | Stripes, Spring Security (implicit) | HSQLDB 2.7.4 | Session-based Authentication | Authentication Manager, Session Management |
| Product Catalog | Java | Spring 5, MyBatis | HSQLDB 2.7.4 | Synchronous Service Calls | Service Layer, Data Access Object, Search Strategy |
| Testing Framework | Java | JUnit 5, Mockito, AssertJ, Selenide | HSQLDB (Test) | Test Method Calls, HTTP (UI Tests) | Test Pyramid, Mock Objects, Page Object |
| Build & Deployment | Java/XML | Maven, Spring Configuration | HSQLDB 2.7.4 | Maven Dependencies, Spring DI | Factory Pattern, Dependency Injection, Builder |
```