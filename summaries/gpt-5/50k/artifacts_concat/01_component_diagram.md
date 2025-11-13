```mermaid
graph TB

  subgraph Client
    Browser[Web Browser]
  end

  subgraph Web[Web Layer (Stripes MVC)]
    Dispatcher[Stripes Filter + DispatcherServlet (*.action)]
    AccountAB[AccountActionBean (Session)]
    CatalogAB[CatalogActionBean (Session)]
    CartAB[CartActionBean (Session)]
    OrderAB[OrderActionBean (Session)]
    Views[JSP Views (/WEB-INF/jsp)]
  end

  subgraph Service[Service Layer (Spring)]
    AccountSvc[AccountService (@Transactional)]
    CatalogSvc[CatalogService]
    OrderSvc[OrderService (@Transactional)]
    TxMgr[DataSourceTransactionManager]
  end

  subgraph Persistence[Persistence (MyBatis)]
    SqlSession[SqlSessionFactory]
    AccountMapper[AccountMapper]
    CategoryMapper[CategoryMapper]
    ProductMapper[ProductMapper]
    ItemMapper[ItemMapper]
    OrderMapper[OrderMapper]
    LineItemMapper[LineItemMapper]
    SequenceMapper[SequenceMapper]
  end

  subgraph DB[Database (HSQLDB In-Memory)]
    Accounts[(ACCOUNT / PROFILE / SIGNON)]
    CatalogTbls[(CATEGORY / PRODUCT / ITEM)]
    Inventory[(INVENTORY)]
    OrdersTbls[(ORDERS / ORDERSTATUS / LINEITEM)]
    Sequence[(SEQUENCE)]
  end

  %% Client -> Web
  Browser -->|HTTP *.action| Dispatcher
  Dispatcher --> AccountAB
  Dispatcher --> CatalogAB
  Dispatcher --> CartAB
  Dispatcher --> OrderAB

  %% Web -> Views
  AccountAB -->|forward/render| Views
  CatalogAB -->|forward/render| Views
  CartAB -->|forward/render| Views
  OrderAB -->|forward/render| Views

  %% Web -> Services
  AccountAB --> AccountSvc
  CatalogAB --> CatalogSvc
  CartAB --> CatalogSvc
  OrderAB --> OrderSvc

  %% Transaction boundaries
  AccountSvc --> TxMgr
  OrderSvc --> TxMgr

  %% Services -> Mappers
  AccountSvc -->|CRUD/auth| AccountMapper
  CatalogSvc -->|list/search| ProductMapper
  CatalogSvc -->|get/list| CategoryMapper
  CatalogSvc -->|get/isInStock| ItemMapper
  OrderSvc -->|orders| OrderMapper
  OrderSvc -->|lines| LineItemMapper
  OrderSvc -->|getNextId("ordernum")| SequenceMapper
  OrderSvc -->|item load + inventory decrement| ItemMapper

  %% Mappers -> DB tables
  AccountMapper --> Accounts
  CategoryMapper --> CatalogTbls
  ProductMapper --> CatalogTbls
  ItemMapper --> CatalogTbls
  ItemMapper --> Inventory
  OrderMapper --> OrdersTbls
  LineItemMapper --> OrdersTbls
  SequenceMapper --> Sequence

  %% MyBatis plumbing (optional view)
  AccountMapper --- SqlSession
  CategoryMapper --- SqlSession
  ProductMapper --- SqlSession
  ItemMapper --- SqlSession
  OrderMapper --- SqlSession
  LineItemMapper --- SqlSession
  SequenceMapper --- SqlSession
```

Rationale:
The system is a layered monolith where Stripes ActionBeans (session-scoped controllers) synchronously invoke Spring services, which orchestrate MyBatis mappers within @Transactional boundaries. All persistence goes to a single HSQLDB schema; Ordering also reads item details and decrements inventory and uses a DB-backed SEQUENCE for IDs, reflecting tight coupling across catalog/inventory and order data within one process.