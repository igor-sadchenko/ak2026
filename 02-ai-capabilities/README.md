# AI capability portfolio

Five capabilities, and four candidates we rejected because a rule, a query or a calendar
does the job. The rejections are on this page, at the top, because they are the more
useful half of the portfolio: they are what "suitability" (criterion 2) actually looks
like.

## The admission rule

Before any capability is allowed into the portfolio it has to survive one question:

> What deterministic rule or ordinary query would we write instead, and what specifically
> does it fail to do?

If the rule works, the rule ships. This is enforced as a review gate, and it is recorded as
ADR-0006. Each accepted capability's file opens by answering the question rather than by
describing the model.

## Rejected: a rule is enough

| Candidate | The obvious AI framing | What we do instead | Why the simple thing wins |
| --- | --- | --- | --- |
| "Understand which areas are popular" (F9) | Visitor journey clustering, heat-map inference from behaviour | Count entries and exits per zone per 15 minutes, put it on a chart, sort descending | This is a counting problem with a known denominator. A model would add error to a measurement. The requirement is satisfied exactly by FR-5, and it is deliberately boring. |
| Feeding schedule adherence (FR-8) | Predict which feedings will be missed | A schedule, a checklist and an escalation when a feeding is not marked done within its window | The keepers know the schedule. The failure is a missed action, not an unknown pattern. Predicting a miss adds nothing you cannot get from noticing it 20 minutes later. |
| Ride maintenance planning (F3) | Predictive maintenance on 18th-century rides | Fixed inspection intervals from the existing certification regime, plus a fault log | The rides are already certified under a regime with defined intervals (C5), and there is no sensor history to learn from. This is our weakest rejection and we say so: a well-designed advisory-only version with per-ride baselines answers the safety and heterogeneity objections, and one exists in a teammate's variant. What survives is a portfolio-size argument, not a design one. Full reasoning and the revisit condition are in [../05-process/decision-log.md](../05-process/decision-log.md) D11. |
| Staff rostering optimisation | Constraint-solving or learned rostering | A roster built by the operations manager from the demand forecast ([cap-03](cap-03-visitor-flow-forecast.md)) | The forecast is the hard part; the roster is a small problem for one person with a spreadsheet. Automating it would be optimisation for its own sake. |
| Dynamic per-visitor pricing | Price elasticity model, personalised prices | Published, simple price tiers (off-peak and peak, family pass discount) | Discussed at length in [../05-process/decision-log.md](../05-process/decision-log.md) D5. Families punish surprise pricing, the trust cost lands on the return-rate objective we are trying to raise, and the estate has no elasticity data. Rejected on suitability, not on feasibility. |

## Accepted portfolio

| # | Capability | Type of AI | Which metric it moves | Human in the loop | Deterministic fallback | Phase |
| --- | --- | --- | --- | --- | --- | --- |
| [cap-01](cap-01-animal-welfare-anomaly.md) | Welfare anomaly detection | Classical ML on multivariate sensor time series | O4: unflagged welfare incidents | Yes. Middle-confidence alerts go to a keeper; the vet decides treatment. Always. | Threshold rules set by the head keeper, running locally, permanently | 2 |
| [cap-02](cap-02-piranha-population.md) | Piranha population estimation | Computer vision, object detection and counting | O4: population estimate error; satisfies FR-9 from F9 | Yes. The estimate is a suggestion; a keeper confirms before any action on the population. | Scheduled manual counts with a sampling protocol | 1 |
| [cap-03](cap-03-visitor-flow-forecast.md) | Visitor flow forecasting | Time-series forecasting with exogenous features | O1: attendance via queue reduction; O2 indirectly | Yes. The operations manager staffs from it; the model never changes staffing directly. | Last-year-same-weekday plus a weather adjustment | 3 |
| [cap-04](cap-04-return-visit-offers.md) | Return-visit and pass-upgrade offers | Uplift modelling on opted-in visitors | O3: repeat visit rate, pass share | Yes. Marketing approves the offer set; the model chooses who sees which. | Untargeted seasonal promotion to all opted-in holders | 4 |
| [cap-05](cap-05-guest-companion.md) | Grounded visitor and staff assistant | Retrieval-augmented generation over a curated corpus | O1, O3 indirectly; reduces staff interruption | Yes for corpus curation; escalation to a human for anything it cannot ground | Static FAQ, search over the same corpus, signage | 2 |

## Portfolio-level properties

| Property | How it holds across all five |
| --- | --- |
| Nothing safety-relevant is produced by a model | The welfare alert path is rules; the model only enriches and prioritises. Generative AI is confined to U5. NFR-SAFE-1, [FF-20](../04-verification/fitness-functions.md#ff-20). |
| Every capability degrades to a working business behaviour, not to an error | The fallback column above is tested in CI ([FF-16](../04-verification/fitness-functions.md#ff-16)) and exercised in a monthly game day. |
| Only one capability's cost scales with attendance | [cap-05](cap-05-guest-companion.md). The other four are fixed-cost batch jobs. See [../03-delivery/cost-model.md](../03-delivery/cost-model.md). |
| Only one capability touches personal data | [cap-04](cap-04-return-visit-offers.md), opt-in only, ADR-0009. |
| Three of five cannot ship before their data exists | The cold-start table in [../03-delivery/implementation-plan.md](../03-delivery/implementation-plan.md). |
| Two AI types dominate, and they are the boring ones | Classical ML and CV carry the operational value. Generative AI carries the least differentiated capability. We state this rather than lead with the LLM, because it is what the evidence supports. |

## Why the portfolio is this small

Criterion 3 is "appropriate level of detail", and the honest reading for this client is
appropriate *scope*. Five capabilities, staged across three years, with two engineers in
steady state, is roughly one capability that a small team can genuinely own, validate and
keep calibrated at any one time. A portfolio of twelve would look more innovative on paper
and would be less likely to have a single one of them still calibrated in year three.

Where we differ from the other submissions in our team on exactly this point, and why, is
recorded in [../05-process/decision-log.md](../05-process/decision-log.md).
