```markdown
```mermaid
graph TB
    subgraph User Interaction
        User[User Browser]
    end

    subgraph "Web Layer (Stripes ActionBeans)"
        User --> AccountActionBean["AccountActionBean"]
        User --> CatalogActionBean["CatalogActionBean"]
        User --> CartActionBean["CartActionBean"]
        User --> OrderActionBean["OrderActionBean"]
    end

    subgraph "Account Management"
        AccountActionBean --> AccountService
        AccountService[AccountService] --> AccountMapper[AccountMapper]
    end

    subgraph "Catalog & Inventory"
        CatalogActionBean --> CatalogService
        CatalogService[CatalogService] --> ItemMapper
        CatalogService --> ProductMapper
        CatalogService --> CategoryMapper
        ItemMapper[ItemMapper]
        ProductMapper[ProductMapper]
        CategoryMapper[CategoryMapper]
    end

    subgraph "Shopping Cart (Session-Scoped)"
        CartActionBean -- "fetches item data" --> CatalogService
    end

    subgraph "Order Management"
        OrderActionBean --> OrderService
        OrderService[OrderService] --> OrderMapper
        OrderService --> LineItemMapper
        OrderMapper[OrderMapper]
        LineItemMapper[LineItemMapper]
    end
    
    %% Critical cross-domain dependency
    OrderService -.->|transactionally updates inventory| ItemMapper

    subgraph Persistence Layer
        AccountMapper --> Database[(Database)]
        CategoryMapper --> Database
        ProductMapper --> Database
        ItemMapper --> Database
        OrderMapper --> Database
        LineItemMapper --> Database
    end

    style User fill:#f9f,stroke:#333,stroke-width:2px
```

The diagram illustrates a classic three-tier monolithic architecture, with components logically grouped into functional domains (Account, Catalog, Cart, Order) but deployed as a single unit. Communication is strictly synchronous via in-process Java method calls, flowing from Stripes ActionBeans in the presentation layer, through transactional Spring services in the business layer, to MyBatis mappers in the persistence layer. A critical tight coupling exists where the Order service makes a direct, transactional call to the Catalog's Item mapper to update inventory, highlighting a key challenge for service decomposition.
```