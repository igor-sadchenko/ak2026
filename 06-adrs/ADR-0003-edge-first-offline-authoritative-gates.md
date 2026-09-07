# ADR-0003: Edge-first, with offline-authoritative gates

**Status:** accepted
**Deciders:** architect
**Relates to:** F10, F11, NFR-AVAIL-1, NFR-AVAIL-2, NFR-DATA-1, FR-3, FR-4

## Context

Wi-Fi coverage across the estate is patchy (F10) and a link to the cloud has to be built
(F11), which is to say connectivity is a known-bad resource. The two things that must never
stop happen in the field: admitting visitors at a gate (FR-3) and alerting a keeper about an
enclosure (FR-10). A queue at a closed gate is lost revenue on the day the estate is
busiest, which is exactly when the network is most likely to be under strain.

The conventional design puts the entitlement check in the cloud and the scanner on the
network. That design has one failure mode and it is the failure the brief describes.

## Decision

Edge nodes are authoritative for admission while disconnected. A ticket or pass is a
**signed token**; each node holds an allow-list of entitlements valid for today and
tomorrow, plus a revocation delta, and verifies the signature offline against a public key.
Redemptions are logged locally and replayed to U2 on reconnect, where they are reconciled
and conflicts are detected, counted and reported. Telemetry is buffered locally for 72 hours
with per-device ordering. Offline ticket sales issue a locally signed entitlement queued for
settlement.

We accept, explicitly, that during a full disconnection a revoked entitlement may be
admitted once, and the same family pass may be used at two gates. The cost is bounded, it is
measured, and it appears in the reconciliation report.

## Alternatives considered

| Alternative | Why not |
| --- | --- |
| Online validation with a cached fallback that fails closed | Fails closed means refusing paying visitors during an outage. The business cost of a refused visitor exceeds the cost of a duplicated ticket by a wide margin. |
| Online validation with a cache that fails open (admit everyone) | Unbounded loss and no record of what happened. Our design fails open in a *bounded, recorded* way, which is a different thing. |
| Strong consistency across gates via a distributed consensus protocol at the edge | Requires gates to reach each other, which is the assumption that F10 breaks. Adds a consensus system for two engineers to operate. |
| Paper tickets only | The manual fallback exists and is rehearsed, but it cannot support FR-2 family passes or the pre-booking that O1 depends on. |

## Consequences

**Good**

- The estate opens and sells on a bad network day. NFR-AVAIL-1 and NFR-AVAIL-2 are met at the design level, not by redundancy spending.
- Every AI capability inherits a system that already works without the cloud, which is what makes the fallbacks in ADR-0012 credible.
- Edge nodes hold no original data except the unforwarded buffer, so losing one is bounded.

**Bad**

- Duplicate logic: validation rules exist at the edge and in U2, and they must not drift.
- A real reconciliation protocol with real conflict semantics is code we own and must test.
- Revocation has a lag equal to the outage length.

**Risks**

| Risk | Mitigation | Where tracked |
| --- | --- | --- |
| Edge and cloud validation logic diverge | Shared rule definition, contract tests both sides, and a nightly comparison on sampled decisions | [FF-14](../04-verification/fitness-functions.md#ff-14) |
| Fraud through deliberate gate-hopping during outages | Conflict rate is monitored per gate and per entitlement; a pattern triggers a commercial response, not a technical one | [FF-01](../04-verification/fitness-functions.md#ff-01) |
| Clock skew admits an expired entitlement | Node-local time plus a recorded offset, bounded tolerance, tested in the chaos suite | [../04-verification/test-strategy.md](../04-verification/test-strategy.md) |
| A flapping link causes duplicate or partial replay | Idempotent replay, asserted with a property-based test | [../04-verification/test-strategy.md](../04-verification/test-strategy.md) |

## How we will know this was right

[FF-01](../04-verification/fitness-functions.md#ff-01) quarterly: with the uplink
disconnected for 72 hours, 100% of valid entitlements are admitted and the reconciliation
conflict rate stays below 0.5% of redemptions. If the conflict rate exceeds 2% in normal
operation, the allow-list refresh cadence or the validity window is wrong and this ADR is
revisited. If the estate never actually loses connectivity for more than a few minutes in
two seasons, the design was more expensive than it needed to be, and we should say so.
