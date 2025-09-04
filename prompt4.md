| Component Name | Language | Frameworks | Database | Communication | Patterns |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Presentation Layer | Java, JSP | Stripes, Spring Web, JSTL | N/A | HTTP, Method Calls | Model-View-Controller (MVC) |
| Business Logic Layer | Java | Spring Framework (Core, DI, Tx) | N/A | Method Calls (via DI) | Service Layer, Facade, Dependency Injection |
| Data Access Layer | Java, XML | MyBatis, MyBatis-Spring, Spring JDBC | HSQLDB | JDBC | Data Mapper, Repository |
| Domain Model | Java | N/A | N/A | Data Transfer Object (DTO) | Plain Old Java Object (POJO) |
| Build & Deployment | YAML, XML, Shell | Maven, Docker, Cargo, GitHub Actions | N/A | N/A | CI/CD, Infrastructure as Code |