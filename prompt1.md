```mermaid
graph TB
    subgraph "Presentation Layer (Stripes & JSP)"
        Actions[Web Actions]
        Views[JSP Views]
    end

    subgraph "Service Layer (Spring)"
        Services[Business Services]
    end

    subgraph "Data Access Layer (MyBatis)"
        Mappers[Data Mappers]
        Database[(HSQLDB)]
    end

    Actions -- "invokes" --> Services
    Services -- "uses" --> Mappers
    Mappers -- "queries/updates" --> Database
    
    Views -- "submits to / renders" --> Actions

```

### Rationale

The system employs a classic layered monolithic architecture, distinctly separating concerns into presentation, business, and data access tiers. The Web Actions, built with the Stripes framework, handle user interactions and delegate all business logic to the Spring-managed Service Layer. This Service Layer, in turn, communicates with the database exclusively through MyBatis Mappers, enforcing a clean, unidirectional dependency flow and a clear separation of persistence logic.