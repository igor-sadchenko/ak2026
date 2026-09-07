# ADR-0004: MQTT and LoRaWAN sensing, cellular backhaul, no data mule

**Status:** accepted
**Deciders:** architect, field technician
**Relates to:** F10, F11, F12, NFR-SCALE-2, NFR-SEC-2, A1, A5, A10, D4

## Context

There is budget for MQTT-capable devices (F12), a link to the cloud is required (F11), and
Wi-Fi coverage is patchy (F10). The sensing load is roughly 600 devices (A10): about 240
welfare sensors across 55 enclosures, 12 counting sensors, 12 cameras, 8 gate terminals and
associated infrastructure. The estate is historic, so trenching for cable is expensive and
often not permitted (assumption).

## Decision

Three tiers, chosen per hop rather than as one technology:

| Hop | Technology | Reason |
| --- | --- | --- |
| Battery sensor to node | LoRaWAN | Range and multi-year battery life at low data rates. Enclosure readings are a few bytes every minute. |
| Camera and gate to node | Wired or local Wi-Fi | These need bandwidth and are near power anyway. |
| Device to broker | MQTT to a **local** broker on the edge node | F12. Devices never connect to the cloud directly, which keeps device configuration trivial and makes the node the only thing needing a certificate to the outside. |
| Node to cloud | Cellular primary, fixed line at the main buildings where present | No civil works, managed by a carrier, and a satellite swap is a drop-in if A5 fails. |
| Node to node | **Nothing** | Nodes are independent; the cloud is the meeting point. |

Every node-to-cloud connection is mutually authenticated with a per-node client certificate,
and a node's credentials grant access only to its own zone's topics (NFR-SEC-2).

**No data mule and no delay-tolerant transport.** Store-and-forward with cellular meets
NFR-DATA-1 without a scheduled physical process.

## Alternatives considered

| Alternative | Why not |
| --- | --- |
| Estate-wide Wi-Fi covering everything | Directly contradicts F10 as a constraint and would mean an infrastructure project across a historic estate before any software ships. |
| Mesh networking between nodes | Solves a problem we do not have (nodes are independently reachable) and introduces routing failures that are hard to diagnose without a network engineer. |
| Data mule: a vehicle physically carrying data from remote zones | Adds a scheduled physical process with an owner and a failure mode, to a client with two engineers. Correct for an estate with genuinely no coverage, which the site survey will confirm or refute. See D4. |
| Direct device-to-cloud MQTT | 600 device certificates to manage and 600 things that stop working when the uplink drops. The local broker is what makes the edge autonomy in ADR-0003 possible. |
| Cellular per sensor (NB-IoT) | Per-device connectivity cost and worse battery life, for no gain over LoRaWAN into a local node. |

## Consequences

**Good**

- Sensing is a procurement problem with commodity parts, not an engineering project.
- One certificate per node instead of one per device; rotation is a manageable operation.
- Satellite is a drop-in replacement for cellular if coverage fails, with no software change.

**Bad**

- LoRaWAN's low data rate rules out anything richer than periodic scalar readings on that tier. Any future behaviour sensing needs the camera tier and its bandwidth.
- Two radio technologies to stock spares for and to train the field technician on.
- Cellular is a recurring cost per node (240 EUR/month for 8 nodes).

**Risks**

| Risk | Mitigation | Where tracked |
| --- | --- | --- |
| Cellular coverage is worse than the survey suggested (A5) | Store-and-forward absorbs it; the playbook moves to satellite or on-estate hosting | [../03-delivery/implementation-plan.md](../03-delivery/implementation-plan.md) |
| Reconnect burst overwhelms cloud ingest | Sized and tested at 10x | [FF-06](../04-verification/fitness-functions.md#ff-06) |
| A node is physically stolen | Certificate scoped to its own zone; revocation procedure; drill quarterly | [FF-09](../04-verification/fitness-functions.md#ff-09) |
| LoRaWAN interference or range shortfall in a walled estate | Confirmed in the week-1 site survey before ordering | [../03-delivery/first-90-days.md](../03-delivery/first-90-days.md) |

## How we will know this was right

[FF-07](../04-verification/fitness-functions.md#ff-07) shows zero unexplained telemetry gaps
and every gap over an hour has a recorded cause, sustained for 30 days at the Phase 1 exit.
[FF-06](../04-verification/fitness-functions.md#ff-06) clears a full 72-hour, 8-node
reconnect burst within an hour. If sensor battery replacement becomes a recurring keeper
burden (more than one per week across the estate), the sensing tier choice was wrong.
