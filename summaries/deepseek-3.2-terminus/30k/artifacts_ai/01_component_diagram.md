```mermaid
graph TB
    %% Web Layer Components
    subgraph WebLayer[Web Layer - Stripes MVC]
        AccountActionBean[AccountActionBean]
        CatalogActionBean[CatalogActionBean]
        CartActionBean[CartActionBean]
        OrderActionBean[OrderActionBean]
        JSPViews[JSP Views]
    end

    %% Service Layer Components
    subgraph ServiceLayer[Service Layer - Spring Managed]
        AccountService[AccountService]
        CatalogService[CatalogService]
        OrderService[OrderService]
    end

    %% Data Access Layer Components
    subgraph DataAccessLayer[Data Access Layer - MyBatis Mappers]
        AccountMapper[AccountMapper]
        CategoryMapper[CategoryMapper]
        ProductMapper[ProductMapper]
        ItemMapper[ItemMapper]
        OrderMapper[OrderMapper]
        LineItemMapper[LineItemMapper]
        SequenceMapper[SequenceMapper]
    end

    %% Database
    subgraph DatabaseLayer[Database - HSQLDB]
        AccountTables[Account Tables]
        CatalogTables[Catalog Tables]
        OrderTables[Order Tables]
    end

    %% Session Components
    subgraph SessionLayer[Session Management]
        ShoppingCart[Shopping Cart]
        UserSession[User Session]
    end

    %% Interaction Flows
    UserSession --> AccountActionBean
    UserSession --> CartActionBean
    UserSession --> OrderActionBean
    
    AccountActionBean --> AccountService
    CatalogActionBean --> CatalogService
    CartActionBean --> CatalogService
    CartActionBean --> OrderService
    OrderActionBean --> OrderService
    
    AccountService --> AccountMapper
    CatalogService --> CategoryMapper
    CatalogService --> ProductMapper
    CatalogService --> ItemMapper
    OrderService --> OrderMapper
    OrderService --> LineItemMapper
    OrderService --> SequenceMapper
    OrderService --> ItemMapper
    
    AccountMapper --> AccountTables
    CategoryMapper --> CatalogTables
    ProductMapper --> CatalogTables
    ItemMapper --> CatalogTables
    OrderMapper --> OrderTables
    LineItemMapper --> OrderTables
    SequenceMapper --> OrderTables
    
    ShoppingCart --> CartActionBean
    ShoppingCart --> OrderActionBean
    ShoppingCart --> OrderService

    %% Styling
    classDef webLayer fill:#e1f5fe
    classDef serviceLayer fill:#f3e5f5
    classDef dataLayer fill:#e8f5e8
    classDef database fill:#fff3e0
    classDef session fill:#fce4ec
    
    class WebLayer webLayer
    class ServiceLayer serviceLayer
    class DataAccessLayer dataLayer
    class DatabaseLayer database
    class SessionLayer session
```

The component boundaries follow a classic layered architecture with clear separation between presentation (Stripes ActionBeans/JSP), business logic (Spring services), and data access (MyBatis mappers). Communication patterns are strictly top-down with web components delegating to services, which coordinate transactions and business rules before accessing data through mapper interfaces. Session-based components like Shopping Cart maintain user state across the web layer while services remain stateless for scalability.