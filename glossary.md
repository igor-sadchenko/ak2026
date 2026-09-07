# Glossary

Terms used across this submission, in the sense we use them.

| Term | Meaning here |
| --- | --- |
| **Allow-list** | The set of entitlements valid for today and tomorrow, replicated to every edge node so a gate can decide without the cloud. See ADR-0003. |
| **Assumption** | A statement not present in the brief, tagged `(assumption)` and listed with an impact analysis in [00-problem/constraints-assumptions.md](00-problem/constraints-assumptions.md). |
| **Basis** | The evidence part of an AI response: which readings, frames, features or documents produced this result. Mandatory (NFR-AI-3). |
| **Capability** | A named AI-bearing function with an owner, a golden set, promotion gates, a fallback and a budget. Five exist; see [02-ai-capabilities/](02-ai-capabilities/). |
| **Cold start** | The condition of having no historical data at project start, which makes several capabilities unbuildable before a given month. ADR-0013. |
| **Confidence band** | High, middle or low, determining whether a result is acted on, shown to a human, or retained for audit. ADR-0010. |
| **Deployable unit / unit** | One of five parts of the system, U1 to U5. Only U1 and the cloud deployable are separately deployable today; U2-U5 are modules. ADR-0002. |
| **Deterministic fallback** | A rule-based behaviour that keeps a business function working when a model is unavailable, wrong or over budget. Not an error message. ADR-0012. |
| **Drill** | A rehearsed manual exercise with a named owner, a calendar slot and a written result. Used where a check cannot be safely automated on a live estate. |
| **Edge node** | An industrial mini-PC serving a zone cluster: MQTT broker, local rules, gate validation, store-and-forward buffer. |
| **Entitlement** | The right to enter, held as a signed token: a single ticket or a family pass with an admission count. |
| **Exit criterion** | A condition that can be observed and failed, gating the end of a delivery phase. Not a status. |
| **Fitness function** | A metric, an instrument, a threshold, a cadence and a breach action, attached to exactly one NFR. [04-verification/fitness-functions.md](04-verification/fitness-functions.md). |
| **Golden set** | Held-out labelled data used to gate a model version before promotion. Grows as disagreements are adjudicated. |
| **Ground truth, delayed** | The real answer, arriving days or weeks later: a treatment record, a next-day count, a return visit. |
| **Middle band queue** | The work queue of model results a human must judge. Its age is monitored; an unworked queue means the bands are wrong. |
| **Missed-signal review** | The weekly sample of discarded low-confidence results, used to estimate false negatives before ground truth arrives. |
| **Opt-in** | Explicit consent given before any personal data is collected. The only basis on which personalisation exists. ADR-0009. |
| **Promotion gate** | The set of conditions a model version must meet before the registry will bind it to production. ADR-0007. |
| **Pseudonymous key** | An identifier issued by U2 for an opted-in visitor, holding no name or contact details, breakable on erasure. |
| **Reconciliation** | Replaying offline gate decisions to the cloud ledger, detecting duplicates and conflicts, and reporting them. |
| **Shadow period** | Running a model on live inputs with its outputs invisible to users, compared against the incumbent. |
| **Split trigger** | The stated condition under which a module becomes a separate deployable. ADR-0002. |
| **Store-and-forward** | Durable local buffering of telemetry with ordered replay on reconnect. 72 hours of capacity. NFR-DATA-1. |
| **Transition signal** | What must accumulate or be confirmed before the next phase starts, stated in terms of data, not dates. |
| **Unit of AI type** | Classical ML (welfare, forecasting, uplift), computer vision (piranha counting), generative with retrieval (assistant). Verified differently; see [04-verification/ai-validation.md](04-verification/ai-validation.md). |

## Identifier conventions

| Prefix | Meaning | Defined in |
| --- | --- | --- |
| `F1`..`F13` | A fact stated in the brief | [00-problem/brief-facts.md](00-problem/brief-facts.md) |
| `U1`..`U10` | Something the brief does not say | [00-problem/brief-facts.md](00-problem/brief-facts.md) |
| `C1`..`C7` | A constraint from the brief | [00-problem/constraints-assumptions.md](00-problem/constraints-assumptions.md) |
| `A1`..`A14` | An assumption of ours | [00-problem/constraints-assumptions.md](00-problem/constraints-assumptions.md) |
| `FR-n` | Functional requirement | [00-problem/requirements.md](00-problem/requirements.md) |
| `NFR-*` | Non-functional requirement | [00-problem/requirements.md](00-problem/requirements.md) |
| `O1`..`O5` | Objective | [00-problem/okrs.md](00-problem/okrs.md) |
| `FF-nn` | Fitness function | [04-verification/fitness-functions.md](04-verification/fitness-functions.md) |
| `ADR-nnnn` | Architecture decision record | [06-adrs/](06-adrs/) |
| `D1`..`D10` | A contested decision in the decision log | [05-process/decision-log.md](05-process/decision-log.md) |
| `cap-nn` | An AI capability | [02-ai-capabilities/](02-ai-capabilities/) |

Note: `U` is overloaded (units U1-U5 and unknowns U1-U10). Context disambiguates: units
appear in architecture and requirement contexts, unknowns only in
[00-problem/brief-facts.md](00-problem/brief-facts.md) and
[00-problem/constraints-assumptions.md](00-problem/constraints-assumptions.md). We note it
rather than renaming, because both conventions are established in the source documents.
