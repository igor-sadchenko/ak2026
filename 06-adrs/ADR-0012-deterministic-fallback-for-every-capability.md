# ADR-0012: A deterministic fallback for every AI capability

**Status:** accepted
**Deciders:** architect
**Relates to:** NFR-AI-1, NFR-AI-5 (NFR-SAFE-1), [FF-16](../04-verification/fitness-functions.md#ff-16), [FF-20](../04-verification/fitness-functions.md#ff-20)

## Context

Models become unavailable, degrade, drift, exceed budgets and get rolled back. In this
system they also sit behind a constrained uplink (F10), which means "unavailable" is a
normal weekly state, not an incident. Meanwhile the client has 2 engineers and one on-call
rotation, and some of what the models touch is safety-relevant: dangerous animals (F4) and
historic rides (F3).

If a model failure can stop a business function, then every model is an availability
liability, and a 2-person team cannot carry five of those.

## Decision

**Every capability declares a deterministic fallback that is a working business behaviour,
not an error message.** The fallback is tested in CI like any other code path
([FF-16](../04-verification/fitness-functions.md#ff-16)) and exercised in a monthly game day
that disables one capability at random in production.

| Capability | Fallback |
| --- | --- |
| [cap-01](../02-ai-capabilities/cap-01-animal-welfare-anomaly.md) welfare anomaly | Threshold rules on the edge node, permanently running underneath |
| [cap-02](../02-ai-capabilities/cap-02-piranha-population.md) piranha counting | Scheduled manual counts with a documented sampling protocol |
| [cap-03](../02-ai-capabilities/cap-03-visitor-flow-forecast.md) forecasting | Last-year-same-weekday plus a weather adjustment, running continuously in parallel |
| [cap-04](../02-ai-capabilities/cap-04-return-visit-offers.md) offers | Untargeted seasonal offer to all opted-in holders |
| [cap-05](../02-ai-capabilities/cap-05-guest-companion.md) assistant | Cached answers plus search over the same corpus, then the static FAQ |

Two consequences follow, and they are the reason this ADR exists rather than being a line in
each capability file:

- **A model failure is never a page.** It is a next-business-day P3, because the fallback is
  already carrying the business function. This is what makes an AI-bearing system operable
  by two people.
- **Safety-relevant decisions are produced only by the deterministic path.** Welfare alerts,
  containment warnings and ride status never come from a generative model and never require
  a network call (NFR-AI-5). Enforced statically at build time
  ([FF-20](../04-verification/fitness-functions.md#ff-20)).

## Alternatives considered

| Alternative | Why not |
| --- | --- |
| Retry, then show an error | An error is not a business behaviour. The keeper still needs to know the enclosure is too hot. |
| Fall back to a second model | Correlated failure: a bad input, a drift event or a data quality problem breaks both. The fallback must be a different *kind* of thing, not a different weight file. |
| Cache the last model output | Stale confident output is the worst failure mode available, especially for welfare, where the whole point is detecting change. |
| Accept downtime for AI features and treat them as optional extras | Fine for the assistant, unacceptable for welfare, and the distinction is easier to get wrong than to design away. One rule for all five is cheaper to enforce than a per-capability judgement. |

## Consequences

**Good**

- No AI capability is on the critical path of an availability requirement.
- The degradation ladders in the capability files are real, tested behaviours rather than prose.
- The rules layer built in Phase 1 has a permanent purpose, which is why it is not scaffolding.

**Bad**

- Two implementations of every capability's business function, both maintained.
- The fallback rarely runs in anger, so it needs deliberate exercise to stay working. Hence the monthly game day.
- Some capability value is genuinely lost during fallback, and the estate feels it.

**Risks**

| Risk | Mitigation | Where tracked |
| --- | --- | --- |
| The fallback rots because it is never used | Monthly game day disabling a random capability in production; CI test on every deploy | [FF-16](../04-verification/fitness-functions.md#ff-16) |
| Users do not notice they are in fallback and misread the output | `fallback_used` is surfaced in the UI, and the dashboard shows the capability as degraded | [../01-architecture/ai-platform.md](../01-architecture/ai-platform.md) |
| A future capability ships without a fallback under deadline pressure | The CI test blocks promotion; a capability without a passing fallback cannot bind, regardless of accuracy | [FF-16](../04-verification/fitness-functions.md#ff-16) |

## How we will know this was right

[FF-16](../04-verification/fitness-functions.md#ff-16): every capability has a passing
fallback test on every deploy, and the monthly game day disables one in production with no
operational interruption, from Phase 1 onward. The operational proof is in the on-call log:
if a model failure ever generates a P1 page, this ADR was not implemented properly.
