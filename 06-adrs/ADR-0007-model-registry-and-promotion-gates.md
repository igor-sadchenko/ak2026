# ADR-0007: Model registry with promotion gates

**Status:** accepted
**Deciders:** architect, capability approvers
**Relates to:** NFR-AI-2, NFR-AI-4, [FF-17](../04-verification/fitness-functions.md#ff-17), [../04-verification/ai-validation.md](../04-verification/ai-validation.md)

## Context

Criterion 6 asks how AI is validated and verified. A model is a component whose correctness
is statistical, which means the usual release control ("the tests passed") does not apply
without being redefined. Without a mechanism, "we evaluate our models" is an assertion, and
in a system operated by 2 people it will decay into "we evaluated it once".

## Decision

A registry records, for every capability version: the artefact or provider and model id, the
prompt version where applicable, the golden set id, the evaluation results, the named
approver, the expected cost per 1,000 calls, and the rollback target. **The registry refuses
a production binding whose evaluation artefact is missing, failing, or older than the model
artefact.** That refusal is the enforcement point; everything else is documentation.

Promotion requires all of: the offline threshold met on a held-out golden set; calibration
within tolerance; a margin over the deterministic fallback; a completed shadow period; a
measured cost inside budget; a passing fallback test; and a named human approver who is the
domain owner, not an engineer.

Rollback is always to the previous **bound version**, because that is a thing the registry
knows, unlike "the last good model".

## Alternatives considered

| Alternative | Why not |
| --- | --- |
| Adopt a full MLOps platform | It would be the largest operational commitment in the system, for five capabilities and two engineers. We build about 3% of one, in roughly a week. The trigger to flip this decision is stated below. |
| Track models in a spreadsheet with a manual checklist | Nothing enforces it. The whole value here is the mechanical refusal to bind an ungated model; a checklist is exactly what decays. |
| Automatic promotion when metrics pass | Removes the domain owner from the loop. For welfare, the head keeper carries the consequence of a bad model and must therefore carry the decision. |
| Promote on offline metrics only, skipping shadow | Offline metrics on 20 labelled events do not predict live behaviour, and the alert-rate constraint that actually protects the keepers can only be measured live. |

## Consequences

**Good**

- "No model in production without a passed gate" is checkable rather than promised ([FF-17](../04-verification/fitness-functions.md#ff-17)).
- Rollback is a defined operation that one engineer can perform under pressure.
- The registry makes the cost of a provider change visible, which is what ADR-0008 depends on.

**Bad**

- We own a small piece of infrastructure that a vendor would otherwise own. It needs maintenance and it will be tempting to extend.
- Promotion takes weeks, not hours, because of the shadow period. This is intentional and it will feel slow.
- A domain owner who is on holiday blocks a promotion. Accepted; nothing here is urgent enough to need an out-of-hours model release.

**Risks**

| Risk | Mitigation | Where tracked |
| --- | --- | --- |
| The registry grows into a platform nobody asked for | Flip to a bought platform when capability count passes about 12 or a second estate appears | Reviewed at each phase gate |
| Golden sets go stale and gates become meaningless | Golden sets grow with each adjudicated disagreement; growth is tracked as a metric | [../04-verification/ai-validation.md](../04-verification/ai-validation.md) |
| Approvers rubber-stamp | The approver is the person who suffers the consequence, which is the strongest available mitigation | - |

## How we will know this was right

[FF-17](../04-verification/fitness-functions.md#ff-17): 100% of production model versions
have a passing gate and a named approver, audited monthly, from Phase 1 onward. A single
production model without a linked evaluation artefact means the enforcement point failed and
the design is wrong, not the process. If maintaining the registry exceeds roughly two
engineer-days per quarter, buy a platform instead.
