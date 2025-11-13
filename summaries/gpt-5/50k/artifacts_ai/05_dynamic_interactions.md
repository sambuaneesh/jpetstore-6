# 1) User Signon

```mermaid
sequenceDiagram
autonumber
participant U as User
participant B as Browser
participant SD as Stripes DispatcherServlet (*.action)
participant AAB as AccountActionBean (session)
participant ACS as AccountService
participant AM as AccountMapper (MyBatis)
participant CS as CatalogService
participant PM as ProductMapper (MyBatis)
database DB as HSQLDB (ACCOUNT/PROFILE/SIGNON/BANNERDATA/PRODUCT)

U->>B: Submit credentials
B->>SD: POST /actions/Account.action?signon
SD->>AAB: dispatch signon(username,password)
AAB->>ACS: getAccount(username,password)
ACS->>AM: getAccountByUsernameAndPassword
AM-->>DB: SELECT join SIGNON/ACCOUNT/PROFILE/BANNERDATA
DB-->>AM: Account row (or null)
AM-->>ACS: Account|null
ACS-->>AAB: Account|null

alt Auth success
  AAB->>AAB: account.password = null
  AAB->>CS: getProductListByCategory(account.favouriteCategoryId)
  CS->>PM: getProductListByCategory
  PM-->>DB: SELECT PRODUCT by category
  DB-->>PM: List<Product>
  PM-->>CS: List<Product>
  CS-->>AAB: myList
  AAB->>SD: set session attribute "accountBean" = this
  SD-->>B: 302 Redirect to /actions/Catalog.action (viewMain)
else Auth failed
  AAB->>SD: add error message
  SD-->>B: 200 SignonForm.jsp with errors
end
```

- Purpose/trigger: Authenticate the user and preload personalized “My List.”
- Communication patterns:
  - HTTP form POST routed by Stripes to AccountActionBean.signon
  - In-process calls: ActionBean -> AccountService/CatalogService
  - Synchronous DB reads via MyBatis mappers (JDBC)
  - No messaging; session state mutated on success
- Error handling:
  - Invalid credentials return SignonForm.jsp with error message
  - Password nulled before storing Account in session to reduce leakage


# 2) Catalog Browsing (Category → Product → Item)

```mermaid
sequenceDiagram
autonumber
participant U as User
participant B as Browser
participant SD as Stripes DispatcherServlet (*.action)
participant CAB as CatalogActionBean (session)
participant CS as CatalogService
participant CGM as CategoryMapper
participant PM as ProductMapper
participant IM as ItemMapper
database DB as HSQLDB (CATEGORY/PRODUCT/ITEM/INVENTORY)

U->>B: Click category
B->>SD: GET /actions/Catalog.action?viewCategory&categoryId={cat}
SD->>CAB: viewCategory(categoryId)
CAB->>CS: getProductListByCategory(categoryId)
CS->>PM: getProductListByCategory
PM-->>DB: SELECT PRODUCT by category
DB-->>PM: List<Product>
PM-->>CS: List<Product>
CS-->>CAB: List<Product>
CAB->>CS: getCategory(categoryId)
CS->>CGM: getCategory
CGM-->>DB: SELECT CATEGORY
DB-->>CGM: Category
CGM-->>CS: Category
CS-->>CAB: Category
SD-->>B: 200 Category.jsp

U->>B: Click product
B->>SD: GET /actions/Catalog.action?viewProduct&productId={pid}
SD->>CAB: viewProduct(productId)
CAB->>CS: getItemListByProduct(productId)
CS->>IM: getItemListByProduct
IM-->>DB: SELECT ITEM join PRODUCT
DB-->>IM: List<Item{Product}]
IM-->>CS: List<Item>
CS-->>CAB: List<Item>
CAB->>CS: getProduct(productId)
CS->>PM: getProduct
PM-->>DB: SELECT PRODUCT
DB-->>PM: Product
PM-->>CS: Product
CS-->>CAB: Product
SD-->>B: 200 Product.jsp

U->>B: Click item
B->>SD: GET /actions/Catalog.action?viewItem&itemId={iid}
SD->>CAB: viewItem(itemId)
CAB->>CS: getItem(itemId)
CS->>IM: getItem
IM-->>DB: SELECT ITEM join PRODUCT + INVENTORY (qty)
DB-->>IM: Item{Product, quantity}
IM-->>CS: Item
CS-->>CAB: Item
SD-->>B: 200 Item.jsp (shows stock and price)
```

- Purpose/trigger: Browse categories, products, and item detail with current inventory.
- Communication patterns:
  - HTTP GET routed to CatalogActionBean events
  - Synchronous service → mapper → DB reads
  - MyBatis second-level cache may serve hot reads
- Error handling:
  - If invalid IDs, mappers return null; JSPs may render empty state or error (default Stripes error page path available)


# 3) Product Search

```mermaid
sequenceDiagram
autonumber
participant U as User
participant B as Browser
participant SD as Stripes DispatcherServlet (*.action)
participant CAB as CatalogActionBean (session)
participant CS as CatalogService
participant PM as ProductMapper
database DB as HSQLDB (PRODUCT)

U->>B: Enter keyword(s) and submit
B->>SD: GET/POST /actions/Catalog.action?searchProducts&keyword="gold fish"
SD->>CAB: searchProducts(keyword)

alt Empty keyword
  CAB->>SD: forward to Error.jsp with message
  SD-->>B: 200 Error.jsp
else Non-empty
  CAB->>CS: searchProductList("gold fish")
  loop For each token t in ["gold","fish"]
    CS->>PM: searchProductList("%t%")
    PM-->>DB: SELECT PRODUCT WHERE lower(name) LIKE :pattern
    DB-->>PM: List<Product>
    PM-->>CS: List<Product> (concatenated, no dedup)
  end
  CS-->>CAB: Aggregated List<Product>
  SD-->>B: 200 SearchProducts.jsp
end
```

- Purpose/trigger: Find products by keyword(s).
- Communication patterns:
  - HTTP GET/POST to CatalogActionBean.searchProducts
  - Synchronous loop of DB queries (one per token)
  - No deduplication across token result sets
- Error handling:
  - Empty keyword → Error.jsp


# 4) Cart Management (Add, Update, Remove)

```mermaid
sequenceDiagram
autonumber
participant U as User
participant B as Browser
participant SD as Stripes DispatcherServlet (*.action)
participant CTB as CartActionBean (session; holds Cart)
participant CS as CatalogService
participant IM as ItemMapper
database DB as HSQLDB (ITEM/INVENTORY)

U->>B: Click "Add to Cart"
B->>SD: POST /actions/Cart.action?addItemToCart&workingItemId={iid}
SD->>CTB: addItemToCart(workingItemId)
alt Item already in cart
  CTB->>CTB: cart.incrementQuantityByItemId(iid)
  SD-->>B: 200 Cart.jsp
else New item
  CTB->>CS: isItemInStock(iid)
  CS->>IM: getInventoryQuantity(iid)
  IM-->>DB: SELECT INVENTORY.qty
  DB-->>IM: qty
  IM-->>CS: qty
  CS-->>CTB: inStock = qty > 0
  CTB->>CS: getItem(iid)
  CS->>IM: getItem(iid) (joins PRODUCT + INVENTORY)
  IM-->>DB: SELECT item/product/inventory
  DB-->>IM: Item{Product, quantity}
  IM-->>CS: Item
  CS-->>CTB: Item
  CTB->>CTB: cart.addItem(item, inStock)
  SD-->>B: 200 Cart.jsp
end

U->>B: Update quantities
B->>SD: POST /actions/Cart.action?updateCartQuantities (form itemId->qty)
SD->>CTB: updateCartQuantities(params)
alt qty < 1
  CTB->>CTB: cart.removeItemById(itemId)
else qty >= 1
  CTB->>CTB: cart.setQuantityByItemId(itemId, qty)
end
SD-->>B: 200 Cart.jsp

U->>B: Remove item
B->>SD: POST /actions/Cart.action?removeItemFromCart&workingItemId={iid}
SD->>CTB: removeItemFromCart(workingItemId)
alt Item present
  CTB->>CTB: cart.removeItemById(iid)
  SD-->>B: 200 Cart.jsp
else Item missing
  CTB->>SD: forward to Error.jsp
  SD-->>B: 200 Error.jsp
end
```

- Purpose/trigger: Build and maintain the shopping cart in session.
- Communication patterns:
  - HTTP POSTs to CartActionBean events
  - In-process cart mutation; DB reads only for add flow (stock check + item fetch)
  - No transactions; session-scoped state
- Error handling:
  - Removing a non-existent item forwards to Error.jsp


# 5) Checkout and Order Placement (Multi-step + Transactional Insert)

```mermaid
sequenceDiagram
autonumber
participant U as User
participant B as Browser
participant SD as Stripes DispatcherServlet (*.action)
participant OAB as OrderActionBean (session)
participant AAB as AccountActionBean (session)
participant CTB as CartActionBean (session)
participant ORS as OrderService
participant IM as ItemMapper
participant OM as OrderMapper
participant LM as LineItemMapper
participant SM as SequenceMapper
database DB as HSQLDB (SEQUENCE/INVENTORY/ORDERS/ORDERSTATUS/LINEITEM)

U->>B: Proceed to checkout
B->>SD: GET /actions/Order.action?newOrderForm
SD->>OAB: newOrderForm()
OAB->>AAB: retrieve account (must be authenticated)
OAB->>CTB: retrieve cart
OAB->>OAB: order.initOrder(account, cart) (copy addresses, line items, totals)
SD-->>B: 200 NewOrderForm.jsp

U->>B: Submit order form (may require shipping, then confirm)
B->>SD: POST /actions/Order.action?newOrder (fields)
SD->>OAB: newOrder()
alt shippingAddressRequired==true
  OAB->>SD: forward to ShippingForm.jsp
  SD-->>B: 200 ShippingForm.jsp
else not confirmed yet
  OAB->>SD: forward to ConfirmOrder.jsp
  SD-->>B: 200 ConfirmOrder.jsp
end

U->>B: Confirm order
B->>SD: POST /actions/Order.action?newOrder&confirmed=true
SD->>OAB: newOrder(confirmed=true)

rect rgba(220,255,220,0.5)
note over ORS: @Transactional
OAB->>ORS: insertOrder(order)
ORS->>SM: getSequence(name="ordernum")
SM-->>DB: SELECT SEQUENCE WHERE name='ordernum'
DB-->>SM: {nextId} or null
SM-->>ORS: nextId|null
alt Sequence present
  ORS->>ORS: order.orderId = nextId; set on each line
  ORS->>SM: updateSequence(name, nextId+1)
  SM-->>DB: UPDATE SEQUENCE set nextid=nextId+1
  loop For each lineItem
    ORS->>IM: updateInventoryQuantity({itemId, increment=quantity})
    IM-->>DB: UPDATE INVENTORY set qty = qty - increment
  end
  ORS->>OM: insertOrder(Order)
  OM-->>DB: INSERT ORDERS
  ORS->>OM: insertOrderStatus(Order)
  OM-->>DB: INSERT ORDERSTATUS
  loop Each lineItem
    ORS->>LM: insertLineItem(LineItem)
    LM-->>DB: INSERT LINEITEM
  end
  ORS-->>OAB: success
  OAB->>CTB: cart.clear()
  OAB->>SD: add success message
  SD-->>B: 200 ViewOrder.jsp
else Sequence null
  ORS-->>OAB: throws RuntimeException
  OAB->>SD: forward to Error.jsp
  SD-->>B: 200 Error.jsp
end
end
```

- Purpose/trigger: Convert the cart into a persisted order; decrement inventory; show confirmation.
- Communication patterns:
  - HTTP GET/POSTs across multi-step UI; final POST triggers transactional service
  - Synchronous DB transaction (Spring @Transactional) spanning:
    - Sequence read/update (select-then-update)
    - Inventory decrements (no guard against negative stock)
    - Inserts to ORDERS, ORDERSTATUS, LINEITEM
  - No async events or messages; all in-process
- Error handling:
  - Missing sequence → RuntimeException → Error.jsp
  - No built-in stock underflow check; DB accepts negative unless constrained
  - On success, cart is cleared in session


# 6) View Orders and Order Detail (Enrichment Read)

```mermaid
sequenceDiagram
autonumber
participant U as User
participant B as Browser
participant SD as Stripes DispatcherServlet (*.action)
participant OAB as OrderActionBean (session)
participant AAB as AccountActionBean (session)
participant ORS as OrderService (@Transactional read)
participant OM as OrderMapper
participant LM as LineItemMapper
participant IM as ItemMapper
database DB as HSQLDB (ORDERS/LINEITEM/ITEM/PRODUCT/INVENTORY)

U->>B: View my orders
B->>SD: GET /actions/Order.action?listOrders
SD->>OAB: listOrders()
OAB->>AAB: get username from session
OAB->>ORS: getOrdersByUsername(username)
ORS->>OM: getOrdersByUsername
OM-->>DB: SELECT ORDERS by userid
DB-->>OM: List<Order>
OM-->>ORS: List<Order>
ORS-->>OAB: List<Order>
SD-->>B: 200 ListOrders.jsp

U->>B: View order detail
B->>SD: GET /actions/Order.action?viewOrder&order.orderId={id}
SD->>OAB: viewOrder(orderId)
OAB->>AAB: ensure order.username == session.username
alt Authorized
  OAB->>ORS: getOrder(orderId)
  rect rgba(220,255,220,0.5)
  note over ORS: @Transactional (read assembly)
  ORS->>OM: getOrder(orderId)
  OM-->>DB: SELECT ORDERS join ORDERSTATUS
  DB-->>OM: Order
  OM-->>ORS: Order
  ORS->>LM: getLineItemsByOrderId(orderId)
  LM-->>DB: SELECT LINEITEM by orderId
  DB-->>LM: List<LineItem>
  LM-->>ORS: List<LineItem>
  loop For each lineItem
    ORS->>IM: getItem(itemId)
    IM-->>DB: SELECT ITEM join PRODUCT + INVENTORY
    DB-->>IM: Item{Product, quantity}
    IM-->>ORS: Item
    ORS->>ORS: lineItem.item = Item; Item.quantity = current inventory
  end
  end
  ORS-->>OAB: Enriched Order
  SD-->>B: 200 ViewOrder.jsp
else Unauthorized
  OAB->>SD: forward to Error.jsp
  SD-->>B: 200 Error.jsp
end
```

- Purpose/trigger: List user’s orders and view an order with current item snapshots and live inventory quantities.
- Communication patterns:
  - HTTP GETs to OrderActionBean
  - Synchronous reads across multiple tables; transactional read to assemble view model
  - Potential benefit from MyBatis second-level caches on hot queries
- Error handling:
  - Username mismatch → Error.jsp
  - Missing order/lines result in partial/null views per mapper return behavior


# 7) Cross-cutting Communication and Error/Recovery Patterns

- Synchronous vs asynchronous:
  - All interactions are synchronous and in-process within the monolith
  - No message brokers, queues, or domain events are emitted/consumed
- Communication forms:
  - HTTP requests to Stripes DispatcherServlet (*.action)
  - In-process method calls: ActionBeans → Services
  - JDBC interactions via MyBatis mappers to HSQLDB
- Transaction boundaries:
  - @Transactional on AccountService (account/profile/signon CRUD)
  - @Transactional on OrderService.getOrder (read assembly) and insertOrder (order creation + inventory decrement + sequence update)
- Caching:
  - MyBatis second-level cache per mapper reduces read load; still synchronous
- Error handling:
  - Validation or missing input (e.g., empty search) → Error.jsp with messages
  - Unauthorized access (viewOrder for another user) → Error.jsp
  - Runtime failure in sequence allocation during order insert → Error.jsp
  - Removing absent cart item → Error.jsp
- Recovery:
  - No retries or compensations; failures surface to UI
  - Session remains authoritative for cart and ActionBean state after errors