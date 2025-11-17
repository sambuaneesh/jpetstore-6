```markdown
# JPetStore Dynamic Interaction Flows

## 1. User Registration and Authentication Workflow

### Purpose
Complete customer account lifecycle management including registration, authentication, and profile updates.

### Communication Patterns
- **Synchronous**: REST-style form submissions
- **Session-based**: HTTP session management for authentication state
- **Database Transactions**: Multi-table operations for account persistence

### Sequence Diagram
```mermaid
sequenceDiagram
    participant U as User
    participant AA as AccountActionBean
    participant AS as AccountService
    participant AM as AccountMapper
    participant DB as Database

    U->>AA: Access Registration Form
    AA->>AA: Initialize empty account form
    AA-->>U: Display registration page
    
    U->>AA: Submit registration form
    AA->>AS: createAccount(accountData)
    AS->>AM: insertAccount(account)
    AM->>DB: INSERT INTO account
    AM->>DB: INSERT INTO profile
    AM->>DB: INSERT INTO signon
    AM-->>AS: Account created
    AS-->>AA: Success confirmation
    AA->>AA: Store user in session
    AA-->>U: Redirect to main page (authenticated)

    Note over U,DB: Authentication Flow
    U->>AA: Submit login credentials
    AA->>AS: getAccount(username, password)
    AS->>AM: getAccountByUsernameAndPassword
    AM->>DB: SELECT account + profile + signon
    AM-->>AS: Account data
    AS-->>AA: Validated account
    AA->>AA: Set authentication session
    AA-->>U: Redirect to personalized catalog
```

## 2. Product Browsing and Search Workflow

### Purpose
Enable customers to discover products through category navigation and keyword search.

### Communication Patterns
- **Synchronous**: Catalog service calls
- **Cached**: Category and product listings
- **Database Queries**: Multi-table joins for catalog data

### Sequence Diagram
```mermaid
sequenceDiagram
    participant U as User
    participant CA as CatalogActionBean
    participant CS as CatalogService
    participant CM as CategoryMapper
    participant PM as ProductMapper
    participant IM as ItemMapper
    participant DB as Database

    U->>CA: Browse main catalog
    CA->>CS: getCategoryList()
    CS->>CM: getCategoryList()
    CM->>DB: SELECT * FROM category
    CM-->>CS: Category list
    CS-->>CA: Category objects
    CA-->>U: Display category menu

    U->>CA: Select category
    CA->>CS: getProductListByCategory(categoryId)
    CS->>PM: getProductListByCategory(categoryId)
    PM->>DB: SELECT * FROM product WHERE category = ?
    PM-->>CS: Product list
    CS-->>CA: Product objects
    CA-->>U: Display products in category

    U->>CA: View product details
    CA->>CS: getProduct(productId)
    CS->>PM: getProduct(productId)
    PM->>DB: SELECT * FROM product WHERE productid = ?
    PM-->>CS: Product details
    CS-->>CA: Product object
    CA->>CS: getItemListByProduct(productId)
    CS->>IM: getItemListByProduct(productId)
    IM->>DB: SELECT * FROM item WHERE productid = ?
    IM-->>CS: Item list with inventory
    CS-->>CA: Item objects
    CA-->>U: Display product with available items

    Note over U,DB: Search Flow
    U->>CA: Search with keywords
    CA->>CS: searchProductList(keywords)
    CS->>PM: searchProductList(keywords)
    PM->>DB: SELECT * FROM product WHERE LOWER(name) LIKE ?
    PM-->>CS: Matching products
    CS-->>CA: Search results
    CA-->>U: Display search results
```

## 3. Shopping Cart Management Workflow

### Purpose
Manage temporary product selections with real-time inventory validation and price calculations.

### Communication Patterns
- **Session-based**: Cart state maintained in HTTP session
- **Synchronous**: Inventory validation calls
- **Event-driven**: Cart update notifications

### Sequence Diagram
```mermaid
sequenceDiagram
    participant U as User
    participant CA as CartActionBean
    participant Cart as Cart Domain
    participant CS as CatalogService
    participant IM as ItemMapper
    participant DB as Database

    U->>CA: Add item to cart
    CA->>CS: getItem(itemId)
    CS->>IM: getItem(itemId)
    IM->>DB: SELECT * FROM item WHERE itemid = ?
    IM-->>CS: Item details
    CS-->>CA: Item object
    
    CA->>Cart: addItem(item, quantity)
    Cart->>Cart: validate quantity vs inventory
    Cart->>Cart: calculate line total
    Cart->>Cart: update cart totals
    Cart-->>CA: Updated cart
    
    CA->>CA: Store cart in session
    CA-->>U: Display updated cart

    U->>CA: Update item quantity
    CA->>Cart: setQuantityByItemId(itemId, quantity)
    Cart->>Cart: validate new quantity
    Cart->>Cart: recalculate line total
    Cart->>Cart: update cart totals
    Cart-->>CA: Updated cart
    CA-->>U: Display updated totals

    U->>CA: Remove item from cart
    CA->>Cart: removeItemById(itemId)
    Cart->>Cart: remove item from collection
    Cart->>Cart: recalculate cart totals
    Cart-->>CA: Updated cart
    CA-->>U: Display updated cart
```

## 4. Order Processing and Checkout Workflow

### Purpose
Complete transactional order creation with inventory management and payment processing.

### Communication Patterns
- **Transactional**: Multi-table database operations
- **Synchronous**: Order service coordination
- **Event-driven**: Inventory update notifications

### Sequence Diagram
```mermaid
sequenceDiagram
    participant U as User
    participant OA as OrderActionBean
    participant OS as OrderService
    participant OM as OrderMapper
    participant LM as LineItemMapper
    participant IM as ItemMapper
    participant SM as SequenceMapper
    participant Cart as Cart Domain
    participant DB as Database

    U->>OA: Initiate checkout
    OA->>OA: Validate user authentication
    OA->>OA: Retrieve cart from session
    OA-->>U: Display shipping form

    U->>OA: Submit shipping/payment info
    OA->>OS: insertOrder(orderData)
    
    OS->>SM: getSequence(name="ordernum")
    SM->>DB: SELECT * FROM sequence WHERE name = ?
    SM-->>OS: Sequence object
    OS->>SM: updateSequence(sequence)
    SM->>DB: UPDATE sequence SET nextid = ? WHERE name = ?
    
    OS->>OM: insertOrder(order)
    OM->>DB: INSERT INTO orders (orderid, userid, ...)
    
    loop For each cart item
        OS->>LM: insertLineItem(lineItem)
        LM->>DB: INSERT INTO lineitem (orderid, ...)
        OS->>IM: updateInventoryQuantity(itemId, -quantity)
        IM->>DB: UPDATE item SET qty = qty - ? WHERE itemid = ?
    end
    
    OM-->>OS: Order created
    OS-->>OA: Order confirmation
    
    OA->>OA: Clear shopping cart from session
    OA-->>U: Display order confirmation
```

## 5. Order History and Tracking Workflow

### Purpose
Provide customers with access to their purchase history and order details.

### Communication Patterns
- **Synchronous**: Order retrieval calls
- **Database Queries**: Multi-table joins for order data
- **Authorization**: User-specific data access controls

### Sequence Diagram
```mermaid
sequenceDiagram
    participant U as User
    participant OA as OrderActionBean
    participant OS as OrderService
    participant OM as OrderMapper
    participant LM as LineItemMapper
    participant DB as Database

    U->>OA: View order history
    OA->>OA: Validate user authentication
    OA->>OS: getOrdersByUsername(username)
    OS->>OM: getOrdersByUsername(username)
    OM->>DB: SELECT * FROM orders WHERE userid = ?
    OM-->>OS: Order list
    OS-->>OA: Order objects
    OA-->>U: Display order history list

    U->>OA: View specific order details
    OA->>OA: Validate order ownership
    OA->>OS: getOrder(orderId)
    OS->>OM: getOrder(orderId)
    OM->>DB: SELECT * FROM orders WHERE orderid = ?
    OM-->>OS: Order header
    OS->>LM: getLineItemsByOrderId(orderId)
    LM->>DB: SELECT * FROM lineitem WHERE orderid = ?
    LM-->>OS: Line items
    OS->>OS: Combine order with line items
    OS-->>OA: Complete order with items
    OA-->>U: Display order details
```

## 6. Inventory Management and Stock Updates

### Purpose
Real-time inventory tracking and stock level management during order processing.

### Communication Patterns
- **Event-driven**: Inventory update triggers
- **Transactional**: Atomic stock operations
- **Synchronous**: Stock validation calls

### Sequence Diagram
```mermaid
sequenceDiagram
    participant IM as ItemMapper
    participant DB as Database
    participant OS as OrderService
    participant CS as CatalogService
    participant CA as CartActionBean

    Note over IM,CA: Inventory Check During Cart Operations
    CA->>CS: getItem(itemId)
    CS->>IM: getItem(itemId)
    IM->>DB: SELECT qty FROM item WHERE itemid = ?
    IM-->>CS: Current stock level
    CS-->>CA: Item with inventory status
    CA->>CA: Validate requested quantity vs available stock

    Note over IM,OS: Inventory Update During Order Processing
    OS->>IM: updateInventoryQuantity(itemId, -quantity)
    IM->>DB: UPDATE item SET qty = qty - ? WHERE itemid = ?
    DB-->>IM: Rows affected
    IM-->>OS: Update result
    
    alt Insufficient Stock
        OS->>OS: Throw InventoryException
        OS-->>OA: Order failed - insufficient stock
        OA-->>U: Display out-of-stock error
    end
```

## 7. Error Handling and Recovery Patterns

### Purpose
Manage system failures, validation errors, and exceptional conditions gracefully.

### Communication Patterns
- **Exception Propagation**: Structured error handling across layers
- **Transaction Rollback**: Database operation reversal
- **User Feedback**: Meaningful error messages

### Sequence Diagram
```mermaid
sequenceDiagram
    participant U as User
    participant AA as AbstractActionBean
    participant Service as Service Layer
    participant Mapper as Mapper Layer
    participant DB as Database

    U->>AA: Submit invalid form data
    AA->>Service: processRequest(invalidData)
    Service->>Service: Validate business rules
    Service-->>AA: ValidationException
    
    AA->>AA: Add error message to context
    AA-->>U: Redisplay form with errors

    Note over U,DB: Database Error Recovery
    U->>AA: Place order
    AA->>Service: insertOrder(orderData)
    Service->>Mapper: databaseOperation()
    Mapper->>DB: SQL operation
    DB-->>Mapper: SQLException
    
    Mapper-->>Service: PersistenceException
    Service-->>AA: ServiceException
    AA->>AA: Log error, rollback transaction
    AA->>AA: Store error message in context
    AA-->>U: Display system error page
    
    Note over U,DB: Session Recovery
    U->>AA: Access page with expired session
    AA->>AA: Detect invalid session
    AA->>AA: Clear session data
    AA-->>U: Redirect to login with message
```

## Communication Pattern Summary

| Pattern Type | Usage Context | Technologies |
|-------------|---------------|-------------|
| **Synchronous** | User interactions, form submissions, immediate data retrieval | Stripes MVC, Service calls, Database queries |
| **Asynchronous** | Not currently implemented in this codebase | (Future enhancement potential) |
| **Event-driven** | Inventory updates, cart modifications, session events | Domain object state changes, Session listeners |
| **Transactional** | Order processing, account creation, inventory updates | Database transactions, Service layer coordination |
| **Session-based** | User authentication, shopping cart, navigation state | HTTP Session, ActionBean context |
| **Database-centric** | All persistent operations, data retrieval | MyBatis, JDBC, Embedded HSQLDB |
```