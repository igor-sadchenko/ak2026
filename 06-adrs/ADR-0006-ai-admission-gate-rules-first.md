# ADR-0006: AI admission gate - state the deterministic alternative first

**Status:** accepted
**Deciders:** architect, with every capability owner
**Relates to:** kata criterion 2, [../02-ai-capabilities/README.md](../02-ai-capabilities/README.md), D3

## Context

The kata's theme creates a pull toward putting AI everywhere, and our own working sessions
produced candidate AI capabilities faster than they produced justifications for them (see
[../05-process/how-we-used-ai.md](../05-process/how-we-used-ai.md)). Meanwhile, the client
has 2 engineers in steady state, and every model added is a thing that must be evaluated,
monitored, recalibrated and eventually retrained by somebody.

An unjustified model is worse than an unnecessary microservice, because its failure is
statistical and silent rather than loud.

## Decision

No capability enters the portfolio until its file has answered, in its opening section:

> What deterministic rule, query or calendar would we write instead, and what specifically
> does it fail to do?

If the deterministic alternative is adequate, it ships and the model does not. If the model
ships, the deterministic alternative **still ships**, as the fallback (ADR-0012). This is a
review gate on every capability proposal and on every ADR that introduces one.

Four candidates died to this rule and are listed at the top of the portfolio page, not
hidden: zone popularity (counters and a chart), feeding adherence (a schedule and an
escalation), ride maintenance (fixed certified intervals), and staff rostering (one person
with a spreadsheet). A fifth, dynamic pricing, was rejected on suitability grounds (D5).

## Alternatives considered

| Alternative | Why not |
| --- | --- |
| Judge each capability on expected business value alone | Value estimates for unbuilt models are guesses, and they are guesses that favour the impressive option. The rule-comparison question is answerable today, with evidence. |
| A governance board reviewing AI proposals | A board is a process for an organisation with more than 6 people. The rule is a checklist item in a document review, which is what fits here. |
| Allow AI anywhere and rely on the fitness functions to catch problems | The fitness functions catch a bad model. They do not catch a model that should not exist, because it may work perfectly and still be a permanent maintenance cost for no gain. |
| No rule; rely on the architect's judgement | It is the architect's judgement that the rule exists to discipline. Writing it down means a reviewer can hold us to it. |

## Consequences

**Good**

- The portfolio is five capabilities rather than twelve, and each has a written justification a judge can check.
- The rejected list is itself evidence of suitability (criterion 2), and it is more persuasive than the accepted list.
- Every accepted capability automatically has a deterministic fallback, because the alternative was written down before the model was built.

**Bad**

- Scores lower on a naive reading of criterion 1 (innovation). Fewer capabilities look like less ambition.
- The rule can be gamed by writing a deliberately weak deterministic alternative. Mitigated only by the reviewer taking the alternative seriously.
- Some genuinely valuable capabilities may be deferred too long while the rule version is measured.

**Risks**

| Risk | Mitigation | Where tracked |
| --- | --- | --- |
| The rule becomes a formality that everything passes | The count of rejected candidates is reviewed at each phase gate. A quarter with zero rejections is a signal the gate has stopped working. | [../03-delivery/implementation-plan.md](../03-delivery/implementation-plan.md) |
| A rule version is shipped and never revisited even when it clearly underperforms | Each rejected candidate carries a revisit condition (for example, dynamic pricing at 12,000 visitors/day) | [../05-process/decision-log.md](../05-process/decision-log.md) |

## How we will know this was right

At each phase gate: every production capability has a documented deterministic alternative
and a fallback that passes [FF-16](../04-verification/fitness-functions.md#ff-16); and the
rejected candidates are still adequately served by their rules, judged by whether the
requirement they cover has generated complaints. If a rejected candidate's rule is failing
in production, the rejection was wrong and we say so in the decision log rather than
quietly reversing it.
