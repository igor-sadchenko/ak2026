# AI validation and verification

Kata criterion 6 asks how AI is validated and verified. Our answer has two halves that are
often confused: **can this model go live** (pre-release) and **is it still fit today**
(production). They use different evidence and are owned by different people.

The rule underneath both: a model is a component with a contract, and it is promoted,
monitored and rolled back like any other component. Its distinguishing property is not
that it is intelligent, but that its correctness is statistical, so the gate is statistical
too.

## Pre-release

### Golden sets

| Capability | Golden set | Size (assumption) | Who owns the labels | The hard part |
| --- | --- | --- | --- | --- |
| [cap-01](../02-ai-capabilities/cap-01-animal-welfare-anomaly.md) welfare anomaly | Historical enclosure windows labelled by outcome: intervention needed / not | 20 positive events minimum, 2,000 negative windows | Head keeper and vet | Positives are rare and precious. We hold them out ruthlessly and never train on the test set, even when it is tempting because there are so few. |
| [cap-02](../02-ai-capabilities/cap-02-piranha-population.md) piranha counting | Frames with witnessed manual counts | 40 counted sessions across 2 tanks and 4 lighting conditions | Keeper doing the manual count | The reference count is itself uncertain. We record the manual count's own spread and never claim accuracy better than it. |
| [cap-03](../02-ai-capabilities/cap-03-visitor-flow-forecast.md) forecast | The final 3 months of history, held out by time, never by random split | 90 days x zones x hours | Nobody; ground truth is the turnstile | A random split leaks the future into the past. Time-based holdout only. |
| [cap-04](../02-ai-capabilities/cap-04-return-visit-offers.md) offers | A randomised holdout group, permanently | >= 20% of the opted-in base | Nobody; it is an experiment | Uplift cannot be measured offline. The golden set here is a live control group, and it never goes away. |
| [cap-05](../02-ai-capabilities/cap-05-guest-companion.md) assistant | 200 curated questions with approved answers and required citations, plus 50 adversarial safety prompts | 250 | Operations manager, reviewed by the head keeper for the safety subset | Judging free text. We score citation correctness and refusal behaviour, which are checkable, rather than "helpfulness", which is not. |

### Promotion gates

A model version binds to production only when all of these hold. The registry enforces the
binding refusal mechanically ([FF-17](fitness-functions.md#ff-17)).

| Gate | Requirement |
| --- | --- |
| Offline threshold | The capability's stated metric thresholds are met on the held-out golden set. Thresholds are in each capability file, not here, because they are domain judgements. |
| Calibration | Stated confidence matches observed frequency within the capability's tolerance. An uncalibrated confidence makes the confidence bands in [../01-architecture/ai-platform.md](../01-architecture/ai-platform.md) meaningless, which would undermine every human-in-the-loop claim we make. |
| Beat the fallback | The model must beat its own deterministic fallback by a stated margin. If it does not, the fallback ships and the model does not. This gate has teeth: it is the reason [cap-03](../02-ai-capabilities/cap-03-visitor-flow-forecast.md) may ship as rules-only in a bad year. |
| Shadow period | The model runs on live inputs with its outputs invisible to users, compared against the incumbent (rules or previous model), for a stated duration. |
| Cost profile | Measured cost per 1,000 calls is inside the capability's budget (ADR-0014). |
| Fallback test | The fallback path passes ([FF-16](fitness-functions.md#ff-16)). |
| Named approver | A human signs the promotion. For welfare it is the head keeper, not an engineer, because they carry the consequence. |

### Shadow and canary

| Stage | Duration | What is compared | Exit |
| --- | --- | --- | --- |
| Shadow | 2-4 weeks per capability | Model output vs incumbent output on identical inputs; disagreements sampled and adjudicated by the domain owner | Disagreement rate and the adjudication verdict meet the capability's threshold |
| Canary | 1-2 weeks | Live for one zone, one tank, or 10% of queries | No regression in the quality proxy or the override rate |
| Full | - | - | Weekly monitoring takes over |

We do not skip shadow to save time. The one place we are explicit about this is the
"what breaks if a phase is skipped" table in
[../03-delivery/implementation-plan.md](../03-delivery/implementation-plan.md): keeper trust
is spent once.

## In production

### What we watch, and why not accuracy

Accuracy is usually unavailable in production because ground truth arrives late or never.
Each capability therefore declares a proxy, and we are explicit that it is a proxy.

| Signal | What it really tells us | Limitation we accept |
| --- | --- | --- |
| Human override rate | Whether the domain expert agrees with the model | Only covers what the human saw. Silent false negatives are invisible here. |
| Low-band audit | An estimate of false negatives, from what we discarded | Sampling only. A weekly review of a sample of low-confidence discards, adjudicated by the domain owner. |
| Delayed ground truth | Real correctness, weeks later | Welfare: treatment records. Forecast: the actual turnstile count the next day. Offers: the return visit, months later. |
| Input drift | Whether the world has moved away from the training window | Drift is not error. It is a reason to look, not to roll back on its own. |
| `fallback_used` rate | Whether the capability is actually working | Rising fallback use is often the first sign of a provider or cost problem, before quality shows anything. |
| Cost per useful outcome | Whether the capability is still worth having | Slow-moving; reviewed at the phase gate. |

### Automatic responses

| Condition | Automatic action | Human follow-up |
| --- | --- | --- |
| Quality proxy breaches its bound for 2 consecutive weeks | Roll back to the previous bound version | Retraining ticket, reviewed at the weekly meeting |
| No previous version qualifies | Route to the deterministic fallback | Capability marked degraded on the operations dashboard |
| Drift beyond the calibrated bound | Alert, no automatic rollback | The domain owner decides whether the world changed or the model did |
| Confidence calibration breaks (predicted vs observed diverges) | Widen the middle band, sending more cases to humans | Recalibrate or retrain |
| Spend passes 100% of ceiling | Route to fallback | Cost review (ADR-0014) |
| Provider outage or timeout | Secondary provider, then fallback | Incident record; feeds the quarterly drill |

Rollback is to the previous *bound version*, not to "the last good model", because the
registry knows what was bound and when, and a vague notion of good is not something a
2-person team can execute at 8 a.m. on a Saturday.

### Retraining

| Rule | Reason |
| --- | --- |
| Retraining is scheduled (quarterly) or triggered (a breach), never continuous. | Continuous retraining removes the human gate that everything above depends on, and nobody here has the capacity to supervise it. |
| Every retrained model goes through the full gate again, including shadow. | A retrained model is a new model. Familiarity is not evidence. |
| The golden set grows with each adjudicated disagreement, and growth is tracked. | This is the compounding asset of the programme. By Phase 4 the golden sets are worth more than the models. |
| Training data comes from the event log, never from the live operational store. | [../01-architecture/decomposition.md](../01-architecture/decomposition.md) data ownership rules. |

## Uncertainty we cannot remove, and what we do instead

Criterion 4 asks how we deal with uncertainty in AI. Three kinds, three different answers.

| Kind of uncertainty | Example here | Our answer |
| --- | --- | --- |
| The model is uncertain about this case | An enclosure reading pattern that is unusual but not clearly bad | Confidence bands. The middle band is routed to a human with evidence; it is not resolved by picking a threshold and hoping. |
| We are uncertain about the model overall | We have 20 labelled welfare events, which is very few | Say so, size the claim to the evidence, keep the rules underneath, and make the golden set grow. We would rather ship a modest claim we can defend than a strong one we cannot. |
| We are uncertain about the supplier | A provider changes a model, raises a price, or withdraws | Provider abstraction, a second provider evaluated on the same golden set, a quarterly drill with a measured switching cost (ADR-0008, [FF-21](fitness-functions.md#ff-21)). |

A fourth kind is worth naming because it is the one that bites teams: **uncertainty about
whether the capability was worth building.** Our answer is the phase exit criteria in
[../03-delivery/implementation-plan.md](../03-delivery/implementation-plan.md), each of
which can be failed, and the permanent holdout group for
[cap-04](../02-ai-capabilities/cap-04-return-visit-offers.md). A capability that cannot
demonstrate uplift is switched off. That has been written into the plan in advance, when it
is still cheap to agree to.
