```markdown
1. ```mermaid
graph TB
    %% Web Layer Components
    subgraph WebLayer [Web Layer - Stripes ActionBeans]
        AA[AccountActionBean]
        CA[CatalogActionBean]
        CTA[CartActionBean]
        OA[OrderActionBean]
    end

    %% Service Layer Components
    subgraph ServiceLayer [Service Layer]
        AS[AccountService]
        CS[CatalogService]
        OS[OrderService]
    end

    %% Data Access Layer Components
    subgraph DataLayer [Data Access Layer - MyBatis Mappers]
        AM[AccountMapper]
        CM[CategoryMapper]
        PM[ProductMapper]
        IM[ItemMapper]
        OM[OrderMapper]
        LIM[LineItemMapper]
        SM[SequenceMapper]
        IVM[InventoryMapper]
    end

    %% Database
    DB[(HSQLDB Database)]

    %% Web Layer Interactions
    AA --> AS
    CA --> CS
    CTA --> CS
    OA --> OS
    OA --> AA
    OA --> CTA

    %% Service Layer Interactions
    AS --> AM
    CS --> CM
    CS --> PM
    CS --> IM
    OS --> OM
    OS --> LIM
    OS --> SM
    OS --> IVM

    %% Data Layer to Database
    AM --> DB
    CM --> DB
    PM --> DB
    IM --> DB
    OM --> DB
    LIM --> DB
    SM --> DB
    IVM --> DB

    %% Cross-service dependencies
    OS -.-> CS
    OS -.-> AS
```

2. The component boundaries follow a classic three-tier architecture with clear separation between presentation (ActionBeans), business logic (Services), and data access (Mappers). Communication patterns are synchronous in-process method calls with session-based state management for user context and shopping cart data. The architecture employs Spring-managed transactions at the service layer, particularly for order processing that spans multiple data operations.
```