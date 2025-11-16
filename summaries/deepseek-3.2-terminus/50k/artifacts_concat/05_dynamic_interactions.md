# MyBatis JPetStore 6 - Dynamic Interaction Flows

## 1. User Authentication Workflow

### Workflow Description
User login process that validates credentials and establishes session authentication state. Triggered when user submits login form.

**Communication Patterns:** REST-like form submission, session management, database authentication check

```mermaid
sequenceDiagram
    participant User as User/Browser
    participant AA as AccountActionBean
    participant AS as AccountService
    participant AM as AccountMapper
    participant DB as HSQLDB
    participant Session as HTTP Session

    User->>AA: POST /actions/Account.action?signon
    Note over User,AA: username/password form data
    
    AA->>AS: getAccount(username, password)
    AS->>AM: getAccountByUsernameAndPassword(username, password)
    AM->>DB: SELECT FROM ACCOUNT, PROFILE, SIGNON
    DB-->>AM: Account data
    AM-->>AS: Account domain object
    AS-->>AA: Account object
    
    alt Authentication Successful
        AA->>Session: setAttribute("account", account)
        AA->>Session: setAttribute("authenticated", true)
        AA->>User: Redirect to main page
    else Authentication Failed
        AA->>User: Display error message
    end
```

## 2. Product Browsing and Search Workflow

### Workflow Description
Catalog navigation flow allowing users to browse categories, view products, and search inventory. Triggered by catalog navigation or search form submission.

**Communication Patterns:** HTTP GET requests, database queries, hierarchical data loading

```mermaid
sequenceDiagram
    participant User as User/Browser
    participant CA as CatalogActionBean
    participant CS as CatalogService
    participant CM as CatalogMapper
    participant IM as ItemMapper
    participant DB as HSQLDB

    User->>CA: GET /actions/Catalog.action?viewCategory&categoryId=FISH
    CA->>CS: getCategory(categoryId)
    CS->>CM: getCategory(categoryId)
    CM->>DB: SELECT FROM CATEGORY WHERE catid=?
    DB-->>CM: Category data
    CM-->>CS: Category object
    CS-->>CA: Category object
    
    CA->>CS: getProductListByCategory(categoryId)
    CS->>CM: getProductListByCategory(categoryId)
    CM->>DB: SELECT FROM PRODUCT WHERE category=?
    DB-->>CM: Product list
    CM-->>CS: List<Product>
    CS-->>CA: Product list
    
    CA->>User: Render Category.jsp with category and products
    
    User->>CA: GET /actions/Catalog.action?viewProduct&productId=FI-SW-01
    CA->>CS: getProduct(productId)
    CS->>CM: getProduct(productId)
    CM->>DB: SELECT FROM PRODUCT WHERE productid=?
    DB-->>CM: Product data
    CM-->>CS: Product object
    CS-->>CA: Product object
    
    CA->>CS: getItemListByProduct(productId)
    CS->>IM: getItemListByProduct(productId)
    IM->>DB: SELECT FROM ITEM WHERE productid=?
    DB-->>IM: Item list
    IM-->>CS: List<Item>
    CS-->>CA: Item list
    
    loop For each item
        CA->>CS: isItemInStock(itemId)
        CS->>IM: getInventoryQuantity(itemId)
        IM->>DB: SELECT qty FROM INVENTORY WHERE itemid=?
        DB-->>IM: Quantity
        IM-->>CS: Stock status
        CS-->>CA: Boolean inStock
    end
    
    CA->>User: Render Product.jsp with product and items
```

## 3. Shopping Cart Management Workflow

### Workflow Description
Complete shopping cart lifecycle including adding items, updating quantities, and removing items. Uses session-based cart storage.

**Communication Patterns:** Session state management, real-time inventory checks, synchronous service calls

```mermaid
sequenceDiagram
    participant User as User/Browser
    participant CA as CartActionBean
    participant CS as CatalogService
    participant IM as ItemMapper
    participant DB as HSQLDB
    participant Session as HTTP Session

    User->>CA: GET /actions/Cart.action?addItemToCart&workingItemId=EST-1
    CA->>Session: getAttribute("cart")
    
    alt Cart doesn't exist
        CA->>CA: create new Cart()
        CA->>Session: setAttribute("cart", cart)
    end
    
    CA->>CS: getItem(workingItemId)
    CS->>IM: getItem(itemId)
    IM->>DB: SELECT FROM ITEM WHERE itemid=?
    DB-->>IM: Item data
    IM-->>CS: Item object
    CS-->>CA: Item object
    
    CA->>CS: isItemInStock(workingItemId)
    CS->>IM: getInventoryQuantity(itemId)
    IM->>DB: SELECT qty FROM INVENTORY WHERE itemid=?
    DB-->>IM: Quantity
    IM-->>CS: Stock status
    CS-->>CA: Boolean inStock
    
    CA->>CA: cart.addItem(item, quantity)
    Note over CA: Updates cart totals and item list
    
    CA->>Session: setAttribute("cart", updatedCart)
    CA->>User: Redirect to cart view
    
    User->>CA: GET /actions/Cart.action?viewCart
    CA->>Session: getAttribute("cart")
    CA->>User: Render Cart.jsp with cart contents
    
    User->>CA: POST /actions/Cart.action?updateCartQuantities
    Note over User,CA: Form data with item quantities
    
    loop For each cart item
        CA->>CS: isItemInStock(itemId)
        CS->>IM: getInventoryQuantity(itemId)
        IM->>DB: SELECT qty FROM INVENTORY WHERE itemid=?
        DB-->>IM: Quantity
        IM-->>CS: Stock status
        CS-->>CA: Boolean inStock
        CA->>CA: cart.updateQuantity(itemId, newQuantity)
    end
    
    CA->>Session: setAttribute("cart", updatedCart)
    CA->>User: Render updated cart view
```

## 4. Order Processing Workflow

### Workflow Description
Complete order lifecycle from checkout initiation to order confirmation and inventory updates. Involves transactional operations across multiple tables.

**Communication Patterns:** Database transactions, sequence generation, inventory updates, session cleanup

```mermaid
sequenceDiagram
    participant User as User/Browser
    participant OA as OrderActionBean
    participant OS as OrderService
    participant OM as OrderMapper
    participant LM as LineItemMapper
    participant SM as SequenceMapper
    participant IM as ItemMapper
    participant DB as HSQLDB
    participant Session as HTTP Session

    User->>OA: GET /actions/Order.action?newOrderForm
    OA->>Session: getAttribute("cart")
    OA->>Session: getAttribute("account")
    OA->>User: Render NewOrderForm.jsp with cart and account data
    
    User->>OA: POST /actions/Order.action?newOrder
    Note over User,OA: Order form data (shipping, billing, payment)
    
    OA->>Session: getAttribute("cart")
    OA->>Session: getAttribute("account")
    OA->>OA: create Order object from form data
    
    OA->>OS: insertOrder(order)
    
    OS->>SM: getNextId("ordernum")
    SM->>DB: SELECT nextid FROM SEQUENCE WHERE name='ordernum'
    DB-->>SM: Current sequence value
    SM->>DB: UPDATE SEQUENCE SET nextid = ? WHERE name='ordernum'
    DB-->>SM: Update confirmation
    SM-->>OS: Generated order ID
    
    OS->>OM: insertOrder(order)
    OM->>DB: INSERT INTO ORDERS (orderid, userid, orderdate, ...)
    DB-->>OM: Order created
    
    loop For each cart item
        OS->>LM: insertLineItem(orderId, lineItem)
        LM->>DB: INSERT INTO LINEITEM (orderid, linenum, itemid, ...)
        DB-->>LM: Line item created
        
        OS->>IM: updateInventoryQuantity(itemId, -quantity)
        IM->>DB: UPDATE INVENTORY SET qty = qty - ? WHERE itemid=?
        DB-->>IM: Inventory updated
    end
    
    OS->>OM: insertOrderStatus(orderId, "P")
    OM->>DB: INSERT INTO ORDERSTATUS (orderid, timestamp, status)
    DB-->>OM: Status recorded
    
    OS-->>OA: Order confirmation
    
    OA->>Session: removeAttribute("cart")
    OA->>User: Render ConfirmOrder.jsp with order details
    
    User->>OA: GET /actions/Order.action?listOrders
    OA->>Session: getAttribute("account")
    OA->>OS: getOrdersByUsername(username)
    OS->>OM: getOrdersByUsername(username)
    OM->>DB: SELECT FROM ORDERS WHERE userid=?
    DB-->>OM: Order list
    
    loop For each order
        OS->>LM: getLineItemsByOrderId(orderId)
        LM->>DB: SELECT FROM LINEITEM WHERE orderid=?
        DB-->>LM: Line items
        LM-->>OS: List<LineItem>
    end
    
    OS-->>OA: List<Order> with populated line items
    OA->>User: Render ListOrders.jsp with order history
```

## 5. Product Search Workflow

### Workflow Description
Keyword-based product search with multi-term tokenization and wildcard matching. Demonstrates business logic in service layer.

**Communication Patterns:** HTTP GET parameters, service-layer algorithm, database pattern matching

```mermaid
sequenceDiagram
    participant User as User/Browser
    participant CA as CatalogActionBean
    participant CS as CatalogService
    participant PM as ProductMapper
    participant DB as HSQLDB

    User->>CA: GET /actions/Catalog.action?searchProducts&keyword=angelfish+tetra
    CA->>CS: searchProductList(keywords)
    
    CS->>CS: String[] terms = keywords.split("\\s+")
    Note over CS: Tokenizes "angelfish tetra" → ["angelfish", "tetra"]
    
    loop For each search term
        CS->>PM: searchProductList("%" + term.toLowerCase() + "%")
        PM->>DB: SELECT FROM PRODUCT WHERE LOWER(name) LIKE ? OR LOWER(descn) LIKE ?
        DB-->>PM: Matching products
        PM-->>CS: List<Product>
        CS->>CS: products.addAll(searchResults)
    end
    
    CS->>CS: removeDuplicateProducts()
    CS-->>CA: List<Product> search results
    
    CA->>User: Render SearchProducts.jsp with results
```

## 6. Error Handling and Recovery Patterns

### Workflow Description
System-wide error handling demonstrating transaction rollback, user feedback, and session recovery mechanisms.

**Communication Patterns:** Exception propagation, transaction management, user feedback

```mermaid
sequenceDiagram
    participant User as User/Browser
    participant OA as OrderActionBean
    participant OS as OrderService
    participant TM as TransactionManager
    participant DB as HSQLDB

    User->>OA: POST /actions/Order.action?newOrder
    OA->>OS: insertOrder(order)
    
    OS->>TM: Begin Transaction
    
    OS->>DB: Generate order ID (SUCCESS)
    OS->>DB: Insert order record (SUCCESS)
    
    loop For first few items
        OS->>DB: Insert line items (SUCCESS)
        OS->>DB: Update inventory (SUCCESS)
    end
    
    OS->>DB: Update inventory for out-of-stock item
    DB-->>OS: SQLException - Negative inventory quantity
    
    OS->>TM: Rollback Transaction
    TM->>DB: Rollback all changes
    
    OS-->>OA: throw OrderException("Insufficient inventory")
    
    OA->>OA: handle OrderException
    OA->>Session: Preserve cart contents
    OA->>User: Display error message with cart preservation
    Note over User,OA: User can adjust quantities and retry
```