### **1. User Registration and Login**

#### **Workflow Description**

This workflow describes the process of a new user creating an account and subsequently logging in. The registration process involves a single, atomic database transaction that creates records in three separate tables (`ACCOUNT`, `PROFILE`, `SIGNON`). The login process validates credentials against the `SIGNON` table and establishes a user session.

*   **Trigger:** User clicks "Sign In" and then "Register Now", or directly fills out the login form.
*   **Communication Patterns:**
    *   Synchronous HTTP POST/GET requests from the browser to the web server.
    *   In-process Java method calls from the `ActionBean` (Controller) to the `Service` layer.
    *   A Spring-managed database transaction (`@Transactional`) at the service layer for the registration process to ensure data integrity across multiple tables.

#### **Sequence Diagram: User Registration**

```mermaid
sequenceDiagram
    actor User
    participant Browser
    participant StripesFilter as Stripes Framework
    participant AccountActionBean as AccountActionBean
    participant AccountService as AccountService
    participant AccountMapper as AccountMapper
    database Database

    User->>Browser: Fills and submits registration form
    Browser->>+StripesFilter: POST /actions/Account.action?newAccount
    StripesFilter->>+AccountActionBean: newAccount(account)
    AccountActionBean->>+AccountService: insertAccount(account)
    
    Note over AccountService, Database: @Transactional block begins

    AccountService->>+AccountMapper: insertAccount(account)
    AccountMapper->>+Database: INSERT INTO ACCOUNT...
    Database-->>-AccountMapper: Success
    AccountMapper-->>-AccountService: void

    AccountService->>+AccountMapper: insertProfile(account)
    AccountMapper->>+Database: INSERT INTO PROFILE...
    Database-->>-AccountMapper: Success
    AccountMapper-->>-AccountService: void

    AccountService->>+AccountMapper: insertSignon(account)
    AccountMapper->>+Database: INSERT INTO SIGNON...
    Database-->>-AccountMapper: Success
    AccountMapper-->>-AccountService: void
    
    Note over AccountService, Database: Transaction commits

    AccountService-->>-AccountActionBean: void
    AccountActionBean->>Browser: Forward to /Catalog.action (Main.jsp)
    deactivate AccountActionBean
    deactivate StripesFilter
    Browser->>User: Renders main page as logged-in user
```

---

### **2. Browse Catalog and Search for Products**

#### **Workflow Description**

This workflow covers how a user navigates the product catalog or uses the search functionality. The interactions are read-only operations. The system retrieves data from the database via the service and persistence layers and displays it to the user.

*   **Trigger:** User clicks on a category link, a product link, or submits the search form.
*   **Communication Patterns:**
    *   Synchronous HTTP GET requests.
    *   In-process Java method calls (`ActionBean` -> `Service` -> `Mapper`).
    *   Direct SQL `SELECT` queries executed via MyBatis mappers.

#### **Sequence Diagram: Product Search**

```mermaid
sequenceDiagram
    actor User
    participant Browser
    participant StripesFilter as Stripes Framework
    participant CatalogActionBean as CatalogActionBean
    participant CatalogService as CatalogService
    participant ProductMapper as ProductMapper
    database Database

    User->>Browser: Enters "Poodle" in search and submits
    Browser->>+StripesFilter: GET /actions/Catalog.action?searchProducts
    StripesFilter->>+CatalogActionBean: searchProducts()
    CatalogActionBean->>+CatalogService: searchProductList("Poodle")
    
    Note over CatalogService: Splits keyword "Poodle" into a list
    
    CatalogService->>+ProductMapper: searchProductList(["%poodle%"])
    ProductMapper->>+Database: SELECT * FROM PRODUCT WHERE lower(name) LIKE ?
    Database-->>-ProductMapper: List<Product>
    ProductMapper-->>-CatalogService: List<Product>
    CatalogService-->>-CatalogActionBean: List<Product>
    
    AccountActionBean-->>Browser: Forward to SearchProducts.jsp
    deactivate CatalogActionBean
    deactivate StripesFilter
    
    Browser->>User: Renders search results page
```

---

### **3. Add Item to Shopping Cart**

#### **Workflow Description**

This workflow shows how a user adds a product to their shopping cart. The cart is a non-persistent, session-scoped object. The system first checks for item availability in the database and then retrieves the item's full details before adding it to the cart object stored in the user's HTTP session.

*   **Trigger:** User clicks the "Add to Cart" button on an item's detail page.
*   **Communication Patterns:**
    *   Synchronous HTTP GET request.
    *   In-process Java method calls.
    *   Interaction with the HTTP Session to read and write the stateful `Cart` object.
    *   Database reads to check stock and fetch item details.

#### **Sequence Diagram: Add to Cart**

```mermaid
sequenceDiagram
    actor User
    participant Browser
    participant StripesFilter as Stripes Framework
    participant CartActionBean as CartActionBean
    participant HttpSession as HTTP Session
    participant CatalogService as CatalogService
    participant ItemMapper as ItemMapper
    database Database

    User->>Browser: Clicks "Add to Cart" for Item 'EST-1'
    Browser->>+StripesFilter: GET /actions/Cart.action?addItemToCart&workingItemId=EST-1
    StripesFilter->>+CartActionBean: addItemToCart()
    
    CartActionBean->>HttpSession: getAttribute("cart")
    HttpSession-->>CartActionBean: Cart object (or null)
    
    alt Cart does not exist in session
        CartActionBean->>HttpSession: new Cart()
        CartActionBean->>HttpSession: setAttribute("cart", newCart)
    end
    
    CartActionBean->>+CatalogService: isItemInStock("EST-1")
    CatalogService->>+ItemMapper: getInventoryQuantity("EST-1")
    ItemMapper->>+Database: SELECT QTY FROM INVENTORY WHERE ITEMID = ?
    Database-->>-ItemMapper: 10
    ItemMapper-->>-CatalogService: 10
    CatalogService-->>-CartActionBean: true
    
    CartActionBean->>+CatalogService: getItem("EST-1")
    CatalogService->>+ItemMapper: getItem("EST-1")
    ItemMapper->>+Database: SELECT * FROM ITEM ... WHERE ITEMID = ?
    Database-->>-ItemMapper: Item object
    ItemMapper-->>-CatalogService: Item object
    CatalogService-->>-CartActionBean: Item object
    
    CartActionBean->>CartActionBean: cart.addItem(item, true)
    
    CartActionBean->>HttpSession: setAttribute("cart", updatedCart)
    
    CartActionBean->>Browser: Forward to Cart.jsp
    deactivate CartActionBean
    deactivate StripesFilter
    
    Browser->>User: Renders updated shopping cart page
```

---

### **4. Place an Order (Successful Checkout)**

#### **Workflow Description**

This is the most critical transactional workflow in the application. It details the process of converting a session-based cart into a persistent order. The entire operation is wrapped in a single database transaction to guarantee atomicity. If any step fails, all previous steps within the transaction are rolled back.

*   **Trigger:** User confirms their order details on the checkout confirmation page.
*   **Communication Patterns:**
    *   Synchronous HTTP POST request.
    *   A single, comprehensive Spring-managed database transaction (`@Transactional`) that spans multiple mapper calls.
    *   **Data Flow:** The `OrderActionBean` creates an `Order` object from session and form data, which is passed to the `OrderService`. The service orchestrates writes to the `SEQUENCE`, `INVENTORY`, `ORDERS`, `ORDERSTATUS`, and `LINEITEM` tables.
    *   **Synchronous Coupling:** There is tight, synchronous coupling between the Order and Catalog/Inventory domains within this transaction.

#### **Sequence Diagram: Successful Order Placement**

```mermaid
sequenceDiagram
    actor User
    participant Browser
    participant StripesFilter as Stripes Framework
    participant OrderActionBean as OrderActionBean
    participant OrderService as OrderService
    participant SequenceMapper as SequenceMapper
    participant ItemMapper as ItemMapper
    participant OrderMapper as OrderMapper
    participant LineItemMapper as LineItemMapper
    database Database

    User->>Browser: Clicks "Confirm Order"
    Browser->>+StripesFilter: POST /actions/Order.action?newOrder&confirmed=true
    StripesFilter->>+OrderActionBean: newOrder()
    OrderActionBean->>OrderActionBean: Retrieves Cart from session, builds Order object
    OrderActionBean->>+OrderService: insertOrder(order)
    
    Note over OrderService, Database: @Transactional block begins

    OrderService->>+SequenceMapper: getNextId("ordernum")
    SequenceMapper->>+Database: UPDATE SEQUENCE SET NEXTID = ...; SELECT NEXTID ...
    Database-->>-SequenceMapper: 1025
    SequenceMapper-->>-OrderService: 1025
    
    loop for each LineItem in Order
        OrderService->>+ItemMapper: updateInventoryQuantity(itemId, -quantity)
        ItemMapper->>+Database: UPDATE INVENTORY SET QTY = QTY - ? WHERE ITEMID = ?
        Database-->>-ItemMapper: Success
        ItemMapper-->>-OrderService: void
    end
    
    OrderService->>+OrderMapper: insertOrder(order)
    OrderMapper->>+Database: INSERT INTO ORDERS ...
    Database-->>-OrderMapper: Success
    OrderMapper-->>-OrderService: void
    
    OrderService->>+OrderMapper: insertOrderStatus(order)
    OrderMapper->>+Database: INSERT INTO ORDERSTATUS ...
    Database-->>-OrderMapper: Success
    OrderMapper-->>-OrderService: void

    loop for each LineItem in Order
        OrderService->>+LineItemMapper: insertLineItem(lineItem)
        LineItemMapper->>+Database: INSERT INTO LINEITEM ...
        Database-->>-LineItemMapper: Success
        LineItemMapper-->>-OrderService: void
    end
    
    Note over OrderService, Database: Transaction commits
    
    OrderService-->>-OrderActionBean: void
    OrderActionBean->>OrderActionBean: Clears Cart from session
    OrderActionBean->>Browser: Forward to ViewOrder.jsp
    deactivate OrderActionBean
    deactivate OrderService
    deactivate StripesFilter
    
    Browser->>User: Renders order confirmation page
```

---

### **5. Order Placement with Error Handling (Rollback)**

#### **Workflow Description**

This diagram illustrates the system's error handling and recovery pattern during the critical checkout process. If any database operation within the `@Transactional` `insertOrder` method fails (e.g., due to a constraint violation, insufficient stock, or database connection issue), an exception is thrown. The Spring Transaction Manager catches this exception and issues a `ROLLBACK` command, ensuring the database is left in a consistent state.

*   **Trigger:** An error occurs during the `insertOrder` service method execution.
*   **Communication Patterns:**
    *   **Exception Handling:** The flow is driven by a runtime exception propagating from the mapper or service layer.
    *   **Transactional Rollback:** The key pattern is the automatic rollback of the database transaction managed by the Spring framework, which reverts all changes made within the transaction's scope.

#### **Sequence Diagram: Order Placement with Transaction Rollback**

```mermaid
sequenceDiagram
    participant OrderActionBean as OrderActionBean
    participant OrderService as OrderService
    participant ItemMapper as ItemMapper
    database Database
    participant SpringTxManager as Spring Transaction Manager

    OrderActionBean->>+OrderService: insertOrder(order)
    OrderService->>+SpringTxManager: Begin Transaction
    
    OrderService->>+ItemMapper: updateInventoryQuantity(item1, -qty)
    ItemMapper->>+Database: UPDATE INVENTORY... (Success)
    Database-->>-ItemMapper: OK
    ItemMapper-->>-OrderService: void
    
    OrderService->>+ItemMapper: updateInventoryQuantity(item2, -qty)
    ItemMapper->>+Database: UPDATE INVENTORY... (Fails due to constraint/error)
    Database-->>-ItemMapper: Exception (e.g., DataAccessException)
    
    ItemMapper-->>-OrderService: Throws DataAccessException
    
    alt Exception Occurs
        OrderService-->>-SpringTxManager: Propagates Exception
        SpringTxManager->>+Database: ROLLBACK
        Database-->>-SpringTxManager: Rollback complete
        
        Note right of Database: All changes (e.g., inventory update for item1) are reverted.
        
        SpringTxManager-->>OrderActionBean: Propagates Exception
    end
    deactivate OrderService
    
    OrderActionBean->>OrderActionBean: Catches exception, adds error message to view
    OrderActionBean->>Browser: Forward to Error.jsp or back to ConfirmOrder.jsp
```