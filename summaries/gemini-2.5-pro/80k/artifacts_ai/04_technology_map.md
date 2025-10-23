| Component Name | Language | Frameworks | Database | Communication | Patterns |
|---|---|---|---|---|---|
| Presentation Layer | Java, JSP | Stripes, Spring Framework (for DI) | N/A | HTTP, In-process method calls (to Service Layer) | Model-View-Controller (MVC) |
| Business Logic Layer | Java | Spring Framework | N/A | In-process method calls (from Presentation Layer, to Data Access Layer) | Transactional Service Layer, Dependency Injection, Facade |
| Data Access Layer | Java, SQL | MyBatis, MyBatis-Spring, Spring JDBC | HSQLDB | In-process method calls (from Service Layer), JDBC | Data Mapper |