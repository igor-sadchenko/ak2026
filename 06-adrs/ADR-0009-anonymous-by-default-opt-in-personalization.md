# ADR-0009: Anonymous by default, personalisation opt-in only

**Status:** accepted
**Deciders:** architect, with the owner
**Relates to:** NFR-PRV-1, NFR-PRV-2, FR-16, A8, U7, D1

## Context

The requirement to understand area popularity (F9) and the requirement to raise the return
rate (F9) pull in different directions on visitor data. Understanding popularity needs
counts. Raising returns needs to contact people. The audience is families with children
(assumption, from the nature of the attraction), the jurisdiction is unknown (U7), and we
assume a GDPR-like regime (A8).

The tempting design is a token or wristband carrying identity and location, which enables
per-visitor journey analysis and a retention flywheel.

## Decision

Three rules, in force across every unit and every phase:

1. **No biometric identification, anywhere, ever.** No face recognition, no gait analysis,
   no re-identification of a person across cameras. Not deferred; excluded (NFR-PRV-1).
2. **Zone understanding is built from anonymous counts.** Entries and exits per zone per
   15-minute window, with no persistent person identifier. Dwell time is derived from
   aggregate flow, not from following an individual.
3. **Personalisation is opt-in and confined to U5.** A visitor who opts in gets a
   pseudonymous key issued by U2; personal data exists in one unit, is used by one
   capability ([cap-04](../02-ai-capabilities/cap-04-return-visit-offers.md)), and is
   erasable within 30 days by a single-unit operation (NFR-PRV-2).

No individual location tracking, opted in or not.

## Alternatives considered

| Alternative | Why not |
| --- | --- |
| Full identification with RFID tokens and location, as a retention flywheel | Continuous location data on children, a large consent obligation, and a trust cost that lands on the same return-rate metric it is meant to raise. Variant B in our team chooses this; the disagreement is set out honestly in D1, and their commercial argument is real. |
| Strict anonymity with no personalisation at all | Loses FR-14 and most of objective O3. Variant A in our team chooses this. Our position takes its default and adds a consented path. |
| Collect now, decide later ("we may need the data") | The most common and the worst option: it creates the obligation before the use case and makes erasure a retrofit. |
| Location tracking for opted-in visitors only | Even with consent, tracking families through a park is a different category of collection from remembering that someone visited. The marginal modelling gain does not justify it ([cap-04](../02-ai-capabilities/cap-04-return-visit-offers.md) alternatives). |

## Consequences

**Good**

- The privacy blast radius is one unit and one phase. An erasure request is a single operation, which is what makes NFR-PRV-2 achievable rather than aspirational.
- "We do not track you" is a statement the estate can make truthfully to families, which is itself worth something on the objective we are trying to move.
- Phases 0 through 3 hold no personal data at all, so three years of the plan carry no privacy risk.

**Bad**

- [cap-04](../02-ai-capabilities/cap-04-return-visit-offers.md) works on a subset (assumption: 20-35% opt-in), which reduces its reach and its statistical power.
- No per-visitor journey analysis, so some genuine operational insight is unavailable.
- If the retention target cannot be met on an opted-in subset, this decision has a cost we will have to own.

**Risks**

| Risk | Mitigation | Where tracked |
| --- | --- | --- |
| Opt-in rate too low for the capability to work | The Phase 4 dependency requires at least 2,000 consenting holders before the model ships; below that, a rules-based segment offer instead | [../03-delivery/implementation-plan.md](../03-delivery/implementation-plan.md) |
| Scope creep: a future feature quietly needs identity | [FF-10](../04-verification/fitness-functions.md#ff-10) fails the build if identifying features appear; [FF-11](../04-verification/fitness-functions.md#ff-11) fails on personal records without consent | pipeline |
| Commercial pressure to widen collection | The honest response is to reopen this ADR in the decision log, not to widen quietly | [../05-process/decision-log.md](../05-process/decision-log.md) |

## How we will know this was right

[FF-10](../04-verification/fitness-functions.md#ff-10) and
[FF-11](../04-verification/fitness-functions.md#ff-11) stay green on every deploy from Phase
0, and the Phase 4 erasure drill completes end to end. The commercial test comes at month
36: if the return-visit objective is met on an opted-in subset, the decision cost nothing. If
it is missed and the analysis shows reach was the binding factor, this ADR is where the
conversation restarts.
