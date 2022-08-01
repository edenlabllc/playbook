# When Should I Write an Architecture Decision Record

## TL;DR Have you made a significant decision that impacts how engineers write software? Write an ADR!
An Architecture Decision Record (ADR) is a document that captures a decision, including the context of how the decision was made and the consequences of adopting the decision. Teams should use ADRs to document their decisions related to system design and engineering best practices. We typically arrive at these decisions through discussion during our engineering meetings.

## What are the benefits?
Since adopting ADRs, the team observed many benefits, some of which include:

### Onboarding
Future team members are able to read a history of decisions and quickly get up to speed on how and why a decision is made, and the impact of that decision.

### Ownership Handover
We work in an agile environment where we aim to learn fast. In this environment, we do not hesitate to implement organizational change to meet the needs of our users as it evolves. When our organization changes, we sometimes have to move ownership of systems from one team to another. In the past, a lot of context/knowledge was lost when this took place, triggering a decrease in productivity. This problem can be less severe with the introduction of ADRs. New owners of a system can quickly get up to speed with how and why the system’s architecture evolved in the way it did simply by reading through the ADRs.

### Alignment
ADRs have made it easier for teams to align on best practices across Edenlab. Alignment has the benefit of removing duplicative efforts, making code reusable across projects, and reducing the variance of solutions that central teams need to support.

## Ok I’m in! But when should I write one?
So you might be thinking, “ADRs sound great! It’s exactly what my team needs to facilitate decision-making while keeping a log of decisions for future teammates, as well as our future selves! But when should I write one?“

An ADR should be written whenever a decision of significant impact is made; it is up to each team to align on what defines a significant impact.

## Decisions
Sometimes a decision was made, or an implicit standard forms naturally on its own, but because it was never documented, it’s not clear to everyone (especially new hires) that this decision exists. If a decision was made but it was never recorded, can it be a standard? One way to identify an undocumented decision is during **PR Review**. The introduction of a competing code pattern or library could lead the reviewer to discover an undocumented decision.

## Proposing large changes
Over the lifecycle of a system, you will have to make decisions that have a large impact on how it is designed, maintained, extended, and more. As requirements evolve, you may need to introduce a breaking change to your API, which would require a migration from your consumers. We have system design reviews, architecture reviews to facilitate agreements on an approach or implementation. As a result of such reviews, it would be nice to have an ADR!

## Proposing small/no changes
In our day-to-day, we make small decisions that have little to no impact. The cost of undocumented decisions is hard to measure, but the effects usually include duplicated efforts (other engineers try to solve the same problems) or competing solutions (two third-party libraries that do the same thing). Enough small decisions can compound into a future problem that requires a large process or effort (ie. migration). Documenting these decisions doesn’t have to cost much. ADRs can be lightweight.

When should I write an Architecture Decision Record? Almost always!

1. Do I have a problem? **Yes**
2. Is there a blessed solution? **Yes**
3. Is it documented? **No**
4. **_Write an ADR!_**
