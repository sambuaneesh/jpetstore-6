### 1. New User Registration

-   **Workflow Purpose and Trigger:** This workflow is triggered when a new user fills out and submits the registration form. Its purpose is to create a complete user profile, including account details, preferences, and sign-on credentials, ensuring all related data is stored atomically.
-   **Communication Patterns:**
    -   **Web Interaction:** Synchronous HTTP POST request from the user's browser to the server.
    -   **Internal Communication:** Synchronous, in-process Java method calls between the Web (`AccountActionBean`), Service (`AccountService`), and Data Access (`AccountMapper`) layers.
    -   **Data Persistence:** A single, declarative database transaction (`@Transactional`) managed by Spring at the service layer. This ensures that inserts into the `ACCOUNT`, `PROFILE`, and `SIGNON` tables either all succeed or all fail together, maintaining data consistency.

```mermaid
sequenceDiagram
    participant User as User (Browser)
    participant Stripes as Stripes Framework
    participant AAB as AccountActionBean
    participant AS as AccountService
    participant Spring as Spring Framework
    participant AM as AccountMapper
    participant DB as Database (HSQLDB)

    User->>Stripes: POST /actions/Account.action (newAccount)
    Note over User, AAB: User submits registration form data.

    Stripes->>AAB: newAccount(account)
    AAB->>AS: insertAccount(account)

    note right of AS: @Transactional starts here
    Spring->>DB: BEGIN TRANSACTION

    AS->>AM: insertAccount(account)
    AM->>DB: INSERT INTO ACCOUNT ...
    DB-->>AM: 
    AM-->>AS: 

    AS->>AM: insertProfile(account)
    AM->>DB: INSERT INTO PROFILE ...
    DB-->>AM: 
    AM-->>AS: 

    AS->>AM: insertSignon(account)
    AM->>DB: INSERT INTO SIGNON ...
    DB-->>AM: 
    AM-->>AS: 

    Spring->>DB: COMMIT TRANSACTION
    note right of AS: Transaction commits successfully.

    AS-->>AAB: 
    AAB->>Stripes: RedirectResolution("/actions/Catalog.action")
    Stripes-->>User: HTTP 302 Redirect to Main Page
```

### 2. User Login and Session Management

-   **Workflow Purpose and Trigger:** Triggered when a user submits their credentials via the sign-on form. The system authenticates the user against the database and, upon success, establishes a stateful session by storing the user's account information in the HTTP session.
-   **Communication Patterns:**
    -   **Web Interaction:** Synchronous HTTP POST request.
    -   **Internal Communication:** Synchronous method calls.
    -   **State Management:** The workflow relies heavily on the stateful HTTP session. The `AccountActionBean` is `@SessionScope`, meaning its instance (and the `account` object it holds) persists across subsequent requests from the same user, keeping them authenticated.

```mermaid
sequenceDiagram
    participant User as User (Browser)
    participant Stripes as Stripes Framework
    participant AAB as AccountActionBean
    participant AS as AccountService
    participant AM as AccountMapper
    participant DB as Database (HSQLDB)

    User->>Stripes: POST /actions/Account.action (signon)
    Note over User, AAB: User submits username and password.

    Stripes->>AAB: signon()
    AAB->>AS: getAccount(username, password)
    AS->>AM: getAccountByUsernameAndPassword(username, password)
    AM->>DB: SELECT * FROM ACCOUNT, SIGNON WHERE ...
    DB-->>AM: Account Data
    AM-->>AS: Account object
    
    alt Credentials are valid
        AS-->>AAB: Returns populated Account object
        AAB->>AAB: setAccount(account)
        note right of AAB: Account object is stored in the<br/>@SessionScope AccountActionBean,<br/>establishing the user's session.
        AAB->>Stripes: RedirectResolution("/actions/Catalog.action")
        Stripes-->>User: HTTP 302 Redirect to Main Page
    else Credentials are invalid
        AS-->>AAB: Returns null
        AAB->>AAB: Set error message
        AAB->>Stripes: ForwardResolution("/account/SignonForm.jsp")
        Stripes-->>User: HTTP 200 OK (Renders SignonForm with error)
    end
```

### 3. Adding an Item to the Shopping Cart

-   **Workflow Purpose and Trigger:** Triggered when a user clicks the "Add to Cart" button for a product item. The system retrieves the item's data and adds it to the user's personal shopping cart, which is managed entirely within the user's HTTP session.
-   **Communication Patterns:**
    -   **Web Interaction:** Synchronous HTTP GET request.
    -   **State Management:** This is a prime example of event-driven logic within a stateful, session-based architecture. The `CartActionBean` and its internal `Cart` object are session-scoped. The entire operation is performed in memory on the server; there is no database write operation for modifying the cart.
    -   **Data Retrieval:** A synchronous call is made to the `CatalogService` to fetch item details before adding them to the in-memory cart.

```mermaid
sequenceDiagram
    participant User as User (Browser)
    participant Stripes as Stripes Framework
    participant CAB as CartActionBean
    participant CS as CatalogService
    participant IM as ItemMapper
    participant DB as Database (HSQLDB)

    User->>Stripes: GET /actions/Cart.action?addItemToCart=&workingItemId=EST-1
    note left of Stripes: User clicks "Add to Cart".

    Stripes->>CAB: addItemToCart()
    note right of CAB: CartActionBean is @SessionScope.<br/>It holds the user's Cart object.

    CAB->>CS: isItemInStock("EST-1")
    CS->>IM: getInventoryQuantity("EST-1")
    IM->>DB: SELECT QTY FROM INVENTORY WHERE ITEMID = ?
    DB-->>IM: Quantity > 0
    IM-->>CS: 
    CS-->>CAB: true
    
    CAB->>CS: getItem("EST-1")
    CS->>IM: getItem("EST-1")
    IM->>DB: SELECT * FROM ITEM WHERE ITEMID = ?
    DB-->>IM: Item data
    IM-->>CS: Item object
    CS-->>CAB: Item object

    CAB->>CAB: cart.addItem(item, isInStock)
    note right of CAB: The Cart object, stored in the HTTP session,<br/>is updated in memory. No DB write occurs.

    CAB->>Stripes: ForwardResolution("/cart/Cart.jsp")
    Stripes-->>User: HTTP 200 OK (Renders updated Cart page)
```

### 4. Placing an Order (Checkout)

-   **Workflow Purpose and Trigger:** This is the application's most critical business transaction, triggered when a user confirms their order. It converts the session-based cart into a permanent order, updates inventory levels, and saves all order details to the database within a single, atomic operation.
-   **Communication Patterns:**
    -   **Web Interaction:** Synchronous HTTP POST request.
    -   **Tightly Coupled Transaction:** The core of this workflow is a single database transaction on the `OrderService.insertOrder` method. This transaction tightly couples the **Order** domain with the **Inventory** domain (part of Catalog).
    -   **Synchronous Cross-Domain Calls:** The `OrderService` makes direct, synchronous calls to the `ItemMapper` to update inventory. A failure here will cause the entire order placement to fail and roll back.
    -   **Error Handling:** The transactional boundary ensures data consistency. If the inventory update fails for any item (e.g., insufficient stock), the entire transaction is rolled back, and no order records are created.

```mermaid
sequenceDiagram
    participant User as User (Browser)
    participant Stripes as Stripes Framework
    participant OAB as OrderActionBean
    participant OS as OrderService
    participant Spring as Spring Framework
    participant Mapper as Mappers (Order, LineItem, Item, Sequence)
    participant DB as Database (HSQLDB)

    User->>Stripes: POST /actions/Order.action (newOrder)
    note over User, OAB: User confirms order details and submits.

    Stripes->>OAB: newOrder()
    note right of OAB: Order object is created from the<br/>session-scoped Cart and user form data.
    OAB->>OS: insertOrder(order)
    
    note right of OS: @Transactional starts here.<br/>This is a critical, tightly coupled transaction.
    Spring->>DB: BEGIN TRANSACTION

    OS->>Mapper: getNextId("ordernum")
    Mapper->>DB: SELECT nextid FROM SEQUENCE WHERE NAME = 'ordernum' FOR UPDATE
    Mapper->>DB: UPDATE SEQUENCE SET nextid = ? WHERE NAME = 'ordernum'
    DB-->>Mapper: New Order ID
    Mapper-->>OS: 
    note right of OS: Fetches and increments unique order ID atomically.

    loop For each LineItem in Order
        OS->>Mapper: updateInventoryQuantity(lineItem)
        note over OS, DB: **Cross-domain coupling**: OrderService calls ItemMapper.
        Mapper->>DB: UPDATE INVENTORY SET QTY = QTY - ? WHERE ITEMID = ?
        DB-->>Mapper: 
        Mapper-->>OS: 
    end

    OS->>Mapper: insertOrder(order)
    Mapper->>DB: INSERT INTO ORDERS ...
    DB-->>Mapper: 
    Mapper-->>OS: 

    OS->>Mapper: insertOrderStatus(order)
    Mapper->>DB: INSERT INTO ORDERSTATUS ...
    DB-->>Mapper: 
    Mapper-->>OS: 

    loop For each LineItem in Order
        OS->>Mapper: insertLineItem(lineItem)
        Mapper->>DB: INSERT INTO LINEITEM ...
        DB-->>Mapper: 
        Mapper-->>OS: 
    end

    alt All operations succeed
        Spring->>DB: COMMIT TRANSACTION
        note right of OS: Transaction commits. Order is successfully placed.
        OS-->>OAB: 
        OAB->>OAB: Clear the session cart.
        OAB->>Stripes: ForwardResolution("/order/ViewOrder.jsp")
        Stripes-->>User: HTTP 200 OK (Renders Order Confirmation)
    else Any DB operation fails
        Spring->>DB: ROLLBACK TRANSACTION
        note right of OS: The entire transaction is rolled back.<br/>No order is created, and inventory is not changed.
        OS-->>OAB: Throws Exception
        OAB->>OAB: Set error message
        OAB->>Stripes: ForwardResolution("/common/Error.jsp")
        Stripes-->>User: HTTP 200 OK (Renders Error Page)
    end
```