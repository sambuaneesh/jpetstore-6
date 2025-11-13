| Component Name | Language | Frameworks | Database | Communication | Patterns |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Presentation Layer (Web Actions)** | Java, JSP | Stripes, Spring | N/A | HTTP/HTTPS, Java Method Calls | Model-View-Controller (MVC), Abstract Base Class, Separation of Concerns |
| **Service Layer** | Java | Spring, Mockito, JUnit | N/A | Java Method Calls | Service Layer, Façade, Dependency Injection, Domain-Driven Design (DDD) concepts |
| **Domain Layer** | Java | JUnit | N/A | Java Method Calls (as Data Transfer Objects) | Domain Model, Entity, POJO, Composition over Inheritance, Serializable |
| **Persistence Layer (MyBatis Mappers)** | Java, XML (for SQL) | MyBatis, Spring, JUnit | Relational Database (HSQLDB for tests), JDBC | Java Method Calls, JDBC | Data Access Object (DAO), Repository, Interface-based Contracts |
| **Build Support (Maven Wrapper)** | Java | Apache Maven | N/A | HTTP/HTTPS | Build Automation, Utility-First Design, Environment Variable Configuration |
| **UI Integration Testing** | Java | JUnit, UI Automation Framework (e.g., Selenium) | Relational Database (via application stack) | HTTP/HTTPS (via browser automation) | End-to-End Testing, User Journey Simulation, UI-Driven Validation |