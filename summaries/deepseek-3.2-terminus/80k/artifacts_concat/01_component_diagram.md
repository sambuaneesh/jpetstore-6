```mermaid
graph TB
    subgraph "Web Layer (Stripes ActionBeans)"
        AA[AccountActionBean]
        CA[CartActionBean]
        CATA[CatalogActionBean]
        OA[OrderActionBean]
    end

    subgraph "Service Layer"
        AS[AccountService]
        CS[CatalogService]
        OS[OrderService]
    end

    subgraph "Data Access Layer (MyBatis Mappers)"
        AM[AccountMapper]
        CM[CategoryMapper]
        PM[ProductMapper]
        IM[ItemMapper]
        OM[OrderMapper]
        LIM[LineItemMapper]
        SM[SequenceMapper]
    end

    subgraph "Database"
        DB[(HSQLDB)]
    end

    AA --> AS
    CA --> OS
    CA --> CS
    CATA --> CS
    OA --> OS

    AS --> AM
    CS --> CM
    CS --> PM
    CS --> IM
    OS --> OM
    OS --> LIM
    OS --> IM
    OS --> SM

    AM --> DB
    CM --> DB
    PM --> DB
    IM --> DB
    OM --> DB
    LIM --> DB
    SM --> DB

    CA -.-> |Session State| SESS[HTTP Session]
    AA -.-> |Authentication| SESS
```

The component boundaries follow a classic three-tier architecture with clear separation between presentation (ActionBeans), business logic (Services), and data access (Mappers) layers. Communication patterns are strictly hierarchical with web components calling services, which in turn coordinate data access through MyBatis mappers. Session state is managed at the web layer for shopping cart and authentication, while transactional boundaries are enforced at the service layer for operations like order processing.