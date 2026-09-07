# Fitness functions

A requirement with no way to check it is not a requirement. Every NFR in
[../00-problem/requirements.md](../00-problem/requirements.md) has exactly one fitness
function here, and every fitness function names a threshold, a cadence, and what happens
when it is breached.

Automation status: `auto` runs in the pipeline or on a schedule with no human;
`drill` is a rehearsed manual exercise; `review` is a human judgement over automated data.

## Index

| ID | NFR | What it checks | Cadence | Mode | From phase |
| --- | --- | --- | --- | --- | --- |
| [FF-01](#ff-01) | NFR-AVAIL-1 | Gates admit with the uplink down for 72 h | Quarterly | drill | 0 |
| [FF-02](#ff-02) | NFR-AVAIL-2 | Ticket sales survive a cloud outage | Quarterly | drill | 0 |
| [FF-03](#ff-03) | NFR-PERF-1 | Scan-to-decision p95 under 500 ms | Every deploy + daily | auto | 0 |
| [FF-04](#ff-04) | NFR-PERF-2 | Threshold breach reaches a keeper in 60 s offline | Weekly | auto | 1 |
| [FF-05](#ff-05) | NFR-SCALE-1 | 3,000 entries/hour across the fleet | Pre-season + on change | auto | 0 |
| [FF-06](#ff-06) | NFR-SCALE-2 | Telemetry ingest and 10x reconnect burst | Weekly | auto | 1 |
| [FF-07](#ff-07) | NFR-DATA-1 | No telemetry loss up to 72 h, ordering preserved | Continuous | auto | 0 |
| [FF-08](#ff-08) | NFR-SEC-1 | No cardholder data anywhere in our systems | Every deploy | auto | 0 |
| [FF-09](#ff-09) | NFR-SEC-2 | Mutual auth; a stolen node reads only its own zone | Every deploy + quarterly | auto + drill | 0 |
| [FF-10](#ff-10) | NFR-PRV-1 | No biometric identification exists in the codebase or data | Every deploy | auto | 0 |
| [FF-11](#ff-11) | NFR-PRV-2 | Opt-in gating and 30-day erasure | Every deploy + quarterly | auto + drill | 4 |
| [FF-12](#ff-12) | NFR-OPS-1 | Operable by <= 6 people, <= 2 engineers | Monthly | review | 0 |
| [FF-13](#ff-13) | NFR-OPS-2 | Non-engineer cold restore in 30 min | Quarterly | drill | 0 |
| [FF-14](#ff-14) | NFR-OPS-3 | One-person deploy and rollback; event schemas stay compatible | Every deploy | auto | 0 |
| [FF-15](#ff-15) | NFR-COST-1 | IT <= 1.5% and AI <= 0.3% of revenue | Monthly | auto + review | 0 |
| [FF-16](#ff-16) | NFR-AI-1 | Every capability has a working deterministic fallback | Every deploy + monthly | auto + drill | 1 |
| [FF-17](#ff-17) | NFR-AI-2 | No model in production without a passed gate | Every promotion | auto | 1 |
| [FF-18](#ff-18) | NFR-AI-3 | Every AI output carries result, confidence, basis, and is retained | Continuous | auto | 1 |
| [FF-19](#ff-19) | NFR-AI-4 | Drift and quality regression detected within 7 days | Weekly | auto | 2 |
| [FF-20](#ff-20) | NFR-AI-5 | No safety decision from a generative model or from the cloud | Every deploy | auto | 1 |
| [FF-21](#ff-21) | NFR-MOD-1 | Provider replaceable in 5 working days | Quarterly | drill | 2 |
| [FF-22](#ff-22) | NFR-USE-1 | Purchase completed in 3 minutes on a weak connection | Monthly | auto | 0 |

Coverage check: 22 NFRs, 22 fitness functions, no gaps. This is verified mechanically as
part of the documentation check described at the end of this page.

---

### FF-01

| Field | Value |
| --- | --- |
| Requirement | NFR-AVAIL-1 (gate entry survives 72 h without the uplink) |
| Metric | Admission success rate and reconciliation conflict rate during a simulated outage |
| Instrument | Scheduled chaos exercise: cellular interface disabled on all edge nodes; synthetic and real scans; replay measured on reconnect |
| Threshold | 100% of valid entitlements admitted; conflict rate below 0.5% of redemptions |
| Cadence | Quarterly, plus once before each season opening |
| On breach | Season opening is blocked until fixed. This is a P1 defect: it invalidates the top architectural driver. |

### FF-02

| Field | Value |
| --- | --- |
| Requirement | NFR-AVAIL-2 (on-site sales survive a cloud outage) |
| Metric | Offline sales completed and later settled without loss |
| Instrument | Same exercise as FF-01; 50 offline sales issued, then settlement reconciled |
| Threshold | 100% settle; zero duplicate charges; zero lost entitlements |
| Cadence | Quarterly |
| On breach | Fall back to the printed manual sales procedure and treat as P1. |

### FF-03

| Field | Value |
| --- | --- |
| Requirement | NFR-PERF-1 (500 ms p95 scan to decision) |
| Metric | p95 and p99 of scan-to-decision latency, measured on the terminal, not on the server |
| Instrument | Terminal-side timing emitted with every `GateAdmission` event; pipeline test against a reference terminal |
| Threshold | p95 <= 500 ms, p99 <= 1,200 ms |
| Cadence | Every deploy (synthetic) and daily (production percentiles) |
| On breach | Block the deploy. In production, alert and investigate the same day: gate latency turns directly into queue length. |

### FF-04

| Field | Value |
| --- | --- |
| Requirement | NFR-PERF-2 (60 s from breach to keeper, offline) |
| Metric | Time from a synthetic out-of-band reading to acknowledged delivery on a keeper device |
| Instrument | Weekly synthetic breach injected on each edge node, with the uplink disabled for the test window |
| Threshold | <= 60 s on every node; zero missed injections |
| Cadence | Weekly, all nodes |
| On breach | Page. Keepers revert to manual rounds for the affected cluster until green. This is a safety path (NFR-SAFE-1). |

### FF-05

| Field | Value |
| --- | --- |
| Requirement | NFR-SCALE-1 (15,000/day, 3,000 entries/hour peak) |
| Metric | Sustained admissions per hour across the gate fleet with latency held |
| Instrument | Load generator against staging with the production gate count, plus a replay of the busiest real day at 3x |
| Threshold | 3,000/hour sustained for 2 hours with FF-03 thresholds still met |
| Cadence | Before each season, and after any change to the entitlement or allow-list path |
| On breach | Add a scan lane rather than re-architect; if the bottleneck is server-side, block the release. |

### FF-06

| Field | Value |
| --- | --- |
| Requirement | NFR-SCALE-2 (600 devices, 10x reconnect burst) |
| Metric | Ingest throughput, broker queue depth, end-to-end lag |
| Instrument | Synthetic device fleet; a reconnect burst test that releases 72 h of buffered messages from 8 nodes at once |
| Threshold | Backlog cleared within 60 minutes; no message loss; no consumer restart |
| Cadence | Weekly synthetic, full burst test quarterly |
| On breach | Increase consumer parallelism or broker tier. Cost impact recorded against FF-15. |

### FF-07

| Field | Value |
| --- | --- |
| Requirement | NFR-DATA-1 (no telemetry loss up to 72 h; per-device ordering) |
| Metric | Gap count per device per day; out-of-order arrival count |
| Instrument | Every device emits a monotonic sequence number; a continuous job detects gaps and inversions |
| Threshold | Zero unexplained gaps; every gap over 1 hour has a recorded cause; zero ordering violations per device |
| Cadence | Continuous, reported daily |
| On breach | Investigate the node. Sustained gaps are a Phase 1 exit blocker. |

### FF-08

| Field | Value |
| --- | --- |
| Requirement | NFR-SEC-1 (no cardholder data) |
| Metric | Presence of card-shaped data in code, logs, database or events |
| Instrument | Pipeline scanner: PAN-pattern detection with Luhn validation across the repository, log sinks and a sample of stored payloads |
| Threshold | Zero findings |
| Cadence | Every deploy, plus a weekly scan of production logs |
| On breach | Block the deploy. A finding in production is a security incident with a defined purge procedure. |

### FF-09

| Field | Value |
| --- | --- |
| Requirement | NFR-SEC-2 (mutual auth; zone isolation) |
| Metric | Rejected connections without a valid client certificate; cross-zone access attempts |
| Instrument | Automated test that a node certificate from zone A cannot subscribe to zone B topics; certificate expiry monitor |
| Threshold | 100% rejection of unauthenticated and cross-zone attempts; no certificate within 30 days of expiry unrenewed |
| Cadence | Every deploy; a physical "stolen node" drill quarterly |
| On breach | Block the deploy; revoke and reissue certificates. |

### FF-10

| Field | Value |
| --- | --- |
| Requirement | NFR-PRV-1 (no biometric identification) |
| Metric | Presence of face or gait recognition libraries, models, or person-identifying features anywhere in the system |
| Instrument | Dependency allow-list in the pipeline; a schema check that no table or event carries a persistent person identifier derived from sensing |
| Threshold | Zero findings |
| Cadence | Every deploy |
| On breach | Block the deploy. This is a stated architectural commitment (ADR-0009), not a tunable. |

### FF-11

| Field | Value |
| --- | --- |
| Requirement | NFR-PRV-2 (opt-in only; erasure within 30 days) |
| Metric | Personal records without a recorded consent; time from erasure request to full removal |
| Instrument | Nightly job scanning for personal records lacking a consent reference; a quarterly end-to-end erasure drill including backups and the analytics store |
| Threshold | Zero records without consent; erasure completed within 30 days, verified by search |
| Cadence | Nightly (auto) and quarterly (drill) |
| On breach | Suspend the offers capability until resolved; the fallback is untargeted promotion, so the business function continues. |

### FF-12

| Field | Value |
| --- | --- |
| Requirement | NFR-OPS-1 (operable by <= 6 people, <= 2 engineers) |
| Metric | Number of deployable units on call; pages per engineer per week; number of distinct runbooks required in a month |
| Instrument | On-call statistics plus a monthly review against the operating model |
| Threshold | <= 2 deployable shapes; <= 3 pages per engineer per week averaged over a month; no undocumented manual intervention |
| Cadence | Monthly review |
| On breach | The response is architectural, not operational: remove a component or automate a runbook. A sustained breach is grounds to revisit ADR-0002. |

### FF-13

| Field | Value |
| --- | --- |
| Requirement | NFR-OPS-2 (non-engineer cold restore in 30 minutes) |
| Metric | Wall-clock time for a member of estate staff to restore a failed edge node or gate terminal using only the printed runbook |
| Instrument | Timed drill with a deliberately bricked device and no engineer present |
| Threshold | <= 30 minutes, and the staff member did not need to phone anyone |
| Cadence | Quarterly, and with every new seasonal staff intake |
| On breach | Rewrite the runbook, not the person. Two consecutive failures mean the node design is too clever. |

### FF-14

| Field | Value |
| --- | --- |
| Requirement | NFR-OPS-3 (one-person deploy with rollback; contracts stay compatible) |
| Metric | Deploy duration and steps requiring a second person; event schema compatibility |
| Instrument | Pipeline enforces: no manual step, a rollback target for every release, and a schema check rejecting any non-additive change to a published event within a major version ([../01-architecture/core-platform.md](../01-architecture/core-platform.md)) |
| Threshold | Zero manual steps; rollback demonstrated in staging on every release; zero incompatible schema changes |
| Cadence | Every deploy |
| On breach | Block the deploy. |

### FF-15

| Field | Value |
| --- | --- |
| Requirement | NFR-COST-1 (IT <= 1.5%, AI <= 0.3% of revenue) |
| Metric | Monthly IT spend and AI spend, each divided by monthly revenue; spend per capability against its declared ceiling |
| Instrument | Cloud and provider billing tagged per capability, joined to revenue from U2 |
| Threshold | Alert at 70% of a capability ceiling, restrict at 90%, fall back at 100% (ADR-0014) |
| Cadence | Automated daily accrual, reviewed monthly |
| On breach | The gateway acts automatically before a human sees it; the review decides whether the ceiling or the design was wrong. |

### FF-16

| Field | Value |
| --- | --- |
| Requirement | NFR-AI-1 (every capability has a deterministic fallback) |
| Metric | Existence and correctness of a declared fallback; `fallback_used` rate in production |
| Instrument | A pipeline test per capability that disables the model and asserts the business function still completes with a correct deterministic result; production metric on `fallback_used` |
| Threshold | Every capability has a passing fallback test; a monthly game-day disables one capability at random in production with no operational interruption |
| Cadence | Every deploy (test), monthly (game day) |
| On breach | Block the deploy. A capability without a working fallback may not be promoted, regardless of its accuracy. |

### FF-17

| Field | Value |
| --- | --- |
| Requirement | NFR-AI-2 (no model in production without a passed gate) |
| Metric | Promotion records with a linked, signed evaluation artefact |
| Instrument | The registry refuses a production binding whose evaluation artefact is missing, failing, or older than the model artefact |
| Threshold | 100% of production model versions have a passing gate and a named approver |
| Cadence | Every promotion; audited monthly |
| On breach | Automatic rollback to the previous bound version. Details in [ai-validation.md](ai-validation.md). |

### FF-18

| Field | Value |
| --- | --- |
| Requirement | NFR-AI-3 (`{result, confidence, basis}` and 12-month retention) |
| Metric | Share of AI responses with a complete contract and a stored audit record |
| Instrument | Gateway-side schema validation on every response; a nightly retention and completeness check |
| Threshold | 100% complete; zero records lost inside the retention window |
| Cadence | Continuous |
| On breach | The gateway rejects the malformed response and serves the fallback, so a contract failure degrades rather than propagates. |

### FF-19

| Field | Value |
| --- | --- |
| Requirement | NFR-AI-4 (drift and regression detected within 7 days) |
| Metric | Input distribution distance against the training window; quality proxy per capability; human override rate |
| Instrument | Weekly drift job; override rate from the operational UI; per-capability quality proxies defined in each capability file |
| Threshold | Per capability; for welfare, override rate above 40% over 2 weeks or a drift distance above the calibrated bound triggers action |
| Cadence | Weekly |
| On breach | Automatic rollback to the previous version, or to the deterministic fallback if no previous version qualifies, plus a retraining ticket. |

### FF-20

| Field | Value |
| --- | --- |
| Requirement | NFR-AI-5 (no safety decision from generative AI or from the cloud) |
| Metric | Call graph from any safety-relevant output back to its sources |
| Instrument | Static check: the modules that emit welfare alerts, containment warnings and ride status may not import the AI gateway's generative client, and may not make a network call on the alerting path. Plus a red-team suite that tries to make the assistant answer a safety question. |
| Threshold | Zero violations; the assistant refuses or defers 100% of red-team safety prompts to a fixed, human-written answer |
| Cadence | Every deploy |
| On breach | Block the deploy. This is the line the regulator and the keepers both care about. |

### FF-21

| Field | Value |
| --- | --- |
| Requirement | NFR-MOD-1 (provider replaceable in 5 working days) |
| Metric | Elapsed time and quality delta of a full failover to the secondary provider |
| Instrument | Quarterly drill: route production assistant traffic to the secondary provider for one day, re-run the golden set, record quality and cost delta |
| Threshold | Failover completed within one working day at the drill; full replacement including re-evaluation estimated at <= 5 working days; quality delta on the golden set within 10% |
| Cadence | Quarterly |
| On breach | If the delta exceeds 10%, the secondary is not a viable fallback and either the prompt is made more portable or a different secondary is chosen. Honest reporting of this number is the point of the drill (ADR-0008). |

### FF-22

| Field | Value |
| --- | --- |
| Requirement | NFR-USE-1 (purchase in 3 minutes on a weak connection) |
| Metric | Time to complete a first purchase on a throttled connection |
| Instrument | Synthetic browser journey on a simulated 3G profile with 300 ms added latency |
| Threshold | <= 3 minutes at p90; page weight budget respected |
| Cadence | Monthly and on any change to the purchase flow |
| On breach | Not a release blocker (this NFR is `should`), but it is tracked against O1, since pre-booked share is a Phase 0 key result. |

---

## How this stays alive

| Rule | Reason |
| --- | --- |
| `auto` functions run in the pipeline or on a schedule and fail loudly. Nobody has to remember them. | A fitness function that depends on discipline decays in a season. |
| `drill` functions are calendar events with a named owner and a written result. | The 72-hour outage test cannot be automated safely on a live estate, so it is scheduled instead. |
| A new NFR without a fitness function fails the documentation check. | The check parses [../00-problem/requirements.md](../00-problem/requirements.md) and this file and compares the two ID sets. |
| A breach either changes the system or changes the threshold, and the change is recorded. | A permanently red check is worse than no check, because it teaches people to ignore red. |
| The monthly review is the same meeting as the OKR review. | Two dashboards that nobody reads are worse than one that somebody does. |
