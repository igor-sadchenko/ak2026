# System context

```mermaid
flowchart TB
  V["Visitor / family<br/>buys, enters, asks questions"]
  K["Keeper<br/>feeds, checks, triages alerts"]
  VET["Veterinarian<br/>decides treatment"]
  GS["Gate staff<br/>admits, overrides"]
  OM["Operations manager<br/>staffs, reports"]
  OWN["Countess (owner)<br/>decides whether to keep spending"]
  REG["Safety inspectorate<br/>rides and containment"]

  SYS["<b>Von Digitalis Estate System</b><br/>edge tier + cloud modules<br/>5 units, 5 AI capabilities"]

  PSP["Payment service provider<br/>(external, holds all card data)"]
  LLM["Model provider<br/>(external, assistant only,<br/>two configured)"]
  WX["Weather data<br/>(external)"]
  NOTIF["Email / message delivery<br/>(external)"]

  V -->|"buys tickets, passes,<br/>asks questions"| SYS
  GS -->|"scans, overrides"| SYS
  SYS -->|"admit / refuse,<br/>answers with citations"| V
  SYS -->|"alerts with evidence"| K
  K -->|"confirms, overrides,<br/>records feeding"| SYS
  SYS -->|"escalations"| VET
  VET -->|"treatment outcomes<br/>(delayed ground truth)"| SYS
  SYS -->|"forecast, counts,<br/>ride status"| OM
  SYS -->|"attendance, revenue,<br/>return rate"| OWN
  SYS -->|"inspection and<br/>incident records"| REG

  SYS <-->|"hosted checkout"| PSP
  SYS -.->|"grounded answers,<br/>fails to cached search"| LLM
  SYS -.->|"forecast feature,<br/>optional"| WX
  SYS -->|"opt-in offers only"| NOTIF
```

## Legend

| Element | Meaning |
| --- | --- |
| Solid arrow | Normal flow the system depends on |
| Dashed arrow | An external dependency the system is designed to work without. Both dashed links have declared fallbacks (ADR-0012). |
| Double-headed arrow | Bidirectional exchange |
| External box | Outside our control and outside our operational responsibility (ADR-0005) |

## What this diagram is asserting

| Assertion | Where it is decided |
| --- | --- |
| Card data never enters the system; the PSP holds it | NFR-SEC-1, ADR-0005 |
| Only one external model provider dependency exists, it serves one capability, and it is dashed | ADR-0008, [../02-ai-capabilities/cap-05-guest-companion.md](../02-ai-capabilities/cap-05-guest-companion.md) |
| The veterinarian is inside the loop, not downstream of it: their outcomes are the training signal | ADR-0010, [../02-ai-capabilities/cap-01-animal-welfare-anomaly.md](../02-ai-capabilities/cap-01-animal-welfare-anomaly.md) |
| The visitor is never identified unless they opt in, so no arrow carries visitor identity by default | ADR-0009 |
| The owner is a stakeholder of the system's output, because the decision to continue funding is a real interface | [../00-problem/stakeholders.md](../00-problem/stakeholders.md) |
