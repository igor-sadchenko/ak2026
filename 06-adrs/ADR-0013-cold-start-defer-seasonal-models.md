# ADR-0013: Cold start - defer seasonal models until the history exists

**Status:** accepted
**Deciders:** architect
**Relates to:** A9, FR-13, [cap-03](../02-ai-capabilities/cap-03-visitor-flow-forecast.md), [../03-delivery/implementation-plan.md](../03-delivery/implementation-plan.md), D8

## Context

The estate has just changed hands. There is no attendance history, no enclosure telemetry
and no offer-response history (A9). Three of the five capabilities depend on history that
does not exist:

| Capability | Needs | First available |
| --- | --- | --- |
| [cap-02](../02-ai-capabilities/cap-02-piranha-population.md) | A few thousand labelled frames | About 8 weeks after camera installation (month 5) |
| [cap-01](../02-ai-capabilities/cap-01-animal-welfare-anomaly.md) | 6 months of telemetry plus >= 20 labelled events | Month 9 |
| [cap-03](../02-ai-capabilities/cap-03-visitor-flow-forecast.md) | 12 months covering a full season and a holiday period | Month 15 |

Sensors can be bought in a week. A year of readings cannot be bought at any price.

## Decision

A capability whose value depends on seasonal or historical patterns does not ship until the
history exists, and the plan says so rather than implying otherwise. Concretely:

- The deterministic version ships first and is the product until the model beats it. For
  forecasting, that is last-year-same-weekday plus a weather adjustment; in year one, when
  there is no last year, it is a hand-built calendar model from the operations manager.
- **"Beat the fallback" is a promotion gate** (ADR-0007), not an aspiration. A model trained
  on 4 months of data will not clear it, which is the mechanism that enforces this ADR
  automatically rather than by discipline.
- Phase transition signals are stated in terms of data accumulated, not calendar dates: 30
  days of clean telemetry to leave Phase 0, 6 months plus 20 labelled events to leave Phase
  1, 12 months of attendance to leave Phase 2.
- If the history never materialises, the fallback becomes the permanent product and the
  phase exit criterion changes to "the baseline is in daily use". We do not ship a model
  trained on 4 months and call it a forecast.

## Alternatives considered

| Alternative | Why not |
| --- | --- |
| Ship a model early on thin data and improve it | It would be presented as a forecast while being noise, the operations manager would staff from it, and the first bad week would end their trust in the capability permanently. Trust is spent once. |
| Buy or synthesise historical data | Another estate's attendance does not transfer; synthetic data cannot contain the seasonal pattern we are trying to learn, only the one we assume. |
| Use a pretrained or foundation time-series model to skip the cold start | Worth testing at month 15 as a candidate against the same gate. It does not remove the need for local history, because it still has to beat a local baseline on local data, and that comparison needs the data. |
| Start the whole programme later, when data exists | Data only starts existing because Phase 0 shipped. The cold start is not a reason to delay; it is a reason to sequence. |

## Consequences

**Good**

- Every model in production has been evaluated against real estate data, which is what makes the claims in [../04-verification/ai-validation.md](../04-verification/ai-validation.md) defensible.
- Phase 0 has a clear purpose beyond software: it is the data-collection phase, and its transition signal says so.
- The deterministic baselines are built anyway as fallbacks (ADR-0012), so nothing is wasted by waiting.

**Bad**

- The most visible AI capabilities arrive in year 2 and 3, which reads as slow.
- Two of the three years of the F7 horizon pass before forecasting influences staffing.
- A judge skimming the phase table sees little AI early.

**Risks**

| Risk | Mitigation | Where tracked |
| --- | --- | --- |
| Data collected in Phase 0 turns out to be unusable for training (gaps, quality) | The Phase 0 transition signal requires 30 days of *clean* telemetry, and data quality gates run before ingestion | [FF-07](../04-verification/fitness-functions.md#ff-07), [../04-verification/test-strategy.md](../04-verification/test-strategy.md) |
| Commercial pressure to ship a model early | The promotion gate blocks it mechanically, which is why the gate is worth more than the policy | [FF-17](../04-verification/fitness-functions.md#ff-17) |
| A season is lost, pushing everything a year | The A9 playbook makes the baseline the permanent product for that year, with no sunk cost | [../03-delivery/implementation-plan.md](../03-delivery/implementation-plan.md) |

## How we will know this was right

At month 15, [cap-03](../02-ai-capabilities/cap-03-visitor-flow-forecast.md) beats the rules
baseline by at least 20% relative on MAPE using 12 months of history. If a model trained on 6
months would have cleared the same gate, we waited too long and the transition signals should
be shortened for the next capability. If the 12-month model fails the gate too, the deferral
was right and the baseline stays.
