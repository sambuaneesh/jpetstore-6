```mermaid
graph TB

  subgraph Client
    Browser[Browser]
  end

  subgraph Web_MVC["Web/MVC Layer (Stripes ActionBeans)"]
    AbstractActionBean[AbstractActionBean (base)]
    AccountActionBean[AccountActionBean]
    CatalogActionBean[CatalogActionBean]
    CartActionBean[CartActionBean]
    OrderActionBean[OrderActionBean]
    JSPs[JSP Views (under WEB-INF)]
  end

  subgraph Session["HTTP Session (per user)"]
    SessionAccount[Account (session-scoped)]
    SessionCart[Cart (session-scoped)]
  end

  subgraph Service["Service Layer (Spring, transactional)"]
    AccountService[AccountService]
    CatalogService[CatalogService]
    OrderService[OrderService]
  end

  subgraph Persistence["Persistence Layer (MyBatis Mappers)"]
    AccountMapper[AccountMapper]
    CategoryMapper[CategoryMapper]
    ProductMapper[ProductMapper]
    ItemMapper[ItemMapper]
    OrderMapper[OrderMapper]
    LineItemMapper[LineItemMapper]
    SequenceMapper[SequenceMapper]
  end

  subgraph Domain["Domain Model (POJOs)"]
    Account[Account]
    Category[Category]
    Product[Product]
    Item[Item]
    Cart[Cart]
    CartItem[CartItem]
    Order[Order]
    LineItem[LineItem]
    Sequence[Sequence]
  end

  DB[(Relational Database)]

  %% Client ↔ Web
  Browser --> AccountActionBean
  Browser --> CatalogActionBean
  Browser --> CartActionBean
  Browser --> OrderActionBean
  AccountActionBean --> JSPs
  CatalogActionBean --> JSPs
  CartActionBean --> JSPs
  OrderActionBean --> JSPs
  JSPs --> Browser

  %% Inheritance (base action)
  AccountActionBean -.-> AbstractActionBean
  CatalogActionBean -.-> AbstractActionBean
  CartActionBean -.-> AbstractActionBean
  OrderActionBean -.-> AbstractActionBean

  %% Web ↔ Session
  AccountActionBean <--> SessionAccount
  CartActionBean <--> SessionCart
  OrderActionBean --> SessionAccount
  OrderActionBean --> SessionCart

  %% Web → Services
  AccountActionBean --> AccountService
  AccountActionBean --> CatalogService
  CatalogActionBean --> CatalogService
  CartActionBean --> CatalogService
  OrderActionBean --> OrderService

  %% Services → Mappers
  AccountService --> AccountMapper
  CatalogService --> CategoryMapper
  CatalogService --> ProductMapper
  CatalogService --> ItemMapper
  OrderService --> SequenceMapper
  OrderService --> OrderMapper
  OrderService --> LineItemMapper
  OrderService --> ItemMapper

  %% Mappers → DB
  AccountMapper --> DB
  CategoryMapper --> DB
  ProductMapper --> DB
  ItemMapper --> DB
  OrderMapper --> DB
  LineItemMapper --> DB
  SequenceMapper --> DB

  %% Domain sharing across layers
  Domain -.-> Web_MVC
  Domain -.-> Service
  Domain -.-> Persistence
```

The system follows a classic layered architecture: Stripes ActionBeans handle HTTP requests, session state (Account/Cart), and view forwarding, delegating business workflows to Spring-managed, transactional services. Services orchestrate operations (e.g., checkout, sequencing, inventory updates) and use MyBatis mappers for persistence against a relational database, while domain POJOs flow across all layers as shared data carriers. This separates concerns cleanly, keeping controllers thin, services cohesive, and persistence isolated.