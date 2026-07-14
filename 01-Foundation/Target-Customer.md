# Target Customer

## Metadata

| Field | Value |
|-------|-------|
| Title | Kairo Target Customer Profile |
| Document ID | KAI-FND-006 |
| Status | Draft |
| Version | 0.1 |
| Target Release | N/A |
| Owner | Founder |
| Created | 2026-07-14 |
| Last Updated | 2026-07-14 |
| Reviewers | TODO |
| Related Documents | [Vision](./Vision.md), [Mission](./Mission.md), [Product Philosophy](./Product-Philosophy.md) |
| Dependencies | None |

---

## Purpose

This document defines who Kairo serves. It establishes the ideal customer profile so that product decisions, feature prioritization, and communication are aligned around a clear audience. Every capability in the platform should serve the needs described here.

---

## Ideal Customer

### Primary Customers

**Ecommerce Agencies**

Teams that build commerce experiences for multiple clients. They need reusable infrastructure that accelerates delivery across projects without rebuilding foundational systems for each engagement.

- Build custom storefronts and commerce experiences for clients
- Manage multiple concurrent commerce projects
- Value reusable backend infrastructure that reduces per-project effort
- Have strong frontend and integration capabilities but prefer not to maintain backend commerce logic

**Software Companies**

Engineering organizations building commerce-enabled products. They need composable backend services that integrate into their existing systems without forcing a platform migration.

- Build products that include commerce functionality
- Require APIs that integrate with existing technical architecture
- Value stability, documentation, and backward compatibility
- Evaluate platforms on technical merit and long-term viability

**Development Teams**

Internal engineering teams within mid-market and enterprise organizations. They have the technical capability to build custom experiences but lack the time or justification to build commerce infrastructure from scratch.

- Staff of experienced developers capable of API integration
- Need to deliver commerce functionality within broader product initiatives
- Prioritize time-to-delivery without sacrificing control
- Require the flexibility to customize business logic

**Mid-Market Brands with Custom Commerce Needs**

Businesses that have outgrown template-based platforms and need commerce infrastructure that matches their unique operational requirements.

- Revenue and operational complexity that exceeds basic platform capabilities
- Unique business rules that cannot be configured in off-the-shelf solutions
- Engineering resources or agency partnerships to build custom experiences
- Long-term view of their commerce infrastructure

### Secondary Customers

**Growing Direct-to-Consumer Brands**

DTC brands reaching the point where their commerce requirements exceed what template platforms offer. They need infrastructure that grows with them without requiring re-architecture.

- Experiencing limitations with current platforms
- Need custom checkout flows, pricing logic, or integration patterns
- Have or are hiring technical resources
- Planning for long-term growth beyond current platform constraints

---

## Non-Ideal Customer

Kairo is not designed for:

- **Non-technical merchants** who need a working store without writing code. They are better served by hosted, all-in-one commerce platforms.
- **Businesses seeking a visual site builder** who prioritize drag-and-drop design over programmatic control.
- **Teams without development resources** who cannot consume APIs or build custom frontends.
- **Organizations that require a fully managed, turnkey solution** with no technical integration effort.
- **Businesses with simple commerce needs** that are well served by existing template-based platforms and have no requirement for customization.

These customers have legitimate needs, but those needs are served by different types of platforms. Attempting to serve them would compromise the developer-first philosophy.

---

## Customer Problems

Problems that Kairo's target customers experience consistently:

| Problem | Impact |
|---------|--------|
| Rebuilding commerce infrastructure for every project | Wasted engineering effort and delayed delivery |
| Monolithic platforms that resist customization | Workarounds that create technical debt and fragility |
| Backend systems that assume a specific frontend | Loss of creative and technical freedom |
| Scaling limitations discovered after launch | Costly re-architecture under production pressure |
| Poor API design in existing commerce platforms | Increased integration time and ongoing maintenance burden |
| Vendor lock-in through proprietary data models | Inability to evolve or migrate without significant cost |
| Inconsistent behavior across commerce platform features | Defensive coding and unpredictable outcomes |
| Inadequate documentation requiring trial-and-error integration | Lost developer productivity and frustration |

---

## Customer Goals

What target customers are trying to achieve:

- **Deliver faster** — Reduce the time from project start to production launch by eliminating undifferentiated backend work.
- **Maintain control** — Own the customer experience completely without platform-imposed limitations on design or behavior.
- **Build reliably** — Depend on infrastructure that is correct, stable, and well-documented.
- **Scale confidently** — Know that the backend will handle growth without requiring re-architecture.
- **Reduce repetition** — Stop rebuilding the same commerce logic across projects and invest that effort in differentiation.
- **Integrate cleanly** — Work with APIs that are consistent, predictable, and designed for developer consumption.

---

## Buying Motivation

What drives the decision to adopt Kairo:

- **Time pressure** — A project deadline requires commerce infrastructure faster than it can be built internally.
- **Repetition fatigue** — The team has built the same systems too many times and wants to stop.
- **Platform limitations** — The current solution cannot accommodate the business requirements without unacceptable workarounds.
- **Quality requirements** — The business needs commerce logic that is correct, auditable, and reliable beyond what a hastily built internal system provides.
- **Long-term cost** — Maintaining custom-built commerce infrastructure is more expensive than adopting proven, maintained infrastructure.
- **Technical alignment** — The platform's API-first, composable approach matches the team's preferred way of building.

---

## Future Expansion

As the platform matures, the target customer profile may expand to include:

- Enterprise organizations with complex multi-brand, multi-region commerce requirements.
- Platform companies building commerce capabilities into non-commerce products.
- System integrators serving large-scale commerce transformations.

This expansion will follow the platform's growth naturally. The product must serve the primary customer exceptionally before broadening its audience. Expansion is driven by demonstrated capability, not by premature ambition.

The core commitment remains unchanged: Kairo serves developers who build commerce experiences.
