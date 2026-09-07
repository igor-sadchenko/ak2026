# ADR-0001: Phase-gated delivery with no AI in Phase 0

**Status:** accepted
**Deciders:** architect, with the owner
**Relates to:** F7, F8, [../03-delivery/implementation-plan.md](../03-delivery/implementation-plan.md), D8

## Context

The client must reach 15,000 visitors per day within three years or sell the carnivorous
plant collection (F7, F8). The failure mode is commercial and dated. At project start there
is no attendance history, no enclosure telemetry and no baseline for any metric in
[../00-problem/okrs.md](../00-problem/okrs.md): nine of twelve key results read `unknown`.

Two consequences collide. The kata's theme is AI, which creates pressure to show AI early.
The data situation means that anything shipped in the first quarter cannot be evaluated
against anything.

## Decision

Delivery is organised into five phases, each with an exit criterion written as a condition
that can be observed and failed, and an explicit transition signal describing what must
accumulate before the next phase starts. **Phase 0 contains no models at all.** It ships
ticketing, offline-authoritative gates, zone counting and a report, and its own success is
measured by whether the estate can sell, admit, count, and produce the baseline that every
later phase is evaluated against.

## Alternatives considered

| Alternative | Why not |
| --- | --- |
| A single release at the end of year one | The owner's decision horizon is one season. A design that shows nothing until month 12 spends a third of the runway in F7 before producing evidence that it works. |
| A roadmap as a list of features by quarter | A list has no exit criteria, so nothing can fail. Every phase in our plan can be failed, which is the property that makes it a plan rather than an intention. |
| Ship a small AI capability in Phase 0 for demonstration | No data exists at month 0. The capability could not pass its own gate, which would break NFR-AI-2 in the first quarter and make every later claim about gates unbelievable. |
| Continuous delivery with no phases | The phase boundaries are not release boundaries; they are decision points about spending the client's money. Removing them removes the owner's opportunity to stop. |

## Consequences

**Good**

- The owner can stop after any phase and still hold a working system. 78,000 of 86,000 EUR of CapEx buys estate operations, not AI.
- Every AI capability arrives after the data it needs exists, so every one can be evaluated.
- The plan states its own kill switch: if attendance is flat at month 24, the software programme stops (see the assumption-failure table).

**Bad**

- Some Phase 0 code is knowingly replaced in Phase 2 (the rules layer stays, but the alerting UI evolves).
- Reads as unambitious to a reader who skims the phase table and looks for AI in the first row.
- The three-year shape means the most interesting capabilities are furthest from the submission date.

**Risks**

| Risk | Mitigation | Where tracked |
| --- | --- | --- |
| A phase exit is fudged under season pressure | Exit criteria are witnessed tests with recorded results, not sign-offs | [../04-verification/fitness-functions.md](../04-verification/fitness-functions.md) |
| The owner loses patience before Phase 2 | Phase 0 and 1 both close brief requirements (F9) on their own | [../00-problem/okrs.md](../00-problem/okrs.md) |

## How we will know this was right

At month 3, all four Phase 0 exit conditions pass in a witnessed test on the live estate,
and the `unknown` column in the OKR table is filled. If Phase 0 cannot pass its own exit
criterion by month 4, the phase model is not the problem, but our estimation is, and the
remaining phase durations are re-derived from the actual Phase 0 velocity before Phase 1
starts.
