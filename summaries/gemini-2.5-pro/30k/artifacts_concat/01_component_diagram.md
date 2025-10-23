```markdown
```mermaid
graph TB
    subgraph "User Facing"
        Browser[Web Browser]
    end

    subgraph "Presentation Layer (Stripes ActionBeans)"
        AccAB["AccountActionBean\n(SessionScope)"]
        CatAB[CatalogActionBean]
        CartAB["CartActionBean\n(SessionScope)"]
        OrdAB[OrderActionBean]
    end

    subgraph "Business Logic Layer (Spring Services)"
        AccSvc[AccountService]
        CatSvc[CatalogService]
        OrdSvc[OrderService]
    end

    subgraph "Data Access Layer (MyBatis Mappers)"
        AccountMappers["Account Mappers\n(Account, Profile, Signon)"]
        CatalogMappers["Catalog Mappers\n(Category, Product, Item, Inventory)"]
        OrderMappers["Order Mappers\n(Order, LineItem, Sequence)"]
    end

    subgraph "Database"
        DB[(HSQLDB)]
    end

    %% Define Styles
    style OrdSvc fill:#f9f,stroke:#333,stroke-width:2px
    style CatalogMappers fill:#f9f,stroke:#333,stroke-width:2px

    %% User to Presentation
    Browser --> AccAB & CatAB & CartAB & OrdAB

    %% Presentation to Service
    AccAB --> AccSvc
    CatAB --> CatSvc
    CartAB --> CatSvc
    OrdAB --> OrdSvc

    %% Service to Data Access (Primary Domain)
    AccSvc --> AccountMappers
    CatSvc --> CatalogMappers
    OrdSvc --> OrderMappers

    %% Cross-Domain Coupling
    OrdSvc -- "Updates Inventory (Transactional)" --> CatalogMappers

    %% Data Access to DB
    AccountMappers --> DB
    CatalogMappers --> DB
    OrderMappers --> DB
```

The diagram illustrates a classic three-tier monolithic architecture, with components grouped logically by business domain (Account, Catalog, Order) within each layer. Communication is synchronous and in-process, flowing downwards from the Presentation layer to the Database. A critical cross-domain coupling exists where the `OrderService` directly modifies inventory data via the `CatalogMappers` within a single shared transaction, highlighting a tight bond between the Order and Catalog domains.
```