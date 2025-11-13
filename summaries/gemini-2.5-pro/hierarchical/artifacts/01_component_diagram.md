```mermaid
graph TB
    subgraph "External Actors & Tests"
        User
        UI_Tests[UI Integration Tests<br>org.mybatis.jpetstore]
    end

    subgraph "Presentation Layer (Stripes MVC)"
        Actions[web.actions<br>ActionBeans<br>e.g., CartActionBean, OrderActionBean]
    end

    subgraph "Service Layer (Business Logic)"
        Services[service<br>AccountService, CatalogService, OrderService]
    end

    subgraph "Persistence Layer (MyBatis)"
        Mappers[mapper<br>AccountMapper, ProductMapper, OrderMapper]
    end

    subgraph "Domain Model (POJOs)"
        Domain[domain<br>Account, Product, Order, Cart]
    end
    
    subgraph "Data Store"
        Database[(Database)]
    end

    subgraph "Build Support"
        Build[MavenWrapperDownloader]
    end
    
    User -- HTTP Request --> Actions
    UI_Tests -- Simulates User --> Actions

    Actions -- Invokes Business Logic --> Services
    Services -- Orchestrates Persistence --> Mappers
    Mappers -- Executes SQL --> Database

    Actions -- Uses/Returns --> Domain
    Services -- Uses/Returns --> Domain
    Mappers -- Maps to/from --> Domain

    style Build fill:#f9f,stroke:#333,stroke-width:2px
```

The system employs a classic layered architecture with a clear separation of concerns. User requests are handled by the Presentation Layer (`ActionBeans`), which delegates business operations to the Service Layer. The Service Layer encapsulates core logic and orchestrates data access by invoking methods on the Persistence Layer (`Mappers`), with all layers using the shared `Domain` objects as the common data transfer model.