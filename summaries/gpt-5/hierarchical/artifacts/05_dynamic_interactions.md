## Workflow 1: Catalog browsing and search

- Purpose and triggers:
  - Users browse categories/products/items or search by keyword to discover products.
  - Triggered by HTTP GET to catalog endpoints (home, category page, product page, item page, search).
- Communication patterns:
  - Synchronous HTTP request/response (Browser → Stripes Dispatcher → ActionBean → JSP).
  - Intra-process synchronous calls (ActionBean → CatalogService).
  - Synchronous DB reads via MyBatis mappers (Category/Product/Item).
  - No asynchronous/event-driven messaging.
- Data flow highlights:
  - Reads category/product/item data; search tokenizes keywords and aggregates product results.
  - No session writes except transient browsing state (selected IDs, keyword).

```mermaid
sequenceDiagram
  autonumber
  actor U as User
  participant B as Browser
  participant FC as Stripes Dispatcher
  participant CAB as CatalogActionBean
  participant CAS as CatalogService
  participant CatM as CategoryMapper
  participant ProdM as ProductMapper
  participant ItemM as ItemMapper
  participant DB as Database
  participant V as JSP View

  U->>B: Navigate / click / submit search
  B->>FC: HTTP GET /catalog (category|product|item|search)
  FC->>CAB: Dispatch to event method

  alt Category page
    CAB->>CAS: getCategory(categoryId)
    CAS->>CatM: getCategory(categoryId)
    CatM->>DB: SELECT category
    DB-->>CatM: Category
    CatM-->>CAS: Category
    CAS-->>CAB: Category
    CAB->>CAS: getProductListByCategory(categoryId)
    CAS->>ProdM: getProductListByCategory(categoryId)
    ProdM->>DB: SELECT products
    DB-->>ProdM: Products[]
    ProdM-->>CAS: Products[]
    CAS-->>CAB: Products[]
    CAB-->>FC: Forward to /WEB-INF/jsp/catalog/Category.jsp
  else Product page
    CAB->>CAS: getProduct(productId)
    CAS->>ProdM: getProduct(productId)
    ProdM->>DB: SELECT product
    DB-->>ProdM: Product
    ProdM-->>CAS: Product
    CAS-->>CAB: Product
    CAB->>CAS: getItemListByProduct(productId)
    CAS->>ItemM: getItemListByProduct(productId)
    ItemM->>DB: SELECT items + join product
    DB-->>ItemM: Items[]
    ItemM-->>CAS: Items[]
    CAS-->>CAB: Items[]
    CAB-->>FC: Forward to /WEB-INF/jsp/catalog/Product.jsp
  else Item page
    CAB->>CAS: getItem(itemId)
    CAS->>ItemM: getItem(itemId)
    ItemM->>DB: SELECT item + join product
    DB-->>ItemM: Item
    ItemM-->>CAS: Item
    CAS-->>CAB: Item
    CAB-->>FC: Forward to /WEB-INF/jsp/catalog/Item.jsp
  else Search
    CAB->>CAB: tokenize(keyword) -> tokens
    loop For each token
      CAB->>CAS: searchProductList(token)
      CAS->>ProdM: searchProductList('%token%')
      ProdM->>DB: SELECT products LIKE
      DB-->>ProdM: Products[]
      ProdM-->>CAS: Products[]
      CAS-->>CAB: Products[] (concatenate)
    end
    CAB-->>FC: Forward to /WEB-INF/jsp/catalog/SearchProducts.jsp
  end

  FC-->>V: Server-side forward with model
  V-->>B: Rendered HTML
```


## Workflow 2: Add to cart and update quantities (with stock checks)

- Purpose and triggers:
  - Users add items to cart, remove items, or adjust quantities from cart view.
  - Triggered by HTTP POST/GET to cart endpoints.
- Communication patterns:
  - Synchronous HTTP request/response.
  - Intra-process synchronous calls (CartActionBean → CatalogService).
  - Synchronous DB reads for item details and stock checks (ItemMapper).
  - Session state mutation (Cart in HttpSession).
- Data flow highlights:
  - Reads item + inventory; writes cart lines (CartItem) into session; recalculates totals client-side in domain objects.

```mermaid
sequenceDiagram
  autonumber
  actor U as User
  participant B as Browser
  participant FC as Stripes Dispatcher
  participant CtAB as CartActionBean
  participant CAS as CatalogService
  participant ItemM as ItemMapper
  participant DB as Database
  participant S as HttpSession
  participant V as JSP View

  U->>B: Click "Add to Cart" / update quantities
  B->>FC: HTTP POST /cart (addItem|removeItem|updateQty)
  FC->>CtAB: Dispatch to event method

  alt Add item
    CtAB->>CAS: getItem(itemId)
    CAS->>ItemM: getItem(itemId)
    ItemM->>DB: SELECT item + product
    DB-->>ItemM: Item
    ItemM-->>CAS: Item
    CAS-->>CtAB: Item

    CtAB->>CAS: isItemInStock(itemId)
    CAS->>ItemM: getInventoryQuantity(itemId)
    ItemM->>DB: SELECT quantity
    DB-->>ItemM: qty
    ItemM-->>CAS: qty
    CAS-->>CtAB: inStock (qty > 0)

    CtAB->>S: Load Cart from session
    alt Item already in cart
      CtAB->>S: cart.incrementQuantity(itemId)
    else New item
      CtAB->>S: cart.addItem(Item, inStock)
    end
    CtAB-->>FC: Forward to /WEB-INF/jsp/cart/Cart.jsp

    note over CtAB,S: Cart subtotal/line totals computed in domain (BigDecimal)
  else Update quantities
    CtAB->>S: Load Cart from session
    loop For each line to update
      CtAB->>S: cart.setQuantityByItemId(itemId, newQty)
    end
    CtAB-->>FC: Forward to /WEB-INF/jsp/cart/Cart.jsp
  else Remove item
    CtAB->>S: cart.removeItemById(itemId)
    CtAB-->>FC: Forward to /WEB-INF/jsp/cart/Cart.jsp
  end

  FC-->>V: Server-side forward with cart model
  V-->>B: Rendered HTML

  alt Out of stock on add
    CtAB->>CtAB: addActionMessage("Item is currently out of stock")
    CtAB-->>FC: Forward to Cart.jsp
  end
```


## Workflow 3: User sign-on (authentication) and session management

- Purpose and triggers:
  - Authenticate users and load personalized context (recommended list).
  - Triggered by HTTP POST from sign-in form; logout via link/action.
- Communication patterns:
  - Synchronous HTTP request/response.
  - Intra-process synchronous calls (AccountActionBean → AccountService; CatalogService for recommendations).
  - Synchronous DB reads via AccountMapper and ProductMapper.
  - Session mutation for authentication state; session invalidation on sign-off.
- Data flow highlights:
  - Reads account + profile + signon; optionally reads product list by favorite category; writes AccountActionBean/auth flags to session.

```mermaid
sequenceDiagram
  autonumber
  actor U as User
  participant B as Browser
  participant FC as Stripes Dispatcher
  participant AAB as AccountActionBean
  participant ACS as AccountService
  participant CatS as CatalogService
  participant AccM as AccountMapper
  participant ProdM as ProductMapper
  participant DB as Database
  participant S as HttpSession
  participant V as JSP View

  U->>B: Submit Sign-in (username, password)
  B->>FC: HTTP POST /account/signon
  FC->>AAB: signon()

  AAB->>ACS: getAccount(username, password)
  ACS->>AccM: getAccountByUsernameAndPassword(u, p)
  AccM->>DB: SELECT account + profile + signon
  DB-->>AccM: Account
  AccM-->>ACS: Account (or null)
  ACS-->>AAB: Account (or null)

  alt Auth success
    AAB->>CatS: getProductListByCategory(account.favouriteCategoryId)
    CatS->>ProdM: getProductListByCategory(catId)
    ProdM->>DB: SELECT products
    DB-->>ProdM: Products[]
    ProdM-->>CatS: Products[]
    CatS-->>AAB: myList
    AAB->>S: Set authenticated=true, store AccountActionBean in session
    AAB-->>FC: Forward to /WEB-INF/jsp/catalog/Main.jsp
  else Auth failure
    AAB->>AAB: addActionMessage("Username or password invalid.")
    AAB-->>FC: Forward to /WEB-INF/jsp/account/Signon.jsp
  end

  FC-->>V: Forwarded JSP renders
  V-->>B: HTML response

  opt Sign-off
    U->>B: Click "Sign-out"
    B->>FC: HTTP GET /account/signoff
    FC->>AAB: signoff()
    AAB->>S: session.invalidate()
    AAB-->>FC: Redirect to /catalog/main
    FC-->>B: HTTP 302
  end
```


## Workflow 4: Checkout and place order (transactional with inventory updates)

- Purpose and triggers:
  - Guide users from cart to order submission, persist the order, adjust inventory atomically, and show confirmation.
  - Triggered by starting checkout, entering shipping, confirming, and submitting.
- Communication patterns:
  - Synchronous HTTP request/response.
  - Intra-process synchronous orchestration (OrderActionBean → OrderService).
  - Transactional DB writes via MyBatis (OrderMapper, LineItemMapper, ItemMapper, SequenceMapper) under Spring transaction manager.
  - No asynchronous events; all updates occur within a single transaction.
- Data flow highlights:
  - Reads and updates sequence for orderId.
  - Writes order header, status, line items; decrements inventory per line item.
  - On success, clears cart in session.

```mermaid
sequenceDiagram
  autonumber
  actor U as User
  participant B as Browser
  participant FC as Stripes Dispatcher
  participant OAB as OrderActionBean
  participant S as HttpSession
  participant ORS as OrderService
  participant TM as TransactionManager
  participant SeqM as SequenceMapper
  participant OrdM as OrderMapper
  participant LIM as LineItemMapper
  participant ItemM as ItemMapper
  participant DB as Database
  participant V as JSP View

  U->>B: Proceed to Checkout
  B->>FC: HTTP GET /order/newOrderForm
  FC->>OAB: newOrderForm()
  OAB->>S: Load Account + Cart
  OAB->>OAB: order.initOrder(account, cart)
  OAB-->>FC: Forward to /WEB-INF/jsp/order/NewOrderForm.jsp
  FC-->>V: Render form
  V-->>B: HTML

  U->>B: Enter shipping details, confirm
  B->>FC: HTTP POST /order/confirm
  FC->>OAB: confirmOrder()
  OAB-->>FC: Forward to /WEB-INF/jsp/order/ConfirmOrder.jsp
  FC-->>V: Render confirmation
  V-->>B: HTML

  U->>B: Submit Order
  B->>FC: HTTP POST /order/submit
  FC->>OAB: submitOrder()
  OAB->>ORS: insertOrder(order)

  ORS->>TM: beginTransaction()

  ORS->>SeqM: getSequence("ordernum")
  SeqM->>DB: SELECT nextId FROM SEQUENCE WHERE name='ordernum'
  DB-->>SeqM: Sequence or null
  SeqM-->>ORS: Sequence or null

  alt Sequence found
    ORS->>SeqM: updateSequence(newNextId)
    SeqM->>DB: UPDATE SEQUENCE SET nextId=nextId+1
    DB-->>SeqM: OK
    SeqM-->>ORS: OK
    ORS->>OrdM: insertOrder(order with generated orderId)
    OrdM->>DB: INSERT ORDER (...)
    DB-->>OrdM: OK
    ORS->>OrdM: insertOrderStatus(orderId, "P")
    OrdM->>DB: INSERT ORDERSTATUS (...)
    DB-->>OrdM: OK

    loop For each line item
      ORS->>LIM: insertLineItem(orderId, line)
      LIM->>DB: INSERT LINEITEM (...)
      DB-->>LIM: OK

      ORS->>ItemM: updateInventoryQuantity({itemId, decrement=qty})
      ItemM->>DB: UPDATE INVENTORY SET qty = qty - ?
      DB-->>ItemM: OK
    end

    ORS->>TM: commit()
    OAB->>S: Clear Cart
    OAB-->>FC: Forward to /WEB-INF/jsp/order/ViewOrder.jsp (confirmation)
    FC-->>V: Render confirmation with orderId
    V-->>B: HTML
  else Sequence missing
    ORS->>TM: rollback()
    ORS-->>OAB: throw RuntimeException("null (sequence) – cannot generate orderId")
    OAB-->>FC: Forward to /WEB-INF/jsp/common/Error.jsp
    FC-->>V: Render error
    V-->>B: HTML
  end

  alt Any DB failure during inserts/updates
    ORS->>TM: rollback()
    ORS-->>OAB: Exception
    OAB-->>FC: Forward to Error.jsp
  end
```


## Workflow 5: View order history and order details (aggregate hydration + authorization)

- Purpose and triggers:
  - Users review past orders and inspect a specific order’s details.
  - Triggered by navigation to order list or clicking an order ID.
- Communication patterns:
  - Synchronous HTTP request/response.
  - Intra-process synchronous calls (OrderActionBean → OrderService).
  - Synchronous DB reads via MyBatis (orders, line items, item details, inventory quantities).
  - Simple authorization check in controller (ownership enforcement).
- Data flow highlights:
  - getOrdersByUsername returns list of order headers.
  - getOrder hydrates: order header, line items, each item’s details, current inventory quantity for display.

```mermaid
sequenceDiagram
  autonumber
  actor U as User
  participant B as Browser
  participant FC as Stripes Dispatcher
  participant OAB as OrderActionBean
  participant ORS as OrderService
  participant OrdM as OrderMapper
  participant LIM as LineItemMapper
  participant ItemM as ItemMapper
  participant DB as Database
  participant S as HttpSession
  participant V as JSP View

  U->>B: Open "My Orders"
  B->>FC: HTTP GET /order/listOrders
  FC->>OAB: listOrders()
  OAB->>S: Get current username
  OAB->>ORS: getOrdersByUsername(username)
  ORS->>OrdM: getOrdersByUsername(username)
  OrdM->>DB: SELECT * FROM ORDERS WHERE username=?
  DB-->>OrdM: Orders[]
  OrdM-->>ORS: Orders[]
  ORS-->>OAB: Orders[]
  OAB-->>FC: Forward to /WEB-INF/jsp/order/ListOrders.jsp
  FC-->>V: Render
  V-->>B: HTML

  U->>B: Click orderId link
  B->>FC: HTTP GET /order/viewOrder?orderId=...
  FC->>OAB: viewOrder(orderId)
  OAB->>ORS: getOrder(orderId)
  ORS->>OrdM: getOrder(orderId)
  OrdM->>DB: SELECT * FROM ORDERS WHERE orderId=?
  DB-->>OrdM: Order
  OrdM-->>ORS: Order

  ORS->>LIM: getLineItemsByOrderId(orderId)
  LIM->>DB: SELECT * FROM LINEITEM WHERE orderId=?
  DB-->>LIM: LineItems[]
  LIM-->>ORS: LineItems[]

  loop For each LineItem
    ORS->>ItemM: getItem(itemId)
    ItemM->>DB: SELECT item + product
    DB-->>ItemM: Item
    ItemM-->>ORS: Item
    ORS->>ItemM: getInventoryQuantity(itemId)
    ItemM->>DB: SELECT qty FROM INVENTORY WHERE itemId=?
    DB-->>ItemM: qty
    ItemM-->>ORS: qty (for in-stock flag)
  end

  ORS-->>OAB: Hydrated Order (header + lines + items + inventory)

  alt Ownership OK
    OAB->>S: Get current username
    OAB->>OAB: assert(order.username == currentUser)
    OAB-->>FC: Forward to /WEB-INF/jsp/order/ViewOrder.jsp
  else Unauthorized
    OAB-->>FC: Forward to /WEB-INF/jsp/common/Error.jsp ("You may only view your own orders")
  end

  FC-->>V: Render
  V-->>B: HTML
```


## Workflow 6: Account registration and profile update (multi-table, transactional)

- Purpose and triggers:
  - Create a new user account or update existing profile/credentials.
  - Triggered by submitting New Account or Edit Account forms.
- Communication patterns:
  - Synchronous HTTP request/response.
  - Intra-process synchronous calls (AccountActionBean → AccountService).
  - Transactional DB writes across multiple tables (account, profile, signon).
- Data flow highlights:
  - Insert path: insert into all three tables; update path: update account, profile, and optionally signon if password present; reload account after persist.

```mermaid
sequenceDiagram
  autonumber
  actor U as User
  participant B as Browser
  participant FC as Stripes Dispatcher
  participant AAB as AccountActionBean
  participant ACS as AccountService
  participant TM as TransactionManager
  participant AccM as AccountMapper
  participant DB as Database
  participant S as HttpSession
  participant V as JSP View

  U->>B: Submit New Account / Edit Account
  B->>FC: HTTP POST /account/save
  FC->>AAB: newAccount() or editAccount()

  alt Create new account
    AAB->>ACS: insertAccount(Account)
    ACS->>TM: beginTransaction()
    ACS->>AccM: insertAccount(Account)
    AccM->>DB: INSERT ACCOUNT (...)
    DB-->>AccM: OK
    ACS->>AccM: insertProfile(Account)
    AccM->>DB: INSERT PROFILE (...)
    DB-->>AccM: OK
    ACS->>AccM: insertSignon(Account)
    AccM->>DB: INSERT SIGNON (...)
    DB-->>AccM: OK
    ACS->>TM: commit()
    ACS-->>AAB: OK
    AAB->>ACS: getAccount(username)
    ACS->>AccM: getAccountByUsername(username)
    AccM->>DB: SELECT account + profile + signon
    DB-->>AccM: Account
    AccM-->>ACS: Account
    ACS-->>AAB: Account (fresh)
    AAB-->>FC: Forward to /WEB-INF/jsp/account/EditAccountForm.jsp (success msg)
  else Update existing account
    AAB->>ACS: updateAccount(Account, maybe new password)
    ACS->>TM: beginTransaction()
    ACS->>AccM: updateAccount(Account)
    AccM->>DB: UPDATE ACCOUNT (...)
    DB-->>AccM: OK
    ACS->>AccM: updateProfile(Account)
    AccM->>DB: UPDATE PROFILE (...)
    DB-->>AccM: OK
    alt Password provided
      ACS->>AccM: updateSignon(Account)
      AccM->>DB: UPDATE SIGNON (...)
      DB-->>AccM: OK
    end
    ACS->>TM: commit()
    ACS-->>AAB: OK
    AAB-->>FC: Forward to /WEB-INF/jsp/account/EditAccountForm.jsp (success msg)
  end

  FC-->>V: Render
  V-->>B: HTML

  alt Any persistence error
    ACS->>TM: rollback()
    ACS-->>AAB: Exception
    AAB-->>FC: Forward to /WEB-INF/jsp/common/Error.jsp
  end
```