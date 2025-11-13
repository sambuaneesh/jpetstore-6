```mermaid
graph TB
  %% Component-level architecture (current monolith)
  subgraph Client
    Browser[User Browser]
  end

  subgraph Presentation[Presentation (Stripes MVC)]
    direction TB
    Stripes[Stripes Dispatcher (*.action)]
    AccountAB[AccountActionBean]
    CatalogAB[CatalogActionBean]
    CartAB[CartActionBean]
    OrderAB[OrderActionBean]
    JSP[JSP Views (JSTL/Stripes tags)]
    Session[(HTTP Session: account, cart, order)]
  end

  subgraph Service[Service Layer (Spring @Service)]
    direction TB
    AccountSvc[AccountService]
    CatalogSvc[CatalogService]
    OrderSvc[OrderService]
    Spring[Spring DI + Transactions]
  end

  subgraph Persistence[Persistence (MyBatis Mappers)]
    direction TB
    AccountMapper[AccountMapper]
    CategoryMapper[CategoryMapper]
    ProductMapper[ProductMapper]
    ItemMapper[ItemMapper]
    OrderMapper[OrderMapper]
    LineItemMapper[LineItemMapper]
    SequenceMapper[SequenceMapper]
  end

  subgraph DB[Database (Embedded HSQLDB)]
    direction TB
    AccountDB[(ACCOUNT, PROFILE, SIGNON, BANNERDATA)]
    CatalogDB[(CATEGORY, PRODUCT, ITEM, INVENTORY)]
    OrderDB[(ORDERS, ORDERSTATUS, LINEITEM)]
    SeqDB[(SEQUENCE)]
  end

  %% Flows
  Browser -->|HTTP requests (*.action)| Stripes
  Stripes --> AccountAB
  Stripes --> CatalogAB
  Stripes --> CartAB
  Stripes --> OrderAB

  AccountAB -.->|forward/redirect| JSP
  CatalogAB -.->|forward/redirect| JSP
  CartAB -.->|forward/redirect| JSP
  OrderAB -.->|forward/redirect| JSP

  AccountAB <--> Session
  CartAB <--> Session
  OrderAB <--> Session

  %% Web -> Services
  AccountAB -->|in-process call| AccountSvc
  CatalogAB -->|in-process call| CatalogSvc
  CartAB -->|in-process call| CatalogSvc
  OrderAB -->|in-process call| OrderSvc

  %% Spring cross-cutting
  Spring -.-> AccountSvc
  Spring -.-> CatalogSvc
  Spring -.-> OrderSvc

  %% Services -> Mappers
  AccountSvc --> AccountMapper
  CatalogSvc --> CategoryMapper
  CatalogSvc --> ProductMapper
  CatalogSvc --> ItemMapper
  OrderSvc --> OrderMapper
  OrderSvc --> LineItemMapper
  OrderSvc --> ItemMapper
  OrderSvc --> SequenceMapper

  %% Mappers -> DB tables
  AccountMapper --> AccountDB
  CategoryMapper --> CatalogDB
  ProductMapper --> CatalogDB
  ItemMapper --> CatalogDB
  OrderMapper --> OrderDB
  LineItemMapper --> OrderDB
  SequenceMapper --> SeqDB
```

Component boundaries follow a classic layered monolith: Stripes ActionBeans (presentation) orchestrate user flows and session-scoped state, domain-focused Spring services (Account, Catalog, Order) provide transactional facades, and MyBatis mappers encapsulate SQL per table family. Communication is synchronous and in-process: controllers call services via DI, services call mappers, and mappers access HSQLDB; notable cross-domain coupling includes OrderService’s use of ItemMapper for inventory adjustment and SequenceMapper for ID generation.