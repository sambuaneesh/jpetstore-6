```markdown
# JPetStore Dynamic Interaction Flows

## 1. User Registration Workflow

### Purpose
Complete user account creation including profile setup and authentication credentials.

### Triggers
- New user accesses registration form
- Form submission with account details

### Communication Patterns
- Synchronous REST calls (Stripes ActionBeans)
- Database transactions (Spring @Transactional)
- Session management

### Sequence Diagram

```mermaid
sequenceDiagram
    participant User as Web User
    participant AB as AccountActionBean
    participant AS as AccountService
    participant AM as AccountMapper
    participant DB as HSQLDB

    User->>AB: GET /actions/Account.action?newAccount
    AB->>User: Return NewAccountForm.jsp
    
    User->>AB: POST account data (username, password, profile)
    AB->>AB: Stripes validation
    AB->>AS: insertAccount(Account)
    
    AS->>AS: Begin Transaction
    AS->>AM: insertAccount(account)
    AM->>DB: INSERT INTO account
    AS->>AM: insertProfile(account)
    AM->>DB: INSERT INTO profile
    AS->>AM: insertSignon(account)
    AM->>DB: INSERT INTO signon
    AS->>AS: Commit Transaction
    
    AS-->>AB: Success response
    AB->>AB: Store user in session
    AB->>User: Redirect to main page
```

## 2. User Authentication Workflow

### Purpose
User login with credential verification and session establishment.

### Triggers
- User submits login form
- Session timeout requiring re-authentication

### Communication Patterns
- Synchronous authentication calls
- Session-based state management
- Password verification

### Sequence Diagram

```mermaid
sequenceDiagram
    participant User as Web User
    participant AB as AccountActionBean
    participant AS as AccountService
    participant AM as AccountMapper
    participant DB as HSQLDB

    User->>AB: POST /actions/Account.action?signon
    Note over User,AB: username + password
    
    AB->>AS: getAccount(username, password)
    AS->>AM: getAccountByUsernameAndPassword(username, password)
    AM->>DB: SELECT account, profile, signon
    DB-->>AM: User data
    AM-->>AS: Account object
    AS-->>AB: Authenticated account
    
    alt Authentication Successful
        AB->>AB: Store account in session
        AB->>User: Redirect to main page with welcome
    else Authentication Failed
        AB->>User: Return to login with error message
    end
```

## 3. Product Browsing and Search Workflow

### Purpose
Catalog navigation, product discovery, and search functionality.

### Triggers
- User navigates categories
- User performs keyword search
- Product detail viewing

### Communication Patterns
- Synchronous catalog queries
- Keyword tokenization and search
- Hierarchical data retrieval

### Sequence Diagram

```mermaid
sequenceDiagram
    participant User as Web User
    participant CB as CatalogActionBean
    participant CS as CatalogService
    participant CM as CategoryMapper
    participant PM as ProductMapper
    participant IM as ItemMapper
    participant DB as HSQLDB

    User->>CB: GET /actions/Catalog.action?viewCategory=CATEGORY_ID
    CB->>CS: getCategoryList()
    CS->>CM: getCategoryList()
    CM->>DB: SELECT * FROM category
    CM-->>CS: List<Category>
    
    CB->>CS: getProductListByCategory(CATEGORY_ID)
    CS->>PM: getProductListByCategory(CATEGORY_ID)
    PM->>DB: SELECT * FROM product WHERE category = CATEGORY_ID
    PM-->>CS: List<Product>
    
    CS-->>CB: Category data with products
    CB->>User: Render Category.jsp
    
    User->>CB: Click product for details
    CB->>CS: getItemListByProduct(PRODUCT_ID)
    CS->>IM: getItemListByProduct(PRODUCT_ID)
    IM->>DB: SELECT item.* FROM item WHERE productid = PRODUCT_ID
    IM-->>CS: List<Item>
    CS-->>CB: Product with items
    CB->>User: Render Product.jsp
```

## 4. Shopping Cart Management Workflow

### Purpose
Add items to cart, manage quantities, and calculate pricing.

### Triggers
- User adds item to cart
- User updates cart quantities
- User views cart contents

### Communication Patterns
- Session-based cart storage
- Synchronized cart operations
- Real-time inventory validation
- BigDecimal price calculations

### Sequence Diagram

```mermaid
sequenceDiagram
    participant User as Web User
    participant CartB as CartActionBean
    participant CS as CatalogService
    participant IM as ItemMapper
    participant DB as HSQLDB
    participant Session as HttpSession

    User->>CartB: POST /actions/Cart.action?addItemToCart
    Note over User,CartB: itemId, quantity
    
    CartB->>Session: getAttribute("cart")
    alt Cart doesn't exist
        CartB->>CartB: Create new Cart()
        CartB->>Session: setAttribute("cart", cart)
    end
    
    CartB->>CS: getItem(itemId)
    CS->>IM: getItem(itemId)
    IM->>DB: SELECT * FROM item WHERE itemid = ITEM_ID
    IM-->>CS: Item object
    CS-->>CartB: Item with pricing
    
    CartB->>Session: get cart
    CartB->>Cart: addItem(Item, quantity)
    
    Cart->>Cart: synchronized addItem
    Cart->>Cart: validate quantity > 0
    Cart->>Cart: calculate subtotal with BigDecimal
    Cart->>Cart: update item quantities
    
    Cart-->>CartB: Updated cart
    CartB->>Session: update cart
    CartB->>User: Redirect to Cart.jsp
```

## 5. Order Creation and Checkout Workflow

### Purpose
Complete order processing from cart to confirmed order with inventory updates.

### Triggers
- User initiates checkout
- User confirms order details
- Order submission

### Communication Patterns
- Transactional order creation
- Inventory quantity updates
- Sequence-based ID generation
- Multi-step form processing

### Sequence Diagram

```mermaid
sequenceDiagram
    participant User as Web User
    participant OB as OrderActionBean
    participant OS as OrderService
    participant OM as OrderMapper
    participant LM as LineItemMapper
    participant IM as ItemMapper
    participant SM as SequenceMapper
    participant DB as HSQLDB
    participant Session as HttpSession

    User->>OB: GET /actions/Order.action?newOrderForm
    OB->>Session: get cart, get account
    OB->>User: Render NewOrderForm.jsp
    
    User->>OB: POST order details (shipping, billing, payment)
    OB->>OB: Stripes validation
    OB->>Session: get cart, get account
    OB->>OB: create Order from cart + account + form data
    
    OB->>OS: insertOrder(Order)
    
    OS->>OS: Begin Transaction
    OS->>SM: getSequence("ordernum")
    SM->>DB: SELECT nextid FROM sequence WHERE name = 'ordernum'
    SM-->>OS: nextOrderId
    OS->>SM: updateSequence("ordernum")
    
    OS->>OM: insertOrder(order)
    OM->>DB: INSERT INTO orders
    OS->>OM: insertOrderStatus(order)
    OM->>DB: INSERT INTO orderstatus
    
    loop For each line item in order
        OS->>LM: insertLineItem(lineItem)
        LM->>DB: INSERT INTO lineitem
        OS->>IM: updateInventoryQuantity(itemId, -quantity)
        IM->>DB: UPDATE inventory SET qty = qty - QUANTITY
    end
    
    OS->>OS: Commit Transaction
    OS-->>OB: Order confirmation
    
    OB->>Session: remove cart
    OB->>User: Render ConfirmOrder.jsp with order details
```

## 6. Product Search Workflow

### Purpose
Keyword-based product search with tokenization and wildcard matching.

### Triggers
- User enters search terms in header
- Search form submission

### Communication Patterns
- Keyword tokenization
- Wildcard SQL search patterns
- Case-insensitive matching

### Sequence Diagram

```mermaid
sequenceDiagram
    participant User as Web User
    participant CB as CatalogActionBean
    participant CS as CatalogService
    participant PM as ProductMapper
    participant DB as HSQLDB

    User->>CB: POST /actions/Catalog.action?searchProducts
    Note over User,CB: keywords="dog food treats"
    
    CB->>CS: searchProductList(keywords)
    
    CS->>CS: tokenize keywords
    Note over CS: Split by space, remove empty
    
    loop For each keyword
        CS->>CS: create search pattern
        Note over CS: Convert to "%keyword%"
        CS->>PM: searchProductList(pattern)
        PM->>DB: SELECT * FROM product WHERE lower(name) LIKE pattern
        PM-->>CS: List<Product>
    end
    
    CS->>CS: combine and deduplicate results
    CS-->>CB: List<Product> search results
    
    CB->>User: Render SearchProducts.jsp with results
```

## 7. Inventory Stock Validation Workflow

### Purpose
Real-time inventory checking during cart operations and order processing.

### Triggers
- Item added to cart
- Cart quantity updates
- Order submission

### Communication Patterns
- Synchronous inventory queries
- Stock availability validation
- Error handling for out-of-stock items

### Sequence Diagram

```mermaid
sequenceDiagram
    participant User as Web User
    participant CartB as CartActionBean
    participant CS as CatalogService
    participant IM as ItemMapper
    participant DB as HSQLDB

    User->>CartB: Update cart quantity for item
    
    CartB->>CS: isItemInStock(itemId)
    CS->>IM: getInventoryQuantity(itemId)
    IM->>DB: SELECT qty FROM inventory WHERE itemid = ITEM_ID
    IM-->>CS: currentStock
    
    alt Sufficient Stock
        CS-->>CartB: true - item available
        CartB->>Cart: updateQuantity(itemId, newQuantity)
        CartB->>User: Success - cart updated
    else Insufficient Stock
        CS-->>CartB: false - insufficient stock
        CartB->>User: Error - "Insufficient stock available"
    end
```

## 8. Order History Retrieval Workflow

### Purpose
Display user's previous orders with complete details and line items.

### Triggers
- User accesses order history page
- Click on specific order for details

### Communication Patterns
- User-specific order queries
- Order-line item aggregation
- Status timeline retrieval

### Sequence Diagram

```mermaid
sequenceDiagram
    participant User as Web User
    participant OB as OrderActionBean
    participant OS as OrderService
    participant OM as OrderMapper
    participant LM as LineItemMapper
    participant DB as HSQLDB
    participant Session as HttpSession

    User->>OB: GET /actions/Order.action?listOrders
    OB->>Session: get account (username)
    
    OB->>OS: getOrdersByUsername(username)
    OS->>OM: getOrdersByUsername(username)
    OM->>DB: SELECT * FROM orders WHERE userid = USERNAME
    OM-->>OS: List<Order>
    
    loop For each order
        OS->>LM: getLineItemsByOrderId(orderId)
        LM->>DB: SELECT * FROM lineitem WHERE orderid = ORDER_ID
        LM-->>OS: List<LineItem>
        OS->>OS: setLineItems(order, lineItems)
    end
    
    OS-->>OB: List<Order> with line items
    OB->>User: Render ListOrders.jsp
```

## Communication Patterns Summary

### Synchronous Patterns
- **REST-like calls** between ActionBeans and Services
- **Database transactions** via MyBatis Mappers
- **Session management** for user state

### Asynchronous Patterns
- **Event-driven UI updates** in cart operations
- **Background inventory validation** during browsing

### Data Flow Patterns
- **Request-response** for web interactions
- **Transactional boundaries** at service layer
- **Session persistence** for shopping cart
- **Database sequences** for order ID generation

### Error Handling Patterns
- **Transaction rollback** on order failures
- **Stock validation** errors in cart operations
- **Authentication failures** with user feedback
- **Form validation** with client-side and server-side checks
```