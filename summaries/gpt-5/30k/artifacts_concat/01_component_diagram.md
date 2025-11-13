```mermaid
graph TB

%% Client
Client[Browser] --> Stripes

%% Presentation Layer
subgraph WEB[Presentation Layer (Stripes + JSP)]
  Stripes[Stripes Dispatcher/Filter (*.action)]
  JSP[JSP Views (JSTL)]
  Session[(HTTP Session: accountBean, cart, order draft)]

  AccountAB[AccountActionBean (@SessionScope)]
  CatalogAB[CatalogActionBean (@SessionScope)]
  CartAB[CartActionBean (@SessionScope)]
  OrderAB[OrderActionBean (@SessionScope)]

  Stripes --> AccountAB
  Stripes --> CatalogAB
  Stripes --> CartAB
  Stripes --> OrderAB

  AccountAB --> JSP
  CatalogAB --> JSP
  CartAB --> JSP
  OrderAB --> JSP

  AccountAB <--> Session
  CatalogAB <--> Session
  CartAB <--> Session
  OrderAB <--> Session
end

%% Service Layer
subgraph SVC[Service Layer (Spring @Service)]
  AccountSvc[AccountService]
  CatalogSvc[CatalogService]
  OrderSvc[OrderService]

  AccountAB --> AccountSvc
  AccountAB --> CatalogSvc
  CatalogAB --> CatalogSvc
  CartAB --> CatalogSvc
  OrderAB --> OrderSvc
end

%% Persistence Layer
subgraph PERS[Persistence Layer (MyBatis Mappers)]
  AccountMapper[AccountMapper]
  CategoryMapper[CategoryMapper]
  ProductMapper[ProductMapper]
  ItemMapper[ItemMapper]
  OrderMapper[OrderMapper]
  LineItemMapper[LineItemMapper]
  SequenceMapper[SequenceMapper]

  AccountSvc --> AccountMapper
  CatalogSvc --> CategoryMapper
  CatalogSvc --> ProductMapper
  CatalogSvc --> ItemMapper
  OrderSvc --> SequenceMapper
  OrderSvc --> ItemMapper
  OrderSvc --> OrderMapper
  OrderSvc --> LineItemMapper
end

%% Database
subgraph DB[HSQLDB Demo Database]
  Accounts[(ACCOUNT / PROFILE / SIGNON)]
  CatalogTbls[(CATEGORY / PRODUCT / ITEM / INVENTORY)]
  OrdersTbls[(ORDERS / ORDERSTATUS / LINEITEM)]
  SeqTbl[(SEQUENCE)]
end

AccountMapper --> Accounts
CategoryMapper --> CatalogTbls
ProductMapper --> CatalogTbls
ItemMapper --> CatalogTbls
OrderMapper --> OrdersTbls
LineItemMapper --> OrdersTbls
SequenceMapper --> SeqTbl
```

The system is a layered monolith: Stripes ActionBeans (session-scoped) handle web requests and forward to JSPs, invoking Spring services for business logic. Services orchestrate transactional operations and delegate persistence to MyBatis mappers that operate on a shared HSQLDB schema; notably, OrderService updates both order and inventory data within a single transaction, reflecting tight coupling via the shared database. All communication is synchronous and intra-process.