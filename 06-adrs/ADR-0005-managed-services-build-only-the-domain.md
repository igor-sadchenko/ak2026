# ADR-0005: Managed services by default; build only the domain

**Status:** accepted
**Deciders:** architect
**Relates to:** F13, A7, A14, NFR-OPS-1, NFR-SEC-1, [../03-delivery/build-vs-buy.md](../03-delivery/build-vs-buy.md)

## Context

Cloud is permitted (F13). The steady-state team is 2 engineers (A7, NFR-OPS-1). The cost
model shows IT at well under 1% of revenue in every scenario, including pessimistic ones,
which means **the scarce resource is attention, not money**. Anything that must be patched,
backed up, monitored or kept available by us competes directly with the work that only we
can do.

## Decision

Buy or rent anything a vendor already operates: databases, message broker, object storage,
container runtime, identity, observability, CI, BI, notification delivery, LLM inference,
weather data, and payment processing. Build only what encodes the estate's own domain: the
offline entitlement and reconciliation mechanism, the welfare rule and alerting layer, the
capability portfolio with its gates, and a thin edge node agent.

Two specific consequences of this rule:

- **No cardholder data touches our systems.** Payment goes through a hosted checkout at an
  external PSP (NFR-SEC-1). PCI scope is the single most expensive compliance burden a
  2-engineer team could take on, and this decision removes it entirely.
- **No Kubernetes.** A managed container runtime with two environments covers the need. The
  trigger to revisit is a team above 5 engineers or more than two deployable shapes.

## Alternatives considered

| Alternative | Why not |
| --- | --- |
| Self-host on estate hardware to save OpEx | Saves perhaps 1,500 EUR/month and costs a person. The arithmetic in [../03-delivery/cost-model.md](../03-delivery/cost-model.md) says the money is not the problem. Kept as the contingency if A5 fails badly. |
| Kubernetes for portability | Portability we do not need, at an operational cost we cannot pay. A container image is already portable enough for the on-estate contingency. |
| Handle card payments directly | Brings PCI scope onto the estate, grows Phase 0 by about a quarter, and would require rewriting NFR-SEC-1. We would push back hard (A14). |
| Adopt a full MLOps platform | The largest operational commitment in the system, for five capabilities. See ADR-0007. |
| Multi-cloud from the start | Two engineers, one estate. Provider risk is addressed where it is real (model providers, ADR-0008), not everywhere. |

## Consequences

**Good**

- The team spends its time on the parts nobody sells: offline admission and welfare alerting.
- Backups, patching and availability of the boring infrastructure are somebody else's job with a contract behind them.
- NFR-SEC-1 is satisfied by architecture rather than by controls.

**Bad**

- Vendor lock-in on the cloud platform. Accepted deliberately for infrastructure and explicitly rejected for model providers (ADR-0008), because the switching costs and the rates of change are different.
- Higher monthly cost than self-hosting.
- Some capabilities are shaped by what the managed services offer.

**Risks**

| Risk | Mitigation | Where tracked |
| --- | --- | --- |
| A managed service is withdrawn or repriced | Standard interfaces (SQL, MQTT, object storage, OIDC) rather than proprietary features wherever the cost is small | Reviewed at each phase gate |
| The cloud region has an outage | Accepted as a degraded day; the edge keeps admitting. Stated as an accepted risk in [../04-verification/test-strategy.md](../04-verification/test-strategy.md) | - |
| Managed costs grow with scale faster than expected | [FF-15](../04-verification/fitness-functions.md#ff-15) monthly | [../03-delivery/cost-model.md](../03-delivery/cost-model.md) |

## How we will know this was right

[FF-12](../04-verification/fitness-functions.md#ff-12): the system is operated by at most 6
people with at most 2 engineers, with no undocumented manual intervention, sustained through
one full season. If the engineers report spending more than 20% of their time on
infrastructure rather than on the domain, we bought the wrong things and this ADR is
revisited.
