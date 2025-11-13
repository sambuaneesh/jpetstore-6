```mermaid
graph TB

  subgraph Client
    Browser[Web Browser]
  end

  subgraph Container[Servlet Container (Tomcat)]
    subgraph Web[Presentation: Stripes MVC + JSP/JSTL]
      Dispatcher[StripesFilter + DispatcherServlet (*.action)]
      subgraph ActionBeans[ActionBeans (@SessionScope)]
        AAB[AccountActionBean]
        CAB[CatalogActionBean]
        CtAB[CartActionBean]
        OAB[OrderActionBean]
      end
      JSP[JSP Views]
      Session[HTTP Session\n(state: AccountActionBean, Cart, Order)]
    end

    subgraph Spring[Spring Application Context]
      subgraph Services[Service Layer (@Service, @Transactional)]
        AccSvc[AccountService]
        CatSvc[CatalogService]
        OrdSvc[OrderService]
      end
      Tx[DataSourceTransactionManager]
    end

    subgraph Persistence[Persistence: MyBatis + MyBatis-Spring]
      SSF[SqlSessionFactory]
      DS[DataSource (Embedded HSQLDB)]
      Cache[MyBatis 2nd-level Cache\n(per-mapper, in-memory)]
      subgraph Mappers[Mapper Interfaces + XML]
        AccMap[AccountMapper]
        CatMap[CategoryMapper]
        ProdMap[ProductMapper]
        ItemMap[ItemMapper]
        OrdMap[OrderMapper]
        LiMap[LineItemMapper]
        SeqMap[SequenceMapper]
      end
    end
  end

  DB[(HSQLDB\n(JPetStore schema))]

  Browser -->|HTTP requests| Dispatcher
  Dispatcher -->|Resolves & invokes events| AAB
  Dispatcher --> CAB
  Dispatcher --> CtAB
  Dispatcher --> OAB
  AAB -->|Forward/Redirect| JSP
  CAB -->|Forward/Redirect| JSP
  CtAB -->|Forward/Redirect| JSP
  OAB -->|Forward/Redirect| JSP

  AAB <--> Session
  CAB <--> Session
  CtAB <--> Session
  OAB <--> Session

  %% Stripes-Spring DI into ActionBeans
  AAB -->|DI| AccSvc
  AAB -->|DI| CatSvc
  CAB -->|DI| CatSvc
  CtAB -->|DI| CatSvc
  OAB -->|DI| OrdSvc

  %% Transaction boundaries at services
  AccSvc -. @Transactional .-> Tx
  CatSvc -. @Transactional .-> Tx
  OrdSvc -. @Transactional .-> Tx

  %% Service to mapper calls
  AccSvc --> AccMap
  CatSvc --> CatMap
  CatSvc --> ProdMap
  CatSvc --> ItemMap
  OrdSvc --> SeqMap
  OrdSvc -->|decrement inventory| ItemMap
  OrdSvc --> OrdMap
  OrdSvc --> LiMap

  %% MyBatis plumbing
  Mappers --> SSF
  SSF --> DS
  DS --> DB

  %% Mapper cache usage
  AccMap -. uses .- Cache
  CatMap -. uses .- Cache
  ProdMap -. uses .- Cache
  ItemMap -. uses .- Cache
  OrdMap -. uses .- Cache
  LiMap -. uses .- Cache
  SeqMap -. uses .- Cache
```

Rationale:
- The system is a layered monolith: Stripes ActionBeans (session-scoped) handle web requests and forward to JSP views; they delegate to Spring @Transactional services, which orchestrate business operations and call MyBatis mapper interfaces. Mappers use a shared SqlSessionFactory/DataSource to access the HSQLDB schema, with per-mapper in-memory second-level caches.
- Communication is synchronous and in-process; HTTP session stores conversational state (account, cart, order), while transaction boundaries are enforced at the service layer (notably, order creation spans inventory decrement and order persistence).