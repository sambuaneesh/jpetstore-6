# Dynamic Interaction Flows and Sequence Diagrams — MyBatis JPetStore 6

## Common Request Lifecycle (Stripes + Spring + MyBatis)
```mermaid
sequenceDiagram
  autonumber
  participant U as User
  participant B as Browser (JSP)
  participant S as Stripes Dispatcher (*.action)
  participant AB as ActionBean (Session-scoped)
  participant TX as Spring TxManager
  participant SV as Service (@Service)
  participant MB as MyBatis Mapper
  participant DB as HSQLDB

  U->>B: Click/Submit (HTTP GET/POST)
  B->>S: HTTP request to /actions/*.action
  S->>AB: Resolve event method (e.g., signon, viewItem)
  alt Handler calls read-only service
    AB->>SV: service.read(...)
    SV->>MB: mapper.select(...)
    MB->>DB: SQL SELECT
    DB-->>MB: ResultSet
    MB-->>SV: Domain object(s)
    SV-->>AB: Assembled data
  else Handler calls @Transactional write service
    AB->>TX: Begin transaction (proxy)
    TX->>SV: service.write(...)
    SV->>MB: mapper.update/insert/delete(...)
    MB->>DB: SQL DML
    DB-->>MB: Update counts/keys
    MB-->>SV: Ack
    SV-->>TX: Return
    TX-->>DB: Commit
    TX-->>AB: Return
  end
  AB-->>S: ForwardResolution or RedirectResolution
  S-->>B: HTTP response (JSP rendered or redirect)
```
- Purpose: Baseline call path and where transactions are opened/committed.
- Communication patterns:
  - HTTP synchronous requests to Stripes.
  - In-process method calls ActionBean -> Service -> Mapper.
  - MyBatis executes SQL over JDBC to HSQLDB.
  - Transactions managed by Spring AOP proxies on @Transactional services.
  - No asynchronous messaging or event bus.

---

## WF1: Sign In (AccountActionBean.signon)
```mermaid
sequenceDiagram
  autonumber
  participant U as User
  participant B as Browser (SignonForm.jsp)
  participant S as Stripes Dispatcher
  participant AA as AccountActionBean (session)
  participant CAS as CatalogService
  participant AS as AccountService
  participant AM as AccountMapper
  participant PM as ProductMapper
  participant DB as HSQLDB
  participant Sess as HTTP Session

  U->>B: Enter username/password, submit signon
  B->>S: POST /actions/Account.action?signon
  S->>AA: signon()
  AA->>AS: getAccount(username,password)
  AS->>AM: getAccountByUsernameAndPassword(u,p)
  AM->>DB: SELECT JOIN SIGNON/ACCOUNT/PROFILE/BANNER
  DB-->>AM: Account row or empty
  AM-->>AS: Account|null
  AS-->>AA: Account|null
  alt Authentication failure
    AA-->>S: Forward to SignonForm.jsp with message
    S-->>B: 200 SignonForm.jsp
  else Success
    AA->>CAS: getProductListByCategory(account.favouriteCategoryId)
    CAS->>PM: getProductListByCategory(catId)
    PM->>DB: SELECT PRODUCT WHERE CATEGORY=?
    DB-->>PM: Product[]
    PM-->>CAS: Product[]
    CAS-->>AA: Product[]
    AA->>Sess: Put "accountBean" and AA (session-scoped)
    AA-->>S: RedirectResolution to /actions/Catalog.action
    S-->>B: 302 Redirect
    B->>S: GET /actions/Catalog.action
    S-->>B: 200 Main.jsp (Welcome ...)
  end
```
- Purpose: Authenticate user and initialize personalized state (favorites, session).
- Triggers: POST to /actions/Account.action?signon from SignonForm.jsp.
- Communication patterns:
  - Synchronous HTTP request/response.
  - Synchronous read via AccountService -> AccountMapper (SELECT).
  - On success, read via CatalogService -> ProductMapper (SELECT).
  - Session mutation: stores accountBean and ActionBean in HTTP session.
  - No transaction (reads only). No events.

---

## WF2: New Account (AccountActionBean.newAccount)
```mermaid
sequenceDiagram
  autonumber
  participant U as User
  participant B as Browser (NewAccountForm.jsp)
  participant S as Stripes Dispatcher
  participant AA as AccountActionBean (session)
  participant AS as AccountService (@Transactional)
  participant AM as AccountMapper
  participant CAS as CatalogService
  participant PM as ProductMapper
  participant DB as HSQLDB
  participant TX as Spring TxManager
  participant Sess as HTTP Session

  U->>B: Fill account details, submit newAccount
  B->>S: POST /actions/Account.action?newAccount
  S->>AA: newAccount()
  AA->>TX: Begin transaction
  TX->>AS: insertAccount(account)
  AS->>AM: insertSignon(account)
  AM->>DB: INSERT SIGNON
  DB-->>AM: OK
  AS->>AM: insertAccount(account)
  AM->>DB: INSERT ACCOUNT
  DB-->>AM: OK
  AS->>AM: insertProfile(account)
  AM->>DB: INSERT PROFILE
  DB-->>AM: OK
  AS-->>TX: Return
  TX-->>DB: COMMIT
  TX-->>AA: Return
  AA->>AS: getAccount(username)  // reload
  AS->>AM: getAccountByUsername(u)
  AM->>DB: SELECT JOIN
  DB-->>AM: Account
  AM-->>AS: Account
  AS-->>AA: Account
  AA->>CAS: getProductListByCategory(favouriteCategoryId)
  CAS->>PM: getProductListByCategory(cat)
  PM->>DB: SELECT PRODUCTS
  DB-->>PM: Product[]
  PM-->>CAS: Product[]
  CAS-->>AA: Product[]
  AA->>Sess: Put "accountBean", authenticated=true
  AA-->>S: Redirect to /actions/Catalog.action
  S-->>B: 302 then 200 Main.jsp
```
- Purpose: Create a new account across SIGNON, ACCOUNT, PROFILE atomically.
- Triggers: POST to /actions/Account.action?newAccount.
- Communication patterns:
  - Synchronous HTTP.
  - Single ACID transaction (three INSERTs).
  - Follow-up reads to reload account and favorites.
  - No events.

---

## WF3: Browse Catalog (Categories -> Products -> Items)
```mermaid
sequenceDiagram
  autonumber
  participant U as User
  participant B as Browser (JSP)
  participant S as Stripes Dispatcher
  participant CA as CatalogActionBean
  participant CS as CatalogService
  participant CM as CategoryMapper
  participant PM as ProductMapper
  participant IM as ItemMapper
  participant DB as HSQLDB

  U->>B: Click category
  B->>S: GET /actions/Catalog.action?viewCategory&categoryId=CAT
  S->>CA: viewCategory()
  CA->>CS: getProductListByCategory(CAT)
  CS->>PM: getProductListByCategory(CAT)
  PM->>DB: SELECT PRODUCTS WHERE CATEGORY=CAT
  DB-->>PM: Product[]
  PM-->>CS: Product[]
  CS-->>CA: Product[]
  CA->>CS: getCategory(CAT)
  CS->>CM: getCategory(CAT)
  CM->>DB: SELECT CATEGORY
  DB-->>CM: Category
  CM-->>CS: Category
  CS-->>CA: Category
  CA-->>S: Forward to Category.jsp

  U->>B: Click product
  B->>S: GET /actions/Catalog.action?viewProduct&productId=PID
  S->>CA: viewProduct()
  CA->>CS: getItemListByProduct(PID)
  CS->>IM: getItemListByProduct(PID)
  IM->>DB: SELECT ITEM JOIN PRODUCT WHERE PRODUCTID=PID
  DB-->>IM: Item[]
  IM-->>CS: Item[]
  CS-->>CA: Item[]
  CA->>CS: getProduct(PID)
  CS->>PM: getProduct(PID)
  PM->>DB: SELECT PRODUCT
  DB-->>PM: Product
  PM-->>CS: Product
  CS-->>CA: Product
  CA-->>S: Forward to Product.jsp

  U->>B: Click item
  B->>S: GET /actions/Catalog.action?viewItem&itemId=IID
  S->>CA: viewItem()
  CA->>CS: getItem(IID)
  CS->>IM: getItem(IID)
  IM->>DB: SELECT ITEM JOIN INVENTORY+PRODUCT WHERE ITEMID=IID
  DB-->>IM: Item(+quantity)
  IM-->>CS: Item
  CS-->>CA: Item
  CA-->>S: Forward to Item.jsp
```
- Purpose: Navigate and read catalog entities.
- Triggers: GET viewCategory/viewProduct/viewItem.
- Communication patterns:
  - Synchronous reads (SELECTs) via mappers.
  - No transactions.
  - No events.
  - MyBatis 2nd-level cache may serve repeated reads within the same app instance.

---

## WF4: Search Products (CatalogActionBean.searchProducts)
```mermaid
sequenceDiagram
  autonumber
  participant U as User
  participant B as Browser (SearchProducts.jsp)
  participant S as Stripes Dispatcher
  participant CA as CatalogActionBean
  participant CS as CatalogService
  participant PM as ProductMapper
  participant DB as HSQLDB

  U->>B: Enter keyword(s), submit searchProducts
  B->>S: GET /actions/Catalog.action?searchProducts&keyword="large fish"
  S->>CA: searchProducts()
  alt Empty keyword
    CA-->>S: Forward to Error.jsp ("Please enter a keyword to search for, then press the search button.")
  else Valid keyword(s)
    CA->>CS: searchProductList("large fish")
    CS->>CS: Split tokens -> ["large","fish"]
    loop tokens
      CS->>PM: searchProductList(%token%)
      PM->>DB: SELECT PRODUCT WHERE lower(name) LIKE %token%
      DB-->>PM: Product[]
      PM-->>CS: Product[]
    end
    CS-->>CA: Concatenated Product[] (no dedupe)
    CA-->>S: Forward to SearchProducts.jsp
    S-->>B: 200 SearchProducts.jsp
  end
```
- Purpose: Full-text-like search across product names.
- Triggers: GET searchProducts with keyword.
- Communication patterns:
  - Multiple synchronous SELECTs, one per token.
  - No transaction, no dedupe logic, no events.

---

## WF5: Add to Cart and View Cart (CartActionBean.addItemToCart/viewCart)
```mermaid
sequenceDiagram
  autonumber
  participant U as User
  participant B as Browser (JSP)
  participant S as Stripes Dispatcher
  participant CTA as CartActionBean (session)
  participant CS as CatalogService
  participant IM as ItemMapper
  participant DB as HSQLDB
  participant Sess as HTTP Session

  U->>B: Click "Add to cart"
  B->>S: GET /actions/Cart.action?addItemToCart&workingItemId=IID
  S->>CTA: addItemToCart()
  alt Item already in session cart
    CTA->>CTA: incrementQuantityByItemId(IID)
  else New item
    CTA->>CS: getItem(IID)
    CS->>IM: getItem(IID)
    IM->>DB: SELECT ITEM JOIN INVENTORY+PRODUCT
    DB-->>IM: Item(+quantity)
    IM-->>CS: Item
    CS-->>CTA: Item
    CTA->>CS: isItemInStock(IID)
    CS->>IM: getInventoryQuantity(IID)
    IM->>DB: SELECT INVENTORY.QTY
    DB-->>IM: qty
    IM-->>CS: qty
    CS-->>CTA: qty>0?
    CTA->>CTA: addItem(Item, inStockFlag)
  end
  CTA->>Sess: Cart stored in session (updated)
  CTA-->>S: Forward to Cart.jsp
  S-->>B: 200 Cart.jsp

  U->>B: View cart
  B->>S: GET /actions/Cart.action?viewCart
  S->>CTA: viewCart()
  CTA-->>S: Forward to Cart.jsp
  S-->>B: 200 Cart.jsp
```
- Purpose: Build and maintain a session-scoped shopping cart.
- Triggers: GET addItemToCart/viewCart.
- Communication patterns:
  - Read-only queries to validate stock and load item.
  - No transaction (cart is in-memory session state).
  - No events.

---

## WF6: Update Cart Quantities / Remove Item (CartActionBean.updateCartQuantities/removeItemFromCart)
```mermaid
sequenceDiagram
  autonumber
  participant U as User
  participant B as Browser (Cart.jsp)
  participant S as Stripes Dispatcher
  participant CTA as CartActionBean (session)
  participant Sess as HTTP Session

  U->>B: Edit quantities (inputs named by itemId), submit updateCartQuantities
  B->>S: POST /actions/Cart.action?updateCartQuantities
  S->>CTA: updateCartQuantities()
  CTA->>CTA: setQuantityByItemId(itemId, qty) for each input
  CTA->>CTA: remove items if qty < 1
  CTA->>Sess: Persist updated cart in session
  CTA-->>S: Forward to Cart.jsp
  S-->>B: 200 Cart.jsp

  U->>B: Click "Remove" item
  B->>S: GET /actions/Cart.action?removeItemFromCart&workingItemId=IID
  S->>CTA: removeItemFromCart()
  alt workingItemId missing
    CTA-->>S: Forward to Error.jsp ("An order could not be found.")
  else ok
    CTA->>CTA: removeItemById(IID)
    CTA-->>S: Forward to Cart.jsp
  end
  S-->>B: 200 JSP
```
- Purpose: Adjust or remove items in the cart.
- Triggers: POST updateCartQuantities; GET removeItemFromCart.
- Communication patterns:
  - Pure session manipulation; no DB calls.
  - No transactions. No events.

---

## WF7: Checkout and Place Order (OrderActionBean.newOrderForm/newOrder + OrderService.insertOrder)
```mermaid
sequenceDiagram
  autonumber
  participant U as User
  participant B as Browser (Checkout/NewOrder/Confirm JSPs)
  participant S as Stripes Dispatcher
  participant OA as OrderActionBean (session)
  participant AA as AccountActionBean (session)
  participant CTA as CartActionBean (session)
  participant OS as OrderService (@Transactional)
  participant SM as SequenceMapper
  participant IM as ItemMapper
  participant OM as OrderMapper
  participant LM as LineItemMapper
  participant DB as HSQLDB
  participant TX as Spring TxManager

  U->>B: Proceed to checkout
  B->>S: GET /actions/Cart.action?checkOut
  S-->>B: 200 Checkout.jsp

  U->>B: Start new order
  B->>S: GET /actions/Order.action?newOrderForm
  S->>OA: newOrderForm()
  OA->>AA: Verify authenticated
  alt Not authenticated
    OA-->>S: Forward to Account.signonForm
    S-->>B: 200 SignonForm.jsp
  else Authenticated
    OA->>CTA: Read Cart
    OA->>OA: order.initOrder(account, cart)
    OA-->>S: Forward to NewOrderForm.jsp
    S-->>B: 200 NewOrderForm.jsp
  end

  U->>B: Submit order (possibly via Shipping/Confirm steps)
  B->>S: POST /actions/Order.action?newOrder
  S->>OA: newOrder(shippingAddressRequired, confirmed)
  alt shippingAddressRequired=true
    OA-->>S: Forward to ShippingForm.jsp
  else confirmed=false
    OA-->>S: Forward to ConfirmOrder.jsp
  else confirmed=true
    OA->>TX: Begin transaction
    TX->>OS: insertOrder(order)
    OS->>SM: getSequence({name:"ordernum"})
    SM->>DB: SELECT SEQUENCE WHERE NAME='ordernum'
    DB-->>SM: nextId or null
    alt Sequence found
      OS->>SM: updateSequence(nextId+1)
      SM->>DB: UPDATE SEQUENCE
      DB-->>SM: OK
      OS->>OS: orderId = nextId
      loop For each line item
        OS->>IM: updateInventoryQuantity({itemId, increment=qty})
        IM->>DB: UPDATE INVENTORY SET QTY = QTY - qty
        DB-->>IM: OK
      end
      OS->>OM: insertOrder(order)
      OM->>DB: INSERT ORDERS
      DB-->>OM: OK
      OS->>OM: insertOrderStatus(order)
      OM->>DB: INSERT ORDERSTATUS (LINENUM=ORDERID)
      DB-->>OM: OK
      loop For each line item
        OS->>LM: insertLineItem(line)
        LM->>DB: INSERT LINEITEM
        DB-->>LM: OK
      end
      OS-->>TX: Return
      TX-->>DB: COMMIT
      TX-->>OA: Return
      OA->>CTA: Clear cart
      OA-->>S: Forward to ViewOrder.jsp (Thank you)
      S-->>B: 200 ViewOrder.jsp
    else Sequence null (fatal)
      OS-->>TX: throw RuntimeException
      TX-->>DB: ROLLBACK
      TX-->>OA: Propagate exception
      OA-->>S: Unhandled -> Container error page (500)
      S-->>B: 500 Error
    end
  end
```
- Purpose: End-to-end order placement with atomic writes and inventory decrement.
- Triggers: newOrderForm (GET) then newOrder (POST) with step flags.
- Communication patterns:
  - One ACID transaction spanning:
    - Sequence read/update (ID allocation).
    - Inventory decrements (UPDATE).
    - Order header/status/line item inserts.
  - All synchronous; no events/outbox.
  - Risk: inventory can go negative under concurrency; no optimistic checks.
  - MyBatis mapper-level cache invalidated by DML on respective mappers (not distributed).

---

## WF8: List Orders and View Order Details (OrderActionBean.listOrders/viewOrder + OrderService.getOrder)
```mermaid
sequenceDiagram
  autonumber
  participant U as User
  participant B as Browser (ListOrders/ViewOrder JSPs)
  participant S as Stripes Dispatcher
  participant OA as OrderActionBean (session)
  participant AA as AccountActionBean (session)
  participant OS as OrderService (@Transactional read)
  participant OM as OrderMapper
  participant LM as LineItemMapper
  participant IM as ItemMapper
  participant DB as HSQLDB

  U->>B: My Orders
  B->>S: GET /actions/Order.action?listOrders
  S->>OA: listOrders()
  OA->>AA: Get current username
  OA->>OS: getOrdersByUsername(username)
  OS->>OM: getOrdersByUsername(username)
  OM->>DB: SELECT ORDERS JOIN ORDERSTATUS WHERE USERID=?
  DB-->>OM: Order[]
  OM-->>OS: Order[]
  OS-->>OA: Order[]
  OA-->>S: Forward to ListOrders.jsp
  S-->>B: 200 ListOrders.jsp

  U->>B: Click an Order
  B->>S: GET /actions/Order.action?viewOrder&orderId=OID
  S->>OA: viewOrder()
  OA->>OS: getOrder(OID)
  OS->>OM: getOrder(OID)
  OM->>DB: SELECT ORDER JOIN STATUS WHERE ORDERID=OID
  DB-->>OM: Order
  OM-->>OS: Order
  OS->>LM: getLineItemsByOrderId(OID)
  LM->>DB: SELECT LINEITEM WHERE ORDERID=OID
  DB-->>LM: LineItem[]
  LM-->>OS: LineItem[]
  loop enrich each LineItem
    OS->>IM: getItem(itemId)
    IM->>DB: SELECT ITEM JOIN INVENTORY+PRODUCT WHERE ITEMID=?
    DB-->>IM: Item(+current qty)
    IM-->>OS: Item
    OS->>OS: lineItem.item = Item; lineItem.total = unitPrice*qty
  end
  OS-->>OA: Order with line items and current inventory
  OA->>AA: Verify order.username == current username
  alt Authorized
    OA-->>S: Forward to ViewOrder.jsp
    S-->>B: 200 ViewOrder.jsp
  else Unauthorized
    OA-->>S: Forward to Error.jsp ("You may only view your own orders.")
    S-->>B: 200 Error.jsp
  end
```
- Purpose: Retrieve user order history and order details, enriching with current item inventory.
- Triggers: listOrders (GET) and viewOrder (GET).
- Communication patterns:
  - Synchronous reads, no writes.
  - getOrder uses a transactional read to assemble aggregate with multiple SELECTs.
  - Authorization check in ActionBean before rendering.

---

## WF9: Sign Out (AccountActionBean.signoff)
```mermaid
sequenceDiagram
  autonumber
  participant U as User
  participant B as Browser
  participant S as Stripes Dispatcher
  participant AA as AccountActionBean (session)
  participant Sess as HTTP Session

  U->>B: Click "Sign Out"
  B->>S: GET /actions/Account.action?signoff
  S->>AA: signoff()
  AA->>Sess: invalidate()
  AA->>AA: Clear state (account=null, authenticated=false)
  AA-->>S: Redirect to /actions/Catalog.action
  S-->>B: 302 then 200 Main.jsp
```
- Purpose: End user session and clear server-side state.
- Triggers: GET signoff.
- Communication patterns:
  - Session invalidation; no DB operations. Synchronous redirect. No events.

---

## Error and Recovery Patterns (selected)
```mermaid
sequenceDiagram
  autonumber
  participant U as User
  participant S as Stripes Dispatcher
  participant AA as AccountActionBean
  participant OA as OrderActionBean
  participant AS as AccountService
  participant AM as AccountMapper
  participant OS as OrderService
  participant SM as SequenceMapper
  participant DB as HSQLDB

  rect rgba(255,240,240,0.8)
  Note over AA: Invalid Sign-in
  U->>S: POST /Account.action?signon
  S->>AA: signon()
  AA->>AS: getAccount(u,p)
  AS->>AM: SELECT ...
  AM->>DB: SELECT
  DB-->>AM: no row
  AM-->>AS: null
  AS-->>AA: null
  AA-->>S: Forward SignonForm.jsp with error message
  end

  rect rgba(255,240,240,0.8)
  Note over OA: Order ID sequence missing (fatal)
  U->>S: POST /Order.action?newOrder (confirmed=true)
  S->>OA: newOrder()
  OA->>OS: insertOrder(order)
  OS->>SM: getSequence("ordernum")
  SM->>DB: SELECT SEQUENCE
  DB-->>SM: null
  SM-->>OS: null
  OS-->>S: RuntimeException(...)
  S-->>U: 500 Container error page (no recovery)
  end

  rect rgba(255,240,240,0.8)
  Note over OA: Unauthorized order view
  U->>S: GET /Order.action?viewOrder&orderId=OID
  S->>OA: viewOrder()
  OA->>OA: if (order.username != current.username)
  OA-->>S: Forward Error.jsp ("You may only view your own orders.")
  end
```
- Purpose: Capture main error flows and outcomes.
- Triggers: Invalid credentials; missing sequence row; cross-user order access.
- Communication patterns:
  - All synchronous; errors surfaced via forwards to Error.jsp or container 500.
  - Transaction behavior:
    - Sequence null -> transaction rollback via exception; no compensation needed.
  - No events.

---

## Communication Pattern Summary by Workflow
- WF1 Sign In:
  - HTTP POST, synchronous read; session mutation; no transaction.
- WF2 New Account:
  - HTTP POST; single transaction across SIGNON/ACCOUNT/PROFILE; synchronous.
- WF3 Browse Catalog:
  - HTTP GET; synchronous reads; optional MyBatis L2 cache hits; no transaction.
- WF4 Search Products:
  - HTTP GET; multiple synchronous SELECTs; no transaction.
- WF5 Add to Cart / View Cart:
  - HTTP GET; reads for item/inventory; cart stored in HTTP session; no transaction.
- WF6 Update Cart / Remove:
  - HTTP POST/GET; session-only operations; no DB; no transaction.
- WF7 Checkout and Place Order:
  - HTTP POST; single ACID transaction spanning inventory and order tables; synchronous; no messaging; potential race on inventory decrements.
- WF8 List Orders / View Order:
  - HTTP GET; transactional read to assemble aggregate; synchronous; authorization check in ActionBean.
- WF9 Sign Out:
  - HTTP GET; session invalidation; synchronous; no DB.

Notes on events and asynchrony:
- The monolith does not publish/consume messages; all flows are synchronous request/response with in-process calls.
- MyBatis second-level cache is per-mapper, in-memory, non-distributed; write operations on a mapper invalidate its local cache only.