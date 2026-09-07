# How we used AI to design this

The kata is called AI-Assisted Software Architecture. Most submissions answer the question
"where is the AI in your system". This page answers the other half: **where was the AI in
our own work, what did it get wrong, and how did we catch it.**

It is written as a record, not as an advertisement. Where an entry needs a concrete example
from our actual working sessions, there is a `[TODO]` marker; those are filled in by a
human from the session transcripts, because inventing plausible examples on this page in
particular would be self-refuting.

## Tools and where they were used

| Stage | Tool class | What it did | Human role |
| --- | --- | --- | --- |
| Understanding the brief | Conversational LLM | Extracted the stated facts and, more usefully, produced a list of what the brief does *not* say | Human decided which gaps mattered enough to become tracked assumptions ([../00-problem/constraints-assumptions.md](../00-problem/constraints-assumptions.md)) |
| Option generation | Conversational LLM | Generated candidate architecture styles, decomposition options, AI capability ideas | Human selected, rejected and set the evaluation criteria before seeing the options |
| Drafting | Coding agent with file access | Produced first drafts of structured documents from a human-written outline | Human wrote the outline, the angle and the constraints; the model wrote prose into that shape |
| Consistency checking | Coding agent | Cross-reference checking, ID coverage (every NFR has a fitness function), formatting rules | Human defined the invariants; this is the task the model did best |
| Adversarial review | Second LLM, different vendor | Was asked to attack the design as a hostile reviewer | Human judged which criticisms were real |
| Numbers | LLM plus a spreadsheet | Draft cost model | Human recomputed every figure by hand. See "what we did not trust" below. |

`[TODO: fill in with facts from the actual sessions - specific tools, models and versions used, and roughly how many working hours at each stage.]`

## Division of labour, and where the line is

| Work | Owner | Why the line is here |
| --- | --- | --- |
| Choosing the angle of the submission | Human | This is a judgement about what a specific audience values. The model has no stake and no taste. |
| Deciding what the client can operate | Human | It depends on facts about people, not about software. Every model we asked defaulted to designs a small team could not run. |
| Generating options | Model | Cheap, fast, and breadth is exactly where a model beats a tired human at 11 p.m. |
| Evaluating options against criteria | Human, with the model as a devil's advocate | Models agree with the framing they are given. Ask "which of these is best" and you get a confident answer to a question you may have posed badly. |
| Writing structured prose from a decided outline | Model | The highest-value use. Turning a decided structure into tables and paragraphs is real work that the model does well. |
| Inventing numbers | Neither | The model will produce plausible figures on request. Every number here is either a brief fact or a human-checked assumption. |
| Cross-reference and consistency checks | Model | Mechanical, tedious, and the model does not get bored on the 40th link. |
| Deciding what to cut | Human | Models add. Asked to improve a document, they lengthen it. Every deletion in this submission was a human decision. |

## Where we rejected what the AI proposed

The prompt to a model shapes what it produces, so a list of rejections says as much about
our process as about the model. At least five concrete cases, with reasons:

**1. `[TODO: fill in with a real rejected proposal from the sessions]`**
Pattern it belongs to: the model proposed a component the brief did not require.
Reason for rejection: `[TODO]`.

**2. `[TODO: fill in]`**
Pattern: the model proposed a finer decomposition than the client's team can operate.
This one is the most common failure mode we saw, and it is why ADR-0002 exists and is
argued rather than asserted. Reason for rejection: the operating model in
[../03-delivery/team-and-operating-model.md](../03-delivery/team-and-operating-model.md)
puts a hard ceiling on the number of things that can be on call.

**3. `[TODO: fill in]`**
Pattern: the model proposed an AI capability where a deterministic rule is sufficient.
The four rejections in [../02-ai-capabilities/README.md](../02-ai-capabilities/README.md)
are the surviving record of this. Reason for rejection: ADR-0006.

**4. `[TODO: fill in]`**
Pattern: the model proposed an impressive capability whose validation strategy it could not
describe when asked. Reason for rejection: criterion 6. If we cannot say how we would know
it works, we do not ship it.

**5. `[TODO: fill in]`**
Pattern: the model proposed a technology by name (a specific platform, framework or managed
product) without connecting it to a requirement. Reason for rejection: our own rule that
every component answers a requirement.

**6. `[TODO: fill in]`**
Pattern: the model proposed a privacy-invasive mechanism because it improves a metric.
Reason for rejection: ADR-0009, and the fact that the metric it improves is the same one
the trust cost damages.

## Where the model was wrong or invented things

| What happened | How we caught it |
| --- | --- |
| `[TODO: a specific hallucinated fact, figure or citation, and what it was]` | `[TODO: what caught it - manual check, second model, source lookup]` |
| `[TODO: a case where the model was confidently wrong about a technical mechanism]` | `[TODO]` |
| `[TODO: a case where the model silently dropped a constraint from an earlier instruction]` | `[TODO]` |
| Plausible numbers produced on request without a stated basis | We recomputed the cost model by hand. This is a general property, not an incident: a request for a figure produces a figure. |
| Agreement with a premise we had stated badly | Caught by re-asking the same question from the opposite premise and seeing whether the answer flipped. It sometimes did. |

## How we checked the model's output

| Method | What it caught | Cost |
| --- | --- | --- |
| Recomputing every number by hand | Arithmetic that looked right and was not; unit and currency slips | Low, and non-negotiable |
| Asking a second model from a different vendor to attack the design | Framing blind spots. A single model tends to defend the design it just produced. | Low |
| Asking for the strongest argument *against* a decision we had already made | Weak reasoning in our own ADRs, which is why the "Alternatives considered" tables are specific rather than dismissive | Low, and the highest value per minute of anything on this list |
| Mechanical consistency checks over IDs and links | Broken cross-references, requirements with no fitness function | Very low, fully automated |
| Reading the whole thing end to end as a human | Repetition, drift in terminology, sections that sounded authoritative and said nothing | High, and irreplaceable |

## What AI was cheap and good at, and what cost more than doing it by hand

| Cheap and good | Why |
| --- | --- |
| Turning a decided outline into structured tables | Genuinely fast, and the output needed light editing rather than rewriting |
| Enumerating alternatives we had not considered | Breadth on demand; a few of them were good, which is enough |
| Consistency and cross-reference checking | Tireless, mechanical, exactly the right task shape |
| Rewriting for a stated constraint (shorter, ASCII only, tables not prose) | Reliable and instant |
| First-pass adversarial review | Cheaper than a colleague's time and available at any hour |

| More expensive than doing it by hand | Why |
| --- | --- |
| Anything numeric | Every figure had to be recomputed anyway, so the model's contribution was a formatted guess. Faster to derive it once and have the model format it. |
| Deciding what to remove | Consistently additive. Asking for a shorter document produced a differently long one. |
| Judgements about people and organisations ("can a 2-person team run this") | Confidently wrong in a direction that is hard to notice, because the answer sounds professional |
| Deep domain specifics we could not verify | Any claim about exotic animal husbandry took longer to check than to leave out. We left most of it out; this is why [cap-01](../02-ai-capabilities/cap-01-animal-welfare-anomaly.md) defers species specifics to the head keeper. |
| `[TODO: an instance where prompting and correcting took longer than writing it directly]` | `[TODO]` |

## Effect on the architecture itself

Two things in this submission exist *because* of how we worked, not despite it:

1. **The admission rule for AI capabilities (ADR-0006).** We noticed that our own sessions
   produced AI capabilities faster than they produced justifications for them. The rule
   "state the deterministic alternative first" was written to discipline us, and it turned
   out to be the most useful design constraint in the whole portfolio. Four capabilities
   died to it, and they are listed at the top of
   [../02-ai-capabilities/README.md](../02-ai-capabilities/README.md).
2. **Fitness functions as the deliverable rather than the appendix.** Working with a
   generator of plausible text for several days makes the difference between an assertion
   and a check very concrete. That experience is why
   [../04-verification/fitness-functions.md](../04-verification/fitness-functions.md)
   exists in this shape.

## What we would repeat, and what we would not

| Would repeat | Reason |
| --- | --- |
| Human writes the outline and the constraints; model writes into that shape | The one pattern that was reliably faster and produced better documents |
| Adversarial second model from a different vendor | Cheap, and it found real problems |
| Recompute every number by hand | Not optional |
| Mechanical checks over IDs and links | Free correctness |
| Keep a rejection log while working | This page would have been impossible to write afterwards, and its absence is the gap we found in every other submission we reviewed |

| Would not repeat | Reason |
| --- | --- |
| Asking a model to generate an architecture from the brief in one pass | It produces something complete-looking and generic, and the effort to make it specific exceeded the effort of starting from a decided angle |
| Long unstructured conversation as the working mode | Decisions get made implicitly and are then hard to find. Writing decisions into files as they are made is slower per hour and much faster overall |
| Trusting a model's judgement about scope | Every scope judgement we accepted without checking was too large |
| `[TODO: anything else from the sessions worth not repeating]` | `[TODO]` |

## Honest limitations of this page

- It is written from memory and from session transcripts, not from an instrumented log. A
  rigorous version would record every prompt and every accepted or rejected suggestion at
  the time. We recommend that to anyone doing this again; see the "would repeat" row about
  keeping a rejection log.
- Rejection counts are not a quality measure. A model asked for ten ideas produces ten
  ideas, and rejecting eight of them is the expected outcome, not a finding.
- We cannot separate the model's contribution from the prompt's. A better-posed question
  produced better output every time, which means most of what looks like model performance
  on this page is actually a property of how we asked.
