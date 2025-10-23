```markdown
```mermaid
graph TB
    subgraph "User"
        User([fa:fa-user User])
    end

    subgraph "JPetStore Monolithic Application"
        direction TB
        
        subgraph "Presentation Layer (Stripes Framework)"
            ActionBeans["ActionBeans<br>(AccountActionBean, CatalogActionBean, etc.)"]
            JSPViews["JSP Views<br>(Cart.jsp, Order.jsp, etc.)"]
        end

        subgraph "Business Logic Layer (Spring Framework)"
            Services["Services<br>(AccountService, CatalogService, OrderService)"]
        end

        subgraph "Data Access Layer (MyBatis Framework)"
            Mappers["MyBatis Mappers<br>(AccountMapper, OrderMapper, etc.)"]
        end
        
    end

    subgraph "Database"
        DB[(HSQLDB)]
    end

    %% Flow of interactions
    User -- HTTP Request --> ActionBeans
    ActionBeans -- Calls business logic --> Services
    Services -- Manages transactions & uses --> Mappers
    Mappers -- Executes SQL --> DB
    DB -- Returns data --> Mappers
    Mappers -- Maps to Domain Objects --> Services
    Services -- Returns Domain Objects --> ActionBeans
    ActionBeans -- Forwards data to --> JSPViews
    JSPViews -- Renders HTML Response --> User

    %% Styling
    style ActionBeans fill:#cde4ff
    style JSPViews fill:#cde4ff
    style Services fill:#d5e8d4
    style Mappers fill:#fff2cc
```

The system is a classic three-tiered monolithic web application, where the presentation layer (Stripes ActionBeans/JSPs) is decoupled from the business logic (Spring Services) and data persistence (MyBatis Mappers). Communication follows a strict layered pattern: ActionBeans invoke Services, which in turn use Mappers to interact with the database, with Spring managing dependency injection and transactions. This structure cleanly separates concerns, with each layer having a distinct responsibility.
```