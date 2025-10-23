| Component Name | Language | Frameworks | Database | Communication | Patterns |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Web / Presentation Layer** | Java | Stripes, JavaServer Pages (JSP), JSTL | N/A (Delegates to Business Logic Layer) | HTTP, In-process Method Invocation | Model-View-Controller (MVC), Front Controller, Session State Management |
| **Business Logic Layer** | Java | Spring Framework | N/A (Delegates to Persistence Layer) | In-process Method Invocation | Service Layer, Dependency Injection (DI), Transactional Service |
| **Persistence Layer** | Java, XML | MyBatis, mybatis-spring | HSQLDB | JDBC, In-process Method Invocation | Data Mapper |
| **Domain Model** | Java | N/A (Plain Old Java Objects - POJOs) | N/A | N/A | Domain Model |
| **Testing** | Java | JUnit 5, Mockito, Selenide | HSQLDB (In-memory) | N/A | Unit Testing, Integration Testing, Browser Automation Testing |
| **Build & CI/CD Pipeline** | YAML, XML | Maven, GitHub Actions, Docker, SonarCloud, Coveralls, Tomcat (embedded) | N/A | N/A | Continuous Integration (CI), Continuous Deployment (CD) |