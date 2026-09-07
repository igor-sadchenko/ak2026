# ADR-0010: Confidence bands and human-in-the-loop

**Status:** accepted
**Deciders:** architect, head keeper
**Relates to:** NFR-AI-3, kata criterion 4, [cap-01](../02-ai-capabilities/cap-01-animal-welfare-anomaly.md)

## Context

Every model in the portfolio is uncertain about individual cases, and in the highest-value
one the cost of the two error types is wildly asymmetric: a missed welfare event may kill an
animal, while a false alarm costs a keeper a walk. Picking a single threshold forces a
single trade-off across all cases, and it hides the uncertainty rather than using it.

The humans who act on these outputs (keepers, the vet, the operations manager) can withdraw
their cooperation simply by ignoring notifications, and that decision is irreversible in
practice.

## Decision

Every capability returns `{result, confidence, basis, fallback_used, ...}` and every one
declares three bands:

| Band | Behaviour |
| --- | --- |
| High | Act automatically; log the action; review in aggregate weekly |
| Middle | Present to a human as a suggestion, **with the evidence**; the human decides |
| Low | Remove from the active path, retain, and sample weekly in a missed-signal review |

Three supporting rules:

- **The basis is mandatory.** A result a human cannot interrogate is not presentable. The
  UI cannot show a model output without showing why.
- **Confidence must be calibrated**, and calibration is a promotion gate (ADR-0007). An
  uncalibrated confidence makes the bands meaningless and every human-in-the-loop claim we
  make hollow.
- **The low band is audited, not discarded.** It is the only cheap estimate of false
  negatives available before delayed ground truth arrives.

The human is never removed. For welfare, treatment decisions are the vet's, always.

## Alternatives considered

| Alternative | Why not |
| --- | --- |
| A single threshold per capability | Forces one trade-off across cases with very different costs, and throws away the information the model already computed. |
| Show everything to the human and let them triage | Alert fatigue. The keeper alert-rate budget in [cap-01](../02-ai-capabilities/cap-01-animal-welfare-anomaly.md) is the constraint that actually protects the deployment, and unfiltered output breaks it immediately. |
| Fully automatic action on all confidences | Not acceptable for animal welfare, and the estate would have no way to notice a systematically wrong model. |
| Discard low-confidence outputs entirely | Loses the only pre-ground-truth signal about false negatives. Storage is cheap; the information is not. |

## Consequences

**Good**

- Human effort is spent on the cases where the model is genuinely unsure, which is the whole point of computing a confidence.
- False negatives become estimable before ground truth arrives.
- Override rate becomes a first-class quality signal ([FF-19](../04-verification/fitness-functions.md#ff-19)).

**Bad**

- Three thresholds per capability to calibrate and maintain instead of one.
- The middle band creates a queue that needs an owner, and a queue nobody works is worse than no queue.
- Calibration is real work and it degrades with drift.

**Risks**

| Risk | Mitigation | Where tracked |
| --- | --- | --- |
| The middle-band queue is ignored | Queue age is monitored; a persistently unworked queue means the bands are set wrong, and the response is to widen the high band or narrow the middle one, not to nag the keepers | [FF-19](../04-verification/fitness-functions.md#ff-19) |
| Calibration silently breaks | Calibration is a promotion gate and a monitored production metric; on breach the middle band widens automatically, sending more to humans | [../04-verification/ai-validation.md](../04-verification/ai-validation.md) |
| Humans defer to the model rather than judging it (automation bias) | The basis is always shown, and override rate is watched in both directions: a rate near zero is as suspicious as a high one | weekly review |

## How we will know this was right

[FF-18](../04-verification/fitness-functions.md#ff-18): 100% of AI responses carry a
complete contract, continuously. And the human signal: keeper override rate stays inside its
band (not near zero, not above 40%), and the middle-band queue is worked within its target
age. If keepers stop opening middle-band items entirely, the design failed regardless of
what the model metrics say.
