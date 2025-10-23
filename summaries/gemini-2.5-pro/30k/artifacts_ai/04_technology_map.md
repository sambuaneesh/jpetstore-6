| Component Name | Language | Frameworks | Database | Communication | Patterns |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Web Presentation Layer** | Java, JSP | Stripes (MVC), JSTL, Spring (for DI) | N/A (Delegates to services) | HTTP, In-process method calls | Model-View-Controller (MVC), Server-Side Rendering (SSR), Dependency Injection |
| **Account Management** | Java | Spring (DI, Transactions), MyBatis | HSQLDB | In-process method calls | Layered Architecture, Transactional Service, Data Mapper |
| **Product Catalog** | Java | Spring (DI), MyBatis | HSQLDB | In-process method calls | Layered Architecture, Data Mapper |
| **Shopping Cart** | Java | Stripes (`@SessionScope`) | In-memory (Session-scoped) | HTTP Session State Management | Stateful Session Object |
| **Order Processing** | Java | Spring (DI, Transactions), MyBatis | HSQLDB | In-process method calls | Layered Architecture, Transactional Service, Data Mapper |