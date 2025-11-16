```markdown
# MyBatis JPetStore 6 - Dynamic Interaction Flows

## 1. User Registration Workflow

### Workflow Description
**Purpose:** New user account creation with profile setup
**Triggers:** User accesses registration form and submits account details
**Communication Patterns:** Synchronous REST calls, database transactions, session management

### Sequence Diagram
```mermaid
sequenceDiagram
    participant User as User/Browser
    participant Web as Web Layer (AccountActionBean)
    participant Service as AccountService
    participant Mapper as AccountMapper
    participant DB as Database
    
    User->>Web: GET /actions/Account.action?newAccount
    Web->>User: Render registration form
    
    User->>Web: POST account details (username, password, profile)
    Web->>Service: insertAccount(account)
    
    Service->>Mapper: getAccountByUsername(username)
    Mapper->>DB: SELECT FROM ACCOUNT
    DB-->>Mapper: Return null (username available)
    
    Service->>Mapper: insertAccount(account)
    Mapper->>DB: INSERT INTO ACCOUNT
    Mapper->>DB: INSERT INTO PROFILE  
    Mapper->>DB: INSERT INTO SIGNON
    DB-->>Mapper: Success
    
    Service-->>Web: Account created
    Web->>Web: Create user session
    Web->>User: Redirect to main page with success message
```

## 2. User Authentication Workflow

### Workflow Description
**Purpose:** User login and session establishment
**Triggers:** User submits credentials on signon form
**Communication Patterns:** Synchronous authentication, session creation, password validation

### Sequence Diagram
```mermaid
sequenceDiagram
    participant User as User/Browser
    participant Web as AccountActionBean
    participant Service as AccountService
    participant Mapper as AccountMapper
    participant DB as Database
    participant Session as HTTP Session
    
    User->>Web: POST /actions/Account.action?signon (username/password)
    Web->>Service: getAccount(username, password)
    
    Service->>Mapper: getAccountByUsernameAndPassword(username, password)
    Mapper->>DB: SELECT FROM ACCOUNT, PROFILE, SIGNON (JOIN)
    DB-->>Mapper: Return account data
    
    alt Valid Credentials
        Service-->>Web: Account object
        Web->>Session: Store authenticated user
        Web->>Session: Initialize shopping cart
        Web->>User: Redirect to main page
    else Invalid Credentials
        Service-->>Web: null/exception
        Web->>User: Display error message
    end
```

## 3. Catalog Browsing Workflow

### Workflow Description
**Purpose:** Product discovery and navigation through category hierarchy
**Triggers:** User clicks category links or uses search functionality
**Communication Patterns:** Synchronous database queries, hierarchical data retrieval

### Sequence Diagram
```mermaid
sequenceDiagram
    participant User as User/Browser
    participant Web as CatalogActionBean
    participant Service as CatalogService
    participant CatMapper as CategoryMapper
    participant ProdMapper as ProductMapper
    participant ItemMapper as ItemMapper
    participant DB as Database
    
    User->>Web: GET /actions/Catalog.action?viewCategory&categoryId=FISH
    Web->>Service: getCategory(categoryId)
    Service->>CatMapper: getCategory(categoryId)
    CatMapper->>DB: SELECT FROM CATEGORY
    DB-->>CatMapper: Return category data
    CatMapper-->>Service: Category object
    
    Web->>Service: getProductListByCategory(categoryId)
    Service->>ProdMapper: getProductListByCategory(categoryId)
    ProdMapper->>DB: SELECT FROM PRODUCT WHERE category=?
    DB-->>ProdMapper: Return product list
    ProdMapper-->>Service: List<Product>
    
    Web->>User: Render Category.jsp with category and product list
    
    User->>Web: GET /actions/Catalog.action?viewProduct&productId=FI-SW-01
    Web->>Service: getProduct(productId)
    Service->>ProdMapper: getProduct(productId)
    ProdMapper->>DB: SELECT FROM PRODUCT
    DB-->>ProdMapper: Return product
    ProdMapper-->>Service: Product object
    
    Web->>Service: getItemListByProduct(productId)
    Service->>ItemMapper: getItemListByProduct(productId)
    ItemMapper->>DB: SELECT FROM ITEM WHERE productid=?
    DB-->>ItemMapper: Return item list
    ItemMapper-->>Service: List<Item>
    
    Web->>User: Render Product.jsp with product and item details
```

## 4. Product Search Workflow

### Workflow Description
**Purpose:** Keyword-based product discovery with multi-term search
**Triggers:** User enters search terms in search box
**Communication Patterns:** Synchronous search with wildcard pattern matching, result aggregation

### Sequence Diagram
```mermaid
sequenceDiagram
    participant User as User/Browser
    participant Web as CatalogActionBean
    participant Service as CatalogService
    participant ProdMapper as ProductMapper
    participant DB as Database
    
    User->>Web: GET /actions/Catalog.action?searchProducts&keyword=angelfish
    Web->>Service: searchProductList(keywords)
    
    Service->>Service: Tokenize keywords: ["angelfish"]
    loop For each keyword
        Service->>ProdMapper: searchProductList("%angelfish%")
        ProdMapper->>DB: SELECT FROM PRODUCT WHERE LOWER(name) LIKE ? OR LOWER(descn) LIKE ?
        DB-->>ProdMapper: Return matching products
        ProdMapper-->>Service: List<Product>
    end
    
    Service->>Service: Aggregate and deduplicate results
    Service-->>Web: List<Product> searchResults
    
    Web->>User: Render SearchProducts.jsp with results
```

## 5. Shopping Cart Management Workflow

### Workflow Description
**Purpose:** Add, remove, and update items in shopping cart with real-time inventory validation
**Triggers:** User interactions with cart (add item, update quantities, remove item)
**Communication Patterns:** Session-based state management, synchronous inventory checks

### Sequence Diagram
```mermaid
sequenceDiagram
    participant User as User/Browser
    participant Web as CartActionBean
    participant Service as CatalogService
    participant Mapper as ItemMapper
    participant DB as Database
    participant Session as HTTP Session
    
    User->>Web: GET /actions/Cart.action?addItemToCart&workingItemId=EST-1
    Web->>Session: Get current cart
    Web->>Service: getItem(itemId)
    Service->>Mapper: getItem(itemId)
    Mapper->>DB: SELECT FROM ITEM WHERE itemid=?
    DB-->>Mapper: Return item details
    Mapper-->>Service: Item object
    
    Web->>Service: isItemInStock(itemId)
    Service->>Mapper: getInventoryQuantity(itemId)
    Mapper->>DB: SELECT qty FROM INVENTORY
    DB-->>Mapper: Return quantity
    Mapper-->>Service: Integer quantity
    
    alt Item in stock
        Web->>Session: cart.addItem(item, quantity=1)
        Web->>Session: Recalculate cart totals
        Web->>User: Redirect to cart view with success
    else Out of stock
        Web->>User: Display out-of-stock error
    end
    
    User->>Web: GET /actions/Cart.action?viewCart
    Web->>Session: Get cart with items
    Web->>User: Render Cart.jsp with cart contents
```

## 6. Order Processing Workflow

### Workflow Description
**Purpose:** Complete purchase transaction with inventory management and order creation
**Triggers:** User proceeds to checkout from shopping cart
**Communication Patterns:** Transactional database operations, sequence generation, inventory updates

### Sequence Diagram
```mermaid
sequenceDiagram
    participant User as User/Browser
    participant Web as OrderActionBean
    participant Service as OrderService
    participant SeqMapper as SequenceMapper
    participant OrderMapper as OrderMapper
    participant LineItemMapper as LineItemMapper
    participant InvMapper as InventoryMapper
    participant DB as Database
    participant Session as HTTP Session
    
    User->>Web: POST /actions/Order.action?newOrder (order details)
    Web->>Session: Get cart and user details
    Web->>Web: Build Order object from form data
    
    Web->>Service: insertOrder(order)
    
    Service->>SeqMapper: getNextId("ordernum")
    SeqMapper->>DB: SELECT nextid FROM SEQUENCE WHERE name=?
    DB-->>SeqMapper: Return current sequence
    SeqMapper->>DB: UPDATE SEQUENCE SET nextid=? WHERE name=? AND nextid=?
    DB-->>SeqMapper: Update success
    
    Service->>OrderMapper: insertOrder(order)
    OrderMapper->>DB: INSERT INTO ORDERS
    DB-->>OrderMapper: Success
    
    loop For each cart item
        Service->>LineItemMapper: insertLineItem(lineItem)
        LineItemMapper->>DB: INSERT INTO LINEITEM
        DB-->>LineItemMapper: Success
        
        Service->>InvMapper: updateInventoryQuantity(itemId, -quantity)
        InvMapper->>DB: UPDATE INVENTORY SET qty = qty - ? WHERE itemid=?
        DB-->>InvMapper: Update success
    end
    
    Service->>OrderMapper: insertOrderStatus(orderId, "Confirmed")
    OrderMapper->>DB: INSERT INTO ORDERSTATUS
    DB-->>OrderMapper: Success
    
    Service-->>Web: Order created successfully
    Web->>Session: Clear shopping cart
    Web->>User: Redirect to order confirmation page
```

## 7. Order History Retrieval Workflow

### Workflow Description
**Purpose:** Display user's order history with detailed order information
**Triggers:** User accesses order history page
**Communication Patterns:** Synchronous database queries, order-line item aggregation

### Sequence Diagram
```mermaid
sequenceDiagram
    participant User as User/Browser
    participant Web as OrderActionBean
    participant Service as OrderService
    participant OrderMapper as OrderMapper
    participant LineItemMapper as LineItemMapper
    participant DB as Database
    participant Session as HTTP Session
    
    User->>Web: GET /actions/Order.action?listOrders
    Web->>Session: Get authenticated user
    Web->>Service: getOrdersByUsername(username)
    
    Service->>OrderMapper: getOrdersByUsername(username)
    OrderMapper->>DB: SELECT FROM ORDERS WHERE userid=? ORDER BY orderdate DESC
    DB-->>OrderMapper: Return order list
    OrderMapper-->>Service: List<Order>
    
    loop For each order
        Service->>LineItemMapper: getLineItemsByOrderId(orderId)
        LineItemMapper->>DB: SELECT FROM LINEITEM WHERE orderid=?
        DB-->>LineItemMapper: Return line items
        LineItemMapper-->>Service: List<LineItem>
        Service->>Service: order.setLineItems(lineItems)
    end
    
    Service-->>Web: List<Order> with line items
    Web->>User: Render ListOrders.jsp with order history
```

## 8. Inventory Management Workflow

### Workflow Description
**Purpose:** Real-time inventory checking and stock level management
**Triggers:** Cart operations, order processing, product browsing
**Communication Patterns:** Synchronous stock queries, transactional inventory updates

### Sequence Diagram
```mermaid
sequenceDiagram
    participant User as User/Browser
    participant Web as CartActionBean/CatalogActionBean
    participant Service as CatalogService
    participant InvMapper as InventoryMapper
    participant DB as Database
    
    User->>Web: Add item to cart
    Web->>Service: isItemInStock(itemId)
    
    Service->>InvMapper: getInventoryQuantity(itemId)
    InvMapper->>DB: SELECT qty FROM INVENTORY WHERE itemid=?
    DB-->>InvMapper: Return current quantity
    InvMapper-->>Service: Integer stockLevel
    
    alt StockLevel > 0
        Service-->>Web: true (in stock)
        Web->>User: Allow add to cart
    else StockLevel <= 0
        Service-->>Web: false (out of stock)
        Web->>User: Display out of stock message
    end
    
    Note over Service,DB: During order processing
    Service->>InvMapper: updateInventoryQuantity(itemId, -orderedQty)
    InvMapper->>DB: UPDATE INVENTORY SET qty = qty - ? WHERE itemid=?
    DB-->>InvMapper: Update applied
    InvMapper-->>Service: Update count
```

## 9. Error Handling and Recovery Patterns

### Workflow Description
**Purpose:** System-wide error management and user-friendly error reporting
**Triggers:** Database errors, validation failures, business rule violations
**Communication Patterns:** Exception propagation, transaction rollback, user notification

### Sequence Diagram
```mermaid
sequenceDiagram
    participant User as User/Browser
    participant Web as ActionBeans
    participant Service as Service Layer
    participant Mapper as Data Mappers
    participant DB as Database
    participant Tx as Transaction Manager
    
    User->>Web: Submit order request
    Web->>Service: insertOrder(order)
    Service->>Tx: Begin transaction
    
    Service->>Mapper: insertOrder(order)
    Mapper->>DB: INSERT INTO ORDERS
    DB-->>Mapper: Success
    
    Service->>Mapper: insertLineItems(lineItems)
    Mapper->>DB: INSERT INTO LINEITEM
    DB-->>Mapper: Success
    
    Service->>Mapper: updateInventory(itemId, -quantity)
    Mapper->>DB: UPDATE INVENTORY SET qty = qty - ?
    DB-->>Mapper: Error: Constraint violation (negative quantity)
    
    Mapper->>Service: Throw SQLException
    Service->>Tx: Rollback transaction
    Service->>Web: Throw OrderException
    
    Web->>Web: Log error details
    Web->>User: Render Error.jsp with friendly message
    Web->>User: Preserve cart contents for retry
```

## 10. Session Management Workflow

### Workflow Description
**Purpose:** User session lifecycle management including creation, maintenance, and cleanup
**Triggers:** Login, logout, session timeout, cart operations
**Communication Patterns:** HTTP session management, state synchronization

### Sequence Diagram
```mermaid
sequenceDiagram
    participant User as User/Browser
    participant Web as ActionBeans
    participant Session as HTTP Session
    participant Service as Service Layer
    
    User->>Web: Successful login
    Web->>Session: Create new session
    Web->>Session: Store authenticated user
    Web->>Session: Initialize empty shopping cart
    Web->>Session: Set session timeout (30 min)
    Web->>User: Set session cookie
    
    User->>Web: Add item to cart
    Web->>Session: Get cart object
    Web->>Session: Update cart with new item
    Web->>Session: Update last access time
    
    User->>Web: Browse catalog (multiple requests)
    Web->>Session: Validate session exists
    Web->>Session: Update last access time
    Web->>User: Serve requested content
    
    alt User logs out
        User->>Web: GET /actions/Account.action?signoff
        Web->>Session: Invalidate session
        Web->>User: Clear session cookie
        Web->>User: Redirect to main page
    else Session timeout
        Note over Session: 30 minutes inactivity
        Session->>Session: Automatically invalidate
        User->>Web: Next request
        Web->>User: Redirect to login with timeout message
    end
```