# ADR-0016: Retrieval over a curated corpus, not fine-tuning

**Status:** accepted
**Deciders:** architect
**Relates to:** [cap-05](../02-ai-capabilities/cap-05-guest-companion.md), NFR-AI-5, [FF-20](../04-verification/fitness-functions.md#ff-20)

## Context

The assistant (FR-12) must answer open natural language questions about the estate from
information the estate controls: opening times, ride restrictions, species facts, safety
rules, procedures, accessibility, catering. Some of that information changes weekly (opening
times, ride closures) and some of it is safety-relevant, where a plausible wrong answer does
real harm.

The corpus is small: a few thousand short documents (assumption). There is no proprietary
text corpus of any size worth training on.

## Decision

Retrieval-augmented generation over a curated corpus, with an **embedded index inside the
cloud deployable, rebuilt nightly**. No fine-tuning. No self-hosted model. No managed vector
database.

Three properties this buys, and they are the reasons rather than side effects:

1. **Citations.** Every answer names the passages it came from, and a guardrail rejects
   uncited or out-of-corpus claims. The whole verification approach for this capability rests
   on citation correctness being checkable, which a fine-tuned model cannot offer.
2. **Instant updates.** A changed opening time is a document edit and a nightly rebuild.
   Under fine-tuning it would be a retraining cycle, which guarantees the model would be
   stale exactly on the facts that change most.
3. **Instant withdrawal.** A factual error is fixed by removing the source from the index in
   minutes (level 4 of the capability's degradation ladder), not by a release.

Safety-relevant topics never reach generation at all: a classifier routes them to fixed,
human-written answers before the model is called (NFR-AI-5,
[FF-20](../04-verification/fitness-functions.md#ff-20)).

## Alternatives considered

| Alternative | Why not |
| --- | --- |
| Fine-tune a model on the estate corpus | More expensive, stale on the facts that change most, and it destroys the citation property that makes this capability verifiable. Criterion 6 asks how we validate; "the model learned it" is not an answer. |
| Self-host an open-weights model on the estate | GPU hardware and an operational burden for a spiky, low-volume workload, on a team of two. Revisit only if provider cost or data residency becomes binding. |
| Managed vector database | The corpus fits in memory. A second system to operate, with its own availability and cost, for a dataset this size. |
| Long-context prompting with the whole corpus in every request | Cost scales with corpus size on every call, and retrieval quality gets worse as the corpus grows, not better. |
| No assistant at all: better search and signage | A serious option, and it is precisely the fallback. We ship the assistant because the long tail of questions is real, and we note in [../02-ai-capabilities/README.md](../02-ai-capabilities/README.md) that if one capability had to be cut, this is the one. |

## Consequences

**Good**

- Answers are checkable against a source, so the quality gate is objective rather than a judgement of helpfulness.
- Corpus edits take effect overnight and withdrawals take effect in minutes.
- No training pipeline, no GPU, no vector database to operate. The whole capability is a nightly index build plus a gateway call.

**Bad**

- Answer quality is bounded by the corpus. The most common real failure will be a missing page, not a model error, which means the ongoing cost is writing rather than engineering.
- Retrieval failures produce a "here are some search results" experience that is visibly weaker than a confident answer.
- Nightly rebuild means an urgent correction needs a manual index refresh, which is a documented operation.

**Risks**

| Risk | Mitigation | Where tracked |
| --- | --- | --- |
| Corpus gaps concentrated in the most-asked topics | Escalation and thumbs-down rates tracked per topic; gaps become writing tasks | [cap-05](../02-ai-capabilities/cap-05-guest-companion.md) |
| The corpus is never maintained after handover | Corpus review is an assigned duty in the operating model, and the top-question cache makes the highest-traffic answers the most reviewed | [../03-delivery/team-and-operating-model.md](../03-delivery/team-and-operating-model.md) |
| Someone adds safety advice to the corpus, making it generatable | The safety classifier runs before retrieval, and the red-team suite runs on every deploy | [FF-20](../04-verification/fitness-functions.md#ff-20) |

## How we will know this was right

Offline gate at Phase 2: at least 95% of 200 golden questions answered with a correct
citation, and 100% of 50 adversarial safety prompts refused or deflected. In production, the
grounded-citation rate stays above 90% weekly. If the dominant failure mode turns out to be
retrieval quality rather than corpus gaps, the index approach is too simple and a managed
retrieval service becomes worth its operational cost.
