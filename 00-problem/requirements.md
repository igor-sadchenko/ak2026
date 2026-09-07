# Requirements

Two rules apply to this page:

1. Every functional requirement carries its **source**: `F9` (stated in brief section D),
   `derived` (a direct consequence of a stated requirement), or `(assumption)`.
2. Every non-functional requirement carries a **verified by** column pointing at a fitness
   function in [../04-verification/fitness-functions.md](../04-verification/fitness-functions.md).
   A requirement with no way to check it is not a requirement; it is a wish, and we cut it.

## Functional requirements

| ID | Requirement | Source | Delivered in | Realised by |
| --- | --- | --- | --- | --- |
| FR-1 | Sell single-visit tickets online and at the gate. | F9 | Phase 0 | U2 Ticketing and Access |
| FR-2 | Sell and manage family passes (multi-visit, multi-person). | F9 | Phase 0 | U2 |
| FR-3 | Validate an entitlement at a gate and admit or refuse, with no dependency on the uplink. | derived from FR-1, FR-2, F10 | Phase 0 | U1 Edge Node, U2, ADR-0003 |
| FR-4 | Reconcile offline gate decisions with the ledger once connectivity returns, including duplicate-use detection. | derived from FR-3 | Phase 0 | U2, ADR-0003 |
| FR-5 | Count and report visitor presence per zone over time. | F9 ("understanding of area popularity") | Phase 0 | U3 Park Operations |
| FR-6 | Report ride availability and downtime for the 40 rides. | derived from F3, F9 | Phase 0 | U3 |
| FR-7 | Monitor enclosure environmental conditions (temperature, humidity, water quality where applicable) for 55 enclosures. | F9 | Phase 1 | U4 Animal Welfare |
| FR-8 | Record and schedule feeding per enclosure, and flag missed or abnormal feeding. | F9 | Phase 1 | U4 |
| FR-9 | Estimate the piranha population per tank and raise an alert when it leaves the agreed band. | F9 | Phase 1 | U4, [cap-02](../02-ai-capabilities/cap-02-piranha-population.md) |
| FR-10 | Raise a welfare alert to a named human with a reason and the evidence behind it. | derived from F9 | Phase 1 (rules), Phase 2 (model) | U4, [cap-01](../02-ai-capabilities/cap-01-animal-welfare-anomaly.md) |
| FR-11 | Dispatch and track operational tasks (feeding, welfare check, ride fault) to staff, including on a phone with no signal. | derived from FR-8, FR-10, F10 | Phase 1 | U1, U3 |
| FR-12 | Answer visitor questions about the estate (opening times, where things are, what is safe to touch) in natural language. | derived from F9 (return rate, area popularity) | Phase 2 | U5, [cap-05](../02-ai-capabilities/cap-05-guest-companion.md) |
| FR-13 | Forecast visitor volume per zone and per hour for the next 1 to 7 days. | derived from F9 (attendance and profitability growth) | Phase 3 | U5, [cap-03](../02-ai-capabilities/cap-03-visitor-flow-forecast.md) |
| FR-14 | Recommend a next-visit or pass-upgrade offer to a visitor who has opted in. | F9 (return rate) | Phase 4 | U5, [cap-04](../02-ai-capabilities/cap-04-return-visit-offers.md) |
| FR-15 | Report the commercial picture: revenue, attendance, pass share, repeat rate, per day and per zone. | F9 (profitability) | Phase 0 (basic), Phase 3 (full) | U5 |
| FR-16 | Allow a visitor to opt in to personalization and to erase that data on request. | derived from ADR-0009 | Phase 4 | U5, U2 |

Explicitly **not** a functional requirement, despite being technically attractive:
dynamic per-visitor pricing, autonomous ground transport, and RFID location tracking.
Reasons are in [../05-process/decision-log.md](../05-process/decision-log.md).

## Non-functional requirements

Priority: `must` blocks a phase exit; `should` is tracked but does not block.

| ID | Requirement | Priority | Driver | Verified by | Related decision |
| --- | --- | --- | --- | --- | --- |
| NFR-AVAIL-1 | Gate entry continues to work with the cloud uplink down for 72 hours. | must | F10, F11 | [FF-01](../04-verification/fitness-functions.md#ff-01) | ADR-0003 |
| NFR-AVAIL-2 | On-site ticket sales continue during a cloud outage, with deferred settlement. | must | F10 | [FF-02](../04-verification/fitness-functions.md#ff-02) | ADR-0003 |
| NFR-PERF-1 | Gate decision (scan to green light) at most 500 ms at p95, measured on the terminal. | must | Queue throughput | [FF-03](../04-verification/fitness-functions.md#ff-03) | ADR-0003 |
| NFR-PERF-2 | An enclosure threshold breach reaches a keeper's device within 60 s while the uplink is down. | must | F4, safety | [FF-04](../04-verification/fitness-functions.md#ff-04) | ADR-0012 |
| NFR-SCALE-1 | Sustain 15,000 admissions per day with a peak of 3,000 entries per hour. | must | F7 | [FF-05](../04-verification/fitness-functions.md#ff-05) | ADR-0002 |
| NFR-SCALE-2 | Ingest sustained telemetry from at least 600 devices at 1 message per device per minute, with 10x burst tolerance on reconnect. | must | F12 | [FF-06](../04-verification/fitness-functions.md#ff-06) | ADR-0004 |
| NFR-DATA-1 | No telemetry loss for outages up to 72 hours; ordering preserved per device. | must | F10 | [FF-07](../04-verification/fitness-functions.md#ff-07) | ADR-0003 |
| NFR-SEC-1 | No cardholder data is processed, stored or transmitted by our systems. | must | Cost of PCI scope | [FF-08](../04-verification/fitness-functions.md#ff-08) | ADR-0005 |
| NFR-SEC-2 | Every device-to-cloud connection is mutually authenticated; a stolen edge node cannot read another zone's data. | must | Physical exposure | [FF-09](../04-verification/fitness-functions.md#ff-09) | ADR-0004 |
| NFR-PRV-1 | No biometric identification of visitors anywhere in the system. | must | Family audience, U7 | [FF-10](../04-verification/fitness-functions.md#ff-10) | ADR-0009 |
| NFR-PRV-2 | Personal data is collected only after opt-in and erased within 30 days of a request. | must | U7 | [FF-11](../04-verification/fitness-functions.md#ff-11) | ADR-0009 |
| NFR-OPS-1 | The whole system is operable by at most 6 people, of whom at most 2 are engineers. | must | U6 | [FF-12](../04-verification/fitness-functions.md#ff-12) | ADR-0002, ADR-0005 |
| NFR-OPS-2 | A failed edge node is restored by a non-engineer following a printed runbook in 30 minutes or less. | must | Seasonal staff | [FF-13](../04-verification/fitness-functions.md#ff-13) | ADR-0003 |
| NFR-OPS-3 | Any component can be deployed by one person from a pipeline, with rollback, without vendor involvement. | should | U6 | [FF-14](../04-verification/fitness-functions.md#ff-14) | ADR-0005 |
| NFR-COST-1 | Total IT operating cost stays at or below 1.5% of revenue; AI inference and platform at or below 0.3%. | must | F8 | [FF-15](../04-verification/fitness-functions.md#ff-15) | ADR-0014 |
| NFR-AI-1 | Every AI capability has a deterministic fallback that keeps the business function running when the model is unavailable, wrong or too expensive. | must | Kata criterion 4 | [FF-16](../04-verification/fitness-functions.md#ff-16) | ADR-0012 |
| NFR-AI-2 | No model reaches production without passing its golden-set gate and a shadow period. | must | Kata criterion 6 | [FF-17](../04-verification/fitness-functions.md#ff-17) | ADR-0007 |
| NFR-AI-3 | Every AI output carries `{result, confidence, basis}` and is retained for audit for 12 months. | must | Kata criterion 4 | [FF-18](../04-verification/fitness-functions.md#ff-18) | ADR-0010 |
| NFR-AI-4 | Input drift and quality regression are detected within 7 days and trigger an automatic rollback to the previous model or to the fallback. | must | Kata criterion 6 | [FF-19](../04-verification/fitness-functions.md#ff-19) | ADR-0007 |
| NFR-AI-5 | Safety-relevant decisions (containment, ride status, dangerous-animal alerts) are never produced by a generative model and never depend on the cloud. | must | F3, F4 | [FF-20](../04-verification/fitness-functions.md#ff-20) | ADR-0012 |
| NFR-MOD-1 | A model provider can be replaced within 5 working days, including re-evaluation, with a documented cost. | should | Kata criterion 4 | [FF-21](../04-verification/fitness-functions.md#ff-21) | ADR-0008 |
| NFR-USE-1 | A first-time visitor completes a ticket purchase in at most 3 minutes on a phone on a weak connection. | should | F7 | [FF-22](../04-verification/fitness-functions.md#ff-22) | - |

## Traceability

The full chain is requirement -> unit -> ADR -> risk -> fitness function. Entry points:

- Requirement to unit: the "Realised by" column above and
  [../01-architecture/decomposition.md](../01-architecture/decomposition.md).
- Requirement to phase: [../03-delivery/implementation-plan.md](../03-delivery/implementation-plan.md).
- NFR to check: [../04-verification/fitness-functions.md](../04-verification/fitness-functions.md).
- Decision to consequence: [../06-adrs/README.md](../06-adrs/README.md).
