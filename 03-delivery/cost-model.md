# Cost model

Every figure on this page is `(assumption)` except attendance (F6: 5,000/day; F7: 15,000/day).
The purpose is not precision. It is to establish orders of magnitude well enough to answer
one question: **is money the constraint here, or is something else?**

Currency: EUR. Basis: European mid-market managed cloud and industrial hardware pricing,
2026 (assumption).

## Capital expenditure by phase

| Item | Unit price | Qty | Total | Phase |
| --- | --- | --- | --- | --- |
| Gate terminal (rugged tablet, scanner, mount) | 900 | 8 | 7,200 | 0 |
| Edge node (industrial mini-PC, UPS, enclosure) | 1,400 | 6 | 8,400 | 0 |
| Cellular router and installation per node | 500 | 6 | 3,000 | 0 |
| People-counting sensor (bidirectional beam or ToF) | 250 | 12 | 3,000 | 0 |
| Network materials and installation labour | - | - | 6,400 | 0 |
| **Phase 0 subtotal** | | | **28,000** | |
| Welfare sensor (temperature, humidity, water quality, door) | 110 | 240 | 26,400 | 1 |
| LoRaWAN gateway | 700 | 6 | 4,200 | 1 |
| Tank and enclosure camera | 400 | 12 | 4,800 | 1 |
| Edge GPU node for tank vision | 3,500 | 1 | 3,500 | 1 |
| Additional edge nodes (animal zones) | 1,400 | 2 | 2,800 | 1 |
| Installation labour, 55 enclosures | - | - | 8,300 | 1 |
| **Phase 1 subtotal** | | | **50,000** | |
| Training compute, labelling workstation | - | - | 5,000 | 2 |
| Refresh and spares allowance | - | - | 3,000 | 3 |
| **Total CapEx over 3 years** | | | **86,000** | |

Spares policy: 1 spare edge node and 2 spare gate terminals are held on site from week 10
(included in the quantities above). This is what makes the 30-minute cold-restore target
in NFR-OPS-2 achievable by a non-engineer: the procedure is "swap, not diagnose".

## Operating expenditure

Monthly, at steady state within each phase.

| Line | P0 | P1 | P2 | P3 | P4 |
| --- | --- | --- | --- | --- | --- |
| Cloud compute and storage | 700 | 1,050 | 1,300 | 1,500 | 1,600 |
| Cellular connectivity (8 nodes) | 180 | 240 | 240 | 240 | 240 |
| Managed broker and observability | 150 | 200 | 250 | 280 | 300 |
| External data feeds (weather) | 0 | 0 | 0 | 40 | 40 |
| **AI inference and platform** | **0** | **80** | **330** | **420** | **520** |
| **Total per month** | **1,030** | **1,570** | **2,120** | **2,480** | **2,700** |
| **Total per year** | 12,360 | 18,840 | 25,440 | 29,760 | 32,400 |

Payment processing fees are excluded because they scale with revenue and are a cost of
selling, not a cost of IT (assumption: 1.4% plus 0.25 EUR per transaction, passed through).

## AI cost, isolated

Criterion 2 (suitability) invites the question "can this client afford AI at all". The
answer is that the AI line is the smallest line in the table.

| Capability | Where it runs | Cost driver | Monthly at 5,000/day | Monthly at 15,000/day |
| --- | --- | --- | --- | --- |
| [cap-02](../02-ai-capabilities/cap-02-piranha-population.md) piranha counting | Edge GPU node | Amortised hardware only, no per-call cost | 0 (CapEx) | 0 |
| [cap-01](../02-ai-capabilities/cap-01-animal-welfare-anomaly.md) welfare anomaly | Cloud, small model, batch every 5 min | Fixed compute, independent of visitors | 120 | 120 |
| [cap-05](../02-ai-capabilities/cap-05-guest-companion.md) assistant | External LLM provider | Per query, scales with visitors | 210 | 630 |
| [cap-03](../02-ai-capabilities/cap-03-visitor-flow-forecast.md) forecasting | Cloud, nightly batch | Fixed, one run per night | 90 | 90 |
| [cap-04](../02-ai-capabilities/cap-04-return-visit-offers.md) offers | Cloud, weekly batch | Fixed | 100 | 100 |
| **Total** | | | **520** | **940** |

Only the assistant scales with attendance. That is why it is the only capability with a
hard per-query budget and an automatic downgrade path (ADR-0014): assumption of 0.9 queries
per visitor per day at a 15% engagement rate, at roughly 0.008 EUR per grounded answer with
caching. The other four are fixed-cost batch jobs, which is a deliberate design property,
not an accident of pricing.

## Against revenue

| Scenario | Visitors/day | Operating days | Spend per visitor | Revenue/year | IT OpEx/year | IT as % of revenue | AI as % of revenue |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Today (F6) | 5,000 | 200 | 18 | 18.0M | 25,440 | 0.14% | 0.03% |
| Year 2 | 10,000 | 200 | 19 | 38.0M | 29,760 | 0.08% | 0.02% |
| Target (F7) | 15,000 | 210 | 20 | 63.0M | 32,400 | 0.05% | 0.02% |
| Pessimistic: spend per visitor is a third of the estimate | 5,000 | 160 | 6 | 4.8M | 25,440 | 0.53% | 0.12% |
| Very pessimistic: attendance halves and spend is a third | 2,500 | 160 | 6 | 2.4M | 25,440 | 1.06% | 0.24% |

## The conclusion this table forces

Even in the very pessimistic scenario, IT sits at about 1% of revenue and AI at a quarter
of a percent, inside NFR-COST-1. **Money is not the constraint on this architecture.
Team capacity is.**

That conclusion is load-bearing, and it is the reason this submission spends its detail on
[team-and-operating-model.md](team-and-operating-model.md) and
[../04-verification/](../04-verification/) rather than on cost optimisation. It also
disciplines the design in a way a cost constraint would not: we cannot justify a component
by saying it is cheap. It has to be justified by someone having time to own it.

Two caveats we state rather than bury:

| Caveat | Effect |
| --- | --- |
| The 18 EUR spend per visitor (A3) is our least evidenced number, and the whole table pivots on it. | If it is wrong by a factor of 5 downward, IT passes 2% and the Phase 4 capabilities are cut first. The playbook is in [implementation-plan.md](implementation-plan.md). |
| CapEx is front-loaded into Phases 0 and 1 (78,000 of 86,000), before any AI capability exists. | An owner who wants to stop after Phase 1 has still bought a working estate system. That is a property of the phasing, and it is intentional. |

## Cost levers, in the order we would pull them

| Lever | Saving | What it costs |
| --- | --- | --- |
| Cache and template the top 20 assistant questions instead of generating them | Up to 60% of assistant cost | Slightly less natural answers for the most common questions. We would do this anyway; the fixed answers are also more auditable. |
| Move the welfare anomaly model to a smaller architecture and run it every 15 minutes instead of every 5 | About 60 EUR/month | Slower detection of slow-developing conditions. Acceptable, because acute conditions are caught by the rules layer within 60 seconds regardless. |
| Reduce telemetry retention in the hot store from 90 to 30 days | About 150 EUR/month | Slower ad hoc investigation. Cold storage keeps everything for training. |
| Drop the secondary LLM provider | About 5% of assistant cost | Loses NFR-MOD-1. We would not do this; the drill is cheap and the lock-in is not. |
| Defer Phase 1 cameras until the piranha requirement is prioritised | 8,300 CapEx | FR-9 unmet, and FR-9 comes from the brief (F9). Only pull this lever if A11 has already failed. |
