```markdown
| Component Name | Responsibility | Interfaces (key endpoints or methods) | Depends On | Technologies |
|---------------|----------------|--------------------------------------|------------|-------------|
| **AccountActionBean** | User authentication, registration, profile management | `/actions/Account.action` (login, register, editProfile) | AccountService, Session | Stripes MVC, Spring DI, JSP |
| **CartActionBean** | Shopping cart operations and management | `/actions/Cart.action` (addItem, updateCart, removeItem) | Cart domain, CatalogService | Stripes MVC, Session scope, Thread-safe maps |
| **CatalogActionBean** | Product browsing, searching, category navigation | `/actions/Catalog.action` (viewCategory, viewProduct, searchProducts) | CatalogService | Stripes MVC, JSP, JSTL |
| **OrderActionBean** | Order creation, processing, and management | `/actions/Order.action` (newOrder, confirmOrder, listOrders) | OrderService, AccountService | Stripes MVC, Spring Transactions |
| **AccountService** | User registration, authentication, profile management | `getAccount()`, `insertAccount()`, `updateAccount()`, `authenticate()` | AccountMapper, ProfileMapper, SignonMapper | Spring @Transactional, MyBatis |
| **CatalogService** | Product catalog browsing, searching, inventory checking | `getCategoryList()`, `searchProductList()`, `isItemInStock()`, `getItem()` | CategoryMapper, ProductMapper, ItemMapper, InventoryMapper | Spring Service, MyBatis, Search algorithms |
| **OrderService** | Order processing, inventory updates, sequence generation | `insertOrder()`, `getOrder()`, `getOrdersByUsername()` | OrderMapper, LineItemMapper, ItemMapper, SequenceMapper | Spring @Transactional, ACID compliance, Sequence generation |
| **AccountMapper** | User account CRUD operations | `getAccountByUsername()`, `insertAccount()`, `updateAccount()` | Database (ACCOUNT table) | MyBatis 3.5, HSQLDB, XML mappers |
| **CategoryMapper** | Product category data access | `getCategory()`, `getCategoryList()` | Database (CATEGORY table) | MyBatis 3.5, HSQLDB |
| **ProductMapper** | Product information and search | `getProductListByCategory()`, `searchProductList()`, `getProduct()` | Database (PRODUCT table) | MyBatis 3.5, HSQLDB, Wildcard search |
| **ItemMapper** | Inventory item management and details | `getItem()`, `getItemListByProduct()`, `updateInventory()` | Database (ITEM, INVENTORY tables) | MyBatis 3.5, HSQLDB |
| **OrderMapper** | Order data management | `insertOrder()`, `getOrder()`, `getOrdersByUsername()` | Database (ORDERS table) | MyBatis 3.5, HSQLDB |
| **LineItemMapper** | Order line item management | `insertLineItem()`, `getLineItemsByOrderId()` | Database (LINEITEM table) | MyBatis 3.5, HSQLDB |
| **SequenceMapper** | ID sequence generation for orders | `getSequence()`, `updateSequence()` | Database (SEQUENCE table) | MyBatis 3.5, HSQLDB, ID generation algorithm |
| **Cart Domain** | Shopping cart with thread-safe item management | `addItem()`, `removeItemById()`, `getSubTotal()`, `incrementQuantity()` | Item domain, Product domain | Java Collections, Synchronized maps, BigDecimal |
| **Order Domain** | Order processing with line items | Order validation, total calculation | Account domain, Item domain | Java POJO, BigDecimal for monetary calculations |
| **Account Domain** | User profile and preferences management | Profile validation, address management | Profile domain, Signon domain | Java POJO, Session management |
| **Catalog Domain** | Product hierarchy and inventory management | Category→Product→Item hierarchy | Inventory domain | Java POJO, Data relationships |
```