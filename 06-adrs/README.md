# Architecture Decision Records

Sixteen decisions. Format: Context, Decision, Alternatives considered (with a specific
reason for each rejection), Consequences (good, bad, risks), and **How we will know this
was right** - a check with a date, not a hope. Template: [template.md](template.md).

Statuses: `accepted` (in force), `proposed` (not yet exercised), `superseded`.

## Delivery and scope

| ID | Title | Status | Ties to |
| --- | --- | --- | --- |
| [ADR-0001](ADR-0001-phase-gated-delivery-no-ai-in-phase-0.md) | Phase-gated delivery with no AI in Phase 0 | accepted | [implementation-plan](../03-delivery/implementation-plan.md), D8 |
| [ADR-0002](ADR-0002-five-deployable-units.md) | Five deployable units, not thirteen quanta | accepted | [decomposition](../01-architecture/decomposition.md), D2 |
| [ADR-0005](ADR-0005-managed-services-build-only-the-domain.md) | Managed services by default; build only the domain | accepted | [build-vs-buy](../03-delivery/build-vs-buy.md) |
| [ADR-0013](ADR-0013-cold-start-defer-seasonal-models.md) | Cold start: defer seasonal models until the history exists | accepted | [cap-03](../02-ai-capabilities/cap-03-visitor-flow-forecast.md), D8 |

## Edge and connectivity

| ID | Title | Status | Ties to |
| --- | --- | --- | --- |
| [ADR-0003](ADR-0003-edge-first-offline-authoritative-gates.md) | Edge-first with offline-authoritative gates | accepted | NFR-AVAIL-1, [FF-01](../04-verification/fitness-functions.md#ff-01) |
| [ADR-0004](ADR-0004-mqtt-lorawan-cellular-backhaul.md) | MQTT and LoRaWAN sensing, cellular backhaul, no data mule | accepted | F10, F12, D4 |

## AI discipline

| ID | Title | Status | Ties to |
| --- | --- | --- | --- |
| [ADR-0006](ADR-0006-ai-admission-gate-rules-first.md) | AI admission gate: state the deterministic alternative first | accepted | [portfolio](../02-ai-capabilities/README.md) |
| [ADR-0007](ADR-0007-model-registry-and-promotion-gates.md) | Model registry with promotion gates | accepted | NFR-AI-2, [FF-17](../04-verification/fitness-functions.md#ff-17) |
| [ADR-0008](ADR-0008-provider-abstraction-two-provider-rule.md) | Provider abstraction and a two-provider rule with a quarterly drill | accepted | NFR-MOD-1, [FF-21](../04-verification/fitness-functions.md#ff-21) |
| [ADR-0010](ADR-0010-confidence-bands-and-human-in-the-loop.md) | Confidence bands and human-in-the-loop | accepted | NFR-AI-3, [cap-01](../02-ai-capabilities/cap-01-animal-welfare-anomaly.md) |
| [ADR-0012](ADR-0012-deterministic-fallback-for-every-capability.md) | A deterministic fallback for every AI capability | accepted | NFR-AI-1, [FF-16](../04-verification/fitness-functions.md#ff-16) |
| [ADR-0014](ADR-0014-per-capability-ai-budget.md) | Per-capability AI budget with automatic downgrade | accepted | NFR-COST-1, D9 |
| [ADR-0016](ADR-0016-retrieval-not-fine-tuning.md) | Retrieval over a curated corpus, not fine-tuning | accepted | [cap-05](../02-ai-capabilities/cap-05-guest-companion.md) |

## Privacy, verification and process

| ID | Title | Status | Ties to |
| --- | --- | --- | --- |
| [ADR-0009](ADR-0009-anonymous-by-default-opt-in-personalization.md) | Anonymous by default, personalisation opt-in only | accepted | NFR-PRV-1/2, D1 |
| [ADR-0011](ADR-0011-fitness-functions-as-ci-gates.md) | Fitness functions are CI gates, owned like tests | accepted | [fitness-functions](../04-verification/fitness-functions.md), D10 |
| [ADR-0015](ADR-0015-document-the-ai-assisted-process.md) | The AI-assisted design process is a deliverable | accepted | [how-we-used-ai](../05-process/how-we-used-ai.md) |

## Reading order for a judge with ten minutes

ADR-0001 (why phases), ADR-0002 (why five units), ADR-0006 (why AI at all, in each case),
ADR-0012 (what happens when AI fails), ADR-0011 (how we know any of this works).
