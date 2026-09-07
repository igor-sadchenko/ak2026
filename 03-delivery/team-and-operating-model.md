# Team and operating model

The binding constraint on this architecture is not money and not technology. It is that
the client is a poor estate with no IT department (U6, A7). Every structural decision in
[../01-architecture/decomposition.md](../01-architecture/decomposition.md) is downstream
of this page.

## Headcount by phase

| Role | P0 | P1 | P2 | P3 | P4 | Steady state |
| --- | --- | --- | --- | --- | --- | --- |
| Lead engineer / architect | 1 | 1 | 1 | 1 | 0.5 | 0 (external) |
| Engineer (full stack, cloud and edge) | 2 | 2 | 2 | 2 | 2 | 2 (client staff) |
| Field technician | 0.5 | 1 | 0.5 | 0.25 | 0.25 | 0.25 (contract) |
| Data engineer | 0 | 1 | 1 | 1 | 0.5 | 0 |
| ML engineer (contract) | 0 | 0.5 | 1 | 0.5 | 0.25 | 0 |
| **Total delivery FTE** | **3.5** | **5.5** | **5.5** | **4.75** | **3.5** | **2.25** |
| Client: operations manager | 0.5 | 0.5 | 0.5 | 0.5 | 0.5 | 0.5 |
| Client: head keeper | 0 | 0.3 | 0.3 | 0.1 | 0.1 | 0.1 |

Two things this table asserts:

1. **The peak is temporary and the ML engineer is a contractor.** A permanent ML engineer
   for five capabilities that retrain quarterly would be idle most of the year and would
   be the first cost cut in a bad season, which is the worst possible moment to lose them.
2. **The steady state is 2 engineers plus estate staff**, which is NFR-OPS-1. If a design
   choice would push the steady state to 3, it needs an ADR arguing why.

## Who does what after handover

| Duty | Owner | Cadence |
| --- | --- | --- |
| Deploys and rollbacks | Client engineer | On demand, pipeline-driven |
| Edge node and terminal swaps | Gate staff, then field technician if the swap fails | On demand, printed runbook (NFR-OPS-2) |
| Welfare alert triage | Keepers | Continuous during opening hours |
| Welfare model override review | Head keeper plus client engineer | Weekly, 30 minutes |
| Fitness function review | Client engineer plus operations manager | Weekly automated report, monthly review meeting |
| Model promotion approval | Named approver per capability (head keeper for welfare, operations manager for forecasting, owner for anything visitor-facing) | Per release, see ADR-0007 |
| Cost review | Operations manager | Monthly, against [cost-model.md](cost-model.md) |
| Provider failover drill | Client engineer | Quarterly, ADR-0008 |
| Security patching | Managed by the platform vendor where possible; client engineer for the deployable | Monthly |

## On-call

One rotation, two people, and a deliberately narrow definition of what wakes someone up.

| Severity | Definition | Response |
| --- | --- | --- |
| P1 | Gates cannot admit, or ticket sales are down, during opening hours. | Page. Target response 15 minutes. Fallback procedure (paper list plus manual gate) is printed and rehearsed. |
| P1 | A welfare alert path is confirmed dead (no heartbeat from an enclosure cluster's rule engine). | Page. Keepers revert to manual rounds for that cluster until resolved. |
| P2 | An edge node is offline but its zone still admits (another node covers the gate) or telemetry is buffering normally. | Next business day. |
| P3 | A model is degraded, drifting or over budget. | Next business day. **A model failure is never a page**, because every model has a deterministic fallback (NFR-AI-1). This is the operational payoff of ADR-0012 and the reason a 2-person team can own an AI-bearing system. |

## Build, buy and manage: the principle

Stated fully in [build-vs-buy.md](build-vs-buy.md). The rule we apply:

> Build only what encodes the estate's own domain. Buy or rent everything that a vendor
> already operates better than two engineers can.

The three things we build, because nobody sells them: the offline entitlement and
reconciliation mechanism, the welfare rule and alerting layer with its escalation model,
and the capability portfolio with its gates. Everything else is bought.

## Knowledge transfer as a deliverable

Handover is scheduled work with acceptance criteria, not a final meeting.

| Phase | Transfer deliverable | How we know it worked |
| --- | --- | --- |
| 0 | Printed runbooks for gate staff; a recorded walkthrough of the deploy pipeline | A staff member performs a cold restore unaided and within 30 minutes ([FF-13](../04-verification/fitness-functions.md#ff-13)) |
| 1 | Threshold configuration owned by the head keeper, edited without an engineer | The head keeper changes a threshold in production and it takes effect |
| 2 | Model promotion runbook; the weekly override review is chaired by the client | Two consecutive promotions approved with the delivery team only observing |
| 3 | One engineer moves onto the client's payroll | That engineer runs a full release cycle alone |
| 4 | External involvement ends | One full season operated with no external on-call |

## Where this differs from a conventional plan

| Conventional | Here | Why |
| --- | --- | --- |
| Staff up for the peak and keep the team | Peak at 5.5 FTE for 12 months, then fall to 2.25 | The client cannot fund a standing team, and a plan that assumes otherwise is fiction. |
| Hire an ML engineer to own the models | Contract one for the build, design the models to be operable without one | Five capabilities that retrain quarterly do not need a full-time specialist; they need good gates ([../04-verification/ai-validation.md](../04-verification/ai-validation.md)). |
| On-call covers everything | On-call covers admission, sales and animal safety only | Model degradation is designed to be a business-hours problem. |
