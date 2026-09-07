# ADR-0008: Provider abstraction and a two-provider rule, with a quarterly drill

**Status:** accepted
**Deciders:** architect
**Relates to:** NFR-MOD-1, [FF-21](../04-verification/fitness-functions.md#ff-21), [cap-05](../02-ai-capabilities/cap-05-guest-companion.md)

## Context

One capability, the assistant ([cap-05](../02-ai-capabilities/cap-05-guest-companion.md)),
uses an external model provider. External providers change models underneath their APIs,
change prices, deprecate versions and occasionally leave the market, on timescales much
shorter than the three-year horizon in F7. The other four capabilities run models we train,
so this decision is scoped to the one place the risk is real.

This is the "uncertainty about the supplier" case in
[../04-verification/ai-validation.md](../04-verification/ai-validation.md), and it is
distinct from uncertainty about a model's output.

## Decision

All model calls go through the AI Gateway. Any capability using an external provider must
have a **second provider configured and evaluated on the same golden set**, and prompts are
versioned artefacts, not strings embedded in code, because a provider swap changes the
prompt version and the prompt version must be re-evaluated.

A **quarterly failover drill** routes production assistant traffic to the secondary provider
for one working day, re-runs the golden set, and records the quality delta and the cost
delta. The drill exists so that the switching cost in NFR-MOD-1 is a measured number rather
than an assertion.

We state plainly that switching is not free: the prompt adapter and the re-evaluation are
real work, and the drill is what keeps the estimate honest.

## Alternatives considered

| Alternative | Why not |
| --- | --- |
| Single provider, accept lock-in | The assistant is visitor-facing; a provider deprecation with 30 days notice would be an emergency for a 2-person team. The drill costs a day a quarter. |
| Self-host an open-weights model to eliminate provider risk | Trades a commercial dependency for a GPU fleet and an operations burden, for a spiky low-volume workload. Revisit if provider cost or data residency becomes binding. |
| Abstract behind a lowest-common-denominator interface so any provider works identically | The abstraction that survives contact with two real providers is thinner than it looks, and pretending otherwise hides the switching cost instead of measuring it. We abstract the call and version the prompt, and we accept that the prompt is provider-shaped. |
| Multi-provider routing on every request | Doubles the evaluation surface and makes quality non-reproducible for a capability whose main risk is inconsistency. |

## Consequences

**Good**

- A provider withdrawal is a planned exercise rather than an incident.
- The switching cost is a measured number reviewed quarterly, which is what makes NFR-MOD-1 verifiable.
- The gateway already carries budget and fallback logic, so provider routing costs almost nothing extra.

**Bad**

- Two provider contracts and two sets of credentials.
- The golden set must be maintained well enough to be meaningful for both, which is real work.
- The drill day carries a small quality risk to visitors, which we accept and time for a quiet weekday.

**Risks**

| Risk | Mitigation | Where tracked |
| --- | --- | --- |
| The secondary provider's quality delta is too large to be a real fallback | If the delta exceeds 10% on the golden set, either make the prompt more portable or choose a different secondary. The drill surfaces this before it matters. | [FF-21](../04-verification/fitness-functions.md#ff-21) |
| A provider silently changes the model behind a version label | Golden set re-run on a schedule and on any detected version change | [../04-verification/ai-validation.md](../04-verification/ai-validation.md) |
| The drill is skipped when the team is busy | It is a calendar event with a named owner and a written result, like the other drills | [../04-verification/fitness-functions.md](../04-verification/fitness-functions.md) |

## How we will know this was right

[FF-21](../04-verification/fitness-functions.md#ff-21) quarterly from Phase 2: failover
completes within one working day, and the golden-set quality delta is within 10%. If a real
provider change ever happens and the switch takes materially longer than the drill
predicted, the drill is not exercising the real path and must be widened.
