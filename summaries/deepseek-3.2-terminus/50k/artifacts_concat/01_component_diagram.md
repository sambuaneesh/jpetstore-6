```mermaid
graph TB
    %% Web Layer Components
    AA[AccountActionBean]
    CA[CatalogActionBean]
    CART[CartActionBean]
    OA[OrderActionBean]
    
    %% Service Layer Components
    AS[AccountService]
    CS[CatalogService]
    OS[OrderService]
    
    %% Data Access Layer Components
    AM[AccountMapper]
    CM[CategoryMapper]
    PM[ProductMapper]
    IM[ItemMapper]
    OM[OrderMapper]
    LM[LineItemMapper]
    SM[SequenceMapper]
    
    %% Database
    DB[(HSQLDB)]
    
    %% Web Layer to Service Layer
    AA --> AS
    CA --> CS
    CART --> CS
    OA --> OS
    
    %% Service Layer to Data Access Layer
    AS --> AM
    CS --> CM
    CS --> PM
    CS --> IM
    OS --> OM
    OS --> LM
    OS --> IM
    OS --> SM
    
    %% Data Access Layer to Database
    AM --> DB
    CM --> DB
    PM --> DB
    IM --> DB
    OM --> DB
    LM --> DB
    SM --> DB
    
    %% Cross-Service Dependencies
    OS -.-> AS
    OA -.-> AA
    OA -.-> CART
```

The component boundaries follow a traditional layered architecture with clear separation between web, service, and data access layers. Communication patterns are primarily synchronous in-process method calls, with the web layer (ActionBeans) delegating to service components that coordinate transactions across multiple data mappers. The architecture shows tight coupling between OrderService and CatalogService through shared ItemMapper usage, indicating a key integration point for microservice decomposition.