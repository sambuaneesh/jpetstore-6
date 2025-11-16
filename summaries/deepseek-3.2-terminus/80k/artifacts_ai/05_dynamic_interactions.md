```markdown
# MyBatis JPetStore 6 - Dynamic Interaction Flows

## 1. User Registration Workflow

### Workflow Description
**Purpose**: New user account creation with profile setup
**Triggers**: User accesses registration form and submits account details
**Communication Patterns**: Synchronous REST-style form submission, database transactions

```mermaid
sequenceDiagram
    participant User as Web User
    participant Stripes as AccountActionBean
    participant Service as AccountService
    participant Mapper as AccountMapper
    participant DB as Database

    User->>Stripes: POST /actions/Account.action (newAccount)
    Stripes->>Stripes: Validate form data
    Stripes->>Service: insertAccount(account)
    Service->>Mapper: getAccount(username)
    Mapper->>DB: SELECT FROM ACCOUNT
    DB-->>Mapper: Account result
    Mapper-->>Service: null (new user)
    
    alt Username Available
        Service->>Mapper: insertAccount(account)
        Mapper->>DB: INSERT INTO ACCOUNT
        Service->>Mapper: insertProfile(profile)
        Mapper->>DB: INSERT INTO PROFILE
        Service->>Mapper: insertSignon(credentials)
        Mapper->>DB: INSERT INTO SIGNON
        DB-->>Mapper: Success
        Mapper-->>Service: Success
        Service-->>Stripes: Success
        Stripes->>Stripes: Set user session
        Stripes-->>User: Redirect to main page
    else Username Exists
        Service-->>Stripes: Error - username exists
        Stripes-->>User: Show registration form with error
    end
```

## 2. User Authentication Workflow

### Workflow Description
**Purpose**: User login and session establishment
**Triggers**: User submits login credentials
**Communication Patterns**: Synchronous authentication, session management

```mermaid
sequenceDiagram
    participant User as Web User
    participant Stripes as AccountActionBean
    participant Service as AccountService
    participant Mapper as AccountMapper
    participant DB as Database

    User->>Stripes: POST /actions/Account.action (login)
    Stripes->>Stripes: Extract username/password
    Stripes->>Service: getAccount(username)
    Service->>Mapper: getAccountByUsername(username)
    Mapper->>DB: SELECT FROM ACCOUNT, PROFILE
    DB-->>Mapper: Account data
    Mapper-->>Service: Account object
    Service->>Mapper: getSignon(username)
    Mapper->>DB: SELECT FROM SIGNON
    DB-->>Mapper: Password hash
    Mapper-->>Service: Credentials
    
    alt Valid Credentials
        Service-->>Stripes: Account object
        Stripes->>Stripes: Create user session
        Stripes->>Stripes: Load user preferences
        Stripes->>Service: getBannerData(favcategory)
        Service->>Mapper: getBannerData(category)
        Mapper->>DB: SELECT FROM BANNERDATA
        DB-->>Mapper: Banner data
        Mapper-->>Service: BannerData
        Service-->>Stripes: BannerData
        Stripes-->>User: Redirect to main page with banner
    else Invalid Credentials
        Service-->>Stripes: Authentication failed
        Stripes-->>User: Show login form with error
    end
```

## 3. Product Browsing and Search Workflow

### Workflow Description
**Purpose**: Catalog navigation and product discovery
**Triggers**: User browses categories or searches for products
**Communication Patterns**: Synchronous database queries, cached category data

```mermaid
sequenceDiagram
    participant User as Web User
    participant Stripes as CatalogActionBean
    participant Service as CatalogService
    participant CategoryMapper as CategoryMapper
    participant ProductMapper as ProductMapper
    participant ItemMapper as ItemMapper
    participant DB as Database

    User->>Stripes: GET /actions/Catalog.action?viewCategory=CAT_ID
    Stripes->>Service: getCategory(categoryId)
    Service->>CategoryMapper: getCategory(categoryId)
    CategoryMapper->>DB: SELECT FROM CATEGORY
    DB-->>CategoryMapper: Category data
    CategoryMapper-->>Service: Category object
    
    Service->>Service: Check category cache
    alt Cache Hit
        Service-->>Stripes: Cached category
    else Cache Miss
        Service->>ProductMapper: getProductListByCategory(categoryId)
        ProductMapper->>DB: SELECT FROM PRODUCT
        DB-->>ProductMapper: Product list
        ProductMapper-->>Service: Product list
        Service->>Service: Update cache
        Service-->>Stripes: Category with products
    end
    
    Stripes->>User: Render category view
    
    User->>Stripes: GET /actions/Catalog.action?viewProduct=PROD_ID
    Stripes->>Service: getProduct(productId)
    Service->>ProductMapper: getProduct(productId)
    ProductMapper->>DB: SELECT FROM PRODUCT
    DB-->>ProductMapper: Product data
    ProductMapper-->>Service: Product object
    Service->>ItemMapper: getItemListByProduct(productId)
    ItemMapper->>DB: SELECT FROM ITEM
    DB-->>ItemMapper: Item list
    ItemMapper-->>Service: Item list
    Service-->>Stripes: Product with items
    Stripes->>User: Render product detail view
```

## 4. Shopping Cart Management Workflow

### Workflow Description
**Purpose**: Add/remove items from shopping cart with real-time inventory validation
**Triggers**: User adds items to cart or modifies quantities
**Communication Patterns**: Session-based cart storage, synchronous inventory checks

```mermaid
sequenceDiagram
    participant User as Web User
    participant Stripes as CartActionBean
    participant Service as CatalogService
    participant Mapper as ItemMapper
    participant DB as Database
    participant Session as HTTP Session

    User->>Stripes: POST /actions/Cart.action?addItemToCart=ITEM_ID
    Stripes->>Session: Get current cart
    Session-->>Stripes: Cart object
    
    Stripes->>Service: getItem(itemId)
    Service->>Mapper: getItem(itemId)
    Mapper->>DB: SELECT FROM ITEM, INVENTORY
    DB-->>Mapper: Item data with quantity
    Mapper-->>Service: Item object
    Service-->>Stripes: Item with stock info
    
    alt Item In Stock
        Stripes->>Stripes: Validate quantity
        Stripes->>Session: Add item to cart
        Session-->>Stripes: Updated cart
        Stripes->>Stripes: Calculate total price
        Stripes-->>User: Show updated cart
    else Item Out of Stock
        Stripes-->>User: Show out-of-stock error
    end
    
    User->>Stripes: POST /actions/Cart.action?updateCartQuantities
    Stripes->>Session: Get current cart
    Session-->>Stripes: Cart object
    loop For each cart item
        Stripes->>Service: isItemInStock(itemId)
        Service->>Mapper: getInventoryQuantity(itemId)
        Mapper->>DB: SELECT FROM INVENTORY
        DB-->>Mapper: Current quantity
        Mapper-->>Service: Quantity
        Service-->>Stripes: Stock status
        Stripes->>Stripes: Update quantity or remove if unavailable
    end
    Stripes->>Session: Update cart
    Stripes-->>User: Show updated cart
```

## 5. Order Processing Workflow

### Workflow Description
**Purpose**: Complete order creation with inventory updates and transaction management
**Triggers**: User proceeds to checkout from shopping cart
**Communication Patterns**: Database transactions, synchronous order processing, inventory updates

```mermaid
sequenceDiagram
    participant User as Web User
    participant Stripes as OrderActionBean
    participant Service as OrderService
    participant SequenceMapper as SequenceMapper
    participant OrderMapper as OrderMapper
    participant LineItemMapper as LineItemMapper
    participant ItemMapper as ItemMapper
    participant DB as Database
    participant Session as HTTP Session

    User->>Stripes: POST /actions/Order.action?newOrderForm
    Stripes->>Session: Get cart and user info
    Session-->>Stripes: Cart and Account data
    Stripes-->>User: Show order form
    
    User->>Stripes: POST /actions/Order.action?confirmOrder
    Stripes->>Session: Get cart and shipping info
    Session-->>Stripes: Complete order data
    
    Stripes->>Service: insertOrder(order)
    
    Service->>SequenceMapper: getSequence("ordernum")
    SequenceMapper->>DB: SELECT FROM SEQUENCE
    DB-->>SequenceMapper: Current sequence
    SequenceMapper-->>Service: Sequence object
    
    Service->>Service: Begin Transaction
    
    loop For each line item in order
        Service->>ItemMapper: getItem(itemId)
        ItemMapper->>DB: SELECT FROM ITEM, INVENTORY
        DB-->>ItemMapper: Item with quantity
        ItemMapper-->>Service: Item object
        
        alt Sufficient Inventory
            Service->>ItemMapper: updateInventory(itemId, newQuantity)
            ItemMapper->>DB: UPDATE INVENTORY
            DB-->>ItemMapper: Success
        else Insufficient Inventory
            Service->>Service: Rollback Transaction
            Service-->>Stripes: Error - insufficient stock
            Stripes-->>User: Show stock error
        end
    end
    
    Service->>OrderMapper: insertOrder(order)
    OrderMapper->>DB: INSERT INTO ORDERS
    DB-->>OrderMapper: Success
    
    Service->>OrderMapper: insertOrderStatus(order)
    OrderMapper->>DB: INSERT INTO ORDERSTATUS
    DB-->>OrderMapper: Success
    
    loop For each line item
        Service->>LineItemMapper: insertLineItem(lineItem)
        LineItemMapper->>DB: INSERT INTO LINEITEM
        DB-->>LineItemMapper: Success
    end
    
    Service->>SequenceMapper: updateSequence("ordernum")
    SequenceMapper->>DB: UPDATE SEQUENCE
    DB-->>SequenceMapper: Success
    
    Service->>Service: Commit Transaction
    Service-->>Stripes: Order confirmation
    
    Stripes->>Session: Clear shopping cart
    Stripes-->>User: Show order confirmation page
```

## 6. Product Search Workflow

### Workflow Description
**Purpose**: Multi-keyword product search with relevance ranking
**Triggers**: User enters search terms in search box
**Communication Patterns**: Database wildcard queries, result aggregation

```mermaid
sequenceDiagram
    participant User as Web User
    participant Stripes as CatalogActionBean
    participant Service as CatalogService
    participant Mapper as ProductMapper
    participant DB as Database

    User->>Stripes: GET /actions/Catalog.action?searchProducts=keywords
    Stripes->>Service: searchProductList(keywords)
    Service->>Service: Split keywords by whitespace
    
    loop For each keyword
        Service->>Mapper: searchProductList("%" + keyword + "%")
        Mapper->>DB: SELECT FROM PRODUCT WHERE LOWER(name) LIKE pattern
        DB-->>Mapper: Product results
        Mapper-->>Service: Product list
    end
    
    Service->>Service: Aggregate and deduplicate results
    Service->>Service: Sort by relevance (keyword frequency)
    Service-->>Stripes: Sorted product list
    
    Stripes->>User: Render search results page
```

## 7. Inventory Management Workflow

### Workflow Description
**Purpose**: Real-time stock validation and inventory updates
**Triggers**: Cart operations and order processing
**Communication Patterns**: Synchronous stock checks, transactional updates

```mermaid
sequenceDiagram
    participant Cart as CartActionBean
    participant Order as OrderService
    participant Catalog as CatalogService
    participant Mapper as ItemMapper
    participant DB as Database

    Cart->>Catalog: isItemInStock(itemId)
    Catalog->>Mapper: getInventoryQuantity(itemId)
    Mapper->>DB: SELECT qty FROM INVENTORY WHERE itemid=?
    DB-->>Mapper: Current quantity
    Mapper-->>Catalog: Quantity
    Catalog-->>Cart: Boolean (quantity > 0)
    
    Order->>Mapper: getItem(itemId)
    Mapper->>DB: SELECT FROM ITEM, INVENTORY
    DB-->>Mapper: Item with stock data
    Mapper-->>Order: Item object
    
    Order->>Mapper: updateInventory(decrement)
    Mapper->>DB: UPDATE INVENTORY SET qty=qty-? WHERE itemid=?
    DB-->>Mapper: Rows affected
    
    alt Update Successful
        Mapper-->>Order: Success
    else Concurrent Modification
        Mapper-->>Order: Failure
        Order->>Order: Retry or rollback
    end
```

## 8. Error Handling and Recovery Patterns

### Workflow Description
**Purpose**: Graceful error handling across different failure scenarios
**Triggers**: Database errors, validation failures, business rule violations
**Communication Patterns**: Exception propagation, transaction rollback, user feedback

```mermaid
sequenceDiagram
    participant User as Web User
    participant Stripes as ActionBean
    participant Service as BusinessService
    participant Mapper as DataMapper
    participant DB as Database
    participant Spring as Spring Container

    User->>Stripes: HTTP Request
    
    alt Validation Error
        Stripes->>Stripes: Form validation
        Stripes-->>User: Show validation errors (immediate)
    else Business Logic Error
        Stripes->>Service: Business operation
        Service->>Service: Business rule validation
        Service-->>Stripes: BusinessException
        Stripes-->>User: Show business error message
    else Database Error
        Stripes->>Service: Data operation
        Service->>Mapper: Database call
        Mapper->>DB: SQL Operation
        DB-->>Mapper: SQLException
        Mapper-->>Service: DataAccessException
        Service-->>Stripes: ServiceException
        Stripes-->>User: Show generic error page
    else Transaction Failure
        Service->>Spring: @Transactional method
        Spring->>Service: Begin transaction
        Service->>Mapper: Multiple DB operations
        Mapper->>DB: SQL 1
        DB-->>Mapper: Success
        Mapper->>DB: SQL 2
        DB-->>Mapper: Constraint violation
        Mapper-->>Service: DataAccessException
        Service->>Spring: Rollback transaction
        Spring->>DB: ROLLBACK
        DB-->>Spring: Success
        Spring-->>Service: Transaction rolled back
        Service-->>Stripes: TransactionException
        Stripes-->>User: Show retry suggestion
    end
```
```