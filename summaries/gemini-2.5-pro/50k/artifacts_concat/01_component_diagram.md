```markdown
```mermaid
graph TB
    subgraph "Presentation Layer (Stripes ActionBeans)"
        AccountAction["AccountActionBean"]
        CatalogAction["CatalogActionBean"]
        CartAction["CartActionBean"]
        OrderAction["OrderActionBean"]
    end

    subgraph "Business Logic Layer (Spring Services)"
        AccountService["AccountService"]
        CatalogService["CatalogService"]
        OrderService["OrderService"]
    end

    subgraph "Persistence Layer (MyBatis Mappers)"
        AccountMapper["AccountMapper"]
        CategoryMapper["CategoryMapper"]
        ProductMapper["ProductMapper"]
        ItemMapper["ItemMapper"]
        OrderMapper["OrderMapper"]
        LineItemMapper["LineItemMapper"]
        SequenceMapper["SequenceMapper"]
    end

    subgraph "Database"
        DB[(JPetStore DB)]
    end

    %% User Entry Point
    User[Browser User] --> AccountAction
    User --> CatalogAction
    User --> CartAction
    User --> OrderAction

    %% Presentation -> Service Layer
    AccountAction --> AccountService
    CatalogAction --> CatalogService
    CartAction -- "Fetches Item Details" --> CatalogService
    OrderAction --> OrderService

    %% Service -> Persistence Layer
    AccountService --> AccountMapper
    CatalogService --> CategoryMapper
    CatalogService --> ProductMapper
    CatalogService --> ItemMapper
    OrderService --> OrderMapper
    OrderService --> LineItemMapper
    OrderService --> SequenceMapper
    OrderService -- "Updates Inventory (Transactional)" --> ItemMapper

    %% Persistence -> Database
    AccountMapper --> DB
    CategoryMapper --> DB
    ProductMapper --> DB
    ItemMapper --> DB
    OrderMapper --> DB
    LineItemMapper --> DB
    SequenceMapper --> DB

    %% Style Definitions
    classDef presentation fill:#e6ffed,stroke:#333,stroke-width:1px
    classDef service fill:#d1e7ff,stroke:#333,stroke-width:1px
    classDef persistence fill:#fff0f0,stroke:#333,stroke-width:1px
    classDef db fill:#fcf8e3,stroke:#333,stroke-width:1px

    class AccountAction,CatalogAction,CartAction,OrderAction presentation
    class AccountService,CatalogService,OrderService service
    class AccountMapper,CategoryMapper,ProductMapper,ItemMapper,OrderMapper,LineItemMapper,SequenceMapper persistence
    class DB db
```

The diagram illustrates a classic three-tier monolithic architecture with distinct Presentation (ActionBeans), Business Logic (Services), and Persistence (Mappers) layers. Communication is synchronous, in-process method invocation, where web actions delegate to services which in turn orchestrate data access via mappers. A key interaction is the `OrderService` directly calling the `ItemMapper` to update inventory, creating a tight, transactional coupling between the order and catalog domains.
```