### 1. User Registration

*   **Description:** A new user fills out and submits a registration form to create an account. This is a critical transactional operation that persists user data across three related database tables (`ACCOUNT`, `PROFILE`, `SIGNON`).
*   **Trigger:** User submits the "New Account" form.
*   **Communication Patterns:**
    *   **HTTP POST Request/Response:** The primary web communication.
    *   **Synchronous Method Calls:** Direct, in-process calls from `ActionBean` -> `Service` -> `Mapper`.
    *   **Database Transaction:** The `insertAccount` service method is annotated with `@Transactional`, ensuring all database writes are performed as a single, atomic unit.

```mermaid
sequenceDiagram
    participant User
    participant Browser
    participant Stripes as Stripes Framework
    participant AccountActionBean
    participant AccountService
    participant AccountMapper
    participant Database as HSQLDB
    
    User->>Browser: Fills and submits registration form
    Browser->>Stripes: POST /actions/Account.action (newAccount)
    Stripes->>AccountActionBean: newAccount(account)
    
    rect rgb(230, 240, 255)
        note over AccountActionBean, Database: Start of Transactional Boundary (@Transactional)
        AccountActionBean->>AccountService: insertAccount(account)
        AccountService->>AccountMapper: insertAccount(account)
        AccountMapper->>Database: EXECUTE INSERT INTO ACCOUNT
        Database-->>AccountMapper: 
        AccountService->>AccountMapper: insertProfile(account)
        AccountMapper->>Database: EXECUTE INSERT INTO PROFILE
        Database-->>AccountMapper: 
        AccountService->>AccountMapper: insertSignon(account)
        AccountMapper->>Database: EXECUTE INSERT INTO SIGNON
        Database-->>AccountMapper: 
        note over AccountActionBean, Database: End of Transactional Boundary (Commit/Rollback)
    end
    
    AccountService-->>AccountActionBean: 
    AccountActionBean->>Stripes: Forward to /actions/Catalog.action
    Stripes-->>Browser: HTTP 302 Redirect
    Browser->>Stripes: GET /actions/Catalog.action
    Stripes-->>Browser: HTTP 200 OK (Main Page HTML)
    Browser->>User: Display main store page
```

### 2. User Login and Session Management

*   **Description:** An existing user signs into the application. Upon successful authentication, the user's `Account` object is retrieved from the database and stored in the HTTP session, establishing an authenticated state for subsequent requests.
*   **Trigger:** User submits the login form.
*   **Communication Patterns:**
    *   **HTTP POST Request/Response:** For submitting credentials.
    *   **Synchronous Method Calls:** For authentication logic.
    *   **Database Read:** A `SELECT` query to validate credentials.
    *   **HTTP Session Interaction:** Writing the `Account` object to the session to maintain state.

```mermaid
sequenceDiagram
    participant User
    participant Browser
    participant Stripes as Stripes Framework
    participant AccountActionBean
    participant HttpSession
    participant AccountService
    participant AccountMapper
    participant Database as HSQLDB
    
    User->>Browser: Fills and submits login form (username, password)
    Browser->>Stripes: POST /actions/Account.action (signon)
    Stripes->>AccountActionBean: signon()
    
    AccountActionBean->>AccountService: getAccount(username, password)
    AccountService->>AccountMapper: getAccountByUsernameAndPassword(username, password)
    AccountMapper->>Database: EXECUTE SELECT ... FROM ACCOUNT, SIGNON WHERE ...
    Database-->>AccountMapper: Account data
    AccountMapper-->>AccountService: return Account object
    
    alt Account found
        AccountService-->>AccountActionBean: return Account object
        AccountActionBean->>HttpSession: setAttribute("accountBean", account)
        note right of AccountActionBean: Store user state in session
        AccountActionBean->>Stripes: Forward to /actions/Catalog.action
        Stripes-->>Browser: HTTP 302 Redirect
        Browser->>Stripes: GET /actions/Catalog.action
        Stripes-->>Browser: HTTP 200 OK (Main Page HTML)
        Browser->>User: Display personalized main page
    else Account not found or password incorrect
        AccountService-->>AccountActionBean: return null
        AccountActionBean->>Stripes: Forward to SignonForm.jsp with error message
        Stripes-->>Browser: HTTP 200 OK (Login Page HTML with error)
        Browser->>User: Display login page with error
    end
```

### 3. Adding an Item to the Cart

*   **Description:** A user adds a specific item to their shopping cart. This workflow retrieves item and inventory data from the database but performs the core "add to cart" logic entirely in-memory by modifying the session-scoped `Cart` object. The state is not persisted in the database until checkout.
*   **Trigger:** User clicks an "Add to Cart" button.
*   **Communication Patterns:**
    *   **HTTP GET Request/Response:** To trigger the action.
    *   **Synchronous Method Calls:** To fetch item data.
    *   **Database Read:** To get item and inventory details.
    *   **In-Memory State Manipulation:** The `Cart` object, held in the `HttpSession`, is modified directly. This is a non-transactional, session-level operation.

```mermaid
sequenceDiagram
    participant User
    participant Browser
    participant Stripes as Stripes Framework
    participant CartActionBean
    participant Cart as Session-Scoped Cart Object
    participant CatalogService
    participant ItemMapper
    participant Database as HSQLDB
    
    User->>Browser: Clicks "Add to Cart" for a specific item
    Browser->>Stripes: GET /actions/Cart.action?addItemToCart=&workingItemId=EST-1
    Stripes->>CartActionBean: addItemToCart()
    
    CartActionBean->>Cart: itemExists("EST-1")
    Cart-->>CartActionBean: false
    
    note over CartActionBean, Database: Fetch item details to add to cart
    CartActionBean->>CatalogService: getItem("EST-1")
    CatalogService->>ItemMapper: getItem("EST-1")
    ItemMapper->>Database: EXECUTE SELECT ... FROM ITEM, INVENTORY WHERE ...
    Database-->>ItemMapper: Item data
    ItemMapper-->>CatalogService: return Item object
    CatalogService-->>CartActionBean: return Item object
    
    CartActionBean->>CatalogService: isItemInStock("EST-1")
    CatalogService->>ItemMapper: getInventoryQuantity("EST-1")
    ItemMapper->>Database: EXECUTE SELECT QTY FROM INVENTORY WHERE ...
    Database-->>ItemMapper: quantity > 0
    ItemMapper-->>CatalogService: return true
    CatalogService-->>CartActionBean: return true
    
    alt Item is in stock
        CartActionBean->>Cart: addItem(item, isInStock)
        note right of CartActionBean: Cart logic is handled in-memory within the<br/>session-scoped Cart domain object.
        Cart-->>CartActionBean: 
    end
    
    CartActionBean->>Stripes: Forward to viewCart()
    Stripes->>CartActionBean: viewCart()
    CartActionBean->>Cart: getCartItemList()
    Cart-->>CartActionBean: List<CartItem>
    
    CartActionBean->>Stripes: Forward to Cart.jsp for rendering
    Stripes-->>Browser: HTTP 200 OK (Cart Page HTML)
    Browser->>User: Display updated shopping cart
```

### 4. Placing an Order (Checkout)

*   **Description:** This is the most critical workflow in the application. It converts the temporary, session-based `Cart` into a persistent `Order`. It executes a complex, multi-table database transaction that generates an order ID, decrements inventory, and creates records in the `ORDERS`, `ORDERSTATUS`, and `LINEITEM` tables, ensuring data consistency across the catalog and order domains.
*   **Trigger:** User confirms their order on the final checkout page.
*   **Communication Patterns:**
    *   **HTTP POST Request/Response:** To submit the final order.
    *   **Synchronous Method Calls:** Orchestrating the order placement.
    *   **Complex Database Transaction:** The `insertOrder` service method wraps all database writes in a single, atomic transaction. A failure at any step (e.g., updating inventory) will cause the entire operation to roll back. This demonstrates a strong, synchronous coupling between the Order and Catalog (Inventory) domains.

```mermaid
sequenceDiagram
    participant User
    participant Browser
    participant Stripes as Stripes Framework
    participant OrderActionBean
    participant HttpSession
    participant Cart as Session-Scoped Cart Object
    participant OrderService
    participant SequenceMapper
    participant ItemMapper
    participant OrderMapper
    participant LineItemMapper
    participant Database as HSQLDB
    
    User->>Browser: Clicks "Confirm" to place order
    Browser->>Stripes: POST /actions/Order.action (newOrder)
    Stripes->>OrderActionBean: newOrder()
    
    OrderActionBean->>HttpSession: getAttribute("cart")
    HttpSession-->>OrderActionBean: return Cart object
    OrderActionBean->>OrderActionBean: Initializes Order object from Cart and Account data
    
    rect rgb(255, 220, 220)
        note over OrderActionBean, Database: Start of Critical Business Transaction (@Transactional)
        OrderActionBean->>OrderService: insertOrder(order)

        note over OrderService, Database: 1. Get unique Order ID
        OrderService->>SequenceMapper: getNextId("ordernum")
        SequenceMapper->>Database: EXECUTE SELECT...UPDATE on SEQUENCE table
        Database-->>SequenceMapper: newOrderId
        SequenceMapper-->>OrderService: return newOrderId
        OrderService->>OrderService: order.setOrderId(newOrderId)

        note over OrderService, Database: 2. Decrement inventory for each item
        loop for each LineItem in Order
            OrderService->>ItemMapper: updateInventoryQuantity(order)
            ItemMapper->>Database: EXECUTE UPDATE INVENTORY SET QTY = QTY - ?
            Database-->>ItemMapper: 
        end
        
        note over OrderService, Database: 3. Insert Order and its LineItems
        OrderService->>OrderMapper: insertOrder(order)
        OrderMapper->>Database: EXECUTE INSERT INTO ORDERS
        Database-->>OrderMapper: 
        
        OrderService->>OrderMapper: insertOrderStatus(order)
        OrderMapper->>Database: EXECUTE INSERT INTO ORDERSTATUS
        Database-->>OrderMapper: 
        
        loop for each LineItem in Order
            OrderService->>LineItemMapper: insertLineItem(lineItem)
            LineItemMapper->>Database: EXECUTE INSERT INTO LINEITEM
            Database-->>LineItemMapper: 
        end
        note over OrderActionBean, Database: End of Transaction (Commit on success, Rollback on failure)
    end
    
    OrderService-->>OrderActionBean: 
    OrderActionBean->>HttpSession: clear cart
    OrderActionBean->>Stripes: Forward to viewOrder()
    Stripes->>OrderActionBean: viewOrder()
    
    OrderActionBean->>OrderService: getOrder(orderId)
    OrderService->>Database: Retrieves full order details for confirmation page
    Database-->>OrderService: Order with LineItems
    OrderService-->>OrderActionBean: return Order object
    
    OrderActionBean->>Stripes: Forward to ViewOrder.jsp
    Stripes-->>Browser: HTTP 200 OK (Order Confirmation Page HTML)
    Browser->>User: Display order confirmation and details
```

### 5. Product Search

*   **Description:** A user enters keywords into the search bar to find relevant products. The service layer tokenizes the input and queries the database using `LIKE` clauses to find matches.
*   **Trigger:** User submits the product search form.
*   **Communication Patterns:**
    *   **HTTP POST Request/Response:** To submit the search query.
    *   **Synchronous Method Calls:** To execute the search logic.
    *   **Database Read:** A `SELECT` query with `LIKE` clauses.

```mermaid
sequenceDiagram
    participant User
    participant Browser
    participant Stripes as Stripes Framework
    participant CatalogActionBean
    participant CatalogService
    participant ProductMapper
    participant Database as HSQLDB
    
    User->>Browser: Enters "Bulldog" in search and clicks "Search"
    Browser->>Stripes: POST /actions/Catalog.action (searchProducts)
    Stripes->>CatalogActionBean: searchProducts()
    
    CatalogActionBean->>CatalogService: searchProductList("bulldog")
    note right of CatalogService: Service logic may tokenize<br/>the keyword string
    CatalogService->>ProductMapper: searchProductList(["bulldog"])
    ProductMapper->>Database: EXECUTE SELECT ... FROM PRODUCT WHERE lower(name) LIKE '%bulldog%' OR ...
    Database-->>ProductMapper: List of matching Product data
    ProductMapper-->>CatalogService: return List<Product>
    CatalogService-->>CatalogActionBean: return List<Product>
    
    CatalogActionBean->>Stripes: Forward to SearchProducts.jsp
    Stripes-->>Browser: HTTP 200 OK (Search Results Page HTML)
    Browser->>User: Display list of matching products
```