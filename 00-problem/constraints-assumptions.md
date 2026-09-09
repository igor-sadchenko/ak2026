# Constraints and assumptions

Constraints come from the brief. Assumptions are ours. The third column is the one that
matters: it says what we would do differently, and how much of the work survives, if the
assumption turns out to be wrong.

## Constraints (from the brief, not negotiable)

| ID | Constraint | Source | Architectural consequence |
| --- | --- | --- | --- |
| C1 | Wi-Fi coverage across the estate is patchy. | F10 | Nothing on the critical path may assume connectivity. Drives ADR-0003 (edge-first, offline-authoritative gates). |
| C2 | A link from the estate to the cloud is required. | F11 | One managed uplink with a documented degraded mode, not a mesh research project. ADR-0004. |
| C3 | Budget exists for MQTT-capable devices. | F12 | Sensing is a solved procurement problem; the risk is in operating 600 devices, not buying them. ADR-0004. |
| C4 | Cloud is permitted. | F13 | Managed services by default; we build only the estate domain. ADR-0005. |
| C5 | The 40 rides are historic and already certified. | F3 | We report on rides, we do not control them. No actuation, no safety interlocks in software. |
| C6 | The animals are exotic and poisonous. | F4 | Welfare alerting is a safety function, so it is rule-based and local. NFR-SAFE-1, ADR-0012. |
| C7 | Failure to reach 15,000 visitors per day costs the client the plant collection. | F7, F8 | Time to first value is an architectural characteristic here, not a project-management concern. ADR-0001. |

## Assumptions

| ID | Assumption | What changes if it is wrong |
| --- | --- | --- |
| A1 | The estate is roughly 40-80 hectares with buildings clustered near the entrance. | Larger: cellular backhaul per zone instead of a single uplink, edge node count roughly doubles, CapEx in [../03-delivery/cost-model.md](../03-delivery/cost-model.md) rises by about 30%. Smaller: fewer edge nodes, no LoRaWAN, same software. Software design is unaffected either way; this is why we put sensing behind a gateway abstraction (ADR-0004). |
| A2 | There are 6 to 8 pedestrian entrances. | More entrances change gate hardware count and Phase 0 CapEx linearly. Fewer entrances raise per-gate peak throughput and make NFR-PERF-1 harder; mitigation is a second scan lane, not a software change. |
| A3 | Average spend per visitor is about 18 EUR, and the season is about 200 operating days. | This drives the "IT cost is not the binding constraint" conclusion in [../03-delivery/cost-model.md](../03-delivery/cost-model.md). If real spend is a third of this, NFR-COST-1 becomes binding and the Phase 4 AI capabilities are cut first, as they have the worst cost-to-certainty ratio. |
| A4 | There is no legacy ticketing or CCTV system worth integrating. | If a legacy ticketing system exists, Phase 0 becomes an integration project: FR-1 and FR-2 are replaced by an adapter, and the gate work (FR-3, FR-4) is unchanged and still the highest-value part. Phase 0 length grows by about 4 weeks. |
| A5 | Cellular coverage reaches at least the main buildings. | If not, the uplink becomes satellite (higher latency, higher cost, same software), or a scheduled physical data transfer. Because the design is store-and-forward, a slower uplink degrades reporting freshness only; FR-3 and FR-10 still work. This is the single assumption that most changes cost and least changes design. |
| A6 | A veterinarian is available on call within a few hours. | If not, the human-in-the-loop welfare design (ADR-0010) has no human. We would then narrow the welfare capability to environmental threshold alerts routed to keepers, drop the anomaly model from Phase 2, and add a documented external escalation contract. See the assumption-failure playbook in [../03-delivery/implementation-plan.md](../03-delivery/implementation-plan.md). |
| A7 | The client can fund 4 to 6 people, of whom 2 are engineers. | Fewer: we cut Phase 4 entirely and buy a hosted analytics product instead of building U5. More: we would still not add units; extra capacity goes into content and operations, not architecture. See ADR-0002. |
| A8 | GDPR or an equivalent regime applies. | A weaker regime does not change our design, because the privacy posture (ADR-0009) is driven by the family audience, not only by law. A stricter regime (for example, biometrics banned outright) also changes nothing, because we already forbid biometrics. This assumption is deliberately non-load-bearing. |
| A9 | Twelve months of attendance history do not exist and cannot be bought. | If usable history exists (for example, from the gnome business or from ticket resellers), the forecasting capability (FR-13) moves from Phase 3 to Phase 2 and the cold-start rules baseline becomes a shorter bridge. Nothing else changes. ADR-0013. |
| A10 | About 55 enclosures need 3 to 5 sensors each, so 165 to 275 welfare sensors; with 8 gate terminals, 12 counting sensors and 12 cameras that is roughly 300 connected devices in total. | Device count drives NFR-SCALE-2 sizing and CapEx only. NFR-SCALE-2 is deliberately sized at 600, twice the expected fleet, so a doubling needs no re-architecture and still stays inside a single managed broker tier. |
| A11 | Piranha tanks are enclosed, artificially lit and camera-observable. | If tanks are outdoor ponds with poor visibility, the CV capability (FR-9) is not feasible as designed; the fallback is periodic manual counts with a sampling protocol, which is what happens today. See [cap-02](../02-ai-capabilities/cap-02-piranha-population.md) risk section. |
| A12 | Keepers carry a phone or a rugged handheld and will look at it. | If not, welfare alerts need physical annunciators at enclosure clusters. That is a hardware addition, not a redesign, and it is priced as an option in the cost model. |
| A13 | Visitors will not install a native app for a single visit. | Drives the choice of a web client over a native app. If an app is mandated later, the same APIs serve it. |
| A14 | Ticket payments go through an external payment service provider. | If the client insists on handling cards directly, PCI scope enters the estate, Phase 0 grows by roughly a quarter, and NFR-SEC-1 has to be rewritten. We would push back hard. ADR-0005. |

## How assumptions are managed after submission

| Rule | Reason |
| --- | --- |
| Each assumption has an owner and a date by which it must be confirmed or replaced by a fact. | An assumption that is never tested becomes a belief. |
| Assumptions A1, A2, A5 and A9 are confirmed during the first two weeks of Phase 0, because they change cost or sequence. | See [../03-delivery/first-90-days.md](../03-delivery/first-90-days.md), weeks 1-2. |
| An assumption that fails triggers the matching playbook, not a replan from scratch. | [../03-delivery/implementation-plan.md](../03-delivery/implementation-plan.md), "When an assumption fails". |
