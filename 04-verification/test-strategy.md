# Test strategy for the non-AI system

Most of this system is not AI, and most of the ways it can hurt the estate are not AI
failures. This page covers how the deterministic parts are tested. The AI parts are in
[ai-validation.md](ai-validation.md); the running checks are in
[fitness-functions.md](fitness-functions.md).

## Shape of the test suite

| Level | Scope | Where it runs | Roughly |
| --- | --- | --- | --- |
| Unit | Entitlement token signing and verification, allow-list matching, redemption rules, reconciliation conflict logic, threshold rule evaluation | Pipeline, every commit | Fast, the bulk of the suite |
| Module contract | Each cloud module against its published interface and its event schemas | Pipeline, every commit | One suite per module |
| Integration | Edge node against a real broker and a real database, on a real image | Pipeline, every merge | Slower, gated |
| System | A full estate simulation: 8 nodes, 600 synthetic devices, a day of traffic | Nightly | One run |
| Field | Real hardware on the estate | Per install wave and before each season | Manual, witnessed |

## The four scenarios that actually matter

These come from the failure modes in the brief and the drivers in
[../01-architecture/drivers-and-characteristics.md](../01-architecture/drivers-and-characteristics.md).
Everything else is ordinary software testing.

### 1. Uplink loss

| Aspect | Detail |
| --- | --- |
| What we simulate | Cellular interface down for 1 minute, 1 hour, 24 hours and 72 hours; and, separately, a flapping link that comes and goes every 30 seconds |
| What must hold | Gates admit; sales continue; telemetry buffers without loss; the offline banner appears within 60 seconds; nothing retries itself into a storm |
| The flapping case is the interesting one | A clean outage is easy. A link that returns for 20 seconds and drops again is what breaks store-and-forward implementations, because partial batches and duplicate sends collide. We test it explicitly and assert idempotent delivery. |
| Automated as | [FF-01](fitness-functions.md#ff-01), [FF-02](fitness-functions.md#ff-02), [FF-07](fitness-functions.md#ff-07) |

### 2. Gate reconciliation

| Aspect | Detail |
| --- | --- |
| What we simulate | Two gates admit the same family pass while both are offline; a pass is revoked centrally during an outage and presented at a gate; a node's buffer replays twice; a node is replaced mid-day and its buffer is lost |
| What must hold | Every case produces a defined outcome and a report line, not an exception. Conflicts are detected, counted and attributed to a gate and a time. The visitor is never refused because of a reconciliation concern; the estate absorbs the cost and measures it. |
| Property tested | Replay is idempotent: applying the same redemption batch twice produces the same ledger state. This is asserted with a property-based test over generated redemption sequences, not with three hand-written cases. |
| Automated as | Part of [FF-01](fitness-functions.md#ff-01); the property test runs on every commit |

### 3. Peak load

| Aspect | Detail |
| --- | --- |
| What we simulate | 3,000 entries per hour across the fleet (NFR-SCALE-1); the 10x reconnect burst when 8 nodes drain 72 hours of buffer simultaneously; a busy-day replay at 3x |
| What must hold | Scan latency holds at p95 500 ms; the backlog clears within an hour; no consumer restarts; no message loss |
| Why the burst matters more than the steady load | Steady load is bounded by physical gates. The reconnect burst is the genuinely spiky event, it happens exactly when everyone is already unhappy, and it is the load case most designs forget to size for. |
| Automated as | [FF-05](fitness-functions.md#ff-05), [FF-06](fitness-functions.md#ff-06) |

### 4. Chaos at the edge

| Aspect | Detail |
| --- | --- |
| What we inject | Power loss mid-write; disk full on a node; clock skew of 10 minutes between a node and the cloud; a sensor stuck reporting its last value; a camera returning black frames; an expired client certificate |
| What must hold | Power loss and disk full degrade to refusing new telemetry rather than corrupting the queue. Clock skew must not admit an expired entitlement or reorder telemetry: timestamps are node-local plus a recorded offset, and validity is evaluated against a monotonic clock with a bounded tolerance. A stuck sensor is detected as stuck (a flat line is a fault, not a reading), and a black-frame camera is detected before its frames reach a model. |
| The two we care most about | The stuck sensor and the black camera, because both look like healthy data. A model trained on or scored against a stuck sensor produces confident nonsense, and the failure is silent. Data quality checks run before ingestion, not after. |
| Automated as | Nightly chaos suite in the system environment; the certificate case is [FF-09](fitness-functions.md#ff-09) |

## Data quality gates

Everything the models depend on passes these before it lands. This is the cheapest defence
against a whole class of AI failures, and it is not itself AI.

| Check | Rejects |
| --- | --- |
| Range | A reading outside the sensor's physical range |
| Flatline | A sensor reporting an identical value for longer than its configured plausibility window |
| Rate of change | A jump larger than physics allows for that metric |
| Freshness | A reading whose timestamp is ahead of now or older than its buffer window |
| Frame quality | A camera frame below a brightness, sharpness or coverage threshold |
| Completeness | An enclosure reporting fewer than its expected sensors for a window |

Rejected records are quarantined with a reason, counted, and surfaced on the operations
dashboard. A rising rejection rate is an early warning of hardware failure and is often the
first signal available.

## Restore and continuity drills

| Drill | Cadence | Success |
| --- | --- | --- |
| Cold restore of an edge node by a non-engineer | Quarterly and with each seasonal intake | [FF-13](fitness-functions.md#ff-13) |
| Database point-in-time restore into a scratch environment | Quarterly | Restored to a chosen minute; row counts verified against a known day |
| Full estate opening-day rehearsal | Before each season | Gates, sales, counting and welfare alerts exercised end to end with real staff |
| Manual fallback rehearsal (paper list, manual gate) | Before each season | Gate staff admit 100 visitors without any system at all, timed |

The last one deserves a note. The manual fallback is the floor beneath the entire
architecture: if everything we build fails, the estate still opens. Rehearsing it costs an
hour a year and it is the reason a P1 at the gate is a bad morning rather than a closed
estate.

## What we do not test, and why

| Not tested | Reason |
| --- | --- |
| Cloud provider regional failover | We accept a regional outage as a degraded day; the edge keeps admitting and the estate keeps operating. Building and testing multi-region for this client would cost more than the risk. Stated as an accepted risk, not an oversight. |
| Browser matrix beyond the two most common mobile browsers | Assumption, revisited if analytics show otherwise. |
| Penetration testing in Phase 0 | Scheduled for Phase 1, once there is more than a ticketing surface to test. The Phase 0 surface is a hosted checkout and a scanner, and the pipeline checks in [FF-08](fitness-functions.md#ff-08) to [FF-10](fitness-functions.md#ff-10) cover its specific risks. |
