| Component Name | Language | Frameworks | Database | Communication | Patterns |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Presentation Layer** | Java | Stripes Framework, JSP, Spring Framework (for DI) | N/A | HTTP, In-process method calls | Model-View-Controller (MVC), Session Management, Dependency Injection (DI) |
| **Business Logic Layer** | Java | Spring Framework | N/A | In-process method calls | Service Layer, Transactional Service, Dependency Injection (DI), Facade |
| **Data Access Layer** | Java, XML | MyBatis, MyBatis-Spring, Spring JDBC | HSQLDB (SQL) | JDBC | Data Mapper, Repository |
| **Domain Model** | Java | N/A (POJOs) | N/A | In-process object passing | Plain Old Java Object (POJO), Data Transfer Object (DTO) |