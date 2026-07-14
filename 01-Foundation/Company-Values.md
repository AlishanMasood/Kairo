# Company Values

## Metadata

| Field | Value |
|-------|-------|
| Title | Kairo Company Values |
| Document ID | KAI-FND-008 |
| Status | Draft |
| Version | 0.1 |
| Target Release | N/A |
| Owner | Founder |
| Created | 2026-07-14 |
| Last Updated | 2026-07-14 |
| Reviewers | TODO |
| Related Documents | [Vision](./Vision.md), [Mission](./Mission.md), [Core Principles](./Core-Principles.md) |
| Dependencies | None |

---

## Purpose

These values define how people at Kairo work. They guide behavior, decision-making, and collaboration. They are not aspirational slogans — they are expectations. Every team member, contributor, and AI agent operating within the Kairo ecosystem is expected to embody these values in daily work.

---

## 1. Integrity

### Meaning

Do the right thing even when no one is watching. Be honest about progress, limitations, and mistakes. Trust is built through consistent truthfulness, not through favorable reporting.

### Expected Behaviors

- Communicate problems early rather than hiding them.
- Acknowledge mistakes and focus on resolution rather than blame.
- Deliver honest estimates and status updates.
- Follow through on commitments or renegotiate them explicitly.

### Examples

- Reporting that a feature is not ready for release rather than shipping known defects silently.
- Acknowledging when a design decision was wrong and proposing a correction.

### Anti-patterns

- Overstating progress to meet expectations.
- Concealing technical debt or known issues.
- Agreeing to timelines while knowing they are unrealistic.

---

## 2. Ownership

### Meaning

Take responsibility for outcomes, not just tasks. Own the result, not just the effort. When something is broken, fix it — do not wait for someone else to notice.

### Expected Behaviors

- Follow work through to completion, including edge cases and cleanup.
- Take responsibility for the systems and documents you create.
- Proactively address problems within your scope without being asked.
- Escalate clearly when a problem exceeds your ability to resolve.

### Examples

- Updating documentation when behavior changes, without a separate ticket.
- Investigating a production anomaly because you recognize it, even if you are not on call.

### Anti-patterns

- Declaring work done without verifying the outcome.
- Treating problems in your area as someone else's responsibility.
- Doing the minimum required by the task description while ignoring obvious issues.

---

## 3. Craftsmanship

### Meaning

Take pride in the quality of work produced. Write code, documentation, and designs that you would be proud to show to any engineer. Quality is not a phase — it is a standard applied to every action.

### Expected Behaviors

- Write clear, maintainable code that others can understand.
- Produce documentation that is accurate, complete, and well-structured.
- Review your own work critically before submitting it for review.
- Invest in getting details right rather than settling for good enough.

### Examples

- Refactoring a working but unclear implementation into something readable before merging.
- Rewriting an error message three times until it communicates the problem precisely.

### Anti-patterns

- Shipping code that works but is incomprehensible.
- Treating documentation as a checkbox rather than a communication tool.
- Accepting poor naming because renaming is inconvenient.

---

## 4. Customer Obsession

### Meaning

Every decision ultimately serves someone building on Kairo. Understand their problems deeply. Design solutions from their perspective, not from the platform's internal perspective.

### Expected Behaviors

- Evaluate features and APIs from the consumer's point of view.
- Seek feedback and use it to improve the developer experience.
- Prioritize work that reduces customer friction over internal convenience.
- Understand the customer's workflow, not just their feature request.

### Examples

- Redesigning an API response structure because customer feedback revealed it was confusing.
- Adding a specific error code because developers reported difficulty debugging a common scenario.

### Anti-patterns

- Designing APIs that mirror internal data structures rather than customer mental models.
- Dismissing feedback because the existing design is technically correct.
- Building features that solve internal problems but create customer complexity.

---

## 5. Developer Empathy

### Meaning

Remember what it feels like to integrate with a new platform, debug an opaque error, or read inadequate documentation. Design every interaction with that memory in mind.

### Expected Behaviors

- Test your own APIs as though you are encountering them for the first time.
- Write documentation that answers the questions you would have as a new user.
- Consider the debugging experience when designing error responses and logging.
- Reduce cognitive load wherever possible.

### Examples

- Including a troubleshooting section in documentation because you know that scenario is confusing.
- Providing a clear example for every non-obvious API parameter.

### Anti-patterns

- Assuming developers will read the entire documentation before starting.
- Returning generic error messages that require source code access to interpret.
- Designing complex setup procedures because "it's obvious if you understand the system."

---

## 6. Long-Term Thinking

### Meaning

Favor decisions that create lasting value over those that solve only the immediate problem. Consider the cost of today's shortcut over the next five years.

### Expected Behaviors

- Evaluate decisions against their long-term maintenance cost.
- Prefer stable, well-understood approaches over novel but unproven ones.
- Build systems that can evolve without requiring replacement.
- Document decisions so future contributors understand the reasoning.

### Examples

- Choosing a more verbose but clearer API design because it will be easier to extend.
- Recording an architecture decision even though the current team all agrees on the rationale.

### Anti-patterns

- Choosing the fastest implementation without considering future modification cost.
- Skipping documentation because the current team already knows the context.
- Adopting trending technology without evaluating long-term support and stability.

---

## 7. Transparency

### Meaning

Share information openly. Default to visibility. Make decisions, reasoning, and progress accessible to everyone who needs them.

### Expected Behaviors

- Document decisions and their rationale in shared, accessible locations.
- Communicate changes that affect others proactively.
- Make work-in-progress visible rather than revealing only finished results.
- Share context broadly rather than hoarding information.

### Examples

- Writing an ADR for a significant technical choice so future contributors understand why.
- Sharing a draft design early for feedback rather than presenting a finished proposal.

### Anti-patterns

- Making decisions in private without recording the reasoning.
- Surprising stakeholders with completed work that affects their area.
- Keeping knowledge in personal notes rather than shared documentation.

---

## 8. Continuous Improvement

### Meaning

The current state is never the final state. Seek opportunities to improve systems, processes, and skills incrementally. Small, consistent improvements compound into transformative change.

### Expected Behaviors

- Identify and address inefficiencies as part of normal work.
- Propose improvements to processes that are not working well.
- Learn from failures and adjust practices accordingly.
- Treat retrospectives and reviews as opportunities, not obligations.

### Examples

- Automating a manual process after performing it three times.
- Proposing a documentation standard improvement based on repeated friction.

### Anti-patterns

- Accepting broken processes because "that's how it's always been."
- Waiting for a dedicated improvement sprint instead of fixing small things continuously.
- Repeating the same mistake without investigating root cause.
