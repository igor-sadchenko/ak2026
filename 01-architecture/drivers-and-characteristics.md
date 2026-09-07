# Architectural drivers and characteristics

Choosing three driving characteristics is only useful if we also say which ones we are
prepared to be bad at. This page does both.

## The three that drive the design

| Rank | Characteristic | Why this one | What it costs us | Verified by |
| --- | --- | --- | --- | --- |
| 1 | **Availability under disconnection** (fault tolerance at the edge) | Patchy Wi-Fi is a stated constraint (F10) and the two things that must never stop, admitting visitors (FR-3) and alerting on a dangerous animal (FR-10), both happen in the field. A system that needs the uplink is a system that closes the gates on a bad day. | Duplicated logic at the edge and in the cloud, a reconciliation protocol, and a real conflict-resolution policy. | [FF-01](../04-verification/fitness-functions.md#ff-01), [FF-04](../04-verification/fitness-functions.md#ff-04), [FF-07](../04-verification/fitness-functions.md#ff-07) |
| 2 | **Operability by a very small team** | The client is a poor estate with no IT department (U6, A7). An architecture the client cannot run is a prototype with a longer README. This is the characteristic that most distinguishes this submission from a larger design. | We refuse decompositions that would be better on paper. Fewer, larger units; managed services even where self-hosting is cheaper per month. | [FF-12](../04-verification/fitness-functions.md#ff-12), [FF-13](../04-verification/fitness-functions.md#ff-13) |
| 3 | **Time to first value** | The failure mode in the brief is commercial and dated: 15,000 per day in 3 years or the collection is sold (F7, F8). An architecture that pays off in year 3 has already failed. | Phase 0 ships deliberately unclever software: counters, gates, a dashboard, no models. Some Phase 0 code is knowingly replaced in Phase 2. | [../03-delivery/implementation-plan.md](../03-delivery/implementation-plan.md) exit criteria |

## What we are deliberately willing to be mediocre at

Naming these is part of the design, not a caveat.

| Characteristic | Position | Reason |
| --- | --- | --- |
| Elasticity | Weak. We size for a known peak (NFR-SCALE-1) and over-provision by a small fixed factor. | Attendance is bounded by physical gates and parking. Autoscaling machinery would cost more in complexity than the compute it saves. |
| Extreme low latency | Not pursued beyond 500 ms at the gate. | Nothing else in the domain is latency-sensitive. An animal enclosure is a minutes-scale problem. |
| Independent deployability at fine grain | Weak by choice. Five units, not thirteen. | See ADR-0002. Deployment independence is bought with operational surface, and the client cannot pay in that currency. |
| Multi-tenancy, internationalisation, white-labelling | Absent. | One estate, one language to start. Adding these speculatively is the failure mode the kata brief punishes under "appropriate level of detail". |
| Real-time personalization | Absent until Phase 4, and opt-in even then. | ADR-0009. The audience is families; the trust cost outweighs the conversion gain. |

## Characteristics per unit

Different parts of this system genuinely have different profiles. This is the honest
version of "quantum boundaries": the boundary is where the profile changes.

| Unit | Availability target | Consistency | Latency | Change rate | Data sensitivity |
| --- | --- | --- | --- | --- | --- |
| U1 Edge Node | Must work standalone, 72 h | Eventually consistent with cloud, locally authoritative | 500 ms local decision | Low (firmware-like) | Low, but physically exposed |
| U2 Ticketing and Access | 99.9% (assumption) during opening hours | Strong within the ledger, eventual toward edge | Seconds | Medium | Financial, no card data (NFR-SEC-1) |
| U3 Park Operations | 99% (assumption) | Eventual | Minutes | Medium | Low, anonymous counts only |
| U4 Animal Welfare | Alerting path must survive cloud loss | Eventual, with local rules authoritative | 60 s to keeper | Medium | Operational, some safety-relevant |
| U5 Guest Engagement and Analytics | 95% (assumption), degrades to static content | Eventual, batch | Minutes to hours | High | Personal data, opt-in only |

The rows differ most in the availability and latency columns, and that is exactly where we
drew the unit boundaries. Where the profiles were similar we merged rather than split; see
[decomposition.md](decomposition.md).

## How AI characteristics attach to this

Kata criterion 5 asks whether the AI characteristics align with the architecture. Our
answer is a constraint, not a feature: **an AI capability inherits the availability
requirement of the unit it lives in, and if it cannot meet that requirement, it does not
live there.**

| Consequence | Where enforced |
| --- | --- |
| Nothing in U1 may call a model over the network on the critical path. | ADR-0003, ADR-0012 |
| The welfare alert path is rule-based and local; the model only enriches and prioritises. | [cap-01](../02-ai-capabilities/cap-01-animal-welfare-anomaly.md), NFR-SAFE-1 |
| Generative AI is confined to U5, the least available and least safety-relevant unit. | [cap-05](../02-ai-capabilities/cap-05-guest-companion.md), ADR-0016 |
| Model unavailability is a normal operating state, not an incident. | ADR-0012, [FF-16](../04-verification/fitness-functions.md#ff-16) |
