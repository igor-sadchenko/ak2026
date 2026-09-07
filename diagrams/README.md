# Diagrams

Mermaid in markdown, so they render in GitHub without an export step. Every diagram has a
legend, because a diagram whose notation has to be guessed is a quiz.

| Diagram | Level | Answers |
| --- | --- | --- |
| [context.md](context.md) | C4 level 1 | Who and what the system talks to, and which of those links can fail |
| [container.md](container.md) | C4 level 2 | What runs where, across the five units |
| [deployment.md](deployment.md) | Deployment | Physical placement on the estate and in the cloud, including the offline path |
| [sequences.md](sequences.md) | Behaviour | Three flows: offline gate entry and reconciliation, welfare anomaly to vet decision, forecast to staffing action |

Diagrams embedded elsewhere rather than duplicated here:

| Diagram | Where |
| --- | --- |
| Unit decomposition | [../01-architecture/decomposition.md](../01-architecture/decomposition.md) |
| AI Services structure | [../01-architecture/ai-platform.md](../01-architecture/ai-platform.md) |
| Delivery phases (Gantt) | [../03-delivery/implementation-plan.md](../03-delivery/implementation-plan.md) |
| Per-capability data flows | each file in [../02-ai-capabilities/](../02-ai-capabilities/) |

## Notation conventions

| Convention | Meaning |
| --- | --- |
| Solid arrow | Normal data or control flow |
| Dashed arrow | A path taken only on failure, degradation or fallback |
| Dotted arrow | A feedback or training loop (ground truth returning to a model) |
| Double-headed arrow | Bidirectional, and designed to survive disconnection |
| A box labelled with a `U`-prefix | One of the five units in [../01-architecture/decomposition.md](../01-architecture/decomposition.md) |
| Diamond | A decision point that a human or a rule makes, never a model on its own |
