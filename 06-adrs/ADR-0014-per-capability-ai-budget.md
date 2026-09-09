# ADR-0014: Per-capability AI budget with automatic downgrade

**Status:** accepted
**Deciders:** architect, operations manager
**Relates to:** NFR-COST-1, [FF-15](../04-verification/fitness-functions.md#ff-15), [../03-delivery/cost-model.md](../03-delivery/cost-model.md), D9

## Context

Our own cost model says IT runs at 0.05% to 1.10% of revenue across every scenario we
modelled, including a very pessimistic one, and AI at 0.01% to 0.21%. **Cost is not the
binding constraint on this architecture; team capacity is.** Stating a hard AI budget as a
headline constraint would therefore be dishonest about our own arithmetic.

But four of the five capabilities are fixed-cost batch jobs and one,
[cap-05](../02-ai-capabilities/cap-05-guest-companion.md), scales with attendance and is
reachable by anyone with a browser. A runaway loop, an abuse case or a provider price change
is a real operational risk even when the baseline cost is trivial.

## Decision

Publish the arithmetic showing cost is not binding, **and** enforce a per-capability monthly
ceiling anyway. Each capability declares a ceiling with an owner; the AI Gateway enforces it
in three steps:

| Utilisation | Action |
| --- | --- |
| 70% | Alert the owner |
| 90% | Restrict to high-value calls: cached answers only for the assistant, batch-only for the others |
| 100% | Route to the deterministic fallback (ADR-0012) |

The purpose is stated explicitly so nobody mistakes it: **the budget is a circuit breaker,
not a savings measure.** It catches the runaway before it becomes an incident, in a system
where nobody is watching the billing dashboard at 3 a.m.

A monthly ceiling alone is a one-shot kill switch, which is exactly the wrong shape against
an anonymous public endpoint: an attacker who exhausts it once removes the capability for the
rest of the calendar month. The assistant therefore also carries:

| Control | Value | Reason |
| --- | --- | --- |
| Daily sub-budget | 1/30th of the monthly ceiling, with its own alert and its own fallback trip | A single bad day degrades one day, not one month. Recovery is automatic at midnight. |
| Per-session rate limit | 20 questions per session (assumption) | Ordinary use never reaches it. |
| Per-IP and per-device limits | Yes, in addition to per-session | Sessions are free to an anonymous visitor, so a per-session limit alone controls nothing. |
| Spend anomaly detector | Alert on a day exceeding 3x the trailing 7-day median | Catches scripted abuse in hours rather than at the ceiling. |

Cost per capability is tagged in billing so
[FF-15](../04-verification/fitness-functions.md#ff-15) can attribute spend.

## Alternatives considered

| Alternative | Why not |
| --- | --- |
| A single global AI budget | A runaway in one capability would silently starve the others, and attribution after the fact is guesswork. |
| No budget, since cost is not binding | Leaves a browser-reachable, per-call-cost capability with no ceiling. The failure is not the monthly bill; it is that nobody notices for two weeks. |
| Alerting only, no automatic action | A 2-person team with one rotation cannot guarantee a response within hours. The gateway acts first and a human reviews afterwards. |
| Hard cutoff with no restrict step | The middle step keeps the most valuable behaviour (cached answers to common questions) available while stopping the expensive tail, which is a much better degraded state. |

## Consequences

**Good**

- A runaway loop or an abuse case degrades a capability instead of producing a surprise invoice.
- Cost per capability is attributable, so the phase-gate question "is this capability worth its cost" has data behind it.
- The degraded state is the fallback, which is already tested ([FF-16](../04-verification/fitness-functions.md#ff-16)), so the breach path needs no separate design.

**Bad**

- Slightly more machinery than the cost arithmetic alone justifies. We think the asymmetry warrants it.
- Ceilings need setting, and a ceiling set too low degrades a working capability for no reason.
- A cost ceiling can mask a quality problem: heavy usage may mean the assistant is failing to answer and users are retrying.

**Risks**

| Risk | Mitigation | Where tracked |
| --- | --- | --- |
| Ceiling set too low, degrading a healthy capability | Ceilings are reviewed monthly against actuals and at each phase gate; the first month runs alert-only | [FF-15](../04-verification/fitness-functions.md#ff-15) |
| High spend is a symptom of poor quality, not of success | Cost is reviewed alongside the escalation and retry rates, never alone | [../04-verification/ai-validation.md](../04-verification/ai-validation.md) |
| The published "cost is not binding" message licenses unbounded additions | The real gate on additions is ADR-0006 and the operating model, not cost. Stated in D9. | [../05-process/decision-log.md](../05-process/decision-log.md) |

## How we will know this was right

[FF-15](../04-verification/fitness-functions.md#ff-15) monthly: IT stays at or below 1.5% and
AI at or below 0.3% of revenue, and no capability exceeds its ceiling without the gateway
acting before a human does. If, over a full season, no capability ever reaches 70%, the
ceilings are decorative and should be lowered to a level that would actually catch a
runaway. If a capability is repeatedly throttled while behaving correctly, the ceiling was
wrong.
