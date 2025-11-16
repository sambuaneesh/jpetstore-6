```markdown
| Component Name | Responsibility | Interfaces (key endpoints or methods) | Depends On | Technologies |
|---------------|----------------|--------------------------------------|------------|--------------|
| AccountActionBean | User authentication and profile management | `/actions/Account.action` (signonForm), `newAccount`, `editAccount`, `signon`, `signoff` | AccountService, CatalogService | Stripes MVC, Spring DI, JSP, Session Management |
| CatalogActionBean | Product catalog browsing and search | Catalog browsing endpoints, product search, item viewing | CatalogService | Stripes MVC, Spring DI, JSP, Internationalization |
| CartActionBean | Shopping cart management | Cart management endpoints, quantity updates, checkout initiation | CatalogService, Cart domain | Stripes MVC, Session Management, Synchronized Maps |
| OrderActionBean | Order processing and management | NewOrder, Shipping, Confirm, Submit, order history | OrderService, AccountService, CartActionBean | Stripes MVC, Spring Transactions, JSP |
| AccountService | User account management and authentication | `getAccount(username)`, `getAccount(username,password)`, `insertAccount(account)`, `updateAccount(account)` | AccountMapper | Spring @Transactional, MyBatis, HSQLDB |
| CatalogService | Product catalog operations and inventory management | `getCategoryList()`, `getProductListByCategory()`, `searchProductList()`, `isItemInStock()` | CategoryMapper, ProductMapper, ItemMapper | Spring @Transactional, MyBatis, Keyword Tokenization |
| OrderService | Order lifecycle management and processing | `insertOrder(order)`, `getOrder(orderId)`, `getOrdersByUsername(username)` | OrderMapper, LineItemMapper, ItemMapper, SequenceMapper | Spring @Transactional, MyBatis, Sequence Generation |
| AccountMapper | User account data access | `getAccountByUsername()`, `getAccountByUsernameAndPassword()`, `insertAccount()`, `updateAccount()` | HSQLDB Database | MyBatis 3, HSQLDB, SQL Mapping |
| CategoryMapper | Product category data access | `getCategory()`, `getCategoryList()` | HSQLDB Database | MyBatis 3, HSQLDB, SQL Mapping |
| ProductMapper | Product data access and search | `getProduct()`, `getProductListByCategory()`, `searchProductList()` | HSQLDB Database | MyBatis 3, HSQLDB, SQL LIKE Pattern Matching |
| ItemMapper | Inventory item management | `getItem()`, `getItemListByProduct()`, `getInventoryQuantity()`, `updateInventoryQuantity()` | HSQLDB Database | MyBatis 3, HSQLDB, SQL Mapping |
| OrderMapper | Order data operations | `insertOrder()`, `getOrder()`, `getOrdersByUsername()`, `insertOrderStatus()` | HSQLDB Database | MyBatis 3, HSQLDB, SQL Mapping |
| LineItemMapper | Order line item management | `insertLineItem()`, `getLineItemsByOrderId()` | HSQLDB Database | MyBatis 3, HSQLDB, SQL Mapping |
| SequenceMapper | ID sequence generation | `getSequence()`, `updateSequence()` | HSQLDB Database | MyBatis 3, HSQLDB, Sequence Table Pattern |
| Cart Domain | Shopping cart business logic | Quantity management, subtotal calculation, inventory validation | Item domain | Java Collections, BigDecimal, Synchronized Maps |
| Order Domain | Order business logic | `initOrder(account, cart)`, total price calculation, address copying | Account domain, Cart domain | Java Beans, BigDecimal, Business Logic |
| HSQLDB Database | Data persistence and storage | JDBC interface, SQL schema | None | HSQLDB, SQL, Foreign Key Constraints |
| Spring Framework | Dependency injection and transaction management | ApplicationContext, @Transactional, @Autowired | None | Spring 5, XML Configuration, AOP |
| MyBatis Framework | Object-relational mapping | SqlSessionFactory, Mapper interfaces, Type aliases | Spring Framework | MyBatis 3, SQL Mapping, ORM Patterns |
| Stripes Framework | MVC web framework | ActionBeans, @UrlBinding, @Validate, Interceptors | Spring Framework | Stripes MVC, Convention over Configuration |
| JSP View Layer | Presentation and user interface | JSP pages (Main.jsp, Cart.jsp, Order forms) | ActionBeans | JSP/JSTL, CSS, Internationalization |
| Test Framework | Application testing and validation | JUnit 5, Mockito, Selenide, Spring Test | Application components | JUnit 5, Mockito, AssertJ, Selenide, HSQLDB |
| CI/CD Pipeline | Build, test, and deployment automation | GitHub Actions workflows, Maven profiles | Application code | GitHub Actions, Maven, Docker, Multiple Containers |
```