```mermaid
graph TB
  Client[Browser] -->|HTTP *.action| Stripes[Stripes Filter/DispatcherServlet]

  subgraph WebContainer["Tomcat (WAR: jpetstore)"]
    subgraph Presentation["Presentation"]
      Stripes --> Actions["ActionBeans\n- AccountActionBean\n- CatalogActionBean\n- CartActionBean\n- OrderActionBean"]
      Actions -->|Forward/Redirect| Views[JSP Views]
    end

    subgraph Business["Services"]
      AccountSvc[AccountService]
      CatalogSvc[CatalogService]
      OrderSvc[OrderService]
    end

    subgraph Persistence["Persistence (MyBatis)"]
      Mappers["Mappers\n- AccountMapper\n- CategoryMapper\n- ProductMapper\n- ItemMapper\n- OrderMapper\n- LineItemMapper\n- SequenceMapper"]
    end

    subgraph Infra["Infrastructure (Spring / MyBatis-Spring)"]
      SpringCtx[Spring ApplicationContext]
      TxMgr[DataSourceTransactionManager]
      SqlSession[SqlSessionFactoryBean]
      DS[BasicDataSource]
    end

    %% Wiring and calls
    Actions -->|@SpringBean| AccountSvc
    Actions -->|@SpringBean| CatalogSvc
    Actions -->|@SpringBean| OrderSvc
    Views -->|HTML| Client

    AccountSvc --> Mappers
    CatalogSvc --> Mappers
    OrderSvc --> Mappers

    Mappers --> SqlSession
    SqlSession --> DS
    SpringCtx -. scans/wires .-> Business
    SpringCtx -. scans/wires .-> Persistence
    SpringCtx -. context/beans .-> Presentation
    TxMgr -. @Transactional .-> Business
  end

  DB[(Database\nMySQL in Docker / HSQLDB in tests)]
  DS --> DB

  %% Deployment hint
  subgraph Runtime["Optional Containerization"]
    AppC["Docker: jpetstore (Tomcat)"]
    DBC["Docker: mysql:8"]
  end
  AppC -. hosts .- WebContainer
  DBC -. provides .- DB
```

Rationale:
The system is a monolithic Stripes/Spring/MyBatis web application packaged as a WAR on Tomcat. Stripes routes *.action requests to session-scoped ActionBeans, which render JSPs or delegate to services; services use MyBatis mapper interfaces to access the database. Spring wires ActionBeans/services/mappers, provides transaction demarcation, and MyBatis-Spring bridges mappers through SqlSessionFactory/DataSource to MySQL (or HSQLDB for tests); Docker Compose can orchestrate the app and MySQL containers.