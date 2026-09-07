# Build, buy or manage

The rule (from [team-and-operating-model.md](team-and-operating-model.md)):

> Build only what encodes the estate's own domain. Buy or rent everything a vendor already
> operates better than two engineers can.

Prices are `(assumption)`, per month unless stated, at Phase 2 scale.

| Component | Decision | Cost | Reason |
| --- | --- | --- | --- |
| Payment processing | **Buy** (hosted checkout, external PSP) | 1.4% + 0.25/txn | Keeps cardholder data entirely outside our systems (NFR-SEC-1). Building this would add a compliance programme to a 2-engineer team. Non-negotiable; see A14. |
| Ticket and entitlement ledger | **Build** | included in team cost | Family passes with shared admissions, offline-verifiable tokens and cross-gate reconciliation are the estate's own rules. Off-the-shelf ticketing assumes an online scanner. |
| Offline gate validation and reconciliation | **Build** | included | This is the mechanism the whole Phase 0 exit criterion rests on, and no vendor sells it for a historic estate with patchy Wi-Fi. It is the most valuable code we write. |
| Gate terminal hardware and OS | **Buy** | 900/unit CapEx | Commodity. |
| Edge node OS image and agent | **Build thin, on a bought base** | included | The image is a standard Linux plus a configuration; the agent is roughly 2,000 lines. Buying an IoT edge platform would add a licence and a vendor dependency for something small. |
| MQTT broker (edge) | **Buy** (open source, self-run on the node) | 0 | Mature, well understood, runs unattended. |
| Message broker (cloud) | **Managed** | 200 | Durability and replay are the two things we least want to operate ourselves. |
| Relational database | **Managed** | 350 | Backups, patching and point-in-time restore for free. Self-hosting saves about 200/month and costs a person. |
| Time-series / telemetry store | **Managed** | 300 | Same reasoning. |
| Object storage and query engine (analytics) | **Managed** | 250 | Cheap history is the substrate for every model. |
| Container runtime | **Managed** | 400 | Explicitly not Kubernetes; see [../01-architecture/core-platform.md](../01-architecture/core-platform.md). |
| Identity and SSO | **Buy** (managed IdP) | 60 | Staff roles for keepers, vet, gate staff, operations, owner. Writing auth is the classic false economy. |
| Observability (metrics, logs, traces) | **Managed** | 150 | One dashboard, one alerting path, one on-call rotation. |
| CI/CD | **Buy** (hosted) | 50 | |
| LLM inference for the assistant | **Buy** (external provider, two configured) | 210-630 | No case for self-hosting: a small, spiky query volume against a general model. Provider risk is handled by ADR-0008, not by running our own. |
| Retrieval index for the assistant | **Build** (embedded index in the deployable) | 0 | The corpus is a few thousand short documents. A managed vector database would be a second system to operate for a dataset that fits in memory. ADR-0016. |
| Welfare anomaly model | **Build** | included | Trained on this estate's enclosures and this estate's keepers' judgements. There is no market product for 55 mixed exotic enclosures. |
| Piranha counting model | **Build on a bought base** (open-source detection backbone, fine-tuned on our frames) | included | The backbone is commodity; the labels are ours. Buying a generic vision API would mean sending tank video off site continuously for a worse result at a higher per-call cost. |
| Forecasting model | **Build** (standard time-series library) | included | Two days of work with a well-known library. A forecasting SaaS costs more than the model is worth and hides the residuals we need to audit. |
| Model registry and evaluation harness | **Build small** | included | About a week of work for five capabilities. An MLOps platform would be the largest operational commitment in the whole system, for the smallest number of models. See [../01-architecture/ai-platform.md](../01-architecture/ai-platform.md). |
| Weather data | **Buy** | 40 | |
| Email and notification delivery | **Buy** | 30 | |
| Reporting and dashboards | **Buy** (hosted BI over the analytics store) | 120 | The operations manager can build their own views without an engineer, which directly serves NFR-OPS-1. |

## Summary

| Category | Count | Share of monthly OpEx |
| --- | --- | --- |
| Built by us | 8 | Team cost only, no licence |
| Bought or managed | 15 | About 2,160/month at Phase 2 |

Everything we build is either the estate's domain rules or a piece small enough that
operating a vendor's version of it would cost more attention than writing it. Everything
with an operational burden (durability, backups, patching, uptime) is somebody else's job.
That is the whole strategy, and it is what makes NFR-OPS-1 achievable.

## The two decisions most likely to be challenged

| Decision | Challenge | Our answer |
| --- | --- | --- |
| Building the ticketing ledger rather than buying a ticketing product | "Ticketing is a solved commercial problem." | It is, for venues with reliable networks. The brief's constraint (F10) breaks the assumption every product makes: that the scanner is online. We evaluated wrapping a product with an offline cache and rejected it, because the reconciliation semantics we need are inside the product's ledger, not outside it. Recorded in [../05-process/decision-log.md](../05-process/decision-log.md) D6. |
| Building the model registry rather than adopting an MLOps platform | "You are rebuilding MLflow." | We are rebuilding about 3% of it, for five capabilities, with two engineers and no platform team. If the capability count passes about 12, or a second estate appears, this decision flips and we say so in ADR-0007. |
