```markdown
```mermaid
graph TB
    subgraph "User Facing"
        User[Web Browser User]
    end

    subgraph "Presentation Layer (Stripes ActionBeans)"
        AccountAction[AccountActionBean]
        CatalogAction[CatalogActionBean]
        CartAction[CartActionBean]
        OrderAction[OrderActionBean]
    end

    subgraph "Business Logic Layer (Spring Services)"
        AccountService["AccountService<br/>(@Transactional)"]
        CatalogService["CatalogService<br/>(@Transactional)"]
        OrderService["OrderService<br/>(@Transactional)"]
    end

    subgraph "Data Access Layer (MyBatis Mappers)"
        AccountMapper[AccountMapper]
        CatalogMappers["Catalog Mappers<br/>(Category, Product)"]
        ItemMapper[ItemMapper]
        OrderMappers["Order Mappers<br/>(Order, LineItem)"]
        SequenceMapper[SequenceMapper]
    end

    subgraph "Data Store"
        Database[(HSQLDB)]
    end

    %% -- Interactions --
    User --> CatalogAction & CartAction & OrderAction & AccountAction

    %% Presentation -> Service
    AccountAction --> AccountService
    CatalogAction --> CatalogService
    CartAction --> CatalogService
    OrderAction --> OrderService

    %% Service -> Mapper
    AccountService --> AccountMapper
    CatalogService --> CatalogMappers & ItemMapper
    OrderService --> OrderMappers & SequenceMapper
    
    %% Cross-Domain Dependency Highlight
    OrderService -- "Updates Inventory" --> ItemMapper
    
    %% Mapper -> Database
    AccountMapper & CatalogMappers & ItemMapper & OrderMappers & SequenceMapper --> Database

    %% Styling for cross-domain call
    linkStyle 6 stroke:red,stroke-width:2px,stroke-dasharray: 5 5;
```

The architecture is a classic three-tier monolith where component boundaries align with distinct layers: Presentation (Stripes ActionBeans), Business Logic (Spring Services), and Data Access (MyBatis Mappers). Communication is synchronous and in-process, flowing top-down from user actions to the database via direct method calls orchestrated by dependency injection. A key tight coupling is visible where the `OrderService` directly invokes the `ItemMapper` (from the catalog domain) to update inventory, demonstrating a cross-domain dependency at the persistence layer.
```