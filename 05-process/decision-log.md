# Decision log

Contested forks. Each entry states the options, what we chose, who decided and why. Where
our answer differs from another submission in our team, that is recorded explicitly:
disagreement between competent designs is information, and hiding it would waste it.

Referenced elsewhere as D1 to D13. Our teammates' variants are referred to as **A**
(`repo/`, edge-first with strict AI discipline), **B** (`repo-andrew/`, thirteen quanta plus
an AI platform and an agentic layer) and **C** (`repo-ivan/`, a three-lever business case
with AI as an advisory overlay).

---

## D1: Anonymous by default, or personalise to drive returns

| Option | Case for | Case against |
| --- | --- | --- |
| Anonymous everywhere: counts only, no visitor identity, no tracking | Nothing to leak, nothing to explain to a family audience, no regulatory exposure, no consent machinery to build | Loses the return-visit objective (O3) almost entirely; FR-14 becomes untargetable |
| Identify visitors (wristband or token) and personalise | Enables retention, richer analytics, per-visitor journey understanding | Continuous location data on children, a large consent and erasure obligation, and a trust cost that lands on the same metric it is meant to improve |
| **Anonymous by default, personalisation strictly opt-in, no location tracking of individuals ever** | Keeps the retention capability, confines personal data to one unit and one phase, and the opt-in is a visible promise rather than a buried policy | Lower reach than full identification; the uplift capability works on a subset |

**Chosen: the third.** Decided by the author of this variant. Recorded as ADR-0009.

Divergence: variant B chooses RFID tokens with location and family linkage (its ADR-013 and
ADR-027) as the basis of a retention flywheel. Variants A and C both choose strict anonymity
(their ADR-0009 and ADR-007). Our position sits between them deliberately: we take the A/C
default and variant B's mechanism, gated behind consent. Note that variant C pairs its
anonymity with a single credential covering both ticket and cashless on-site spend (its
ADR-009), which reintroduces a linkable purchase history through a different door; see D13. We think the middle
position is the stronger answer to a judge, because it shows the trade-off was seen rather
than avoided in either direction. We may be wrong: variant B's flywheel argument is real,
and if the estate's retention target cannot be met on an opted-in subset, the honest
response is to revisit this, not to quietly widen collection.

---

## D2: Five deployable units, or thirteen quanta

| Option | Case for | Case against |
| --- | --- | --- |
| Fine-grained quanta (10+ independently deployable) with a platform layer | Each part scales and evolves independently; boundaries align precisely with differing characteristics; the analysis is more rigorous | The client has at most 2 engineers in steady state (A7). Ten deployables means ten pipelines, ten on-call surfaces and a platform capability nobody can staff. |
| **Five units: one edge shape plus four modules of one cloud deployable** | Matches the operating model; boundaries still follow the characteristic differences that actually exist; module interfaces are enforced by review and schema rather than by network hops | Less deployment independence; a bad release in one module affects the others; would not scale to a second estate without a split |

**Chosen: five.** Recorded as ADR-0002, with explicit split triggers.

Divergence: variant B identifies 13 quanta plus an AI platform. Its analysis of where
characteristics diverge is better than ours, and we borrowed the reasoning. We disagree
only on what follows from it: a boundary in the design does not have to become a boundary in
the deployment. Our units preserve the same boundaries as modules with owned data and
published interfaces, and ADR-0002 names the conditions under which each one should become
a separate deployable. If the client were a commercial park operator with a platform team,
variant B's answer would be better than ours.

---

## D3: An agentic layer, or a fixed capability portfolio

| Option | Case for | Case against |
| --- | --- | --- |
| Tool-using agents per stakeholder, able to act (book, dispatch, adjust) | Genuinely innovative, strong on criterion 1, and a good story | Each action is a new failure mode requiring its own verification. Criterion 6 asks how we validate, and an open action surface cannot be validated by two engineers. |
| **Five fixed capabilities, each with a gate, a fallback and an owner** | Every capability is verifiable and every one has a named approver; the portfolio can be operated by the client's own team | Less impressive. A judge looking for novelty may score it lower on criterion 1. |

**Chosen: the fixed portfolio.** Recorded as ADR-0006 and in
[../01-architecture/ai-platform.md](../01-architecture/ai-platform.md).

Divergence: variant B includes an agentic layer (its ADR-023). We think it is the most
interesting idea across the three submissions and the hardest to defend under criterion 6.
We would rather ship five capabilities we can validate than one we cannot, and we accept
the innovation cost of that choice openly. This is a genuine disagreement about which
criterion dominates, not a claim that the other answer is careless.

---

## D4: Data mule and delay-tolerant networking, or store-and-forward with cellular

| Option | Case for | Case against |
| --- | --- | --- |
| A vehicle physically carrying data from remote zones | Elegant, genuinely appropriate for very large estates with no coverage, and a memorable design element | Adds a scheduled physical process with an owner, a failure mode and a maintenance burden, to a client with two engineers |
| **Store-and-forward at the edge with a cellular uplink** | Meets NFR-DATA-1 with no moving parts; degrades to longer buffering; a satellite uplink is a drop-in if cellular fails (A5 playbook) | Depends on cellular coverage reaching at least the main buildings |

**Chosen: store-and-forward.** Recorded in
[../01-architecture/core-platform.md](../01-architecture/core-platform.md).

Divergence: variant B includes a data mule (its ADR-004 and ADR-015). It is the right answer
for a large estate with genuinely no coverage, which is a scenario the brief does not
confirm or exclude (U1, U10). Our week-1 site survey resolves this. If coverage is as bad as
that scenario assumes, we would adopt variant B's answer.

---

## D5: Dynamic pricing

| Option | Case for | Case against |
| --- | --- | --- |
| AI-driven dynamic or personalised pricing | Direct revenue lever, moves O2, and a well-understood technique | No elasticity data exists; a family audience punishes surprise pricing; the trust cost lands on O3, the return rate, which we are simultaneously trying to raise; and it needs the forecasting capability that does not exist until month 15 |
| **Simple published tiers: off-peak and peak, family pass discount** | Understandable, defensible, movable by hand, and it captures most of the demand-shifting benefit | Leaves money on the table in a mature operation |

**Chosen: published tiers.** Rejected AI pricing on suitability, not feasibility, in
[../02-ai-capabilities/README.md](../02-ai-capabilities/README.md).

Divergence: variant B includes dynamic pricing AI (its ADR-021), and variant C ships dynamic
family-bundle pricing in its Phase 3, gated on a full season of forecast-accuracy data. Our
objection is specific: the estate's core problem is that too few people come, not that the
ones who come pay too little. Pricing sophistication is a lever for a full park, and this
park is a third full. Variant C's gating condition is the right instinct and close to ours;
we would revisit the decision once attendance passes about 12,000 per day, which is a
demand-side trigger rather than a data-readiness one.

---

## D6: Build the ticketing ledger, or buy a ticketing product

| Option | Case for | Case against |
| --- | --- | --- |
| Buy a ticketing product and wrap it with an offline cache | Ticketing is a solved commercial problem; faster to a first sale | Every product assumes the scanner is online. The reconciliation semantics we need (cross-gate duplicate detection during a multi-day outage) live inside the product's ledger, not outside it, so the wrapper would have to duplicate the ledger anyway. |
| **Build the entitlement ledger, buy the payment processing** | The offline entitlement mechanism is the highest-value code in Phase 0, and it is the direct answer to the brief's hardest constraint (F10) | We own a ticketing system, which is not glamorous work |

**Chosen: build the ledger, buy the checkout.** Recorded in
[../03-delivery/build-vs-buy.md](../03-delivery/build-vs-buy.md) and ADR-0005.

---

## D7: Camera-based welfare behaviour analysis

| Option | Case for | Case against |
| --- | --- | --- |
| Cameras in all 55 enclosures with behaviour and posture analysis | Richer welfare signal than environmental sensing; catches conditions no thermometer can | Cameras in 55 mixed enclosures, night vision, and a labelling effort in a domain where no pretrained model has seen these species. The largest single cost in the portfolio, for an unproven marginal gain over sensor-based detection. |
| **Environmental sensing now, cameras only for the piranha tanks (FR-9), behaviour analysis deferred** | Delivers the brief's stated animal requirements at a fraction of the cost; the deferral is reversible | Slower detection of behavioural-only conditions |

**Chosen: defer.** Revisit when [cap-01](../02-ai-capabilities/cap-01-animal-welfare-anomaly.md)
is calibrated and there is evidence about what it misses. Deferred, not rejected.

---

## D8: Whether Phase 0 may contain any AI at all

| Option | Case for | Case against |
| --- | --- | --- |
| Ship one small AI capability in Phase 0 to demonstrate the theme early | The kata is about AI; a first phase with no AI reads as timid | No data exists at month 0, so nothing could be evaluated. Shipping an unevaluable model in the first quarter would break NFR-AI-2 immediately, and every gate we later claim to enforce. |
| **Phase 0 contains no models, and says so as a design statement** | Demonstrates the admission rule under pressure. The most credible way to show discipline about AI is to have a phase without any. | Risks reading as a lack of ambition to a judge skimming the phase table |

**Chosen: no AI in Phase 0.** Recorded as ADR-0001 and ADR-0013.

Convergence, for once: variant C reaches the same conclusion independently, and its Phase 1
opens with "nothing here is AI" for the same cold-start reason. Two variants arriving at the
same answer from different framings (ours from evaluability, theirs from funding each phase
out of the previous one's payback) is the strongest evidence in this log that the conclusion
is not a preference. Variants A and B do not forbid AI in a first release. The cold-start
arithmetic in [../03-delivery/implementation-plan.md](../03-delivery/implementation-plan.md)
is where our version of the argument is made.

---

## D9: How to frame the cost of AI

| Option | Case for | Case against |
| --- | --- | --- |
| "AI must stay under X% of revenue", with alerts and automatic downgrade | Concrete, controllable, and it makes cost a first-class architectural concern | Implies cost is the binding constraint, which our own numbers say it is not |
| "IT is a fraction of a percent of revenue, so cost is not a constraint" | Honest about the arithmetic | Invites unbounded spending on capabilities nobody can operate |
| **Both: publish the arithmetic showing cost is not binding, and keep per-capability budgets with automatic downgrade anyway** | The budgets are not there to save money; they are there to catch a runaway loop or an abuse case before it becomes an incident | Slightly more machinery than the numbers alone justify |

**Chosen: both.** Recorded as ADR-0014 and
[../03-delivery/cost-model.md](../03-delivery/cost-model.md).

Divergence: variant A frames AI cost as a hard budget constraint; variant B computes that IT
is about 0.1% of revenue and concludes cost is not a constraint. Both are right about their
own half. Our contribution is to say what the real binding constraint is instead: team
capacity, which is why NFR-OPS-1 blocks phase exits and NFR-COST-1, in practice, does not.

---

## D10: Whether verification deserves its own top-level block

| Option | Case for | Case against |
| --- | --- | --- |
| Verification distributed through each section (as most designs do) | Keeps each topic self-contained | Coverage becomes unprovable. Nobody can answer "does every NFR have a check" without reading everything. |
| **One block with one fitness function per NFR and a mechanical coverage check** | The question "how do you know" has a single, checkable answer, which is criterion 6 asked directly | Some duplication between the fitness function and the requirement |

**Chosen: a top-level block.** [../04-verification/](../04-verification/), ADR-0011.

This is the decision that most defines this submission, together with
[../03-delivery/implementation-plan.md](../03-delivery/implementation-plan.md) and
[how-we-used-ai.md](how-we-used-ai.md).

---

## D11: Predictive maintenance on the heritage rides

| Option | Case for | Case against |
| --- | --- | --- |
| Retrofit vibration, temperature and cycle-count sensors externally, model each ride against its own baseline, raise advisory inspection recommendations only | Creates the sensor history that does not exist today; per-ride baselines handle mechanical heterogeneity; advisory-only keeps the certified inspection regime authoritative | 40 rides to instrument and maintain; a further year of baseline data before the model means anything; one more capability to keep calibrated with two engineers |
| **Fixed certified inspection intervals plus a fault log** | The regime already exists, is legally authoritative (C5), and costs nothing to run | Misses degradation between inspections |

**Chosen: intervals and a fault log**, listed as a rejected AI candidate in
[../02-ai-capabilities/README.md](../02-ai-capabilities/README.md).

Divergence: variant C designs this capability (its ADR-013 and C5) and its argument is
better than our original rejection. We had objected that the failure cost is safety-critical
and that a model over 40 heterogeneous historic machines is unverifiable; advisory-only
output and per-ride baselines answer both objections cleanly, and non-invasive external
sensors answer the heritage-modification problem we had not considered.

What survives of our objection is narrower and we state it as such: it is a portfolio-size
argument, not a design argument. A sixth capability, needing its own instrumentation
programme and its own year of baseline data before it produces anything, competes for the
same two engineers who must keep the welfare model calibrated. If the estate had capacity for
one more capability, this is the one we would add, and we would adopt variant C's design for
it rather than invent our own.

---

## D12: Per-enclosure baselines or per-animal identity

| Option | Case for | Case against |
| --- | --- | --- |
| Identify individuals (PIT or RFID tags, per-animal feed stations, statistical disaggregation of shared feed scales) and baseline each animal | A sick individual in a group enclosure is detectable; the welfare signal is far sharper | Requires handling exotic and poisonous animals to tag them (F4); tags fail; multi-occupancy attribution stays partly statistical anyway; a large keeper workload |
| **Per-enclosure environmental baselines, with the keeper as the individual-level observer** | No handling, no tags, works from day one on sensors we are installing anyway, and the keepers already know their animals individually | A single sick animal in a shared enclosure may not move the enclosure aggregate at all |

**Chosen: per-enclosure**, see [../02-ai-capabilities/cap-01-animal-welfare-anomaly.md](../02-ai-capabilities/cap-01-animal-welfare-anomaly.md).

Divergence: variant C tags individuals (its ADR-008), and it raises the sharpest criticism of
our design in this log: a per-enclosure baseline is blind to the individual, which for a
collection of 200+ animals in 55 enclosures is a real gap and not a small one. We accept the
criticism and take the trade knowingly, for one reason: our capability is scoped as an
enrichment layer over threshold rules, with the keeper as the individual-level observer, and
we are not willing to justify handling venomous animals to fit tags on the strength of a
model we have not yet shown to work. The honest sequence is to prove the enclosure-level
model first and revisit identity if the misses it produces are individual-level ones. That
revisit condition belongs in [cap-01](../02-ai-capabilities/cap-01-animal-welfare-anomaly.md),
and variant C's mechanism is what we would adopt.

---

## D13: A single credential carrying both admission and cashless spend

| Option | Case for | Case against |
| --- | --- | --- |
| One signed credential for gate entry and on-site payment, verified offline, queued for reconciliation | Removes real friction; the same offline mechanism we already built serves both; a strong on-site spend lever for objective O2 | It creates a per-visitor purchase history tied to an admission credential, which is a linkable behavioural record by another name; it also puts payment inside our reconciliation scope |
| **Admission credential only; payment through an external provider at the point of sale** | Keeps NFR-SEC-1 absolute (no cardholder data, no payment reconciliation risk) and keeps the anonymous-by-default promise intact | Leaves the on-site spend lever unpulled |

**Chosen: admission only.** ADR-0005, ADR-0009.

Divergence: variant C builds this (its ADR-009 and capability C3) and rates it as its fastest
payback. We think it is the most commercially attractive idea in any of the four submissions
and the one with the least examined privacy consequence: variant C states anonymous-by-default
as a principle (its ADR-007) while its credential accumulates exactly the linkable spend
history that principle is meant to prevent. That is a tension to resolve, not a fault, and
resolving it is probably a matter of separating the two balances behind different keys.

Our reason for not building it is separate from the privacy point and simpler: it moves
payment reconciliation inside our scope, which is the one thing ADR-0005 spends the most
effort keeping out, for a team of two.
