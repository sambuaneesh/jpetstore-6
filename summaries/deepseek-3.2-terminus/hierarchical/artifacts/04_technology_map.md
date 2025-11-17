```markdown
| Component Name | Language | Frameworks | Database | Communication | Patterns |
|---------------|----------|------------|----------|---------------|----------|
| Web Controller Layer | Java | Stripes MVC, JSP, Selenium (testing) | None (delegates to services) | HTTP, Session Management, Forward/Redirect | MVC, ActionBean, Template Method, Session State |
| Service Layer | Java | MyBatis, Mockito (testing) | HSQLDB (embedded testing) | Method calls, Transaction boundaries | Service Layer, Transaction Script, Dependency Injection |
| Data Access Layer | Java | MyBatis, HSQLDB (testing), JDBC | HSQLDB (embedded), Database-agnostic | Database connections, SQL mapping | Data Mapper, Repository, Integration Testing |
| Domain Model Layer | Java | JUnit (testing) | None | Object references, Method calls | Domain Model, JavaBean, Rich Domain Object, Value Object |
| Build Infrastructure | Java | Maven Wrapper | None | File I/O, HTTP downloads | Bootstrap, Self-Contained Deployment, Convention over Configuration |
| Integration Testing | Java | Selenium, JUnit | HSQLDB (embedded) | Browser automation, HTTP | End-to-End Testing, Business Process Testing, Integration Testing |
```