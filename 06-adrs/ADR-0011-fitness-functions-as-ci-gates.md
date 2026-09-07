# ADR-0011: Fitness functions are CI gates, owned like tests

**Status:** accepted
**Deciders:** architect
**Relates to:** every NFR, [../04-verification/fitness-functions.md](../04-verification/fitness-functions.md), D10

## Context

Criterion 6 asks how AI is validated, and the same question applies to the rest of the
system: how does anyone know the architecture still holds? Non-functional requirements decay
silently. Nobody notices that gate latency crept from 300 ms to 900 ms until there is a
queue, and nobody notices that a privacy commitment was quietly violated by a new feature
until someone asks.

In a system operated by 2 people, anything that depends on remembering to check will not be
checked.

## Decision

Every NFR has exactly one fitness function with a named metric, instrument, threshold,
cadence and breach action. They are classified as `auto` (runs in the pipeline or on a
schedule and fails loudly), `drill` (a calendar event with a named owner and a written
result) or `review` (a human judgement over automated data).

Three rules give this teeth:

1. **A new NFR without a fitness function fails the documentation check**, which compares the
   ID sets in [../00-problem/requirements.md](../00-problem/requirements.md) and
   [../04-verification/fitness-functions.md](../04-verification/fitness-functions.md).
2. **`auto` functions block the deploy** where the requirement is a `must`. The privacy,
   security, fallback and safety checks (FF-08 to FF-11, FF-16, FF-20) are release blockers,
   not dashboards.
3. **A breach either changes the system or changes the threshold, and the change is
   recorded.** A permanently red check is worse than no check, because it teaches people to
   ignore red.

Verification gets its own top-level block rather than being distributed through the
document, so that coverage is provable by reading one page.

## Alternatives considered

| Alternative | Why not |
| --- | --- |
| Verification described within each topic, as most designs do | Coverage becomes unprovable. Nobody can answer "does every NFR have a check" without reading everything, and that question is criterion 6 asked directly. |
| A quarterly architecture review by a person | A review finds what the reviewer thinks to look for, on the day they look. It complements automation; it cannot replace it for a 2-person team. |
| Monitoring and alerting only | Monitoring tells you production is broken. A fitness function stops you breaking it. The privacy checks in particular have to run before the deploy, not after. |
| Fitness functions as advisory dashboards | Advisory checks are ignored under season pressure, which is exactly when they matter. |

## Consequences

**Good**

- "How do you know" has one answer, in one place, with a coverage guarantee.
- The most consequential commitments (no biometrics, no cardholder data, a working fallback for every model, no generative safety answers) are enforced by the pipeline rather than by intention.
- The client's own team inherits the checks along with the system, so the architecture keeps holding after we leave.

**Bad**

- Real engineering effort: 22 checks to build, several needing test infrastructure (a load generator, a chaos harness, a synthetic device fleet).
- Drills consume calendar time: roughly two days per quarter.
- A blocking check on a `should` requirement would be over-enforcement, so the must/should split has to be maintained honestly.

**Risks**

| Risk | Mitigation | Where tracked |
| --- | --- | --- |
| Thresholds are relaxed to make checks pass | Every threshold change is recorded with a reason and reviewed at the phase gate | [../03-delivery/implementation-plan.md](../03-delivery/implementation-plan.md) |
| Flaky checks erode trust in the whole suite | A flaky check is a defect with the same priority as a failing one | monthly review |
| Drills are skipped under pressure | Named owner, calendar event, written result; a skipped drill is reported to the owner of the phase gate | [../04-verification/fitness-functions.md](../04-verification/fitness-functions.md) |

## How we will know this was right

The coverage check is green (22 NFRs, 22 functions, no gaps) from Phase 0, and the `auto`
subset runs on every deploy. The real test is at month 36: the client's team has operated
one full season with no external on-call and the dashboard has been green for three
consecutive months. If, at any review, more than two checks are red and have been red for
over a month, this mechanism has stopped working and needs fixing before anything else does.
