# ADR-0002: Five deployable units, not thirteen quanta

**Status:** accepted
**Deciders:** architect
**Relates to:** U6, A7, NFR-OPS-1, [../01-architecture/decomposition.md](../01-architecture/decomposition.md), D2

## Context

Analysing where architectural characteristics diverge produces many candidate boundaries in
this domain: ticketing has a consistency and audit profile nothing else shares, welfare
alerting has a safety and availability profile nothing else shares, analytics has a change
rate nothing else shares, and the edge has an availability requirement that is categorically
different. A rigorous quantum analysis lands somewhere above ten independently deployable
parts.

The client has at most 6 people in steady state, of whom at most 2 are engineers (A7,
NFR-OPS-1). There is no platform team and there will not be one.

## Decision

Keep the boundaries; do not make them all deployment boundaries. Five units: **U1 Edge
Node** as its own deployable shape, and **U2 Ticketing and Access, U3 Park Operations, U4
Animal Welfare, U5 Guest Engagement and Analytics** as modules of one cloud deployable, each
with owned data, a published interface and no cross-module database access. AI Services is a
gateway plus a library inside that deployable, not a sixth unit.

Each module has a stated split trigger. When a trigger fires, that module becomes a separate
deployable, and because it already owns its data and publishes its interface, the split is a
deployment change.

| Module | Split trigger |
| --- | --- |
| U5 | Attendance above 25,000/day, or analytics releases blocked by operations releases more than twice a quarter |
| U2 | A second estate or a second sales channel with its own release cadence |
| U4 | A second collection site with its own keeper team |
| U3 | Not expected to split |

## Alternatives considered

| Alternative | Why not |
| --- | --- |
| Thirteen quanta plus an AI platform | Thirteen pipelines, thirteen on-call surfaces and a platform capability for a 2-engineer team. The analysis is sound; the operating consequence is unaffordable. See D2, where we credit the reasoning and disagree with the conclusion. |
| A single monolith with no internal boundaries | The characteristics really do diverge, and a shared database between welfare alerting and marketing analytics would let an analytics change break an animal alert. |
| Microservices for the cloud, thin edge | Fails NFR-AVAIL-1 outright and adds distribution cost. Rejected in [../01-architecture/style-decision.md](../01-architecture/style-decision.md). |
| Serverless functions per capability | Multiplies the number of things to observe and deploy without reducing the number of things to understand. |

## Consequences

**Good**

- Two deployable shapes, one pipeline, one on-call rotation. NFR-OPS-1 is achievable rather than aspirational.
- The boundaries are preserved, so the decomposition remains a real analysis and not an absence of one.
- Splitting later is cheap, because data ownership and interfaces are already in place.

**Bad**

- A bad release in one module affects the others. Mitigated by module contract tests and a rollback target on every release ([FF-14](../04-verification/fitness-functions.md#ff-14)), not eliminated.
- Modules cannot scale independently. Accepted: the load is bounded by physical gates.
- Discipline is enforced by review rather than by the network, so it can erode.

**Risks**

| Risk | Mitigation | Where tracked |
| --- | --- | --- |
| Module boundaries erode into a big ball of mud | Schema separation, no cross-module foreign keys, contract tests per module, and a boundary check in review | [FF-14](../04-verification/fitness-functions.md#ff-14) |
| Growth outruns the decision quietly | Split triggers are explicit and reviewed at each phase gate | [../03-delivery/implementation-plan.md](../03-delivery/implementation-plan.md) |

## How we will know this was right

[FF-12](../04-verification/fitness-functions.md#ff-12) holds every month: at most two
deployable shapes, at most three pages per engineer per week, no undocumented manual
intervention. If pages per engineer exceed three per week for two consecutive months, or a
split trigger fires, this ADR is revisited. If we find ourselves adding a sixth unit
without a trigger firing, that is the signal that this decision was wrong.
