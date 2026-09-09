# Containers

```mermaid
flowchart TB
  subgraph DEV["Devices (about 300; sized for 600)"]
    SEN["Welfare sensors<br/>LoRaWAN, ~240"]
    CNT["Counting sensors<br/>~12"]
    CAM["Cameras<br/>tanks and enclosures, ~12"]
    TERM["Gate terminals<br/>~8, scan and admit"]
  end

  subgraph EDGE["U1 Edge Node (6-8, one per zone cluster)"]
    BRK["Local MQTT broker"]
    SAF["Rules engine<br/>thresholds, ride faults<br/>ALWAYS LOCAL"]
    VAL["Entitlement validator<br/>allow-list + signature<br/>AUTHORITATIVE OFFLINE"]
    SFQ["Store-and-forward queue<br/>72 h durable"]
    GPU["Tank vision inference<br/>(GPU node only)"]
  end

  subgraph CLOUD["Cloud deployable"]
    U2["U2 Ticketing and Access<br/>entitlements, passes,<br/>reconciliation"]
    U3["U3 Park Operations<br/>zone counts, ride status,<br/>tasks, device health"]
    U4["U4 Animal Welfare<br/>enclosure state, feeding,<br/>alerts, piranha counts"]
    U5["U5 Guest Engagement<br/>reporting, forecast,<br/>assistant, offers"]
    AIS["AI Services<br/>gateway, registry, eval<br/>(library + gateway)"]
  end

  subgraph DATA["Managed data stores"]
    BUS["Event backbone<br/>durable topics"]
    ODB["Operational DB<br/>schema per module"]
    TSD["Telemetry store"]
    LAKE["Object storage + query<br/>event history, training data"]
  end

  subgraph CLIENTS["Clients"]
    WEB["Visitor web client<br/>buy, ask"]
    STAFF["Staff app<br/>alerts, tasks, offline capable"]
    BI["Reporting (hosted BI)"]
  end

  SEN --> BRK
  CNT --> BRK
  CAM --> BRK
  CAM --> GPU
  TERM --> VAL
  BRK --> SAF
  BRK --> SFQ
  GPU --> SFQ
  VAL --> SFQ
  SAF ==>|"alert, < 60 s,<br/>no cloud needed"| STAFF
  SFQ -->|"cellular, replay on reconnect"| BUS
  U2 -->|"allow-list sync"| VAL

  BUS --> U2
  BUS --> U3
  BUS --> U4
  U2 --> ODB
  U3 --> TSD
  U4 --> ODB
  U2 --> LAKE
  U3 --> LAKE
  U4 --> LAKE
  LAKE --> U5
  U4 -.-> AIS
  U5 -.-> AIS

  WEB --> U2
  WEB --> U5
  STAFF --> U3
  STAFF --> U4
  BI --> LAKE
```

## Legend

| Element | Meaning |
| --- | --- |
| Solid arrow | Normal data flow |
| Thick arrow (`==>`) | A path that must work with the cloud unreachable |
| Dashed arrow (`-.->`) | A model call, always with a deterministic fallback (ADR-0012) |
| `U`-prefixed box | One of the five units in [../01-architecture/decomposition.md](../01-architecture/decomposition.md) |
| CLOUD subgraph | One deployable. The boxes inside are modules, not services. |

## What this diagram is asserting

| Assertion | Where it is decided |
| --- | --- |
| Devices never talk to the cloud; the local broker terminates every device connection | ADR-0004 |
| The two paths that must never stop, `VAL` and `SAF`, have no cloud dependency at all | ADR-0003, NFR-AVAIL-1, NFR-SAFE-1 |
| Piranha vision inference runs on the estate, so tank video never crosses the uplink | [../02-ai-capabilities/cap-02-piranha-population.md](../02-ai-capabilities/cap-02-piranha-population.md) |
| AI Services is reachable from exactly two modules, U4 and U5, and from nothing at the edge | [../01-architecture/ai-platform.md](../01-architecture/ai-platform.md), [FF-20](../04-verification/fitness-functions.md#ff-20) |
| Every module writes its own data and reads others' through events or the lake, with no cross-module database access | [../01-architecture/decomposition.md](../01-architecture/decomposition.md) |
| Reporting reads the lake, not the operational database, so a slow report cannot slow the gates | [../01-architecture/core-platform.md](../01-architecture/core-platform.md) |
