# Architecture style decision

Decision: **a modular monolith in the cloud, plus an autonomous edge tier, connected by an
event backbone.** Recorded as ADR-0002 (decomposition) and ADR-0003 (edge tier).

The interesting part of this choice is not the cloud style. It is that the estate needs
**two** styles at once, because the edge and the cloud have incompatible constraints, and
pretending otherwise is where these designs usually go wrong.

## Candidates

| Style | Fit for disconnection (driver 1) | Fit for a 2-engineer team (driver 2) | Time to first value (driver 3) | Cost | Verdict |
| --- | --- | --- | --- | --- | --- |
| Single cloud monolith, thin edge devices | 1 - the gate stops when the uplink stops, which violates NFR-AVAIL-1 outright | 5 - one thing to run | 5 - fastest to a first release | 5 | Rejected. Fails the top driver. No amount of operational simplicity compensates. |
| Microservices in the cloud, thin edge | 1 - same failure, plus more moving parts | 1 - needs a platform team the client cannot hire (A7) | 2 - service scaffolding before any visitor sees anything | 2 | Rejected. Worst combination for this client: the cost of distribution without the benefit. |
| Fine-grained quanta (10+ independently deployable units) with a platform layer | 3 - can be made to work | 1 - the operational surface is the problem, not the design | 2 | 2 | Rejected for this client. Defensible for a large operator; see [../05-process/decision-log.md](../05-process/decision-log.md) D2, where we disagree with variant B on exactly this point. |
| Edge-autonomous tier + cloud modular monolith + event backbone | 5 - the edge is designed to be authoritative offline | 4 - two deployable shapes, five modules, one pipeline | 4 - Phase 0 ships gates and counters in about 12 weeks | 4 | **Selected.** |
| Fully decentralised, estate-hosted only, no cloud | 5 | 2 - the client would have to run servers, backups and physical security | 3 | 3 | Rejected. Cloud is explicitly permitted (F13) and removes work the client cannot absorb. Kept as the contingency if A5 (connectivity) fails badly. |

Scores are 1 to 5, higher is better, and they are judgements not measurements. The
justification is in the cell text; the number is a summary of it, not evidence.

## Why the split point is where it is

The line between the two styles is drawn at a single question: **can this decision be
wrong for 72 hours without hurting anyone or losing money?**

| Decision | Answer | Where it lives |
| --- | --- | --- |
| Admit this person through this gate | No. A closed gate is lost revenue and a queue. | Edge, authoritative |
| This enclosure is out of its safe temperature band | No. It is an animal welfare and safety matter. | Edge, rule-based, local |
| This ride is out of service | No. Staff need it now. | Edge, with cloud replication |
| This ticket was used twice | Yes. Detect on reconciliation, act commercially. | Cloud |
| This zone is more popular than that one | Yes. It is a weekly decision. | Cloud |
| This visitor may like a family pass | Yes. It is a marketing decision. | Cloud |
| This enclosure's pattern looks abnormal in a way no threshold catches | Yes, if the thresholds still run locally. | Cloud model, edge rules underneath |

Everything in the "No" rows is at the edge and is deterministic. Everything in the "Yes"
rows may be in the cloud and may use a model. That single test also produces our AI
placement policy, which is why ADR-0006 and ADR-0012 read as consequences of this table
rather than as separate philosophies.

## Event backbone

A managed message broker carries events between the edge tier and the cloud modules and
between cloud modules. We use it for three specific reasons, not as a default.

| Reason | Concrete need |
| --- | --- |
| Buffering across disconnection | Store-and-forward at the edge needs a durable receiver that tolerates a 10x burst on reconnect (NFR-SCALE-2). |
| Decoupling module release cycles | U5 analytics changes weekly; U2 ticketing changes rarely. A queue lets them move at different speeds without a shared release. |
| Replay for model training and for incident review | Every model in [../02-ai-capabilities/](../02-ai-capabilities/) is trained on the same event history the operators see, which is what makes the audit trail in NFR-AI-3 possible. |

We do **not** use it for request-response between cloud modules. Inside the cloud monolith,
modules call each other in process across published interfaces. Introducing asynchrony
where there is no asynchrony to manage adds debugging cost and buys nothing.

## What would make us change our minds

| Trigger | New style |
| --- | --- |
| Attendance exceeds 25,000 per day, or a second estate is added. | Split U5 out first (highest change rate, different scaling profile), then U2. |
| The client hires a platform team of 5 or more. | Finer decomposition becomes affordable; revisit ADR-0002. |
| Assumption A5 fails badly and there is effectively no uplink. | Move the cloud modules to an on-estate server, keep the same code, accept the loss of managed services. The module boundaries are chosen so that this is a deployment change, not a rewrite. |
