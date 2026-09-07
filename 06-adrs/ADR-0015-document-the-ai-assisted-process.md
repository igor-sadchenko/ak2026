# ADR-0015: The AI-assisted design process is a deliverable

**Status:** accepted
**Deciders:** architect
**Relates to:** kata criteria 1 and 4, [../05-process/how-we-used-ai.md](../05-process/how-we-used-ai.md)

## Context

The kata's theme is AI-Assisted Software Architecture. Most submissions, including the two
others produced by our own team, answer only the first half of that: where AI sits inside
the designed system. The second half, how the team used AI while designing, appears as a
one-line disclaimer or not at all.

This matters beyond scoring. A team that used a model to produce an architecture and cannot
say where the model was wrong has no evidence that it checked.

## Decision

Treat the process as a first-class artifact with the same standards as the rest of the
submission:
[../05-process/how-we-used-ai.md](../05-process/how-we-used-ai.md) and
[../05-process/decision-log.md](../05-process/decision-log.md).

Four content rules, because a page like this is easy to write badly:

1. **Record rejections while working, not afterwards.** At least five concrete cases where a
   model's proposal was rejected, each with a reason. Reconstructing these later produces
   flattering fiction.
2. **Record errors and hallucinations specifically**, including what caught them.
3. **Name what cost more than doing it by hand.** A page that reports only successes is
   marketing.
4. **Never invent an example.** Where a real one is missing, the page carries an explicit
   `[TODO]` marker rather than a plausible story. On this page above all others, a fabricated
   example would refute the document's own argument.

The process also shaped the design, and that link is recorded: ADR-0006 (the AI admission
gate) exists because our own sessions produced capabilities faster than justifications, and
[../04-verification/fitness-functions.md](../04-verification/fitness-functions.md) has the
shape it does because several days of working with a generator of plausible text makes the
difference between an assertion and a check very concrete.

## Alternatives considered

| Alternative | Why not |
| --- | --- |
| A disclaimer line ("AI tools were used in preparing this document") | Says nothing verifiable and answers none of the criteria. |
| A section inside the README | It would be read as a footnote. The kata's title puts this at the centre, so it gets its own block. |
| A prompt log appended as an artifact | Long, unreadable, and it shows what we asked rather than what we decided. The decisions and the rejections are the useful record; the transcript is the raw material. |
| Omitting it because the judges asked about the system, not the process | Criterion 1 (innovative use of AI) and criterion 4 (dealing with uncertainty) both read naturally as applying to the team's own use of AI, and no other submission we reviewed covers it. |
| Writing plausible examples where real ones are missing | It would be the one dishonesty that invalidates the whole document, and a reader who has used these tools would spot it. |

## Consequences

**Good**

- Answers the half of the kata's theme that the other submissions leave empty.
- Forced a working practice (keeping a rejection log while designing) that improved the design itself, notably ADR-0006.
- Gives the client's team a record of which parts of the design were most heavily model-assisted and therefore most in need of human re-checking later.

**Bad**

- Publishing your own mistakes is uncomfortable and can be read as a weakness rather than as rigour.
- The `[TODO]` markers are visible in the submitted artifact unless a human fills them in first. We prefer visible gaps to invented content, but a judge may not.
- Keeping the log costs time during design, when the temptation is to move on.

**Risks**

| Risk | Mitigation | Where tracked |
| --- | --- | --- |
| The page reads as self-congratulatory | It is structured around rejections, errors and things that cost more than doing them by hand, and it ends with an honest-limitations section | [../05-process/how-we-used-ai.md](../05-process/how-we-used-ai.md) |
| `[TODO]` markers ship unfilled | They are the only TODOs anywhere in the submission, so they are trivial to find with a grep before submitting | final consistency check |
| The record is reconstructed from memory and is therefore unreliable | Stated openly in the page's own limitations section, with the recommendation to instrument next time | - |

## How we will know this was right

Immediate test: a reader can name three specific things our AI tooling got wrong, and three
things it did well, after reading one page. Longer test: the practice of recording rejections
during design continues into Phases 1 to 4, where the same discipline applies to the
capability portfolio. If the page ships with unfilled `[TODO]` markers, the decision was
right and the execution was not.
