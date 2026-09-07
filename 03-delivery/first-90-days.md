# First 90 days

Week-by-week breakdown of Phase 0 from [implementation-plan.md](implementation-plan.md).
Twelve weeks, 4 FTE. Every week has a proof point: something observable at the end of it.
Durations are `(assumption)`.

Roles: **LE** lead engineer, **E1/E2** engineers, **FT** field technician (part-time),
**OM** the client's operations manager (0.5 FTE, client side).

| Week | Focus | Proof point at the end of the week | Owner |
| --- | --- | --- | --- |
| 1 | Site survey. Walk the estate: entrances, power, existing cabling, cellular signal strength per zone, tank and enclosure locations. Confirm A1 (area), A2 (entrances), A4 (legacy), A5 (coverage). | A signal map and an entrance count, both written down. Any failed assumption triggers its playbook now, not in month 6. | FT, LE, OM |
| 2 | Decide and order. Edge node and gate terminal selection, cellular contracts, counting sensor type. Cloud account, two environments, CI pipeline skeleton. | Hardware ordered with delivery dates. A "hello world" deploy runs through the pipeline into staging. | LE, E1 |
| 3 | Entitlement model. Token format, signing, validity windows, allow-list shape, revocation deltas. This is the design that everything else in Phase 0 hangs from. | A signed token is issued by a test service and verified by a script with no network access. | LE, E2 |
| 4 | Ticketing core. Orders, entitlements, family passes, PSP integration (hosted checkout, so no card data touches us, NFR-SEC-1). | A ticket is bought end to end in staging with a test card, and the PSP holds all card data. | E1, E2 |
| 5 | Edge node image. Base image, MQTT broker, store-and-forward queue, health beacon, configuration pull, certificate enrolment. | A node boots from the image on a bench, enrols, and forwards a synthetic message. Reflash takes under 10 minutes. | LE, FT |
| 6 | Gate application. Scan, offline verify, local redemption log, staff-visible offline banner, manual override with a reason code. | On the bench, with the network cable pulled, a terminal admits a valid token and refuses an expired one. | E2, FT |
| 7 | Reconciliation. Redemption replay, conflict detection, duplicate-use reporting, settlement of offline sales. | A 48-hour offline period is simulated; replay produces a conflict report with correct counts. | E1 |
| 8 | Install wave 1. Three entrances and two zones: edge nodes, gate terminals, counting sensors, uplinks. | Real scans from real hardware appear in staging. | FT, E2, OM |
| 9 | Counting and reporting. Zone in/out aggregation, ride status board, the one-page commercial report (FR-15 basic). | OM opens the report and finds yesterday's numbers on it without being shown how. | E1, OM |
| 10 | Install wave 2 and staff training. Remaining entrances and zones. Printed runbooks for gate staff: node swap, terminal swap, offline procedure, manual override. | A gate staff member restores a terminal from cold using only the printed card, timed. | FT, OM, E2 |
| 11 | Verification week. Run the Phase 0 exit criterion in full: 72-hour uplink disconnection, 3,000 scans/hour load test, two-zone manual count audit, cold restore drill. Set up the fitness function dashboard so these checks keep running. | Signed test results against all four exit conditions. Failures are fixed in week 12, not explained away. | LE, all |
| 12 | Season opening and handover. Go live. Two engineers on site for opening day. Fitness functions running on schedule. Baseline metrics recorded in the OKR table. | The estate opens, sells, admits and counts. The `unknown` column in [../00-problem/okrs.md](../00-problem/okrs.md) is filled in. | all |

## What is explicitly not in the first 90 days

| Not included | Reason |
| --- | --- |
| Enclosure sensors | The keeper team is preparing for the season. Their attention is the scarce input for Phase 1, and it is not available in these 12 weeks. |
| Any model | There is no data and no baseline. A model shipped here could not be evaluated, which would break NFR-AI-2 in the first quarter of the programme. |
| Mobile app | A13. The web client covers FR-1 and FR-12 later. |
| A second cloud region, autoscaling, or a disaster-recovery site | The failure that actually happens here is a lost uplink, and that is handled at the edge. Cloud redundancy at this stage would be insurance against the rarer risk. |

## The three things most likely to go wrong in these 12 weeks

| Risk | Early signal | Response |
| --- | --- | --- |
| Hardware delivery slips past week 8. | Order confirmation in week 2 shows a date after week 7. | Install wave 1 with two entrances instead of three and keep the verification week intact. The exit criterion allows 6 gates; below that, the phase does not exit and go-live slips openly. |
| Cellular coverage is worse than the week 1 survey suggested. | Queue depth on a node climbs and does not drain overnight. | This is the A5 playbook. Nothing in the gate path depends on the uplink, so the season still opens; reporting freshness degrades and is reported as degraded. |
| Gate staff do not trust or use the offline banner and manual override. | Override counts near zero during the training drill, or wildly high. | Redesign the override flow with the staff in week 10-11. This is a usability defect with a revenue impact, and it is cheaper to fix before opening day than after. |
