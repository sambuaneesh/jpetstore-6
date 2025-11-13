```mermaid
graph TB

  subgraph Client
    Browser[Web Browser]
  end

  subgraph Presentation (Servlet container: Stripes + JSP)
    StripesFilter[StripesFilter (with Spring integration)]
    Dispatcher[Stripes DispatcherServlet (*.action)]
    JSPs[JSP/JSTL Views]

    subgraph ActionBeans (session-scoped)
      CatalogAB[CatalogActionBean]
      AccountAB[AccountActionBean]
      CartAB[CartActionBean]
      OrderAB[OrderActionBean]
    end

    HttpSession[HTTP Session (cart, account, order state)]
  end

  subgraph Spring Service Layer
    AccountSvc[AccountService]
    CatalogSvc[CatalogService]
    OrderSvc[OrderService @Transactional]
    TxMgr[Spring DataSourceTransactionManager]
  end

  subgraph Persistence (MyBatis + MyBatis-Spring)
    SqlSess[SqlSessionFactory (MyBatis-Spring)]
    L2Cache[MyBatis 2nd-level cache (per mapper)]

    subgraph Mappers (DAO)
      AccountMapper[AccountMapper]
      CategoryMapper[CategoryMapper]
      ProductMapper[ProductMapper]
      ItemMapper[ItemMapper]
      OrderMapper[OrderMapper]
      LineItemMapper[LineItemMapper]
      SequenceMapper[SequenceMapper]
    end
  end

  subgraph Database
    HSQL[(Embedded HSQLDB)]
  end

  ContextLoader[Spring ContextLoaderListener]

  %% Request flow
  Browser -->|HTTP| StripesFilter -->|dispatch| Dispatcher
  Dispatcher -->|invoke| CatalogAB
  Dispatcher -->|invoke| AccountAB
  Dispatcher -->|invoke| CartAB
  Dispatcher -->|invoke| OrderAB

  %% View rendering
  CatalogAB -->|forward| JSPs
  AccountAB -->|forward| JSPs
  CartAB -->|forward| JSPs
  OrderAB -->|forward| JSPs

  %% Session state
  AccountAB <--> HttpSession
  CartAB <--> HttpSession
  OrderAB <--> HttpSession

  %% DI and wiring
  StripesFilter -.->|inject @SpringBean| CatalogAB
  StripesFilter -.->|inject @SpringBean| AccountAB
  StripesFilter -.->|inject @SpringBean| CartAB
  StripesFilter -.->|inject @SpringBean| OrderAB
  ContextLoader --> AccountSvc
  ContextLoader --> CatalogSvc
  ContextLoader --> OrderSvc
  ContextLoader --> TxMgr
  ContextLoader --> SqlSess
  ContextLoader --> AccountMapper
  ContextLoader --> CategoryMapper
  ContextLoader --> ProductMapper
  ContextLoader --> ItemMapper
  ContextLoader --> OrderMapper
  ContextLoader --> LineItemMapper
  ContextLoader --> SequenceMapper

  %% ActionBeans -> Services
  AccountAB --> AccountSvc
  AccountAB --> CatalogSvc
  CatalogAB --> CatalogSvc
  CartAB --> CatalogSvc
  OrderAB --> OrderSvc

  %% Services -> Mappers (business operations)
  AccountSvc --> AccountMapper
  CatalogSvc --> CategoryMapper
  CatalogSvc --> ProductMapper
  CatalogSvc --> ItemMapper

  OrderSvc --> ItemMapper
  OrderSvc --> OrderMapper
  OrderSvc --> LineItemMapper
  OrderSvc --> SequenceMapper

  %% Transactions
  TxMgr -.->|advises| AccountSvc
  TxMgr -.->|advises| CatalogSvc
  TxMgr -.->|advises| OrderSvc

  %% MyBatis plumbing and DB
  AccountMapper --> SqlSess
  CategoryMapper --> SqlSess
  ProductMapper --> SqlSess
  ItemMapper --> SqlSess
  OrderMapper --> SqlSess
  LineItemMapper --> SqlSess
  SequenceMapper --> SqlSess

  SqlSess -->|JDBC/SQL| HSQL
  TxMgr -->|commit/rollback| HSQL

  %% Caching
  L2Cache -.-> AccountMapper
  L2Cache -.-> CategoryMapper
  L2Cache -.-> ProductMapper
  L2Cache -.-> ItemMapper
  L2Cache -.-> OrderMapper
  L2Cache -.-> LineItemMapper
  L2Cache -.-> SequenceMapper
```

Component boundaries follow a classic layered monolith: Stripes ActionBeans (presentation) invoke Spring @Service components (business), which synchronously call MyBatis mapper DAOs (persistence) against an embedded HSQLDB, with Spring managing transactions across multiple mapper calls. Session-scoped ActionBeans hold conversational state (account, cart, order), and MyBatis provides per-mapper second-level caches; all interactions are in-process method calls except JDBC to the database.