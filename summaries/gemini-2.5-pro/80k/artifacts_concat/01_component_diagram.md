```markdown
graph TB
    subgraph User["User Interface"]
        Browser
    end

    subgraph Presentation["Presentation Layer (Stripes ActionBeans)"]
        AccAB[AccountActionBean]
        CatAB[CatalogActionBean]
        CartAB[CartActionBean]
        OrdAB[OrderActionBean]
    end

    subgraph Business["Business Logic Layer (Spring Services)"]
        AccSvc[AccountService]
        CatSvc[CatalogService]
        OrdSvc[OrderService]
    end

    subgraph Data["Data Access Layer (MyBatis Mappers)"]
        AccMap[AccountMapper]
        CatMappers["Catalog Mappers<br/>(Category, Product, Item)"]
        OrdMappers["Order Mappers<br/>(Order, LineItem, Sequence)"]
    end

    subgraph Persistence["Database"]
        DB[(HSQLDB)]
    end

    %% User to Presentation
    Browser -- HTTP Requests --> CatAB
    Browser -- HTTP Requests --> AccAB
    Browser -- HTTP Requests --> CartAB
    Browser -- HTTP Requests --> OrdAB

    %% Presentation to Business
    AccAB -- calls --> AccSvc
    CatAB -- calls --> CatSvc
    CartAB -- calls --> CatSvc
    OrdAB -- calls --> OrdSvc
    AccAB -- calls --> CatSvc

    %% Business to Data
    AccSvc -- calls --> AccMap
    CatSvc -- calls --> CatMappers
    OrdSvc -- calls --> OrdMappers
    OrdSvc -- cross-domain call --> CatMappers

    %% Data to Database
    AccMap -- SQL --> DB
    CatMappers -- SQL --> DB
    OrdMappers -- SQL --> DB

    classDef layer fill:#f9f9f9,stroke:#333,stroke-width:2px;
    class Presentation,Business,Data,Persistence layer;
```

The architecture is a classic three-tier monolith with clear boundaries between the presentation (Stripes ActionBeans), business logic (Spring Services), and data access (MyBatis Mappers) layers. Communication follows a strict, layered pattern via synchronous, in-process method calls, but exhibits tight coupling, such as the `OrderService` directly invoking the `ItemMapper` to update inventory during checkout.
```