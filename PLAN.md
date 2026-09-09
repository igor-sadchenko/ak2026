# PLAN

Submission for O'Reilly Architectural Katas 2026 (Von Digitalis Estates).
Angle: **buildability and process** - delivery, verification, and the documented
AI-assisted design process. Complements variant A (`repo/`, edge-first + AI discipline),
variant B (`repo-andrew/`, quanta + platform breadth) and variant C (`repo-ivan/`,
three-lever business case + advisory AI); does not duplicate them.

Target: 1500-2500 lines of markdown, English, ASCII punctuation, Mermaid diagrams.

## Files

| File | One-line content |
| --- | --- |
| `README.md` | Thesis, overview diagram, kata-criterion -> where-covered table, 5-minute reading path |
| `00-problem/brief-facts.md` | Brief SSOT plus the explicit list of what the brief does not say |
| `00-problem/stakeholders.md` | Who, what they want, how each measures success, what they veto |
| `00-problem/requirements.md` | FR with source tag (§D / derived / assumption), NFR with a "verified by" column |
| `00-problem/okrs.md` | Objective -> metric -> now -> +12m -> +36m -> what moves it |
| `00-problem/constraints-assumptions.md` | Every assumption with a "what changes if wrong" column |
| `01-architecture/drivers-and-characteristics.md` | Top-3 driving characteristics and why not the other candidates |
| `01-architecture/style-decision.md` | Candidate styles scored in a table, with the reasoning behind each score |
| `01-architecture/decomposition.md` | Five deployable units: boundaries, data ownership, characteristics, why the line is here |
| `01-architecture/core-platform.md` | Non-AI foundation: edge nodes, backhaul, telemetry ingest, event bus, data stores |
| `01-architecture/ai-platform.md` | Provider abstraction, model/prompt registry, eval harness, monitoring, budgets |
| `02-ai-capabilities/README.md` | Portfolio table incl. two candidates rejected as "a rule is enough" |
| `02-ai-capabilities/cap-01-animal-welfare-anomaly.md` | Welfare anomaly detection: why not thresholds, data, uncertainty, V&V, degradation |
| `02-ai-capabilities/cap-02-piranha-population.md` | Piranha counting by CV: why not manual counts, label strategy, error budget |
| `02-ai-capabilities/cap-03-visitor-flow-forecast.md` | Zone/queue forecasting: cold start, rules baseline, promotion criteria |
| `02-ai-capabilities/cap-04-return-visit-offers.md` | Family-pass and return-visit uplift modelling, opt-in only, holdout by design |
| `02-ai-capabilities/cap-05-guest-companion.md` | Grounded assistant over a curated corpus, guardrails, safety answers never generated |
| `03-delivery/implementation-plan.md` | **Main artifact.** Phases 0-4: scope, FR/NFR closed, metric delta, team, cost delta, exit criteria as testable conditions, transition signals, what we skip and why, skip-this-phase breakage table, assumption-failure playbooks |
| `03-delivery/first-90-days.md` | Week-by-week breakdown of Phase 0 with owners and weekly proof points |
| `03-delivery/team-and-operating-model.md` | Roles, headcount per phase, buy vs build vs managed, on-call and escalation |
| `03-delivery/cost-model.md` | CapEx/OpEx per phase and per growth scenario, AI cost isolated, share of revenue, cost levers |
| `03-delivery/build-vs-buy.md` | Decision table per component: buy / build / managed, with price and the reason |
| `04-verification/fitness-functions.md` | **Second main artifact.** Every NFR -> metric -> tool -> threshold -> cadence -> action on breach, automation flag |
| `04-verification/ai-validation.md` | Golden sets, promotion thresholds, shadow runs, drift, override rate, auto-rollback |
| `04-verification/test-strategy.md` | Uplink loss, gate reconciliation, peak load, edge chaos, data quality, restore drills |
| `05-process/how-we-used-ai.md` | **Third main artifact.** Tools per stage, human/model division of labour, >=5 rejected AI proposals, hallucinations caught, cross-checking method, cost verdict. Contains the only TODO placeholders in the repo |
| `05-process/decision-log.md` | Contested forks incl. where this submission diverges from variants A, B and C, and why |
| `06-adrs/README.md` | Index with statuses and grouping |
| `06-adrs/template.md` | Context / Decision / Alternatives (with why-not) / Consequences / How we will know this was right |
| `06-adrs/ADR-0001..0016` | 16 ADRs: phase gating, right-sized decomposition, edge-first, backhaul, managed-first, AI admission gate, model registry, provider abstraction, privacy, HITL, fitness functions as CI gates, deterministic fallback, cold-start deferral, AI cost budget, documented AI process, retrieval-not-fine-tuning |
| `diagrams/README.md` | Diagram index and legend conventions |
| `diagrams/context.md` | System context (C4 L1) with legend |
| `diagrams/container.md` | Containers across the five units with legend |
| `diagrams/deployment.md` | Estate + cloud deployment, including the offline path |
| `diagrams/sequences.md` | Three sequences: offline gate entry + reconciliation, welfare anomaly to vet decision, forecast to staffing action |
| `glossary.md` | Terms used across the submission |

## Working order

00-problem -> 01-architecture -> 03-delivery -> 04-verification -> 02-ai-capabilities ->
05-process -> 06-adrs -> diagrams -> glossary -> final consistency pass (links, ASCII,
NFR-to-fitness-function coverage, criteria table).
