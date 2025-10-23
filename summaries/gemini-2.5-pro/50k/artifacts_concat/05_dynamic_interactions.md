### Workflow 1: New User Registration

*   **Description:** This workflow describes the process of a new user creating an account. The user fills out a registration form, and upon submission, the system creates the necessary user records (`ACCOUNT`, `PROFILE`, `SIGNON`) within a single, atomic database transaction to ensure data integrity.
*   **Communication Patterns:**
    *   **User Interaction:** HTTP POST request from the user's browser to the web server.
    *   **Internal Communication:** Synchronous, in-process Java method calls from the controller (`ActionBean`) to the service layer, and from the service to the data access layer (`Mapper`).
    *   **Data Persistence:** A single, ACID-compliant database transaction managed by the Spring Framework (`@Transactional`) that spans multiple `INSERT` operations across different tables.

```mermaid
sequenceDiagram
    actor User
    participant StripesFramework as Front Controller
    participant AccountActionBean as Controller
    participant AccountService as Service
    participant AccountMapper as Data Mapper
    participant Database

    User->>+StripesFramework: POST /actions/Account.action?newAccount
    StripesFramework->>+AccountActionBean: newAccount(accountDetails)
    AccountActionBean->>+AccountService: insertAccount(account)

    rect rgba(255, 228, 181, 0.5)
        note over AccountService, Database: @Transactional Boundary Starts
        AccountService->>+AccountMapper: insertAccount(account)
        AccountMapper->>+Database: INSERT INTO ACCOUNT ...
        Database-->>-AccountMapper: Success
        AccountMapper-->>-AccountService:
        
        AccountService->>+AccountMapper: insertProfile(account)
        AccountMapper->>+Database: INSERT INTO PROFILE ...
        Database-->>-AccountMapper: Success
        AccountMapper-->>-AccountService:

        AccountService->>+AccountMapper: insertSignon(account)
        AccountMapper->>+Database: INSERT INTO SIGNON ...
        Database-->>-AccountMapper: Success
        AccountMapper-->>-AccountService:
        note over AccountService, Database: Transaction Commits
    end

    alt Successful Registration
        AccountService-->>-AccountActionBean: return
        AccountActionBean->>StripesFramework: Forward to Main View
        StripesFramework-->>-User: HTTP 200 OK (Homepage)
    else Transaction Fails
        AccountService-->>-AccountActionBean: throws Exception
        note over AccountService, Database: All DB changes are rolled back
        AccountActionBean->>StripesFramework: Forward to Error View
        StripesFramework-->>-User: HTTP 500 (Error Page)
    end
    deactivate AccountService
    deactivate AccountActionBean
    deactivate StripesFramework
```

### Workflow 2: User Login and Session Management

*   **Description:** This workflow details how a user signs into the application. Upon successful authentication against the database, the user's `Account` object is stored in the HTTP session. The `AccountActionBean` is session-scoped, meaning it persists for the duration of the user's session, maintaining the logged-in state.
*   **Communication Patterns:**
    *   **User Interaction:** HTTP POST request with user credentials.
    *   **Internal Communication:** Synchronous method calls.
    *   **State Management:** The HTTP Session is used to store the authenticated user's state (`Account` object).
    *   **Data Persistence:** A database `SELECT` query to retrieve user data for authentication.

```mermaid
sequenceDiagram
    actor User
    participant StripesFramework as Front Controller
    participant AccountActionBean as Controller (@SessionScope)
    participant HTTPSession as Session
    participant AccountService as Service
    participant AccountMapper as Data Mapper
    participant Database

    User->>+StripesFramework: POST /actions/Account.action?signon
    StripesFramework->>+AccountActionBean: signon(username, password)
    AccountActionBean->>+AccountService: getAccount(username, password)
    AccountService->>+AccountMapper: getAccountByUsernameAndPassword(username, password)
    AccountMapper->>+Database: SELECT * FROM ACCOUNT, SIGNON WHERE ...
    Database-->>-AccountMapper: Account Data or NULL
    AccountMapper-->>-AccountService: return Account object or null
    AccountService-->>-AccountActionBean: return Account object or null
    
    alt Login Successful
        AccountActionBean->>+HTTPSession: setAttribute("accountBean", this)
        note right of AccountActionBean: Stores Account object in session
        HTTPSession-->>-AccountActionBean: 
        deactivate HTTPSession
        AccountActionBean->>StripesFramework: Redirect to Main View
        StripesFramework-->>-User: HTTP 302 Redirect (Homepage)
    else Login Failed
        AccountActionBean->>StripesFramework: Forward to Signon Form with Error
        StripesFramework-->>-User: HTTP 200 OK (Signon Page + Error Message)
    end
    
    deactivate AccountActionBean
    deactivate StripesFramework
```

### Workflow 3: Adding an Item to the Shopping Cart

*   **Description:** This flow captures the process of a user adding an item to their shopping cart. The cart is a non-persistent, session-scoped object. The system first retrieves the item's details from the database to ensure it exists and then adds it to the `Cart` object stored in the user's HTTP session.
*   **Communication Patterns:**
    *   **User Interaction:** HTTP GET request, typically from a link on a product or item page.
    *   **State Management:** The HTTP Session is the primary mechanism for managing the state of the shopping cart. The cart is not saved to the database at this stage.
    *   **Internal Communication:** Synchronous method calls to fetch product data.

```mermaid
sequenceDiagram
    actor User
    participant StripesFramework as Front Controller
    participant CartActionBean as Controller
    participant HTTPSession as Session
    participant CatalogService as Service
    participant ItemMapper as Data Mapper
    participant Database

    User->>+StripesFramework: GET /actions/Cart.action?addItemToCart&workingItemId=EST-1
    StripesFramework->>+CartActionBean: addItemToCart("EST-1")
    CartActionBean->>+HTTPSession: getAttribute("cart")
    
    alt Cart does not exist in session
        HTTPSession-->>-CartActionBean: return null
        CartActionBean->>CartActionBean: create new Cart()
    else Cart exists
        HTTPSession-->>-CartActionBean: return Cart object
    end
    deactivate HTTPSession

    CartActionBean->>+CatalogService: getItem("EST-1")
    CatalogService->>+ItemMapper: getItem("EST-1")
    ItemMapper->>+Database: SELECT * FROM ITEM WHERE itemid = 'EST-1'
    Database-->>-ItemMapper: Item Data
    ItemMapper-->>-CatalogService: return Item object
    CatalogService-->>-CartActionBean: return Item object

    CartActionBean->>CartActionBean: cart.addItem(item)
    note right of CartActionBean: In-memory logic to add item or increment quantity

    CartActionBean->>+HTTPSession: setAttribute("cart", cart)
    HTTPSession-->>-CartActionBean: 
    deactivate HTTPSession

    CartActionBean->>StripesFramework: Forward to Cart View
    StripesFramework-->>-User: HTTP 200 OK (Display Cart Page)

    deactivate CartActionBean
    deactivate StripesFramework
```

### Workflow 4: Product Search

*   **Description:** This workflow shows how a user searches for products using a keyword. The `CatalogService` implements a simple search algorithm that splits the keyword string and queries the database for each word, aggregating the results. This highlights a potentially inefficient data access pattern.
*   **Communication Patterns:**
    *   **User Interaction:** HTTP POST from a search form.
    *   **Internal Communication:** Synchronous method calls.
    *   **Data Access:** Multiple `SELECT` queries may be executed against the database for a single user search action if multiple keywords are provided.

```mermaid
sequenceDiagram
    actor User
    participant StripesFramework as Front Controller
    participant CatalogActionBean as Controller
    participant CatalogService as Service
    participant ProductMapper as Data Mapper
    participant Database

    User->>+StripesFramework: POST /actions/Catalog.action?searchProducts
    StripesFramework->>+CatalogActionBean: searchProducts(keyword="dog fish")
    CatalogActionBean->>+CatalogService: searchProductList("dog fish")
    
    note over CatalogService: Splits "dog fish" into ["dog", "fish"]
    
    loop For each keyword
        CatalogService->>+ProductMapper: searchProductList("%keyword%")
        ProductMapper->>+Database: SELECT * FROM PRODUCT WHERE name ILIKE '%keyword%'
        Database-->>-ProductMapper: List<Product>
        ProductMapper-->>-CatalogService: returns partial results
    end
    
    note over CatalogService: Aggregates results from all keywords
    CatalogService-->>-CatalogActionBean: return List<Product>
    
    AccountActionBean->>StripesFramework: Forward to Search Results View
    StripesFramework-->>-User: HTTP 200 OK (Display SearchResults.jsp)

    deactivate CatalogActionBean
    deactivate StripesFramework
```

### Workflow 5: Placing an Order (Checkout Process)

*   **Description:** This is the most critical business workflow, showing how a user's shopping cart is converted into a persistent order. The entire process of updating inventory and creating order records is wrapped in a single database transaction to guarantee atomicity. A failure at any step (e.g., insufficient inventory) will cause the entire operation to be rolled back.
*   **Communication Patterns:**
    *   **User Interaction:** HTTP POST request from the order confirmation page.
    *   **State Management:** Reads the `Cart` object from the HTTP Session.
    *   **Internal Communication:** Synchronous method calls between the controller, service, and multiple mappers.
    *   **Data Persistence:** A complex, multi-step, and mission-critical database transaction (`@Transactional`) that involves `SELECT`, `UPDATE`, and `INSERT` statements across the `SEQUENCE`, `INVENTORY`, `ORDERS`, and `LINEITEM` tables. This demonstrates tight coupling between the Order and Inventory/Catalog domains.

```mermaid
sequenceDiagram
    actor User
    participant StripesFramework as Front Controller
    participant OrderActionBean as Controller
    participant HTTPSession as Session
    participant OrderService as Service
    participant SequenceMapper as Data Mapper
    participant ItemMapper as Data Mapper
    participant OrderMapper as Data Mapper
    participant LineItemMapper as Data Mapper
    participant Database

    User->>+StripesFramework: POST /actions/Order.action?newOrder
    StripesFramework->>+OrderActionBean: newOrder(order details)
    OrderActionBean->>+HTTPSession: getAttribute("cart")
    HTTPSession-->>-OrderActionBean: return Cart object
    deactivate HTTPSession
    OrderActionBean->>+OrderService: insertOrder(order, cart)

    rect rgba(210, 105, 30, 0.5)
        note over OrderService, Database: @Transactional Boundary Starts
        
        OrderService->>+SequenceMapper: getNextId("ordernum")
        SequenceMapper->>+Database: SELECT nextid FROM SEQUENCE WHERE name = 'ordernum'
        Database-->>-SequenceMapper: currentId
        SequenceMapper->>+Database: UPDATE SEQUENCE SET nextid = nextid + 1
        Database-->>-SequenceMapper: Success
        SequenceMapper-->>-OrderService: return newOrderId

        loop For each LineItem in Order
            OrderService->>+ItemMapper: updateInventoryQuantity(itemId, quantity)
            ItemMapper->>+Database: UPDATE INVENTORY SET QTY = QTY - ? WHERE itemid = ?
            Database-->>-ItemMapper: Success
            ItemMapper-->>-OrderService: 
        end

        OrderService->>+OrderMapper: insertOrder(order)
        OrderMapper->>+Database: INSERT INTO ORDERS ...
        Database-->>-OrderMapper: Success
        OrderMapper-->>-OrderService: 

        OrderService->>+OrderMapper: insertOrderStatus(order)
        OrderMapper->>+Database: INSERT INTO ORDERSTATUS ...
        Database-->>-OrderMapper: Success
        OrderMapper-->>-OrderService: 

        loop For each LineItem in Order
            OrderService->>+LineItemMapper: insertLineItem(lineItem)
            LineItemMapper->>+Database: INSERT INTO LINEITEM ...
            Database-->>-LineItemMapper: Success
            LineItemMapper-->>-OrderService: 
        end
        note over OrderService, Database: Transaction Commits
    end

    alt Order Placed Successfully
        OrderService-->>-OrderActionBean: return
        OrderActionBean->>+HTTPSession: removeAttribute("cart")
        note right of OrderActionBean: Clears the cart from session
        HTTPSession-->>-OrderActionBean: 
        deactivate HTTPSession
        OrderActionBean->>StripesFramework: Forward to ViewOrder page
        StripesFramework-->>-User: HTTP 200 OK (Display Order Confirmation)
    else Transaction Fails (e.g., Inventory Update Fails)
        OrderService-->>-OrderActionBean: throws Exception
        note over OrderService, Database: All DB changes are rolled back
        OrderActionBean->>StripesFramework: Forward to Error page
        StripesFramework-->>-User: HTTP 500 (Error Page)
    end
    deactivate OrderActionBean
    deactivate StripesFramework
```