# Core platform (the non-AI foundation)

Everything on this page ships in Phase 0 or Phase 1 and contains no models. If the AI
portfolio were deleted entirely, this platform would still sell tickets, admit visitors,
report zone popularity and alert on enclosure conditions. That is deliberate: it is what
makes the degradation ladder in ADR-0012 credible rather than decorative.

## Edge node

One node per zone cluster (assumption: 6 to 8 nodes for a 40-80 ha estate, A1).
Industrial mini-PC, wired power, cellular or fibre uplink where available, local Wi-Fi
access point for staff devices.

| Function | Behaviour | Requirement |
| --- | --- | --- |
| MQTT broker | Local broker terminates all device connections in its zone. Devices never talk to the cloud directly. | F12, NFR-SCALE-2 |
| Store-and-forward | Durable local queue, sized for 72 hours of local telemetry (assumption: about 4 GB per node). On reconnect, forwards oldest-first with per-device ordering and at-least-once delivery. | NFR-DATA-1, [FF-07](../04-verification/fitness-functions.md#ff-07) |
| Gate validation | Validates a signed entitlement token offline against a locally held allow-list and a local redemption log. Admits or refuses without the cloud. | FR-3, NFR-AVAIL-1, ADR-0003 |
| Local rules engine | Evaluates enclosure thresholds and ride fault signals locally, and pages staff over local Wi-Fi or cellular SMS fallback. | FR-10, NFR-PERF-2, NFR-SAFE-1 |
| Offline ticket sale | Issues a locally signed entitlement when the cloud is unreachable, queued for settlement. | FR-1, NFR-AVAIL-2 |
| Health beacon | Reports its own liveness, queue depth and disk headroom. Absence of a beacon is itself an alert. | NFR-OPS-2 |

Recovery model: a node is cattle. It holds no original data except the unforwarded buffer,
boots from an image, pulls its configuration and allow-list on first contact, and can be
swapped by a non-engineer following a printed card (NFR-OPS-2, [FF-13](../04-verification/fitness-functions.md#ff-13)).

## Offline entitlement design

This is the single most load-bearing mechanism in Phase 0, so it is specified rather than
named.

| Element | Design |
| --- | --- |
| Token | A ticket or pass is a signed token containing entitlement id, validity window, admission count and a pseudonymous holder key. It is signed by U2 and verified offline by any edge node with the public key. |
| Allow-list | Each node holds the set of entitlements valid for the current and next day, refreshed whenever connectivity exists. Size is small: 15,000 entitlements is a few megabytes. |
| Revocation | Handled by the validity window plus a small revocation delta list. An entitlement revoked during a disconnection may still be admitted once. We accept that; the cost is one ticket. |
| Double-use | Detected locally within a node (local redemption log) and across nodes at reconciliation. Cross-gate double-use during a full disconnection is possible and is accepted, bounded and reported. | 
| Reconciliation | On reconnect, the node replays redemptions; U2 applies them, detects conflicts, and produces a reconciliation report with a conflict rate. | 
| Why this trade | Refusing a valid visitor is worse than occasionally admitting a duplicate. The failure is commercial, small, measurable and reported, and this is written into ADR-0003 as an accepted consequence rather than hidden. |

## Connectivity

| Layer | Choice | Reason |
| --- | --- | --- |
| Sensor to node | LoRaWAN for low-rate battery sensors (enclosure temperature, humidity, door state), wired or Wi-Fi for cameras and gates | Battery life and range beat bandwidth for 200+ enclosure sensors. Cameras need bandwidth and are near power anyway. |
| Node to node | None by default | A mesh is a research project. Nodes are independent; the cloud is the meeting point. If A5 fails, mesh becomes a considered option, not before. |
| Node to cloud | Cellular (primary), fixed line at the main buildings where present | F11. Cheap, managed, no civil works across a historic estate. |
| Degraded mode | Local operation, defined in ADR-0003, with a staff-visible banner showing "offline since HH:MM" | Staff must know they are offline; silent degradation is how offline systems lose trust. |

We explicitly rejected a data-mule or delay-tolerant transport (a vehicle physically
carrying data). It is elegant and it is real engineering, but it adds a moving part with a
schedule to a client with two engineers, and store-and-forward with cellular already meets
NFR-DATA-1. Recorded in [../05-process/decision-log.md](../05-process/decision-log.md) D4.

## Cloud platform

| Component | Choice | Reason |
| --- | --- | --- |
| Compute | One container service running the cloud deployable, two environments (staging, production) | ADR-0005. Two environments is the maximum a two-engineer team keeps honest. |
| Event backbone | Managed message broker with durable topics per event family | Buffering, decoupling, replay. See [style-decision.md](style-decision.md). |
| Operational store | One managed relational database, schema per module, no cross-module foreign keys | Strong consistency where it is needed (U2), and the module boundary is enforced by review and by schema, not by a network hop. |
| Telemetry store | Managed time-series or columnar store for device readings and counts | Different access pattern and retention from operational data. |
| Analytics store | Object storage plus a query engine over the event log | Cheap history, replayable, and the training substrate for [../02-ai-capabilities/](../02-ai-capabilities/). |
| Identity | Managed identity provider, staff SSO, role per stakeholder group | Keepers, vet, gate staff, operations and the owner see different things. |
| Secrets and certificates | Managed secret store; per-node client certificates with a documented rotation procedure | NFR-SEC-2 |
| Observability | Managed metrics, logs and traces, with a single on-call rotation | NFR-OPS-1, [FF-12](../04-verification/fitness-functions.md#ff-12) |

## Event families

Events are the contract between units and the substrate for every model. Payloads are
illustrative and use assumed field names.

| Event | Producer | Consumers | Example payload |
| --- | --- | --- | --- |
| `GateAdmission` | U1 | U2, U3, U5 | `{event_id, node_id, gate_id, entitlement_id, decision, decided_at, offline: true, admission_seq}` |
| `EntitlementIssued` | U2 | U1, U5 | `{entitlement_id, type, valid_from, valid_to, admissions_allowed, holder_key}` |
| `ZoneCount` | U1 | U3, U5 | `{node_id, zone_id, window_start, window_end, in_count, out_count, method: "beam"}` |
| `EnclosureReading` | U1 | U4 | `{node_id, enclosure_id, metric, value, unit, observed_at, sensor_id, quality}` |
| `WelfareAlert` | U1 (rule) or U4 (model) | U4, U3 | `{alert_id, enclosure_id, severity, source: "rule"\|"model", reason, confidence, basis, raised_at}` |
| `FeedingRecorded` | U4 | U5 | `{enclosure_id, scheduled_at, performed_at, keeper_id, amount, notes}` |
| `PiranhaCountEstimated` | U4 | U5 | `{tank_id, estimate, interval_low, interval_high, method, frames_used, estimated_at}` |
| `RideStatusChanged` | U1 | U3, U5 | `{ride_id, status, reason, changed_at, reported_by}` |

Schema evolution rule: additive only. New fields are optional with defaults; a field is
never removed or retyped within a major version; the major version is in the topic name.
A consumer must ignore unknown fields. This is checked in CI as
[FF-14](../04-verification/fitness-functions.md#ff-14).

## What is deliberately absent

| Absent | Why |
| --- | --- |
| Service mesh, API gateway between internal modules | There is one deployable. There is nothing to mesh. |
| Kubernetes | Two engineers. A managed container runtime covers the need at a fraction of the operational cost. Revisit if the team passes 5 engineers. |
| Feature store as a separate system | Two models in Phase 2, five at most by Phase 4. A curated table in the analytics store with a versioning convention is enough, and it does not need an operator. |
| Real-time streaming analytics engine | The fastest business question is "how long is the queue right now", which is a one-minute aggregate. Batch every minute is real-time enough. |
