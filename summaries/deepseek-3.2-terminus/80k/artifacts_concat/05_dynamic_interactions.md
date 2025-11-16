```markdown
# MyBatis JPetStore 6 - Dynamic Interaction Flows

## 1. User Registration and Authentication Workflow

### Purpose
New user account creation and authentication process, including profile setup and preference management.

### Triggers
- User accesses registration page
- User submits login credentials

### Communication Patterns
- Synchronous REST-style HTTP requests via Stripes Actions
- Database transactions for account creation
- Session-based authentication state management

```mermaid
sequenceDiagram
    actor User
    participant AccountActionBean
    participant AccountService
    participant AccountMapper
    participant ProfileMapper
    participant SignonMapper
    participant Database

    Note over User,Database: Registration Flow
    User->>AccountActionBean: POST /actions/Account.action?newAccount
    AccountActionBean->>AccountService: insertAccount(account, profile)
    AccountService->>AccountMapper: insertAccount(account)
    AccountMapper->>Database: INSERT INTO account
    Database-->>AccountMapper: Success
    AccountService->>ProfileMapper: insertProfile(profile)
    ProfileMapper->>Database: INSERT INTO profile
    Database-->>ProfileMapper: Success
    AccountService->>SignonMapper: updateSignon(username, password)
    SignonMapper->>Database: INSERT INTO signon
    Database-->>SignonMapper: Success
    AccountService-->>AccountActionBean: Account created
    AccountActionBean->>AccountActionBean: Set session attributes
    AccountActionBean-->>User: Redirect to main page

    Note over User,Database: Authentication Flow
    User->>AccountActionBean: POST /actions/Account.action?signon
    AccountActionBean->>AccountService: getAccount(username, password)
    AccountService->>SignonMapper: getPassword(username)
    SignonMapper->>Database: SELECT password FROM signon
    Database-->>SignonMapper: Password hash
    AccountService->>AccountMapper: getAccountByUsername(username)
    AccountMapper->>Database: SELECT * FROM account
    Database-->>AccountMapper: Account data
    AccountService->>ProfileMapper: getProfile(username)
    ProfileMapper->>Database: SELECT * FROM profile
    Database-->>ProfileMapper: Profile data
    AccountService-->>AccountActionBean: Account object
    AccountActionBean->>AccountActionBean: Validate credentials
    AccountActionBean->>AccountActionBean: Set authenticated session
    AccountActionBean-->>User: Redirect to main page
```

## 2. Product Browsing and Search Workflow

### Purpose
Catalog navigation, product discovery, and inventory availability checking.

### Triggers
- User browses categories
- User searches for products
- User views product details

### Communication Patterns
- Synchronous catalog data retrieval
- Database queries with caching
- Multi-keyword search processing

```mermaid
sequenceDiagram
    actor User
    participant CatalogActionBean
    participant CatalogService
    participant CategoryMapper
    participant ProductMapper
    participant ItemMapper
    participant Database

    Note over User,Database: Category Browsing
    User->>CatalogActionBean: GET /actions/Catalog.action?viewCategory=CAT_ID
    CatalogActionBean->>CatalogService: getCategory(categoryId)
    CatalogService->>CategoryMapper: getCategory(categoryId)
    CategoryMapper->>Database: SELECT * FROM category WHERE catid=?
    Database-->>CategoryMapper: Category data
    CatalogService->>ProductMapper: getProductListByCategory(categoryId)
    ProductMapper->>Database: SELECT * FROM product WHERE category=?
    Database-->>ProductMapper: Product list
    CatalogService-->>CatalogActionBean: Category with Products
    CatalogActionBean-->>User: Display category page

    Note over User,Database: Product Search
    User->>CatalogActionBean: GET /actions/Catalog.action?search=keywords
    CatalogActionBean->>CatalogService: searchProductList(keywords)
    CatalogService->>CatalogService: parseKeywords(keywords)
    loop For each keyword
        CatalogService->>ProductMapper: searchProductList("%keyword%")
        ProductMapper->>Database: SELECT * FROM product WHERE (name/desc) LIKE ?
        Database-->>ProductMapper: Matching products
    end
    CatalogService-->>CatalogActionBean: Combined product list
    CatalogActionBean-->>User: Display search results

    Note over User,Database: Product Detail View
    User->>CatalogActionBean: GET /actions/Catalog.action?viewProduct=PROD_ID
    CatalogActionBean->>CatalogService: getProduct(productId)
    CatalogService->>ProductMapper: getProduct(productId)
    ProductMapper->>Database: SELECT * FROM product WHERE productid=?
    Database-->>ProductMapper: Product data
    CatalogService->>ItemMapper: getItemListByProduct(productId)
    ItemMapper->>Database: SELECT * FROM item WHERE productid=?
    Database-->>ItemMapper: Item list
    CatalogService-->>CatalogActionBean: Product with Items
    CatalogActionBean-->>User: Display product detail page
```

## 3. Shopping Cart Management Workflow

### Purpose
Add items to cart, update quantities, and maintain cart state across user sessions.

### Triggers
- User adds item to cart
- User updates item quantities
- User views cart contents

### Communication Patterns
- Session-based cart storage
- Synchronous inventory validation
- Real-time price calculations

```mermaid
sequenceDiagram
    actor User
    participant CartActionBean
    participant CatalogService
    participant ItemMapper
    participant Database
    participant Session

    Note over User,Session: Add Item to Cart
    User->>CartActionBean: POST /actions/Cart.action?addItemToCart=ITEM_ID
    CartActionBean->>CatalogService: isItemInStock(itemId)
    CatalogService->>ItemMapper: getInventoryQuantity(itemId)
    ItemMapper->>Database: SELECT qty FROM inventory WHERE itemid=?
    Database-->>ItemMapper: Quantity
    CatalogService-->>CartActionBean: Stock status
    
    alt Item in stock
        CartActionBean->>Session: getCart()
        Session-->>CartActionBean: Cart object
        CartActionBean->>CartActionBean: addItem(item, quantity)
        CartActionBean->>CartActionBean: calculateSubtotal()
        CartActionBean->>Session: updateCart(cart)
        CartActionBean-->>User: Display updated cart
    else Item out of stock
        CartActionBean-->>User: Display error message
    end

    Note over User,Session: Update Cart Quantities
    User->>CartActionBean: POST /actions/Cart.action?updateCartQuantities
    CartActionBean->>Session: getCart()
    Session-->>CartActionBean: Cart object
    loop For each cart item
        CartActionBean->>CatalogService: isItemInStock(itemId)
        CatalogService->>ItemMapper: getInventoryQuantity(itemId)
        ItemMapper->>Database: SELECT qty FROM inventory
        Database-->>ItemMapper: Quantity
        CatalogService-->>CartActionBean: Stock status
        CartActionBean->>CartActionBean: updateQuantity(itemId, newQty)
    end
    CartActionBean->>CartActionBean: recalculateTotals()
    CartActionBean->>Session: updateCart(cart)
    CartActionBean-->>User: Display updated cart
```

## 4. Order Processing and Checkout Workflow

### Purpose
Complete order creation with inventory updates, sequence generation, and transaction management.

### Triggers
- User proceeds to checkout
- User confirms order placement

### Communication Patterns
- Transactional database operations
- Sequence-based ID generation
- Atomic inventory updates

```mermaid
sequenceDiagram
    actor User
    participant OrderActionBean
    participant OrderService
    participant SequenceMapper
    participant OrderMapper
    participant LineItemMapper
    participant ItemMapper
    participant Database

    Note over User,Database: Order Creation Process
    User->>OrderActionBean: POST /actions/Order.action?newOrder
    OrderActionBean->>OrderService: insertOrder(order)
    
    Note over OrderService,Database: Transaction Boundary Start
    OrderService->>SequenceMapper: getSequence("ordernum")
    SequenceMapper->>Database: SELECT nextid FROM sequence WHERE name=?
    Database-->>SequenceMapper: Current sequence
    OrderService->>SequenceMapper: updateSequence("ordernum", nextId+1)
    SequenceMapper->>Database: UPDATE sequence SET nextid=?
    Database-->>SequenceMapper: Success
    
    OrderService->>OrderMapper: insertOrder(order)
    OrderMapper->>Database: INSERT INTO orders (orderid, userid, ...)
    Database-->>OrderMapper: Success
    
    OrderService->>OrderMapper: insertOrderStatus(order)
    OrderMapper->>Database: INSERT INTO orderstatus (orderid, timestamp, ...)
    Database-->>OrderMapper: Success
    
    loop For each line item
        OrderService->>LineItemMapper: insertLineItem(lineItem)
        LineItemMapper->>Database: INSERT INTO lineitem (orderid, linenum, ...)
        Database-->>LineItemMapper: Success
        
        OrderService->>ItemMapper: updateInventoryQuantity(itemId, -quantity)
        ItemMapper->>Database: UPDATE inventory SET qty = qty - ? WHERE itemid=?
        Database-->>ItemMapper: Success
    end
    Note over OrderService,Database: Transaction Boundary End
    
    OrderService-->>OrderActionBean: Order created successfully
    OrderActionBean->>OrderActionBean: clearCart()
    OrderActionBean-->>User: Display order confirmation
```

## 5. Error Handling and Recovery Patterns

### Purpose
Handle system failures, validation errors, and maintain data consistency.

### Triggers
- Database constraints violations
- Inventory shortages
- Authentication failures

### Communication Patterns
- Transaction rollback on failures
- Session timeout handling
- User-friendly error messages

```mermaid
sequenceDiagram
    actor User
    participant ActionBean
    participant Service
    participant Mapper
    participant Database
    participant TransactionManager

    Note over User,TransactionManager: Order Processing Error Scenario
    User->>ActionBean: Submit order
    ActionBean->>Service: insertOrder(order)
    Service->>TransactionManager: Begin transaction
    
    Service->>Mapper: insertOrder(order)
    Mapper->>Database: INSERT INTO orders
    Database-->>Mapper: Success
    
    Service->>Mapper: insertLineItem(lineItem1)
    Mapper->>Database: INSERT INTO lineitem
    Database-->>Mapper: Success
    
    Service->>Mapper: updateInventory(item1, -quantity)
    Mapper->>Database: UPDATE inventory SET qty = qty - ?
    Database-->>Mapper: Error: Insufficient inventory
    
    alt Inventory shortage
        Service->>TransactionManager: Rollback transaction
        TransactionManager->>Database: Rollback all changes
        Service-->>ActionBean: OrderFailedException
        ActionBean->>ActionBean: addError("Insufficient inventory")
        ActionBean-->>User: Display error with cart restoration
    end

    Note over User,TransactionManager: Database Connection Failure
    User->>ActionBean: Any action
    ActionBean->>Service: businessOperation()
    Service->>Mapper: databaseOperation()
    Mapper->>Database: SQL operation
    Database-->>Mapper: Connection timeout
    
    Mapper-->>Service: DataAccessException
    Service-->>ActionBean: ServiceException
    ActionBean->>ActionBean: logError()
    ActionBean-->>User: Display system error page
```

## Communication Pattern Summary

| Workflow | Primary Pattern | Data Flow | Error Handling |
|----------|-----------------|-----------|----------------|
| User Auth | Synchronous REST | Session-based state | Credential validation |
| Catalog Browse | Synchronous DB queries | Cached catalog data | Fallback to empty results |
| Cart Management | Session state + DB validation | Real-time inventory checks | Stock validation errors |
| Order Processing | Transactional DB operations | Atomic multi-table updates | Full transaction rollback |
| Error Recovery | Exception propagation | Graceful degradation | User-friendly messages |
```