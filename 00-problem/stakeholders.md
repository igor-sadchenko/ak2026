# Stakeholders

The architecture is judged by people, not by diagrams. This table is the shortlist of who
can say no, what they measure, and what they will veto.

| Stakeholder | Fact or assumption | What they want | How they measure it | What they will veto |
| --- | --- | --- | --- | --- |
| Countess von Digitalis (owner) | F1 | The estate becomes self-sustaining before the plant collection has to be sold (F8). | Visitors per day; operating margin; whether the collection is still hers in 3 years. | Any plan whose first payoff arrives after year 1. Any spend she cannot explain in one sentence. |
| Head keeper / animal team | F4, (assumption) 6-10 keepers | Fewer dead or sick animals; less time spent walking 55 enclosures to read a thermometer. | Welfare incidents per quarter; time from an abnormal condition to a human seeing it. | A system that pages them at 3 a.m. for false alarms. A model that overrides their judgement. |
| Veterinarian | (assumption) part-time, on call | To be called early and only when it is real. | Share of alerts that were genuine; time from alert to intervention. | Any design where a model decides treatment. |
| Gate and front-of-house staff | (assumption) seasonal, high turnover, low technical skill | A queue that moves and a scanner that works when the network does not. | Entries per hour; number of manual overrides per day. | Anything that stops selling or admitting when the uplink drops. |
| Ride operators | F3 | Rides stay compliant and open; paperwork is not doubled. | Ride uptime; inspection compliance. | Anything that adds a second system for the same inspection log. |
| Estate operations manager | (assumption) exists, one person | Knows where people are today and where they will be tomorrow; staffs accordingly. | Queue length at peak; overtime hours. | Dashboards nobody has time to read. |
| Marketing / commercial (assumption) | Derived from F9 | Sell more passes; bring visitors back. | Pass share of revenue; repeat visit rate. | A privacy posture so strict that no one can be contacted again. |
| Visitors and families | Derived from F9 | Get in fast, find things, not be tracked. | Queue time; whether they come back. | Face recognition, hidden tracking, surprise pricing. |
| Regulator / safety inspectorate | F3 | Rides and dangerous-animal containment remain inspectable. | Inspection outcomes; incident reports. | Safety decisions that depend on a model or on a cloud round trip. See ADR-0012 and NFR-SAFE-1. |
| Delivery team (us, then the client's own team) | (assumption) <= 6 people | To hand over something a small team can run. | Number of components on call; mean time to recover an edge node. | An architecture that needs a platform team the client cannot hire. See ADR-0002. |

## Consequences that shape the architecture

| Observation | Architectural consequence |
| --- | --- |
| The owner's horizon is one season, not one decade (F7, F8). | Phase 0 must deliver measurable value with no AI at all. See ADR-0001. |
| The keepers are the users of the highest-value AI capability, and they can switch it off by ignoring it. | Welfare alerting is precision-budgeted and human-in-the-loop from day one. See [cap-01](../02-ai-capabilities/cap-01-animal-welfare-anomaly.md) and ADR-0010. |
| Gate staff are seasonal and non-technical. | The gate must be offline-authoritative and recoverable by a runbook, not by an engineer. See NFR-OPS-2 and ADR-0003. |
| Nobody on the client side can run a platform team (U6). | Five deployable units, managed services by default. See ADR-0002 and ADR-0005. |
| The regulator and the visitors both push against surveillance. | Anonymous by default, personalization opt-in only. See ADR-0009. |
