```mermaid
graph TB

%% Client
C[Browser] -->|HTTP| Stripes[Stripes DispatcherServlet (*.action)]

%% Presentation Layer
subgraph Web[Presentation (Stripes + JSP)]
  direction TB
  subgraph Actions[ActionBeans (@SessionScope)]
    AA[AccountActionBean]
    CA[CatalogActionBean]
    CTA[CartActionBean]
    OA[OrderActionBean]
  end
  JSP[JSP Views (JSTL)]
  Session[(HTTP Session)]
end

Stripes -->|dispatch| AA
Stripes -->|dispatch| CA
Stripes -->|dispatch| CTA
Stripes -->|dispatch| OA
AA -->|forwards to| JSP
CA -->|forwards to| JSP
CTA -->|forwards to| JSP
OA -->|forwards to| JSP
JSP -->|HTML| C
AA -->|stores user/cart state| Session
CA -->|reads session| Session
CTA -->|stores cart| Session
OA -->|reads/clears cart| Session

%% Service Layer
subgraph Service[Service Layer (Spring @Service)]
  direction TB
  AS[AccountService]
  CS[CatalogService]
  OS[OrderService]
end

AA -->|calls| AS
AA -->|calls| CS
CA -->|calls| CS
CTA -->|calls| CS
OA -->|calls| OS

%% Persistence Layer
subgraph Persistence[Persistence (MyBatis Mappers)]
  direction TB
  AM[AccountMapper]
  CM[CategoryMapper]
  PM[ProductMapper]
  IM[ItemMapper (incl. Inventory)]
  OM[OrderMapper]
  LIM[LineItemMapper]
  SM[SequenceMapper]
end

AS --> AM
CS --> CM
CS --> PM
CS --> IM
OS --> OM
OS --> LIM
OS --> SM
OS -->|getItem(), getInventory(), updateInventoryQty()| IM

%% Infrastructure
subgraph Infra[Infrastructure (Spring + MyBatis)]
  direction TB
  DI[Spring Context & DI]
  TM[(Spring TxManager)]
  SSF[MyBatis SqlSessionFactory]
  DS[(DataSource)]
end

AM --> SSF
CM --> SSF
PM --> SSF
IM --> SSF
OM --> SSF
LIM --> SSF
SM --> SSF
SSF --> DS
DS --> DB[(HSQLDB)]

DI --> AS
DI --> CS
DI --> OS
TM -.->|@Transactional| AS
TM -.->|@Transactional| OS
```

Rationale:
The monolith is layered: session-scoped Stripes ActionBeans handle HTTP and render JSPs, delegating to Spring-managed services that define transactional boundaries. Services use MyBatis mappers for persistence against a single HSQLDB; notable coupling exists where OrderService directly reads items and adjusts inventory via ItemMapper. This structure centralizes transactions per service call and relies on a shared database and mapper-level caches for read performance.