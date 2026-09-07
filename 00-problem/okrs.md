# Objectives and key results

Only two numbers in this document come from the brief: 5,000 visitors per day today (F6)
and at least 15,000 within three years (F7). Everything else is a target we propose and
tag `(assumption)`. The point of the table is not the numbers; it is that each target
names the capability that moves it and the phase that ships it.

## Objectives

| ID | Objective | Why it exists |
| --- | --- | --- |
| O1 | Triple attendance within 3 years. | F7. Missing it costs the plant collection (F8). |
| O2 | Raise revenue per visitor without raising ticket price. | Attendance alone does not fix a poor estate if each visitor spends nothing. |
| O3 | Make visitors come back. | The cheapest visitor is one who has already been. F9 names return rate directly. |
| O4 | Keep 200+ animals alive and well with a small keeper team. | F4. A dead poisonous animal is a headline, not a metric. |
| O5 | Keep the estate able to run its own systems. | U6. The architecture fails if the client cannot operate it. |

## Key results

| Objective | Metric | Now | +12 months | +36 months | What moves it | Ships in |
| --- | --- | --- | --- | --- | --- | --- |
| O1 | Visitors per day, season average | 5,000 (F6) | 8,000 (assumption) | 15,000 (F7) | Throughput at the gate, queue reduction, staffing to demand | Phase 0, Phase 3 |
| O1 | Peak-hour entries per gate | unknown (U4) | 500/hour/gate (assumption) | 500/hour/gate | FR-3 offline validation, NFR-PERF-1 | Phase 0 |
| O1 | Share of tickets sold before arrival | unknown | 40% (assumption) | 60% (assumption) | FR-1 online sales, pre-booking incentives | Phase 0 |
| O2 | Revenue per visitor | unknown (U2) | +10% vs baseline (assumption) | +25% (assumption) | Zone popularity data driving where food and retail go (FR-5), pass upsell (FR-14) | Phase 0, Phase 4 |
| O2 | Family-pass share of ticket revenue | 0 (new product) | 15% (assumption) | 30% (assumption) | FR-2, FR-14 | Phase 0, Phase 4 |
| O3 | Repeat visit rate within 12 months | unknown | 12% (assumption) | 25% (assumption) | FR-14 opt-in offers, queue experience | Phase 4 |
| O3 | Opt-in rate for personalization | 0 | 20% of pass holders (assumption) | 35% (assumption) | FR-16, honest value exchange, ADR-0009 | Phase 4 |
| O4 | Welfare incidents requiring veterinary intervention that were not flagged in advance | unknown (no data today) | -30% vs Phase 1 baseline (assumption) | -60% (assumption) | FR-7, FR-10, [cap-01](../02-ai-capabilities/cap-01-animal-welfare-anomaly.md) | Phase 1, Phase 2 |
| O4 | Keeper hours spent on routine enclosure checks | unknown | -25% (assumption) | -40% (assumption) | FR-7 continuous sensing replaces manual rounds | Phase 1 |
| O4 | Piranha population estimate error against a manual count | no estimate exists | +/- 10% (assumption) | +/- 5% (assumption) | FR-9, [cap-02](../02-ai-capabilities/cap-02-piranha-population.md) | Phase 1 |
| O5 | People required to operate the system | n/a | <= 6, of whom <= 2 engineers | <= 6 | ADR-0002, ADR-0005, NFR-OPS-1 | all phases |
| O5 | IT operating cost as a share of revenue | n/a | <= 1.5% (assumption) | <= 1.5% | [../03-delivery/cost-model.md](../03-delivery/cost-model.md), ADR-0014 | all phases |

## Baseline problem

Nine of the twelve key results have `unknown` in the "Now" column. That is not sloppiness;
it is the actual situation of an estate that has just changed hands. It has two
architectural consequences:

| Consequence | Where it is handled |
| --- | --- |
| Phase 0 exists mainly to create the baseline. Its own success criterion is measurement coverage, not a business delta. | [../03-delivery/implementation-plan.md](../03-delivery/implementation-plan.md), Phase 0 |
| Any AI capability whose value is measured against a baseline cannot be evaluated before that baseline exists. This is a second, independent reason for the cold-start deferral in ADR-0013. | [../02-ai-capabilities/README.md](../02-ai-capabilities/README.md) |

## How these are reviewed

Monthly for O1 and O2 (attendance and revenue are daily data), quarterly for O3 and O4,
and at every phase gate for O5. The review is the same meeting as the fitness function
review; see [../04-verification/fitness-functions.md](../04-verification/fitness-functions.md).
