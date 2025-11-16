```markdown
# MyBatis JPetStore - Dynamic Interaction Flows

## Workflow 1: User Registration & Authentication

### Description
Complete user registration and authentication flow including account creation, profile setup, and login process. This workflow establishes user identity and personalization preferences.

### Communication Patterns
- **Synchronous**: REST-style form submissions via Stripes ActionBeans
- **Database Transactions**: Atomic account creation across multiple tables
- **Session Management**: HTTP session establishment for authenticated users

### Sequence Diagram

```mermaid
sequenceDiagram
    participant U as User
    participant AB as AccountActionBean
    participant AS as AccountService
    participant AM as AccountMapper
    participant PM as ProfileMapper
    participant SM as SignonMapper
    participant DB as HSQLDB

    U->>AB: GET /actions/Account.action?newAccountForm
    AB->>U: Render NewAccountForm.jsp
    
    U->>AB: POST account data (username, password, profile)
    AB->>AS: insertAccount(account)
    
    AS->>AM: insertAccount(account)
    AM->>DB: INSERT INTO ACCOUNT
    AM-->>AS: Account created
    
    AS->>PM: insertProfile(account)
    PM->>DB: INSERT INTO PROFILE
    PM-->>AS: Profile created
    
    AS->>SM: insertSignon(account)
    SM->>DB: INSERT INTO SIGNON
    SM-->>AS: Signon created
    
    AS-->>AB: Account created successfully
    AB->>AB: Establish user session
    AB->>U: Redirect to main page with personalized banner
    
    Note over U,DB: Authentication Flow
    U->>AB: POST login credentials
    AB->>AS: getAccount(username, password)
    AS->>AM: getAccountByUsernameAndPassword(username, password)
    AM->>DB: SELECT from ACCOUNT, PROFILE, SIGNON
    AM-->>AS: Account with profile
    AS-->>AB: Authenticated account
    AB->>AB: Validate session
    AB->>U: Redirect to personalized main page
```

## Workflow 2: Product Browsing & Search

### Description
Catalog navigation workflow including category browsing, product viewing, item selection, and keyword-based search functionality. This workflow demonstrates the hierarchical catalog structure.

### Communication Patterns
- **Synchronous**: Page navigation with catalog data loading
- **Database Queries**: Hierarchical category→product→item queries
- **Search Algorithm**: Keyword tokenization and pattern matching

### Sequence Diagram

```mermaid
sequenceDiagram
    participant U as User
    participant CB as CatalogActionBean
    participant CS as CatalogService
    participant CM as CategoryMapper
    participant PM as ProductMapper
    participant IM as ItemMapper
    participant DB as HSQLDB

    U->>CB: GET /actions/Catalog.action?viewCategory=&categoryId=FISH
    CB->>CS: getCategoryList()
    CS->>CM: getCategoryList()
    CM->>DB: SELECT * FROM CATEGORY
    CM-->>CS: Category list
    CS-->>CB: Category data
    
    CB->>CS: getProductListByCategory("FISH")
    CS->>PM: getProductListByCategory("FISH")
    PM->>DB: SELECT * FROM PRODUCT WHERE CATEGORY = 'FISH'
    PM-->>CS: Product list
    CS-->>CB: Product data
    
    CB->>U: Render Category.jsp with products
    
    U->>CB: GET /actions/Catalog.action?viewProduct=&productId=FI-SW-01
    CB->>CS: getProduct("FI-SW-01")
    CS->>PM: getProduct("FI-SW-01")
    PM->>DB: SELECT * FROM PRODUCT WHERE PRODUCTID = 'FI-SW-01'
    PM-->>CS: Product details
    CS-->>CB: Product data
    
    CB->>CS: getItemListByProduct("FI-SW-01")
    CS->>IM: getItemListByProduct("FI-SW-01")
    IM->>DB: SELECT * FROM ITEM WHERE PRODUCTID = 'FI-SW-01'
    IM-->>CS: Item list
    CS-->>CB: Item data
    
    CB->>U: Render Product.jsp with items
    
    Note over U,DB: Search Functionality
    U->>CB: POST search keywords "angelfish"
    CB->>CS: searchProductList("angelfish")
    CS->>CS: Tokenize keywords → ["%angelfish%"]
    CS->>PM: searchProductList("%angelfish%")
    PM->>DB: SELECT * FROM PRODUCT WHERE LOWER(NAME) LIKE '%angelfish%'
    PM-->>CS: Search results
    CS-->>CB: Product matches
    CB->>U: Render SearchProducts.jsp with results
```

## Workflow 3: Shopping Cart Management

### Description
Complete shopping cart lifecycle including item addition, quantity updates, real-time inventory validation, and price calculations. This workflow demonstrates session-based cart management with thread-safe operations.

### Communication Patterns
- **Session-based**: Cart persistence in HTTP session
- **Synchronous**: Real-time inventory validation
- **Thread-safe**: Synchronized cart operations
- **Price Calculation**: BigDecimal arithmetic for monetary operations

### Sequence Diagram

```mermaid
sequenceDiagram
    participant U as User
    participant CTB as CartActionBean
    participant CS as CatalogService
    participant IM as ItemMapper
    participant DB as HSQLDB
    participant Cart as ShoppingCart

    U->>CTB: POST add item to cart (itemId=EST-1, quantity=2)
    CTB->>CTB: Get or create cart from session
    CTB->>CS: getItem("EST-1")
    CS->>IM: getItem("EST-1")
    IM->>DB: SELECT item.*, product.* FROM ITEM, PRODUCT WHERE ITEMID='EST-1'
    IM-->>CS: Item details with pricing
    CS-->>CTB: Item object
    
    CTB->>CS: isItemInStock("EST-1")
    CS->>IM: getInventoryQuantity("EST-1")
    IM->>DB: SELECT QTY FROM INVENTORY WHERE ITEMID='EST-1'
    IM-->>CS: Current stock (15)
    CS-->>CTB: true (in stock)
    
    CTB->>Cart: addItem(item, 2)
    Note right of Cart: Synchronized addItem()<br/>ConcurrentHashMap.putIfAbsent()<br/>BigDecimal price calculations
    Cart-->>CTB: Updated cart
    
    CTB->>U: Redirect to cart page
    
    U->>CTB: GET view cart
    CTB->>CTB: Get cart from session
    CTB->>Cart: calculateSubTotal()
    Note right of Cart: Stream reduction across cart items<br/>BigDecimal aggregation
    Cart-->>CTB: Subtotal amount
    CTB->>U: Render Cart.jsp with items and total
    
    Note over U,DB: Quantity Update Flow
    U->>CTB: POST update quantity (itemId=EST-1, quantity=3)
    CTB->>CTB: Get cart from session
    CTB->>CS: isItemInStock("EST-1")
    CS->>IM: getInventoryQuantity("EST-1")
    IM->>DB: SELECT QTY FROM INVENTORY
    IM-->>CS: Current stock (15)
    CS-->>CTB: true (sufficient stock)
    
    CTB->>Cart: setQuantityByItemId("EST-1", 3)
    Note right of Cart: Synchronized quantity update<br/>Real-time price recalculation
    Cart-->>CTB: Updated cart item
    CTB->>U: Updated cart display
```

## Workflow 4: Order Creation & Checkout

### Description
Complete order processing workflow including cart validation, order creation, inventory updates, and payment processing. This demonstrates transactional order management with distributed ID generation.

### Communication Patterns
- **Database Transactions**: Atomic order creation across multiple tables
- **Sequence Generation**: Distributed ID generation for orders
- **Inventory Updates**: Real-time stock reduction
- **Saga Pattern**: Multi-step order creation process

### Sequence Diagram

```mermaid
sequenceDiagram
    participant U as User
    participant OB as OrderActionBean
    participant OS as OrderService
    participant CS as CatalogService
    participant OM as OrderMapper
    participant LM as LineItemMapper
    participant IM as ItemMapper
    participant SM as SequenceMapper
    participant DB as HSQLDB

    U->>OB: GET /actions/Order.action?newOrderForm
    OB->>OB: Validate user authentication
    OB->>OB: Validate cart not empty
    OB->>U: Render NewOrderForm.jsp
    
    U->>OB: POST order data (billing/shipping, payment)
    OB->>OB: Get cart from session
    OB->>OB: initOrder(account, cart)
    Note right of OB: Copy addresses from account<br/>Calculate order totals<br/>Convert cart items to line items
    
    OB->>OS: insertOrder(order)
    
    OS->>SM: getSequence("ordernum")
    SM->>DB: SELECT NEXTID FROM SEQUENCE WHERE NAME='ordernum'
    SM-->>OS: Current sequence (1001)
    
    OS->>SM: updateSequence(sequence)
    SM->>DB: UPDATE SEQUENCE SET NEXTID=1002 WHERE NAME='ordernum'
    SM-->>OS: Sequence updated
    
    OS->>OM: insertOrder(order)
    OM->>DB: INSERT INTO ORDERS (ORDERID, USERID, TOTALPRICE, ...)
    OM-->>OS: Order created
    
    OS->>OM: insertOrderStatus(order)
    OM->>DB: INSERT INTO ORDERSTATUS (ORDERID, STATUS, TIMESTAMP)
    OM-->>OS: Status recorded
    
    loop For each line item in order
        OS->>LM: insertLineItem(lineItem)
        LM->>DB: INSERT INTO LINEITEM (ORDERID, ITEMID, QUANTITY, UNITPRICE)
        LM-->>OS: Line item saved
        
        OS->>IM: updateInventoryQuantity(itemId, -quantity)
        IM->>DB: UPDATE INVENTORY SET QTY = QTY - quantity WHERE ITEMID=?
        IM-->>OS: Inventory updated
    end
    
    OS-->>OB: Order created successfully (orderId=1001)
    OB->>OB: Clear user cart from session
    OB->>U: Redirect to order confirmation page
    
    U->>OB: GET /actions/Order.action?viewOrder=&orderId=1001
    OB->>OS: getOrder(1001)
    OS->>OM: getOrder(1001)
    OM->>DB: SELECT * FROM ORDERS WHERE ORDERID=1001
    OM-->>OS: Order details
    
    OS->>LM: getLineItemsByOrderId(1001)
    LM->>DB: SELECT * FROM LINEITEM WHERE ORDERID=1001
    LM-->>OS: Line items
    OS-->>OB: Complete order with line items
    OB->>U: Render ViewOrder.jsp with order details
```

## Workflow 5: Error Handling & Recovery

### Description
Comprehensive error handling workflow demonstrating exception propagation, transaction rollback, and user-friendly error reporting across different failure scenarios.

### Communication Patterns
- **Exception Propagation**: Service layer to presentation layer
- **Transaction Rollback**: @Transactional automatic rollback on exceptions
- **User Feedback**: Graceful error pages with recovery options
- **Validation**: Client-side and server-side validation coordination

### Sequence Diagram

```mermaid
sequenceDiagram
    participant U as User
    participant AB as AccountActionBean
    participant AS as AccountService
    participant AM as AccountMapper
    participant DB as HSQLDB
    participant EH as ErrorHandler

    U->>AB: POST register with duplicate username
    AB->>AS: insertAccount(account)
    
    AS->>AM: insertAccount(account)
    AM->>DB: INSERT INTO ACCOUNT (USERID, ...)
    DB-->>AM: SQLException - Unique constraint violation
    
    AM->>AS: Throw DataAccessException
    Note right of AS: @Transactional boundary<br/>Automatic rollback triggered
    
    AS->>AB: Propagate exception
    AB->>EH: Handle exception
    EH->>AB: Prepare error context
    
    AB->>U: Render NewAccountForm.jsp with error message<br/>"Username already exists"
    
    Note over U,DB: Inventory Shortage Scenario
    U->>CTB: POST add item with quantity > stock
    CTB->>CS: isItemInStock("EST-1")
    CS->>IM: getInventoryQuantity("EST-1")
    IM->>DB: SELECT QTY FROM INVENTORY WHERE ITEMID='EST-1'
    IM-->>CS: Current stock (2)
    CS-->>CTB: false (insufficient stock)
    
    CTB->>CTB: Create validation error
    CTB->>U: Render Cart.jsp with error:<br/>"Insufficient stock for item EST-1"
    
    Note over U,DB: Order Transaction Failure
    U->>OB: POST create order
    OB->>OS: insertOrder(order)
    OS->>OM: insertOrder(order)
    OM->>DB: INSERT INTO ORDERS...
    OM-->>OS: Success
    
    OS->>LM: insertLineItem(lineItem1)
    LM->>DB: INSERT INTO LINEITEM...
    LM-->>OS: Success
    
    OS->>IM: updateInventoryQuantity("EST-1", -5)
    IM->>DB: UPDATE INVENTORY SET QTY = QTY - 5...
    DB-->>IM: SQLException - Check constraint violation (QTY < 0)
    
    IM->>OS: Throw DataAccessException
    Note right of OS: @Transactional boundary<br/>Automatic rollback of ALL operations
    OS->>OB: Propagate exception
    OB->>EH: Handle order creation failure
    EH->>U: Render Error.jsp with recovery suggestion
```