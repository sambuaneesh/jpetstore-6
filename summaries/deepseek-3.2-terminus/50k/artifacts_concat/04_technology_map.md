```markdown
| Component Name | Language | Frameworks | Database | Communication | Patterns |
|---------------|----------|------------|----------|---------------|----------|
| **Web Layer** | Java | Stripes MVC, JSP, JSTL | HSQLDB | HTTP Requests/Responses, Session State | MVC, Front Controller, Session Facade |
| **Account Service** | Java | Spring 5 (DI, Transactions) | HSQLDB | Synchronous Method Calls | Service Layer, Transaction Script |
| **Catalog Service** | Java | Spring 5 (DI, Transactions), MyBatis 3 | HSQLDB | Synchronous Method Calls | Service Layer, Data Mapper |
| **Order Service** | Java | Spring 5 (DI, Transactions), MyBatis 3 | HSQLDB | Synchronous Method Calls | Service Layer, Transaction Script, Optimistic Locking |
| **Cart Service** | Java | Spring 5 (DI) | Session Storage | Session-based State | Session Facade, Value Object |
| **Data Access Layer** | Java | MyBatis 3, Spring Integration | HSQLDB | JDBC | Data Mapper, DAO Pattern |
| **Domain Models** | Java | Plain Java Objects | HSQLDB | In-memory Objects | Anemic Domain Model, POJO |
| **Inventory Management** | Java | MyBatis 3, Spring Transactions | HSQLDB | Synchronous Method Calls | Transaction Script, Atomic Updates |
| **Sequence Management** | Java | MyBatis 3 | HSQLDB | Database Sequences | Sequence Generator Pattern |
```