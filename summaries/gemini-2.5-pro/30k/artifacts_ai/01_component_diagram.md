```markdown
```mermaid
graph TB
    subgraph "User"
        Browser
    end

    subgraph "Presentation Layer (Stripes ActionBeans)"
        direction LR
        AAB[AccountActionBean]
        CAB[CatalogActionBean]
        CartAB[CartActionBean]
        OAB[OrderActionBean]
    end
    
    subgraph "Web Container"
        Session["HTTP Session (Stateful)"]
    end

    subgraph "Service Layer (Business Logic & Transactions)"
        direction LR
        AS[AccountService]
        CS[CatalogService]
        OS[OrderService]
    end

    subgraph "Persistence Layer (MyBatis Mappers)"
        direction LR
        AM[AccountMapper]
        CM[Catalog Mappers <br/>(Product, Category, Item)]
        OM[Order Mappers <br/>(Order, LineItem, Sequence)]
    end

    subgraph "Database"
        DB[(HSQLDB)]
    end

    %% Interactions
    Browser -- HTTP Requests --> AAB
    Browser -- HTTP Requests --> CAB
    Browser -- HTTP Requests --> CartAB
    Browser -- HTTP Requests --> OAB

    AAB <--> Session
    CartAB <--> Session["Manages Cart object in Session"]

    AAB -- "manages user" --> AS
    CAB -- "browses catalog" --> CS
    CartAB -- "initiates checkout" --> OAB
    OAB -- "places order" --> OS
    
    AS -- "uses" --> AM
    CS -- "uses" --> CM
    OS -- "uses" --> OM
    OS -- "<b>updates inventory (Transactional Coupling)</b>" --> CM

    AM -- "CRUD" --> DB
    CM -- "CRUD" --> DB
    OM -- "CRUD" --> DB
```

The diagram illustrates a classic monolithic, multi-layered architecture with distinct Presentation (Stripes ActionBeans), Service, and Persistence (MyBatis Mappers) layers. Communication is synchronous and in-process, flowing strictly top-down from web request handlers to the database, with the service layer acting as the transactional boundary. The most critical interaction is the tight, transactional coupling between the Order and Catalog domains, where `OrderService` directly calls `ItemMapper` to update inventory when an order is placed.
```