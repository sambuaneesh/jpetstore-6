```markdown
| Component Name | Language | Frameworks | Database | Communication | Patterns |
|---------------|----------|------------|----------|---------------|----------|
| **Web Layer (ActionBeans)** | Java | Stripes 1.6.0, Spring 5.3.39, JSP 2.3, Servlet 4.0 | HSQLDB 2.7.4 | HTTP Requests, Stripes Actions (.action endpoints) | MVC, Front Controller, Session Façade, @SessionScope |
| **Service Layer** | Java | Spring 5.3.39 (Context, JDBC, Web), MyBatis-Spring 3.0.5 | HSQLDB 2.7.4 | Direct Java method calls, Spring dependency injection | Service Layer, Dependency Injection, Declarative Transactions (@Transactional) |
| **Data Access Layer (Mappers)** | Java | MyBatis 3.5.19, Spring JDBC | HSQLDB 2.7.4 | MyBatis SQL mapping, JDBC connections | Data Mapper, Repository, ORM with XML configurations |
| **Domain Models** | Java | Pure Java objects | HSQLDB 2.7.4 | Object serialization/deserialization | Domain Model, POJO, Data Transfer Object |
| **Shopping Cart** | Java | Spring 5.3.39 (Session management) | HTTP Session (in-memory) | Session attributes, synchronized maps | Session State, Thread-safe collections, BigDecimal for monetary calculations |
| **Order Processing** | Java | Spring 5.3.39 (Transactional) | HSQLDB 2.7.4 | Transactional method calls, sequence generation | Transaction Script, ACID compliance, Sequence-based ID generation |
| **Catalog Search** | Java | MyBatis 3.5.19 | HSQLDB 2.7.4 | Database queries with wildcards | Multi-keyword search algorithm, Result aggregation |
| **Authentication System** | Java | Spring 5.3.39, Stripes 1.6.0 | HSQLDB 2.7.4 | Session-based authentication, database credential validation | Session Management, Credential validation |
```