# Dynamic Interaction Flows and Sequence Diagrams

## 1) User Sign-in

Purpose and trigger
- User submits credentials to sign in and start an authenticated session.
- Trigger: POST /actions/Account.action?event=signon from SignonForm.jsp.

Communication patterns
- HTTP POST (Stripes action event)
- Synchronous in-process calls: ActionBean -> Service -> MyBatis Mapper -> DB
- Session state mutation (HttpSession)
- Redirect to CatalogActionBean (HTTP 302), then server-side forward to Main.jsp

Data flow
- Input: username/password
- Query: SIGNON + ACCOUNT + PROFILE (+ BANNERDATA join in mapper)
- Output: Account domain object; optional myList = favorite products by favouriteCategoryId

Error handling
- Invalid credentials -> set message, forward back to SignonForm.jsp
- Successful sign-in -> password nulled in memory; authenticated flag set; session attribute “accountBean” stored

```mermaid
sequenceDiagram
  autonumber
  participant U as User
  participant B as Browser
  participant D as StripesDispatcher (*.action)
  participant A as AccountActionBean
  participant S as AccountService
  participant M as AccountMapper (MyBatis)
  participant DB as HSQLDB
  participant C as CatalogService
  participant PM as ProductMapper
  participant SES as HttpSession
  participant J1 as SignonForm.jsp
  participant J2 as Main.jsp

  U->>B: Enter username/password
  B->>D: POST /actions/Account.action?event=signon
  D->>A: invoke signon()
  A->>S: getAccount(username, password)
  S->>M: getAccountByUsernameAndPassword(u, p)
  M->>DB: SELECT SIGNON + ACCOUNT + PROFILE (+ BANNERDATA)
  DB-->>M: Account row or null
  M-->>S: Account or null
  S-->>A: Account or null

  alt Authentication failed
    A->>A: setMessage("Invalid username or password.")
    A-->>D: Forward -> SignonForm.jsp
    D-->>B: 200 OK (render SignonForm.jsp)
  else Authentication success
    A->>A: account.password = null; authenticated = true
    A->>SES: set("accountBean", this)
    A->>C: getProductListByCategory(account.favouriteCategoryId)
    C->>PM: getProductListByCategory(catId)
    PM->>DB: SELECT PRODUCT by category
    DB-->>PM: List<Product>
    PM-->>C: List<Product>
    C-->>A: myList (favorites)
    A-->>D: Redirect to /actions/Catalog.action
    D-->>B: 302 Found (Location: /actions/Catalog.action)
    B->>D: GET /actions/Catalog.action (default event viewMain)
    D-->>B: 200 OK (render Main.jsp)
  end
```


## 2) Registration / New Account

Purpose and trigger
- Create a new user account with profile and credentials.
- Trigger: POST /actions/Account.action?event=newAccount from NewAccountForm.jsp.

Communication patterns
- HTTP POST (Stripes event)
- Synchronous, transactional service method across multiple tables
- Session state mutation; redirect on success

Data flow
- Inserts: SIGNON, ACCOUNT, PROFILE
- Reads: Account + favorites after creation

Error handling
- Any DB constraint/insert error -> transaction rollback; Stripes will resolve to error page (generic Error.jsp) depending on exception handling; otherwise standard success redirect.

```mermaid
sequenceDiagram
  autonumber
  participant U as User
  participant B as Browser
  participant D as StripesDispatcher
  participant A as AccountActionBean
  participant S as AccountService (@Transactional)
  participant M as AccountMapper
  participant DB as HSQLDB
  participant C as CatalogService
  participant PM as ProductMapper
  participant SES as HttpSession
  participant J as NewAccountForm.jsp

  U->>B: Fill registration fields
  B->>D: POST /actions/Account.action?event=newAccount
  D->>A: invoke newAccount()
  Note over S,DB: Transaction begins
  A->>S: insertAccount(account)
  S->>M: insertSignon(account)
  M->>DB: INSERT into SIGNON
  DB-->>M: OK
  S->>M: insertAccount(account)
  M->>DB: INSERT into ACCOUNT
  DB-->>M: OK
  S->>M: insertProfile(account)
  M->>DB: INSERT into PROFILE (flags 0/1)
  DB-->>M: OK
  Note over S,DB: Transaction commit
  S-->>A: void

  A->>S: getAccount(username)
  S->>M: getAccountByUsername(username)
  M->>DB: SELECT joins
  DB-->>M: Account
  M-->>S: Account
  S-->>A: Account
  A->>A: authenticated = true; account.password = null
  A->>SES: set("accountBean", this)
  A->>C: getProductListByCategory(account.favouriteCategoryId)
  C->>PM: getProductListByCategory(catId)
  PM->>DB: SELECT PRODUCT
  DB-->>PM: List<Product>
  PM-->>C: List<Product>
  C-->>A: myList
  A-->>D: Redirect to /actions/Catalog.action
  D-->>B: 302 Found

  alt DB error during insert
    Note over S,DB: Transaction rollback
    S-->>A: Exception
    A-->>D: Forward -> Error.jsp (message)
    D-->>B: 200 OK Error.jsp
  end
```


## 3) Browse Catalog (Category -> Product -> Item)

Purpose and trigger
- Navigate catalog hierarchy and view item details with live inventory.
- Triggers:
  - GET /actions/Catalog.action?event=viewCategory&categoryId=...
  - GET /actions/Catalog.action?event=viewProduct&productId=...
  - GET /actions/Catalog.action?event=viewItem&itemId=...

Communication patterns
- HTTP GET (Stripes events)
- Read-only synchronous service calls
- Server-side forwards to JSP views

Data flow
- Category view: CATEGORY + PRODUCT list
- Product view: PRODUCT + ITEM list
- Item view: ITEM join PRODUCT + INVENTORY (quantity)

Error handling
- Missing IDs are handled by ActionBean conditionals; typical behavior is forward to Error.jsp with message in edge cases.

```mermaid
sequenceDiagram
  autonumber
  participant U as User
  participant D as StripesDispatcher
  participant CatAB as CatalogActionBean
  participant CS as CatalogService
  participant CM as CategoryMapper
  participant PM as ProductMapper
  participant IM as ItemMapper
  participant DB as HSQLDB
  participant JC as Category.jsp
  participant JP as Product.jsp
  participant JI as Item.jsp

  U->>D: GET ...Catalog.action?viewCategory&categoryId=CAT
  D->>CatAB: viewCategory()
  CatAB->>CS: getCategory(CAT)
  CS->>CM: getCategory(CAT)
  CM->>DB: SELECT CATEGORY
  DB-->>CM: Category
  CM-->>CS: Category
  CS-->>CatAB: Category
  CatAB->>CS: getProductListByCategory(CAT)
  CS->>PM: getProductListByCategory(CAT)
  PM->>DB: SELECT PRODUCT by category
  DB-->>PM: List<Product>
  PM-->>CS: List<Product>
  CS-->>CatAB: List<Product>
  CatAB-->>D: Forward -> Category.jsp
  D-->>U: 200 OK (Category.jsp)

  U->>D: GET ...Catalog.action?viewProduct&productId=PRD
  D->>CatAB: viewProduct()
  CatAB->>CS: getProduct(PRD)
  CS->>PM: getProduct(PRD)
  PM->>DB: SELECT PRODUCT
  DB-->>PM: Product
  PM-->>CS: Product
  CS-->>CatAB: Product
  CatAB->>CS: getItemListByProduct(PRD)
  CS->>IM: getItemListByProduct(PRD)
  IM->>DB: SELECT ITEM join PRODUCT
  DB-->>IM: List<Item>
  IM-->>CS: List<Item>
  CS-->>CatAB: List<Item>
  CatAB-->>D: Forward -> Product.jsp
  D-->>U: 200 OK (Product.jsp)

  U->>D: GET ...Catalog.action?viewItem&itemId=ITM
  D->>CatAB: viewItem()
  CatAB->>CS: getItem(ITM)
  CS->>IM: getItem(ITM)
  IM->>DB: SELECT ITEM join INVENTORY + PRODUCT
  DB-->>IM: Item(quantity, product)
  IM-->>CS: Item
  CS-->>CatAB: Item
  CatAB-->>D: Forward -> Item.jsp
  D-->>U: 200 OK (Item.jsp)
```


## 4) Search Products

Purpose and trigger
- Full-text-like search on product name with multiple tokens.
- Trigger: GET /actions/Catalog.action?event=searchProducts&keyword=...

Communication patterns
- HTTP GET
- Loop over tokens; multiple synchronous mapper calls; aggregate results
- Forward to SearchProducts.jsp

Data flow
- For each token t: ProductMapper.searchProductList("%t%")
- Output: aggregated List<Product>

Error handling
- Empty keyword -> set message and forward to Error.jsp

```mermaid
sequenceDiagram
  autonumber
  participant U as User
  participant D as StripesDispatcher
  participant CatAB as CatalogActionBean
  participant CS as CatalogService
  participant PM as ProductMapper
  participant DB as HSQLDB
  participant JS as SearchProducts.jsp
  participant JE as Error.jsp

  U->>D: GET ...Catalog.action?searchProducts&keyword="fresh water"
  D->>CatAB: searchProducts()
  alt keyword empty
    CatAB->>CatAB: setMessage("Please enter a keyword...")
    CatAB-->>D: Forward -> Error.jsp
    D-->>U: 200 OK (Error.jsp)
  else keyword non-empty
    CatAB->>CS: searchProductList("fresh water")
    loop each token (lowercased)
      CS->>PM: searchProductList("%token%")
      PM->>DB: SELECT PRODUCT WHERE lower(name) like %token%
      DB-->>PM: List<Product>
      PM-->>CS: Partial results
    end
    CS-->>CatAB: Aggregated List<Product>
    CatAB-->>D: Forward -> SearchProducts.jsp
    D-->>U: 200 OK (SearchProducts.jsp)
  end
```


## 5) Add to Cart and Update Cart

Purpose and trigger
- Add items to cart and adjust quantities.
- Triggers:
  - GET/POST /actions/Cart.action?event=addItemToCart&workingItemId=...
  - POST /actions/Cart.action?event=updateCartQuantities

Communication patterns
- HTTP requests to Stripes actions
- Synchronous service calls for stock check and item load
- Session-scoped cart mutated; forward to Cart.jsp

Data flow
- Reads: INVENTORY quantity, ITEM details (with product)
- Writes: Cart object in session

Error handling
- removeItemFromCart with missing workingItemId -> Error.jsp (not shown in this diagram)
- Out-of-stock item -> item can be added with inStock=false flag; UI conveys availability

```mermaid
sequenceDiagram
  autonumber
  participant U as User
  participant D as StripesDispatcher
  participant CartAB as CartActionBean
  participant CS as CatalogService
  participant IM as ItemMapper
  participant DB as HSQLDB
  participant SES as HttpSession
  participant JC as Cart.jsp

  U->>D: POST ...Cart.action?addItemToCart&workingItemId=ITM
  D->>CartAB: addItemToCart()
  alt Item already in cart
    CartAB->>SES: cart.incrementQuantityByItemId(ITM)
    CartAB-->>D: Forward -> Cart.jsp
    D-->>U: 200 OK (Cart.jsp)
  else Item not in cart
    CartAB->>CS: isItemInStock(ITM)
    CS->>IM: getInventoryQuantity(ITM)
    IM->>DB: SELECT INVENTORY.qty
    DB-->>IM: qty
    IM-->>CS: qty
    CS-->>CartAB: inStock = (qty > 0)
    CartAB->>CS: getItem(ITM)
    CS->>IM: getItem(ITM)
    IM->>DB: SELECT ITEM join INVENTORY + PRODUCT
    DB-->>IM: Item
    IM-->>CS: Item
    CS-->>CartAB: Item
    CartAB->>SES: cart.addItem(item, inStock)
    CartAB-->>D: Forward -> Cart.jsp
    D-->>U: 200 OK (Cart.jsp)
  end

  U->>D: POST ...Cart.action?updateCartQuantities
  D->>CartAB: updateCartQuantities()
  loop for each cart item
    CartAB->>SES: read new qty from request param (name=itemId)
    alt qty < 1
      CartAB->>SES: cart.removeItemById(itemId)
    else qty >= 1
      CartAB->>SES: cart.setQuantityByItemId(itemId, qty)
    end
  end
  CartAB-->>D: Forward -> Cart.jsp
  D-->>U: 200 OK (Cart.jsp)
```


## 6) Checkout and Place Order (Multi-step)

Purpose and trigger
- Convert cart into a persisted order, decrement inventory, and present confirmation.
- Triggers:
  - GET /actions/Cart.action?event=checkOut -> proceeds to Order flow
  - GET /actions/Order.action?event=newOrderForm
  - POST /actions/Order.action?event=newOrder (shippingAddressRequired/confirmed flags)

Communication patterns
- HTTP GET/POST for Stripes events
- Synchronous transactional service call for order creation
- Reads/writes across multiple tables; session state dependencies (AccountActionBean, CartActionBean)

Data flow
- Order.initOrder(account, cart): build Order and LineItems in memory
- insertOrder transaction:
  - Read/update SEQUENCE('ordernum')
  - Decrement INVENTORY per item
  - Insert ORDERS + ORDERSTATUS
  - Insert LINEITEM rows

Error handling
- Not authenticated -> forward to signon
- shippingAddressRequired -> ShippingForm.jsp
- Not confirmed -> ConfirmOrder.jsp
- Sequence missing or DB errors -> exception, transaction rollback, Error.jsp

```mermaid
sequenceDiagram
  autonumber
  participant U as User
  participant D as StripesDispatcher
  participant O as OrderActionBean
  participant A as AccountActionBean (session)
  participant CartAB as CartActionBean (session)
  participant S as OrderService (@Transactional)
  participant SeqM as SequenceMapper
  participant IM as ItemMapper
  participant OM as OrderMapper
  participant LM as LineItemMapper
  participant DB as HSQLDB
  participant JN as NewOrderForm.jsp
  participant JS as ShippingForm.jsp
  participant JC as ConfirmOrder.jsp
  participant JV as ViewOrder.jsp
  participant JE as Error.jsp

  U->>D: GET ...Cart.action?checkOut
  D-->>U: 200 OK (Checkout.jsp)

  U->>D: GET ...Order.action?newOrderForm
  D->>O: newOrderForm()
  alt not authenticated (A.authenticated == false)
    O->>O: setMessage("You must sign on before attempting to check out.")
    O-->>D: Forward -> AccountActionBean.signonForm
    D-->>U: 200 OK (SignonForm.jsp)
  else authenticated
    O->>A: read account
    O->>CartAB: read cart
    O->>O: order.initOrder(account, cart)
    O-->>D: Forward -> NewOrderForm.jsp
    D-->>U: 200 OK (NewOrderForm.jsp)
  end

  U->>D: POST ...Order.action?newOrder (flags: shippingAddressRequired, confirmed)
  D->>O: newOrder()
  alt shippingAddressRequired == true
    O-->>D: Forward -> ShippingForm.jsp
    D-->>U: 200 OK (ShippingForm.jsp)
  else confirmed == false
    O-->>D: Forward -> ConfirmOrder.jsp
    D-->>U: 200 OK (ConfirmOrder.jsp)
  else confirmed == true
    Note over S,DB: Transaction begins (@Transactional)
    O->>S: insertOrder(order)
    S->>SeqM: getSequence("ordernum")
    SeqM->>DB: SELECT SEQUENCE where name='ordernum'
    DB-->>SeqM: Sequence(nextId)
    SeqM-->>S: nextId
    S->>SeqM: updateSequence(nextId+1)
    SeqM->>DB: UPDATE SEQUENCE set nextId=nextId+1
    DB-->>SeqM: OK
    S->>S: set orderId on order and line items

    par For each line item
      S->>IM: updateInventoryQuantity({itemId, increment=quantity})
      IM->>DB: UPDATE INVENTORY set qty = qty - increment
      DB-->>IM: OK
    and Persist order header and status
      S->>OM: insertOrder(order)
      OM->>DB: INSERT ORDERS
      DB-->>OM: OK
      S->>OM: insertOrderStatus(order)
      OM->>DB: INSERT ORDERSTATUS (initial status)
      DB-->>OM: OK
    and Persist line items
      loop for each line
        S->>LM: insertLineItem(line)
        LM->>DB: INSERT LINEITEM
        DB-->>LM: OK
      end
    end

    Note over S,DB: Transaction commit
    S-->>O: void
    O->>CartAB: clear cart
    O->>O: setMessage("Thank you, your order has been submitted.")
    O-->>D: Forward -> ViewOrder.jsp
    D-->>U: 200 OK (ViewOrder.jsp)
  end

  alt Sequence row missing or DB error
    Note over S,DB: Transaction rollback
    S-->>O: Exception
    O-->>D: Forward -> Error.jsp (failure message)
    D-->>U: 200 OK (Error.jsp)
  end
```


## 7) List Orders and View Order Details

Purpose and trigger
- List user’s past orders and view order details enriched with current item and inventory data.
- Triggers:
  - GET /actions/Order.action?event=listOrders
  - GET /actions/Order.action?event=viewOrder&orderId=...

Communication patterns
- HTTP GET
- Synchronous service calls; getOrder aggregates multiple reads
- Authorization check for viewOrder

Data flow
- listOrders: ORDERS (+ ORDERSTATUS) by username
- viewOrder: ORDER header + LINEITEMS; for each line: load Item and current inventory quantity

Error handling
- viewOrder: if current user != order.username -> Error.jsp

```mermaid
sequenceDiagram
  autonumber
  participant U as User
  participant D as StripesDispatcher
  participant O as OrderActionBean
  participant A as AccountActionBean (session)
  participant S as OrderService
  participant OM as OrderMapper
  participant LM as LineItemMapper
  participant IM as ItemMapper
  participant DB as HSQLDB
  participant JL as ListOrders.jsp
  participant JV as ViewOrder.jsp
  participant JE as Error.jsp

  U->>D: GET ...Order.action?listOrders
  D->>O: listOrders()
  O->>A: read username from session
  O->>S: getOrdersByUsername(username)
  S->>OM: getOrdersByUsername(username)
  OM->>DB: SELECT ORDERS + ORDERSTATUS WHERE USERID=?
  DB-->>OM: List<Order>
  OM-->>S: List<Order>
  S-->>O: List<Order>
  O-->>D: Forward -> ListOrders.jsp
  D-->>U: 200 OK (ListOrders.jsp)

  U->>D: GET ...Order.action?viewOrder&orderId=OID
  D->>O: viewOrder()
  O->>S: getOrder(OID)
  Note over S,DB: Read-assemble aggregate (not transactional write)
  S->>OM: getOrder(OID)
  OM->>DB: SELECT ORDERS + ORDERSTATUS WHERE ORDERID=OID
  DB-->>OM: Order header
  OM-->>S: Order
  S->>LM: getLineItemsByOrderId(OID)
  LM->>DB: SELECT LINEITEM WHERE ORDERID=OID
  DB-->>LM: List<LineItem>
  LM-->>S: List<LineItem>
  loop for each line item
    S->>IM: getItem(itemId)
    IM->>DB: SELECT ITEM join INVENTORY + PRODUCT
    DB-->>IM: Item (includes quantity)
    IM-->>S: Item
    S->>S: attach Item to LineItem
  end
  S-->>O: Order aggregate with line items

  alt current user matches order.username
    O-->>D: Forward -> ViewOrder.jsp
    D-->>U: 200 OK (ViewOrder.jsp)
  else unauthorized
    O->>O: setMessage("You may only view your own orders.")
    O-->>D: Forward -> Error.jsp
    D-->>U: 200 OK (Error.jsp)
  end
```


## 8) Edit Account Profile

Purpose and trigger
- Update account details and optionally password.
- Trigger: POST /actions/Account.action?event=editAccount from EditAccountForm.jsp.

Communication patterns
- HTTP POST
- Synchronous transactional update across multiple tables
- Redirect to Catalog on success

Data flow
- Updates: ACCOUNT, PROFILE; SIGNON only if password is provided
- Reads: refresh Account; loads favorites

Error handling
- Validation errors by Stripes @Validate; DB errors cause transaction rollback and error forward

```mermaid
sequenceDiagram
  autonumber
  participant U as User
  participant D as StripesDispatcher
  participant A as AccountActionBean
  participant S as AccountService (@Transactional)
  participant M as AccountMapper
  participant DB as HSQLDB
  participant C as CatalogService
  participant PM as ProductMapper
  participant J as EditAccountForm.jsp

  U->>D: POST ...Account.action?editAccount
  D->>A: editAccount()
  Note over S,DB: Transaction begins
  A->>S: updateAccount(account)
  S->>M: updateAccount(account)
  M->>DB: UPDATE ACCOUNT
  DB-->>M: OK
  S->>M: updateProfile(account)
  M->>DB: UPDATE PROFILE (flags)
  DB-->>M: OK
  alt password provided
    S->>M: updateSignon(account)
    M->>DB: UPDATE SIGNON (PASSWORD)
    DB-->>M: OK
  end
  Note over S,DB: Transaction commit
  S-->>A: void

  A->>S: getAccount(username)
  S->>M: getAccountByUsername(username)
  M->>DB: SELECT joins
  DB-->>M: Account
  M-->>S: Account
  S-->>A: Account
  A->>C: getProductListByCategory(account.favouriteCategoryId)
  C->>PM: getProductListByCategory(catId)
  PM->>DB: SELECT PRODUCT
  DB-->>PM: List<Product>
  PM-->>C: List<Product>
  C-->>A: myList
  A-->>D: Redirect -> /actions/Catalog.action
  D-->>U: 302 Found

  alt Validation/DB error
    Note over S,DB: Transaction rollback (if started)
    A-->>D: Forward -> EditAccountForm.jsp (with errors) or Error.jsp
    D-->>U: 200 OK
  end
```


# Communication Patterns Summary (Across Flows)

- Web to Controller: HTTP GET/POST to Stripes action events (*.action). Sync.
- Controller to Service: In-process method calls. Sync.
- Service to Persistence: MyBatis mapper calls (JDBC). Sync.
- Persistence to Database: SQL queries and DML against HSQLDB. Sync.
- Transactions:
  - @Transactional on AccountService.insertAccount/updateAccount
  - @Transactional on OrderService.insertOrder and getOrder (read-assembly may or may not be transactional; writes are transactional)
- View navigation:
  - ForwardResolution (server-side forward to JSP) for most views
  - RedirectResolution (HTTP 302) after sign-in/registration/account edit to avoid form resubmission
- Session state:
  - AccountActionBean, CatalogActionBean, CartActionBean, OrderActionBean are @SessionScope; HttpSession holds conversational state (accountBean, cart, order-in-progress)


# Event-Driven Interactions

- Controller “events” are Stripes action methods (e.g., signon, newOrder, viewItem). They are HTTP-triggered and synchronous; there is no asynchronous message bus.
- No external eventing; all message flows are request/response oriented within a single process and DB transaction boundaries.


# Error Handling and Recovery Patterns

- Authentication failure: message set and forward to SignonForm.jsp (no redirect).
- Authorization failure (viewOrder of another user): message and forward to Error.jsp.
- Input validation failures: Stripes validation errors keep user on form (forward with messages).
- Transactional failures (e.g., DB constraint errors, missing sequence):
  - Service throws exception; Spring triggers transaction rollback; UI resolves to Error.jsp with message.
- Search with empty keyword: message and forward to Error.jsp.
- Cart operations with missing parameters (e.g., remove without workingItemId): forward to Error.jsp.
- Concurrency caveat: inventory decrement has no lower-bound check; negative inventory is possible under concurrent orders; current code relies on DB isolation/ordering but has no explicit recovery logic.