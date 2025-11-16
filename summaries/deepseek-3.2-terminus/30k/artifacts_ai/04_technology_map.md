| Component Name | Language | Frameworks | Database | Communication | Patterns |
|----------------|----------|------------|----------|---------------|----------|
| Web Layer (Presentation) | Java | Stripes MVC, JSP/JSTL, Spring 5 | HSQLDB | HTTP Request/Response, Session Management | MVC Pattern, Action-Based Controllers, Client-Side Validation |
| Service Layer (Business Logic) | Java | Spring 5 (DI, Transactions) | HSQLDB | Method Calls, @SpringBean Injection | Service Layer, Declarative Transactions (@Transactional), Business Logic Encapsulation |
| Data Access Layer | Java | MyBatis 3, Spring 5 | HSQLDB | JDBC, SqlSession | Mapper Pattern, Repository Pattern, SQL Mapping |
| Domain Model | Java | - | HSQLDB | Object Composition, Method Invocation | Domain Model, Entity Pattern, Value Objects |
| Shopping Cart | Java | - | Session Storage | Session Attributes, Synchronized Maps | Session State, Thread-Safe Collections, BigDecimal Calculations |
| Order Processing | Java | Spring 5 (Transactions) | HSQLDB | Method Calls, Transactional Boundaries | Transaction Script, Saga Pattern (implicit), Sequence Generation |
| Account Management | Java | Spring 5 (Transactions) | HSQLDB | Method Calls, Session Context | Authentication, Profile Management, Personalization |
| Product Catalog | Java | MyBatis 3 | HSQLDB | Database Queries, Search Algorithms | Hierarchical Navigation, Search Pattern, Caching (potential) |
| Testing Framework | Java | JUnit 5, Mockito, AssertJ, Spring Test, Selenide | HSQLDB | Test Method Calls, HTTP Requests | Unit Testing, Integration Testing, UI Testing, Mocking |
| Configuration & Deployment | Java, XML | Spring 5, Maven, Tomcat 9+ | HSQLDB | Dependency Injection, WAR Deployment | Convention over Configuration, Layered Architecture |