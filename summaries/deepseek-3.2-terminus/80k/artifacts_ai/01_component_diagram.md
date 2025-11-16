```mermaid
graph TB
    %% Presentation Layer
    subgraph PresentationLayer[Presentation Layer]
        AA[AccountActionBean]
        CA[CartActionBean]
        CTA[CatalogActionBean]
        OA[OrderActionBean]
        JSP[JSP Views]
    end

    %% Business Layer
    subgraph BusinessLayer[Business Layer]
        AS[AccountService]
        CS[CatalogService]
        OS[OrderService]
        DM[Domain Models<br/>Account, Cart, Order, etc.]
    end

    %% Persistence Layer
    subgraph PersistenceLayer[Persistence Layer]
        AM[AccountMapper]
        CM[CategoryMapper]
        PM[ProductMapper]
        IM[ItemMapper]
        OM[OrderMapper]
        LIM[LineItemMapper]
        SM[SequenceMapper]
    end

    %% Database
    DB[(HSQLDB<br/>Database)]

    %% External
    WB[Web Browser]

    %% Interactions
    WB -->|HTTP Requests| AA
    WB -->|HTTP Requests| CA
    WB -->|HTTP Requests| CTA
    WB -->|HTTP Requests| OA
    
    AA -->|@SpringBean| AS
    CA -->|@SpringBean| CS
    CTA -->|@SpringBean| CS
    OA -->|@SpringBean| OS
    
    AS -->|Dependency| AM
    CS -->|Dependency| CM
    CS -->|Dependency| PM
    CS -->|Dependency| IM
    OS -->|Dependency| OM
    OS -->|Dependency| LIM
    OS -->|Dependency| SM
    
    AM -->|SQL Queries| DB
    CM -->|SQL Queries| DB
    PM -->|SQL Queries| DB
    IM -->|SQL Queries| DB
    OM -->|SQL Queries| DB
    LIM -->|SQL Queries| DB
    SM -->|SQL Queries| DB
    
    AA -->|Renders| JSP
    CA -->|Renders| JSP
    CTA -->|Renders| JSP
    OA -->|Renders| JSP
    JSP -->|HTML Response| WB

    %% Cross-layer dependencies
    AS -->|Uses| DM
    CS -->|Uses| DM
    OS -->|Uses| DM
    AM -->|Maps to| DM
    CM -->|Maps to| DM
    PM -->|Maps to| DM
    IM -->|Maps to| DM
    OM -->|Maps to| DM
    LIM -->|Maps to| DM
```

**Rationale:** The component boundaries follow a classic three-tier architecture with clear separation between presentation (Stripes ActionBeans), business logic (Spring Services), and persistence (MyBatis Mappers) layers. Communication patterns are strictly top-down through direct Java method calls with dependency injection, where ActionBeans delegate to Services which coordinate transactions and business rules before calling Mappers for database operations. The domain models serve as the data transfer objects that flow between all layers while maintaining clear responsibility boundaries.