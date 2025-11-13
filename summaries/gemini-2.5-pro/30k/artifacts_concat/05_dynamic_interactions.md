### 1. User Registration Workflow

*   **Workflow Purpose and Triggers:** This workflow describes how a new user creates an account. It is triggered when the user submits the "New Account" registration form. The process is a single, atomic transaction that creates records in three separate but related tables (`ACCOUNT`, `PROFILE`, `SIGNON`).

*   **Communication Patterns:**
    *   **HTTP POST:** The user's browser sends user details to the server.
    *   **Synchronous Method Calls:** The `AccountActionBean` invokes the `AccountService`, which in turn calls the `AccountMapper`. This is a direct, in-process communication flow.
    *   **Database Transaction:** The entire operation within the `AccountService` is wrapped in a Spring-managed declarative transaction (`@Transactional`). If any of the database inserts fail, all previous inserts in the same transaction are rolled back, ensuring data consistency.

```mermaid
sequenceDiagram
    actor User
    participant Browser
    participant StripesMVC as Web Layer (Stripes)
    participant AccountActionBean as Presentation Controller
    participant AccountService as Service Layer
    participant AccountMapper as Data Access Layer
    participant Database

    User->>Browser: Fills and submits registration form
    Browser->>StripesMVC: POST /actions/Account.action?newAccount
    StripesMVC->>AccountActionBean: newAccount(accountData)
    activate AccountActionBean

    AccountActionBean->>AccountService: insertAccount(account)
    activate AccountService

    note over AccountService, Database: Spring begins @Transactional block

    AccountService->>AccountMapper: insertAccount(account)
    activate AccountMapper
    AccountMapper->>Database: INSERT INTO ACCOUNT (...)
    deactivate AccountMapper

    AccountService->>AccountMapper: insertProfile(account)
    activate AccountMapper
    AccountMapper->>Database: INSERT INTO PROFILE (...)
    deactivate AccountMapper

    AccountService->>AccountMapper: insertSignon(account)
    activate AccountMapper
    AccountMapper->>Database: INSERT INTO SIGNON (...)
    deactivate AccountMapper

    note over AccountService, Database: Spring commits transaction
    AccountService-->>AccountActionBean: void
    deactivate AccountService

    AccountActionBean->>AccountActionBean: Puts Account object into HTTP Session
    AccountActionBean-->>StripesMVC: RedirectResolution("/actions/Catalog.action")
    deactivate AccountActionBean
    StripesMVC-->>Browser: HTTP 302 Redirect
    Browser->>User: Renders main catalog page (logged in)

```

### 2. Add Item to Cart Workflow

*   **Workflow Purpose and Triggers:** This workflow captures the process of a user adding an item to their shopping cart. It is triggered when the user clicks an "Add to Cart" button for a specific item. The workflow demonstrates the use of stateful HTTP sessions to manage user-specific data (the cart) without immediate database persistence.

*   **Communication Patterns:**
    *   **HTTP GET:** A request is sent to add a specific item to the cart.
    *   **HTTP Session State:** The `CartActionBean` and the `Cart` object itself are session-scoped. The user's cart is stored in memory on the server, tied to their session. It is not written to the database until an order is placed.
    *   **Synchronous Method Calls:** The action bean calls the `CatalogService` to fetch item details and check stock levels.
    *   **Database Read Operations:** The `CatalogService` uses the `ItemMapper` to perform read-only `SELECT` queries against the database to get item and inventory data.

```mermaid
sequenceDiagram
    actor User
    participant Browser
    participant StripesMVC as Web Layer (Stripes)
    participant CartActionBean as Presentation Controller
    participant CatalogService as Service Layer
    participant ItemMapper as Data Access Layer
    participant Database

    User->>Browser: Clicks "Add to Cart" for an item
    Browser->>StripesMVC: GET /actions/Cart.action?addItemToCart=&workingItemId={id}
    StripesMVC->>CartActionBean: addItemToCart()
    activate CartActionBean

    note over CartActionBean: Retrieves Cart object from HTTP Session. <br/>Creates a new Cart if one does not exist.

    CartActionBean->>CatalogService: getItem(itemId)
    activate CatalogService
    CatalogService->>ItemMapper: getItem(itemId)
    activate ItemMapper
    ItemMapper->>Database: SELECT * FROM ITEM WHERE ITEMID = ?
    Database-->>ItemMapper: Item data
    deactivate ItemMapper
    CatalogService-->>CartActionBean: Item object
    deactivate CatalogService

    CartActionBean->>CatalogService: isItemInStock(itemId)
    activate CatalogService
    CatalogService->>ItemMapper: getInventoryQuantity(itemId)
    activate ItemMapper
    ItemMapper->>Database: SELECT QTY FROM INVENTORY WHERE ITEMID = ?
    Database-->>ItemMapper: Quantity
    deactivate ItemMapper
    CatalogService-->>CartActionBean: boolean (inStock)
    deactivate CatalogService

    note over CartActionBean: Logic to add/increment item in Cart object
    CartActionBean->>CartActionBean: cart.addItem(item, inStock)

    note over CartActionBean: Updated Cart object is stored back in HTTP Session
    CartActionBean-->>StripesMVC: ForwardResolution("/cart/Cart.jsp")
    deactivate CartActionBean

    StripesMVC-->>Browser: HTTP 200 OK (Renders Cart.jsp)
    Browser->>User: Displays updated shopping cart

```

### 3. Place Order Workflow

*   **Workflow Purpose and Triggers:** This is the most critical business workflow, handling the conversion of a user's shopping cart into a persisted order. It is triggered when the user confirms their order on the final checkout page. This flow highlights strong transactional consistency across multiple business domains (Order and Inventory/Catalog).

*   **Communication Patterns:**
    *   **HTTP POST:** The final confirmation is sent to the server to create the order.
    *   **HTTP Session State:** The workflow retrieves the `Account` and `Cart` objects from the user's session to build the `Order` object.
    *   **Synchronous Method Calls:** A chain of direct method calls from the `OrderActionBean` to the `OrderService` and then to multiple mappers (`SequenceMapper`, `ItemMapper`, `LineItemMapper`, `OrderMapper`).
    *   **Database Transaction:** The `OrderService.insertOrder` method is marked as `@Transactional`. This single transaction encompasses multiple database writes: updating the sequence number, decrementing inventory for each item, and inserting records into the `LINEITEM`, `ORDERS`, and `ORDERSTATUS` tables. This ensures that an order is only placed if the inventory can be successfully updated for all items.
    *   **Error Handling (Implicit):** If any database operation within the transaction fails (e.g., an inventory update), the entire transaction is rolled back, preventing partial order creation and ensuring data integrity.

```mermaid
sequenceDiagram
    actor User
    participant Browser
    participant StripesMVC as Web Layer (Stripes)
    participant OrderActionBean as Presentation Controller
    participant OrderService as Service Layer
    participant SequenceMapper as Data Access
    participant ItemMapper as Data Access
    participant LineItemMapper as Data Access
    participant OrderMapper as Data Access
    participant Database

    User->>Browser: Confirms order
    Browser->>StripesMVC: POST /actions/Order.action?newOrder=&confirmed=true
    StripesMVC->>OrderActionBean: newOrder()
    activate OrderActionBean

    note over OrderActionBean: Retrieves Account and Cart from HTTP Session
    OrderActionBean->>OrderActionBean: Creates Order domain object from session data

    OrderActionBean->>OrderService: insertOrder(order)
    activate OrderService

    note over OrderService, Database: Spring begins @Transactional block

    OrderService->>SequenceMapper: getSequence("ordernum")
    SequenceMapper->>Database: SELECT nextid FROM SEQUENCE WHERE name='ordernum'
    Database-->>SequenceMapper: nextId
    OrderService->>SequenceMapper: updateSequence(sequence)
    SequenceMapper->>Database: UPDATE SEQUENCE SET nextid = ? WHERE name='ordernum'

    loop for each LineItem in Order
        OrderService->>ItemMapper: updateInventoryQuantity(itemId, quantity)
        activate ItemMapper
        ItemMapper->>Database: UPDATE INVENTORY SET QTY = QTY - ? WHERE ITEMID = ?
        deactivate ItemMapper
    end

    OrderService->>OrderMapper: insertOrder(order)
    activate OrderMapper
    OrderMapper->>Database: INSERT INTO ORDERS (...)
    deactivate OrderMapper

    OrderService->>OrderMapper: insertOrderStatus(order)
    activate OrderMapper
    OrderMapper->>Database: INSERT INTO ORDERSTATUS (...)
    deactivate OrderMapper

    loop for each LineItem in Order
        OrderService->>LineItemMapper: insertLineItem(lineItem)
        activate LineItemMapper
        LineItemMapper->>Database: INSERT INTO LINEITEM (...)
        deactivate LineItemMapper
    end

    note over OrderService, Database: Spring commits transaction
    OrderService-->>OrderActionBean: void
    deactivate OrderService

    note over OrderActionBean: Clears the Cart from the HTTP Session
    OrderActionBean->>OrderActionBean: cart.clear()
    OrderActionBean-->>StripesMVC: RedirectResolution("/actions/Order.action?viewOrder=&orderId=...")
    deactivate OrderActionBean

    StripesMVC-->>Browser: HTTP 302 Redirect
    Browser->>User: Renders the "View Order" confirmation page
```