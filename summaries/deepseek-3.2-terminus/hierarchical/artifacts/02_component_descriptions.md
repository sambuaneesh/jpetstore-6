```markdown
| Component Name | Responsibility | Interfaces (key endpoints or methods) | Depends On | Technologies |
|---------------|----------------|--------------------------------------|------------|-------------|
| Presentation Layer | Handles web request processing and user interface coordination | ActionBeans (viewCart, addItemToCart, signon, newOrder), JSP views | Service Layer, Domain Model | Stripes MVC, ActionBean pattern, Session management |
| Business Logic Layer | Orchestrates business operations and maintains transactional integrity | AccountService (authenticate, getAccount, insertAccount), CatalogService (getCategoryList, searchProductList), OrderService (insertOrder, getOrdersByUsername) | Data Access Layer, Domain Model | Service Layer pattern, Transaction Script pattern, Mockito (testing) |
| Data Access Layer | Manages database persistence and data retrieval operations | Mapper interfaces (AccountMapper.getAccount, OrderMapper.insertOrder, ProductMapper.getProductList), MyBatis SQL mapping | Domain Model, Database | MyBatis ORM, HSQLDB (testing), Mapper pattern, JDBC |
| Domain Model Layer | Encapsulates business entities and core business logic | Domain classes (Account, Order, Cart, Product), Business methods (Cart.addItem, Order.initialize) | None (foundation layer) | Domain-Driven Design, JavaBean pattern, Serializable, BigDecimal |
| Integration Testing | Validates end-to-end application workflows and user journeys | ScreenTransitionIT (complete user workflow tests) | All application layers | JUnit, Browser automation, Embedded database |
| Build Infrastructure | Manages build environment consistency and dependency management | MavenWrapperDownloader (bootstrap Maven environment) | Maven ecosystem | Maven Wrapper, HTTP client, File I/O |
```