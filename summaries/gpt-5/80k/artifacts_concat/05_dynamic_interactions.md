# Workflow 1: User Sign-in

Purpose and trigger
- Purpose: Authenticate a user and establish session-scoped account state (banner, favorites).
- Trigger: User submits the Sign In form (POST /actions/Account.action?signon).

Communication patterns
- HTTP form POST handled by Stripes MVC (synchronous).
- In-process synchronous calls: AccountActionBean → AccountService → AccountMapper → DB.
- Additional synchronous read for favorites: CatalogService → ProductMapper → DB.
- Database transaction: read-only; no explicit transaction annotation required.

Data flow
- Input: username, password.
- Reads SIGNON, ACCOUNT, PROFILE, BANNERDATA to hydrate Account.
- On success, reads PRODUCT list for favouriteCategoryId; stores AccountActionBean in session; redirects to main catalog.

Error handling and recovery
- On invalid credentials: adds message and forwards to SignonForm.jsp (no redirect, no session set).
- Any mapper/DB error propagates to generic error handling (Stripes/JSP error page).

```mermaid
sequenceDiagram
    autonumber
    participant U as User
    participant B as Browser
    participant S as Stripes Dispatcher
    participant AAB as AccountActionBean
    participant AS as AccountService
    participant AM as AccountMapper
    participant DB as HSQLDB
    participant CS as CatalogService
    participant PM as ProductMapper

    U->>B: Submit Sign In (username/password)
    B->>S: POST /actions/Account.action?signon
    S->>AAB: dispatch signon()
    AAB->>AS: getAccount(username, password)
    AS->>AM: getAccountByUsernameAndPassword(username, password)
    AM->>DB: SELECT SIGNON/ACCOUNT/PROFILE/BANNERDATA JOIN
    DB-->>AM: Account + profile + bannerName
    AM-->>AS: Account or null
    AS-->>AAB: Account or null

    alt Authentication success
        AAB->>CS: getProductListByCategory(account.favouriteCategoryId)
        CS->>PM: getProductListByCategory(categoryId)
        PM->>DB: SELECT PRODUCT by category
        DB-->>PM: Product[]
        PM-->>CS: Product[]
        CS-->>AAB: Product[]
        AAB->>S: Redirect to /actions/Catalog.action (viewMain)
        S-->>B: 302 Found
        B->>S: GET /actions/Catalog.action
        S->>CatalogActionBean: viewMain()
        CatalogActionBean-->>S: Forward MAIN.jsp
        S-->>B: 200 MAIN.jsp
    else Authentication failed
        AAB-->>S: Forward SignonForm.jsp with error message
        S-->>B: 200 SIGNON.jsp
    end
```

---

# Workflow 2: Browse Catalog and Add Item to Cart

Purpose and trigger
- Purpose: Navigate categories/products/items and add an item to the cart with in-stock flag.
- Trigger: User browses catalog and clicks “Add to Cart” on an item page.

Communication patterns
- HTTP GET requests to Stripes actions and JSP forwards (synchronous).
- Catalog retrieval: CatalogActionBean → CatalogService → Mappers → DB (synchronous).
- Add-to-cart: CartActionBean → CatalogService → ItemMapper/ProductMapper → DB (synchronous).
- No async events.

Data flow
- Read-only for browsing: CATEGORY, PRODUCT, ITEM (+PRODUCT join), INVENTORY.
- Add-to-cart reads INVENTORY qty for stock flag and ITEM (joined with PRODUCT, INVENTORY).

Error handling and recovery
- If workingItemId is null on add: forward to Error.jsp.
- Otherwise always forwards to Cart.jsp.

```mermaid
sequenceDiagram
    autonumber
    participant U as User
    participant B as Browser
    participant S as Stripes Dispatcher
    participant CAB as CatalogActionBean
    participant CS as CatalogService
    participant CtM as CategoryMapper
    participant PmM as ProductMapper
    participant ImM as ItemMapper
    participant DB as HSQLDB
    participant CartAB as CartActionBean
    participant Cart as Cart (session)

    U->>B: Click category link
    B->>S: GET /actions/Catalog.action?viewCategory&categoryId=...
    S->>CAB: viewCategory()
    CAB->>CS: getCategory(categoryId), getProductListByCategory(categoryId)
    CS->>CtM: getCategory
    CtM->>DB: SELECT CATEGORY
    DB-->>CtM: Category
    CtM-->>CS: Category
    CS->>PmM: getProductListByCategory
    PmM->>DB: SELECT PRODUCT by category
    DB-->>PmM: Product[]
    PmM-->>CS: Product[]
    CS-->>CAB: Category + Product[]
    CAB-->>S: Forward Category.jsp
    S-->>B: 200 Category.jsp

    U->>B: Click product link
    B->>S: GET /actions/Catalog.action?viewProduct&productId=...
    S->>CAB: viewProduct()
    CAB->>CS: getProduct(productId), getItemListByProduct(productId)
    CS->>PmM: getProduct
    PmM->>DB: SELECT PRODUCT
    DB-->>PmM: Product
    PmM-->>CS: Product
    CS->>ImM: getItemListByProduct(productId)
    ImM->>DB: SELECT ITEM join PRODUCT
    DB-->>ImM: Item[]
    ImM-->>CS: Item[]
    CS-->>CAB: Product + Item[]
    CAB-->>S: Forward Product.jsp
    S-->>B: 200 Product.jsp

    U->>B: Click Add to Cart on item page
    B->>S: GET /actions/Cart.action?addItemToCart&workingItemId=EST-1
    S->>CartAB: addItemToCart()
    alt Item already in cart
        CartAB->>Cart: containsItemId(EST-1) == true
        CartAB->>Cart: incrementQuantityByItemId(EST-1)
        CartAB-->>S: Forward Cart.jsp
        S-->>B: 200 Cart.jsp
    else New item
        CartAB->>Cart: containsItemId(EST-1) == false
        CartAB->>CS: isItemInStock(EST-1)
        CS->>ImM: getInventoryQuantity(itemId)
        ImM->>DB: SELECT INVENTORY.QTY
        DB-->>ImM: qty
        ImM-->>CS: qty
        CS-->>CartAB: inStock = (qty > 0)

        CartAB->>CS: getItem(EST-1)
        CS->>ImM: getItem(itemId)
        ImM->>DB: SELECT ITEM join INVENTORY + PRODUCT
        DB-->>ImM: Item + Product + qty
        ImM-->>CS: Item
        CS-->>CartAB: Item

        CartAB->>Cart: addItem(Item, inStock)
        CartAB-->>S: Forward Cart.jsp
        S-->>B: 200 Cart.jsp
    end
```

---

# Workflow 3: Update Cart Quantities

Purpose and trigger
- Purpose: Adjust item quantities or remove items in the cart.
- Trigger: User submits the cart form with per-item quantity fields; action updateCartQuantities.

Communication patterns
- HTTP POST to Stripes action; no service/DB calls (business in session-scoped Cart).

Data flow
- Inputs: key-value pairs where key=itemId and value=desired quantity.
- Updates Cart’s in-memory state in session.

Error handling and recovery
- If quantity < 1, item is removed.
- Invalid/missing params are ignored per item; forwards to Cart.jsp.

```mermaid
sequenceDiagram
    autonumber
    participant U as User
    participant B as Browser
    participant S as Stripes Dispatcher
    participant CartAB as CartActionBean
    participant Cart as Cart (session)

    U->>B: Submit cart updates (e.g., EST-1=3, EST-2=0)
    B->>S: POST /actions/Cart.action?updateCartQuantities
    S->>CartAB: updateCartQuantities()
    loop For each CartItem
        CartAB->>Cart: setQuantityByItemId(itemId, qty)
        alt qty < 1
            CartAB->>Cart: removeItemById(itemId)
        end
    end
    CartAB-->>S: Forward Cart.jsp
    S-->>B: 200 Cart.jsp
```

---

# Workflow 4: Checkout and Place Order (Multi-step)

Purpose and trigger
- Purpose: Convert the cart into a confirmed order, decrement inventory, record status.
- Triggers:
  - newOrderForm (GET) to initialize Order from Account + Cart.
  - newOrder (POST) to navigate shipping/confirmation, and finally place the order.

Communication patterns
- HTTP GET/POST with Stripes.
- In-process service calls: OrderActionBean → OrderService → Mappers → DB (synchronous).
- Transactional boundary: OrderService.insertOrder is @Transactional (ACID across sequence, inventory, order, line items).

Data flow
- Reads Account + Cart from session; Order.initOrder computes totals and line items.
- Writes: INVENTORY (decrement), ORDERS, ORDERSTATUS, LINEITEM; SEQUENCE updated atomically.

Error handling and recovery
- Not authenticated in newOrderForm: redirect/forward back to sign-on with message.
- newOrder steps:
  - If shippingAddressRequired: show shipping form.
  - If not confirmed: show confirmation.
  - On insertOrder failure (e.g., missing sequence): transaction rollback; forward to Error.jsp.

```mermaid
sequenceDiagram
    autonumber
    participant U as User
    participant B as Browser
    participant S as Stripes Dispatcher
    participant OAB as OrderActionBean
    participant AAB as AccountActionBean (session)
    participant CartAB as CartActionBean (session)
    participant OS as OrderService
    participant SeqM as SequenceMapper
    participant ImM as ItemMapper
    participant OM as OrderMapper
    participant LM as LineItemMapper
    participant DB as HSQLDB

    Note over U,B: Step 1: Initialize order from session state
    U->>B: Click "Proceed to Checkout"
    B->>S: GET /actions/Order.action?newOrderForm
    S->>OAB: newOrderForm()
    OAB->>S: retrieve /actions/Account.action and /actions/Cart.action from session
    alt Not authenticated
        OAB-->>S: Forward to AccountActionBean.signonForm with message
        S-->>B: 200 Signon.jsp
    else Authenticated
        OAB->>OAB: order.initOrder(account, cart) (domain logic)
        OAB-->>S: Forward NewOrder.jsp
        S-->>B: 200 NewOrder.jsp
    end

    Note over U,B: Step 2: Multi-step submit → place order
    U->>B: Submit order (shipping/confirm/submit)
    B->>S: POST /actions/Order.action?newOrder
    S->>OAB: newOrder()

    alt shippingAddressRequired == true
        OAB->>OAB: shippingAddressRequired=false
        OAB-->>S: Forward ShippingForm.jsp
        S-->>B: 200 Shipping.jsp
    else not confirmed
        OAB-->>S: Forward ConfirmOrder.jsp
        S-->>B: 200 Confirm.jsp
    else confirmed
        OAB->>OS: insertOrder(order) [@Transactional]
        OS->>SeqM: getSequence("ordernum")
        SeqM->>DB: SELECT SEQUENCE where name='ordernum'
        DB-->>SeqM: Sequence(nextId)
        SeqM-->>OS: Sequence
        OS->>SeqM: updateSequence(nextId+1)
        SeqM->>DB: UPDATE SEQUENCE set nextid = nextId+1

        loop For each LineItem
            OS->>ImM: updateInventoryQuantity({itemId, increment=quantity})
            ImM->>DB: UPDATE INVENTORY set QTY = QTY - quantity
        end

        OS->>OM: insertOrder(order header)
        OM->>DB: INSERT ORDERS
        OS->>OM: insertOrderStatus(order)
        OM->>DB: INSERT ORDERSTATUS (timestamp, status)

        loop For each LineItem
            OS->>LM: insertLineItem(line)
            LM->>DB: INSERT LINEITEM
        end

        OS-->>OAB: success (orderId assigned)
        OAB->>S: remove /actions/Cart.action from session (clear cart)
        OAB-->>S: Forward ViewOrder.jsp with success message
        S-->>B: 200 ViewOrder.jsp
    end

    opt Failure during insertOrder
        OS->>OS: throw RuntimeException
        note over OS,DB: Spring rolls back the transaction
        OAB-->>S: Forward Error.jsp with failure message
        S-->>B: 500/200 Error.jsp (depending on handler)
    end
```

---

# Workflow 5: View Order History and Order Details

Purpose and trigger
- Purpose: Allow users to list their orders and view details; enforce simple authorization.
- Triggers:
  - listOrders (GET) from navigation.
  - viewOrder (GET) with orderId from link.

Communication patterns
- Synchronous reads to DB via OrderService and mappers.
- Enrichment: OrderService.getOrder hydrates line items with Item and current INVENTORY quantity.

Data flow
- listOrders reads ORDERS + ORDERSTATUS (by username).
- viewOrder reads ORDERS + ORDERSTATUS + LINEITEM; for each line item reads ITEM + PRODUCT + INVENTORY.

Error handling and recovery
- viewOrder rejects access if session username ≠ order.username (forwards to Error.jsp).

```mermaid
sequenceDiagram
    autonumber
    participant U as User
    participant B as Browser
    participant S as Stripes Dispatcher
    participant OAB as OrderActionBean
    participant AAB as AccountActionBean (session)
    participant OS as OrderService
    participant OM as OrderMapper
    participant LM as LineItemMapper
    participant ImM as ItemMapper
    participant DB as HSQLDB

    Note over U,B: List user orders
    U->>B: Click "My Orders"
    B->>S: GET /actions/Order.action?listOrders
    S->>OAB: listOrders()
    OAB->>S: get /actions/Account.action from session
    OAB->>OS: getOrdersByUsername(username)
    OS->>OM: getOrdersByUsername(username)
    OM->>DB: SELECT ORDERS join ORDERSTATUS where userid=?
    DB-->>OM: Order[]
    OM-->>OS: Order[]
    OS-->>OAB: Order[]
    OAB-->>S: Forward ListOrders.jsp
    S-->>B: 200 ListOrders.jsp

    Note over U,B: View order details
    U->>B: Click order link (orderId=1234)
    B->>S: GET /actions/Order.action?viewOrder&orderId=1234
    S->>OAB: viewOrder(orderId)
    OAB->>OS: getOrder(orderId)
    OS->>OM: getOrder(orderId)
    OM->>DB: SELECT ORDERS join ORDERSTATUS where orderid=?
    DB-->>OM: Order
    OM-->>OS: Order
    OS->>LM: getLineItemsByOrderId(orderId)
    LM->>DB: SELECT LINEITEM where orderid=?
    DB-->>LM: LineItem[]
    LM-->>OS: LineItem[]

    loop For each LineItem
        OS->>ImM: getItem(itemId)
        ImM->>DB: SELECT ITEM join PRODUCT + INVENTORY
        DB-->>ImM: Item (with Product, qty)
        ImM-->>OS: Item
        OS->>OS: attach Item to LineItem
    end

    OS-->>OAB: Enriched Order
    OAB->>S: get /actions/Account.action (username)
    alt username matches order.username
        OAB-->>S: Forward ViewOrder.jsp
        S-->>B: 200 ViewOrder.jsp
    else unauthorized
        OAB-->>S: Forward Error.jsp (Unauthorized)
        S-->>B: 403/200 Error.jsp
    end
```

---

# Workflow 6: Account Registration and Profile Update

Purpose and trigger
- Purpose: Create a new account (spanning ACCOUNT/PROFILE/SIGNON) or update existing profile and optionally password.
- Triggers:
  - newAccountForm → newAccount (POST).
  - editAccountForm → editAccount (POST).

Communication patterns
- Synchronous service calls with transactional writes:
  - AccountService.insertAccount: INSERT into ACCOUNT, PROFILE, SIGNON (single transaction).
  - AccountService.updateAccount: UPDATE ACCOUNT, PROFILE, and conditionally SIGNON.

Data flow
- INSERT/UPDATE across ACCOUNT, PROFILE, SIGNON; on success, reload Account and precompute favorites (CatalogService.getProductListByCategory).

Error handling and recovery
- On any write failure, transaction rollback; forwards to error page (Stripes default).
- On success: redirect to main catalog.

```mermaid
sequenceDiagram
    autonumber
    participant U as User
    participant B as Browser
    participant S as Stripes Dispatcher
    participant AAB as AccountActionBean
    participant AS as AccountService
    participant AM as AccountMapper
    participant CS as CatalogService
    participant PM as ProductMapper
    participant DB as HSQLDB

    Note over U,B: Registration
    U->>B: Submit New Account form
    B->>S: POST /actions/Account.action?newAccount
    S->>AAB: newAccount()
    AAB->>AS: insertAccount(Account) [@Transactional]
    AS->>AM: insertAccount(Account)
    AM->>DB: INSERT ACCOUNT
    DB-->>AM: OK
    AS->>AM: insertProfile(Account)
    AM->>DB: INSERT PROFILE
    DB-->>AM: OK
    AS->>AM: insertSignon(Account)
    AM->>DB: INSERT SIGNON
    DB-->>AM: OK
    AS-->>AAB: success

    AAB->>AS: getAccount(username)
    AS->>AM: getAccountByUsername(username)
    AM->>DB: SELECT ACCOUNT/PROFILE/SIGNON/BANNERDATA JOIN
    DB-->>AM: Account
    AM-->>AS: Account
    AS-->>AAB: Account

    AAB->>CS: getProductListByCategory(favouriteCategoryId)
    CS->>PM: getProductListByCategory
    PM->>DB: SELECT PRODUCT by category
    DB-->>PM: Product[]
    PM-->>CS: Product[]
    CS-->>AAB: Product[]
    AAB->>S: Redirect to /actions/Catalog.action
    S-->>B: 302 then 200 MAIN.jsp

    Note over U,B: Profile update (similar)
    U->>B: Submit Edit Account form
    B->>S: POST /actions/Account.action?editAccount
    S->>AAB: editAccount()
    AAB->>AS: updateAccount(Account) [@Transactional]
    AS->>AM: updateAccount(Account)
    AM->>DB: UPDATE ACCOUNT
    DB-->>AM: OK
    AS->>AM: updateProfile(Account)
    AM->>DB: UPDATE PROFILE
    DB-->>AM: OK
    alt password provided
        AS->>AM: updateSignon(Account)
        AM->>DB: UPDATE SIGNON
        DB-->>AM: OK
    end
    AS-->>AAB: success
    AAB->>AS: getAccount(username) + refresh myList via CS->PM as above
    AAB->>S: Redirect to /actions/Catalog.action
    S-->>B: 302 then 200 MAIN.jsp
```

---

# Workflow 7: Product Search

Purpose and trigger
- Purpose: Search products by keyword(s) against product name (case-insensitive).
- Trigger: User submits search form (searchProducts).

Communication patterns
- Synchronous calls: CatalogActionBean → CatalogService → ProductMapper → DB.
- No async events; read-only.

Data flow
- Input: keyword string.
- Service splits on whitespace; for each token calls ProductMapper.searchProductList("%kw%"); concatenates results (no deduplication).

Error handling and recovery
- If keyword empty/null: forward to SearchProducts.jsp with error message.
- DB errors show error page.

```mermaid
sequenceDiagram
    autonumber
    participant U as User
    participant B as Browser
    participant S as Stripes Dispatcher
    participant CAB as CatalogActionBean
    participant CS as CatalogService
    participant PM as ProductMapper
    participant DB as HSQLDB

    U->>B: Submit search (keyword="gold fish")
    B->>S: GET /actions/Catalog.action?searchProducts&keyword=gold%20fish
    S->>CAB: searchProducts()
    alt keyword empty
        CAB-->>S: Forward SearchProducts.jsp with error
        S-->>B: 200 Search.jsp
    else tokens present
        CAB->>CS: searchProductList("gold fish")
        CS->>CS: tokens = ["gold","fish"]
        loop For each token
            CS->>PM: searchProductList("%token%")
            PM->>DB: SELECT PRODUCT WHERE lower(NAME) LIKE ?
            DB-->>PM: Product[] (for token)
            PM-->>CS: Product[]
            CS->>CS: append to results
        end
        CS-->>CAB: Aggregated Product[] (no dedupe)
        CAB-->>S: Forward SearchProducts.jsp
        S-->>B: 200 SearchResults.jsp
    end
```

---

# Cross-cutting Communication and Error Patterns

- Communication types:
  - HTTP requests to Stripes actions (synchronous).
  - In-process service and mapper calls via Spring DI (synchronous).
  - Database access via MyBatis mappers (synchronous SQL).
  - No asynchronous messaging, background jobs, or external APIs.

- Transactions:
  - OrderService.insertOrder and AccountService insert/update methods are transactional. Failures trigger rollback across all involved tables.
  - Reads are non-transactional or default transactional read-only behavior.

- Caching:
  - MyBatis second-level cache per mapper (<cache/>) reduces repeated reads; cache is invalidated on writes to the same namespace.

- Error handling:
  - Validation/flow errors: Forward to relevant JSP with messages (e.g., signon failure, empty search, unauthorized order view).
  - System/DB errors: Propagate to Error.jsp; transactional methods roll back state on exception.

- Session and state:
  - Session-scoped ActionBeans maintain conversational state (Account, Cart, Order) under keys like "/actions/Account.action".
  - On successful sign-in, "accountBean" is also stored for banner and favorites rendering.
  - On order completion, CartActionBean is cleared from session.