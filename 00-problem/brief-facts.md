# Brief facts (single source of truth)

Everything in this submission traces back to this page. A statement that is not here and
is not derived here must carry the tag `(assumption)` and appear in
[constraints-assumptions.md](constraints-assumptions.md).

## Facts stated in the brief

| # | Fact | Used by |
| --- | --- | --- |
| F1 | The client is the 72nd Countess von Digitalis, who inherited the estate unexpectedly. | [stakeholders.md](stakeholders.md) |
| F2 | The previous family business (exploding garden gnomes) has ended. There is no going concern to protect. | [../03-delivery/implementation-plan.md](../03-delivery/implementation-plan.md) |
| F3 | 40 rides dating from the 18th century, all of which have passed safety inspections. | FR-11, [../01-architecture/decomposition.md](../01-architecture/decomposition.md) |
| F4 | An exotic and poisonous animal collection: 200+ individuals across 55 enclosures, aquatic and terrestrial, including jumping piranhas. | FR-5, FR-6, FR-7 |
| F5 | A carnivorous plant collection. | F9, [okrs.md](okrs.md) |
| F6 | Current attendance is about 5,000 visitors per day. | [okrs.md](okrs.md) |
| F7 | Target attendance is at least 15,000 visitors per day within 3 years. | [okrs.md](okrs.md), O1 |
| F8 | If the target is missed, the carnivorous plant collection has to be sold. | Framing of O1; the failure mode is commercial, not technical. |
| F9 | Required (brief section D): ticket and family-pass sales; understanding of area popularity; animal health and feeding monitoring plus piranha population control; growth in attendance and profitability; visitor return rate. | [requirements.md](requirements.md), FR-1 .. FR-10 |
| F10 | Constraint (brief section E): Wi-Fi coverage across the estate is patchy. | Top driver, [../01-architecture/drivers-and-characteristics.md](../01-architecture/drivers-and-characteristics.md) |
| F11 | Constraint (brief section E): a link from the estate to the cloud is required. | ADR-0004 |
| F12 | Constraint (brief section E): there is budget for MQTT-capable devices. | ADR-0004, [../03-delivery/cost-model.md](../03-delivery/cost-model.md) |
| F13 | Constraint (brief section E): cloud is permitted. | ADR-0005 |

## What the brief does not say

Named explicitly so that no reader mistakes our choices for the client's requirements.
Each row is expanded, with an impact analysis, in
[constraints-assumptions.md](constraints-assumptions.md).

| # | Not stated | Why it matters |
| --- | --- | --- |
| U1 | Estate area and layout. | Decides backhaul technology and the number of edge nodes. |
| U2 | Ticket prices and current revenue. | Decides whether IT cost is a real constraint. |
| U3 | Existing systems, if any (legacy ticketing, accounting, CCTV). | Decides how much of Phase 0 is integration versus greenfield. |
| U4 | Number of entrances and their physical arrangement. | Decides gate hardware count and peak throughput per gate. |
| U5 | Which species are in the collection, other than piranhas. | Decides welfare sensing per enclosure and the labelling effort. |
| U6 | Size, skills and budget of the client's staff and IT team. | Decides how much system a team can own. This is our binding constraint. |
| U7 | Jurisdiction and applicable data protection law. | Decides the privacy posture and retention rules. |
| U8 | Opening season length, operating hours, weather exposure. | Decides peak sizing and the forecasting horizon. |
| U9 | Whether veterinary staff are on site or on call. | Decides whether a human-in-the-loop welfare design is viable at all. |
| U10 | Existing connectivity contracts and cellular coverage at the estate. | Decides whether the uplink assumption in ADR-0004 holds. |

## Reading rule

- Facts F1 to F13 are quoted, not paraphrased at length, and are never restated as our
  own analysis.
- Anything numeric that is not F6 or F7 is an assumption and is tagged.
- If a judge disagrees with an assumption, [constraints-assumptions.md](constraints-assumptions.md)
  states what breaks and what we would do instead.
