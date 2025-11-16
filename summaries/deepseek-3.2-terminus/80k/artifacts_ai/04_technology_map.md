```markdown
| Component Name | Language | Frameworks | Database | Communication | Patterns |
|---------------|----------|------------|----------|---------------|----------|
| Presentation Layer | Java, JSP, HTML, CSS | Stripes 1.6.0, JSTL, Servlet 4.0 | - | HTTP, Stripes ActionBeans, Spring Bean Injection | MVC, Front Controller, Session Façade |
| Account Management | Java | Spring 5.3.39, MyBatis 3.5.19 | HSQLDB 2.7.4 (ACCOUNT, PROFILE, SIGNON tables) | Direct method calls, Spring Dependency Injection | Service Layer, Data Mapper, Dependency Injection |
| Catalog Management | Java | Spring 5.3.39, MyBatis 3.5.19 | HSQLDB 2.7.4 (CATEGORY, PRODUCT, ITEM, SUPPLIER, INVENTORY tables) | Direct method calls, Spring Dependency Injection | Service Layer, Data Mapper, Transaction Script |
| Order Management | Java | Spring 5.3.39, MyBatis 3.5.19 | HSQLDB 2.7.4 (ORDERS, ORDERSTATUS, LINEITEM, SEQUENCE tables) | Direct method calls, Spring Dependency Injection | Service Layer, Data Mapper, Transaction Script, Saga Pattern |
| Shopping Cart | Java | Stripes 1.6.0, Spring 5.3.39 | Session-based (persisted to HTTP Session) | HTTP Session, Stripes ActionBeans | Session State, Model View Controller |
| Persistence Layer | Java, SQL | MyBatis 3.5.19, Spring JDBC | HSQLDB 2.7.4 | MyBatis Mapper interfaces, JDBC | Data Mapper, Repository, L2 Caching |
| Testing Framework | Java | JUnit 5, Mockito, Selenide, Spring Test | HSQLDB 2.7.4 (embedded test database) | Direct method calls, Mock objects | Integration Testing, Unit Testing, Mock Testing |
| Build & Deployment | Java, Shell | Maven, Docker, GitHub Actions | - | Maven dependencies, Docker networking | CI/CD, Containerization, Multi-profile Deployment |
```