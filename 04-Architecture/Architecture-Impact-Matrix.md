# Architecture Impact Matrix

## Metadata

| Field | Value |
|-------|-------|
| Title | Kairo Architecture Impact Matrix |
| Document ID | KAI-ARCH-011 |
| Status | Draft |
| Version | 0.1 |
| Target Release | N/A |
| Owner | Chief Software Architect |
| Created | 2026-07-15 |
| Last Updated | 2026-07-15 |
| Reviewers | TODO |
| Related Documents | All architecture documents |
| Dependencies | None |

---

## Purpose

This document is the master traceability map for the architecture layer. It traces every architecture document to its upstream influences (foundation, product, capability) and its downstream dependents (modules, ADRs, roadmap items).

Traceability ensures that architectural decisions can be traced back to the business rationale that drove them, and forward to the implementation decisions they constrain. When a foundation or product document changes, this matrix identifies which architecture documents must be reviewed. When an architecture document changes, this matrix identifies which downstream artifacts are affected.

---

## Document Inventory

| Document ID | Document | Path |
|-------------|----------|------|
| KAI-ARCH-001 | Architecture Overview | [04-Architecture/Architecture-Overview.md](./Architecture-Overview.md) |
| KAI-ARCH-002 | System Architecture | [04-Architecture/System-Architecture.md](./System-Architecture.md) |
| KAI-ARCH-003 | Architecture Principles | [04-Architecture/Architecture-Principles.md](./Architecture-Principles.md) |
| KAI-ARCH-004 | Technology Stack | [04-Architecture/Technology-Stack.md](./Technology-Stack.md) |
| KAI-ARCH-005 | Monolith Strategy | [04-Architecture/Monolith-Strategy.md](./Monolith-Strategy.md) |
| KAI-ARCH-006 | Module Architecture | [04-Architecture/Module-Architecture.md](./Module-Architecture.md) |
| KAI-ARCH-007 | Cross-Cutting Concerns | [04-Architecture/Cross-Cutting-Concerns.md](./Cross-Cutting-Concerns.md) |
| KAI-ARCH-008 | Architecture Roadmap | [04-Architecture/Architecture-Roadmap.md](./Architecture-Roadmap.md) |
| KAI-ARCH-009 | Architecture Constraints | [04-Architecture/Architecture-Constraints.md](./Architecture-Constraints.md) |
| KAI-ARCH-010 | Quality Attributes | [04-Architecture/Quality-Attributes.md](./Quality-Attributes.md) |
| KAI-ARCH-011 | Architecture Impact Matrix | [04-Architecture/Architecture-Impact-Matrix.md](./Architecture-Impact-Matrix.md) |

---

## Upstream Traceability

### Architecture Overview (KAI-ARCH-001)

| Category | Related Documents |
|----------|------------------|
| Foundation | [Vision](../01-Foundation/Vision.md), [Technical Philosophy](../01-Foundation/Technical-Philosophy.md), [Core Principles](../01-Foundation/Core-Principles.md) |
| Product | [Kairo Platform](../02-Products/Kairo-Platform.md), [Product Ecosystem](../02-Products/Product-Ecosystem.md) |
| Capability | [Capability Map](../03-Business-Capabilities/Capability-Map.md), [Bounded Contexts](../03-Business-Capabilities/Bounded-Contexts.md), [Context Relationships](../03-Business-Capabilities/Context-Relationships.md) |
| Future Modules | All module specifications depend on this overview for structural context |
| Future ADRs | ADRs that propose structural changes must reference this document |
| Future Roadmap | Release roadmaps reference the architecture layer model defined here |

### System Architecture (KAI-ARCH-002)

| Category | Related Documents |
|----------|------------------|
| Foundation | [Technical Philosophy](../01-Foundation/Technical-Philosophy.md), [Product Philosophy](../01-Foundation/Product-Philosophy.md) |
| Product | [Kairo Platform](../02-Products/Kairo-Platform.md), [Kairo Commerce](../02-Products/Kairo-Commerce.md), [Product Ecosystem](../02-Products/Product-Ecosystem.md) |
| Capability | [Capability Map](../03-Business-Capabilities/Capability-Map.md), [Shared Capabilities](../03-Business-Capabilities/Shared-Capabilities.md), [Capability Dependencies](../03-Business-Capabilities/Capability-Dependencies.md) |
| Future Modules | All module specifications reference the system layers and deployment boundaries |
| Future ADRs | ADRs on deployment topology, gateway design, and async processing |
| Future Roadmap | Deployment milestones, API gateway evolution, async infrastructure scaling |

### Architecture Principles (KAI-ARCH-003)

| Category | Related Documents |
|----------|------------------|
| Foundation | [Core Principles](../01-Foundation/Core-Principles.md), [Technical Philosophy](../01-Foundation/Technical-Philosophy.md), [Product Philosophy](../01-Foundation/Product-Philosophy.md) |
| Product | [Kairo Platform](../02-Products/Kairo-Platform.md), [Naming Conventions](../02-Products/Naming-Conventions.md) |
| Capability | [Bounded Contexts](../03-Business-Capabilities/Bounded-Contexts.md), [Context Relationships](../03-Business-Capabilities/Context-Relationships.md) |
| Future Modules | All module specifications must comply with these principles |
| Future ADRs | ADRs that propose exceptions to principles must reference the specific principle |
| Future Roadmap | Principle compliance gates are embedded in version milestones |

### Technology Stack (KAI-ARCH-004)

| Category | Related Documents |
|----------|------------------|
| Foundation | [Technical Philosophy](../01-Foundation/Technical-Philosophy.md) |
| Product | [Kairo Platform](../02-Products/Kairo-Platform.md) |
| Capability | [Shared Capabilities](../03-Business-Capabilities/Shared-Capabilities.md) |
| Future Modules | All modules are implemented using the approved stack |
| Future ADRs | ADRs for technology additions, replacements, or exceptions |
| Future Roadmap | Technology upgrade cadence, technology evaluation cycles |

### Monolith Strategy (KAI-ARCH-005)

| Category | Related Documents |
|----------|------------------|
| Foundation | [Technical Philosophy](../01-Foundation/Technical-Philosophy.md) |
| Product | [Kairo Platform](../02-Products/Kairo-Platform.md), [Product Ecosystem](../02-Products/Product-Ecosystem.md), [Product Boundaries](../02-Products/Product-Boundaries.md) |
| Capability | [Capability Dependencies](../03-Business-Capabilities/Capability-Dependencies.md), [Capability Lifecycle](../03-Business-Capabilities/Capability-Lifecycle.md) |
| Future Modules | Module deployment strategy, extraction decisions |
| Future ADRs | ADRs for service extraction decisions |
| Future Roadmap | Extraction milestones, multi-product deployment evolution |

### Module Architecture (KAI-ARCH-006)

| Category | Related Documents |
|----------|------------------|
| Foundation | [Core Principles](../01-Foundation/Core-Principles.md), [Technical Philosophy](../01-Foundation/Technical-Philosophy.md) |
| Product | [Product Boundaries](../02-Products/Product-Boundaries.md), [Commerce Domain](../02-Products/Commerce-Domain.md), [Naming Conventions](../02-Products/Naming-Conventions.md) |
| Capability | [Bounded Contexts](../03-Business-Capabilities/Bounded-Contexts.md), [Context Relationships](../03-Business-Capabilities/Context-Relationships.md), [Capability Map](../03-Business-Capabilities/Capability-Map.md) |
| Future Modules | All module specifications must follow this architecture standard |
| Future ADRs | ADRs for module-specific pattern choices (event sourcing, CQRS) |
| Future Roadmap | Module delivery sequence, contract stability milestones |

### Cross-Cutting Concerns (KAI-ARCH-007)

| Category | Related Documents |
|----------|------------------|
| Foundation | [Core Principles](../01-Foundation/Core-Principles.md), [Technical Philosophy](../01-Foundation/Technical-Philosophy.md) |
| Product | [Kairo Platform](../02-Products/Kairo-Platform.md), [Product Ecosystem](../02-Products/Product-Ecosystem.md) |
| Capability | [Shared Capabilities](../03-Business-Capabilities/Shared-Capabilities.md), [Capability Dependencies](../03-Business-Capabilities/Capability-Dependencies.md) |
| Future Modules | All modules consume cross-cutting concerns through the defined interfaces |
| Future ADRs | ADRs for caching strategy, logging infrastructure, authorization model |
| Future Roadmap | Platform service maturity milestones, observability rollout |

### Architecture Roadmap (KAI-ARCH-008)

| Category | Related Documents |
|----------|------------------|
| Foundation | [Vision](../01-Foundation/Vision.md), [Mission](../01-Foundation/Mission.md) |
| Product | [Product Ecosystem](../02-Products/Product-Ecosystem.md), [Future Products](../02-Products/Future-Products.md) |
| Capability | [Capability Roadmap](../03-Business-Capabilities/Capability-Roadmap.md), [Capability Dependencies](../03-Business-Capabilities/Capability-Dependencies.md), [Capability Maturity](../03-Business-Capabilities/Capability-Maturity.md) |
| Future Modules | Module delivery is sequenced by architecture version gates |
| Future ADRs | ADRs that shift architecture evolution direction |
| Future Roadmap | Release roadmaps align with architecture version gates |

### Architecture Constraints (KAI-ARCH-009)

| Category | Related Documents |
|----------|------------------|
| Foundation | [Core Principles](../01-Foundation/Core-Principles.md), [Product Philosophy](../01-Foundation/Product-Philosophy.md), [Unique Selling Proposition](../01-Foundation/Unique-Selling-Proposition.md), [Success Metrics](../01-Foundation/Success-Metrics.md) |
| Product | [Kairo Platform](../02-Products/Kairo-Platform.md), [Target Customer](../01-Foundation/Target-Customer.md) |
| Capability | [Capability Maturity](../03-Business-Capabilities/Capability-Maturity.md) |
| Future Modules | All module designs must satisfy active constraints |
| Future ADRs | ADRs that propose constraint relaxation |
| Future Roadmap | Budget and compliance milestones |

### Quality Attributes (KAI-ARCH-010)

| Category | Related Documents |
|----------|------------------|
| Foundation | [Core Principles](../01-Foundation/Core-Principles.md), [Success Metrics](../01-Foundation/Success-Metrics.md), [Technical Philosophy](../01-Foundation/Technical-Philosophy.md) |
| Product | [Kairo Platform](../02-Products/Kairo-Platform.md), [Unique Selling Proposition](../01-Foundation/Unique-Selling-Proposition.md) |
| Capability | [Capability Maturity](../03-Business-Capabilities/Capability-Maturity.md), [Shared Capabilities](../03-Business-Capabilities/Shared-Capabilities.md) |
| Future Modules | Module specifications define quality targets per the standards here |
| Future ADRs | ADRs that address quality attribute trade-offs |
| Future Roadmap | Quality milestone tracking per version |

---

## Downstream Impact Matrix

This table shows which downstream artifacts are affected when an architecture document changes.

| Architecture Document | Affected Modules | Affected Development Standards | Affected ADRs | Affected Roadmap |
|----------------------|-----------------|-------------------------------|---------------|-----------------|
| Architecture Overview | All modules (structural context) | Project structure standards | Structural change ADRs | Version milestone definitions |
| System Architecture | All modules (layer assignment) | Deployment standards | Gateway, async boundary ADRs | Deployment milestones |
| Architecture Principles | All modules (compliance) | All development standards | Principle exception ADRs | Principle compliance gates |
| Technology Stack | All modules (implementation) | Coding standards, tooling standards | Technology change ADRs | Technology upgrade schedule |
| Monolith Strategy | All modules (deployment) | Deployment, testing standards | Extraction decision ADRs | Extraction milestones |
| Module Architecture | All modules (internal structure) | Module development standards | Module pattern ADRs | Module delivery sequence |
| Cross-Cutting Concerns | All modules (platform consumption) | Logging, security, caching standards | Cross-cutting implementation ADRs | Platform service milestones |
| Architecture Roadmap | All modules (version alignment) | Release process standards | Roadmap shift ADRs | All release roadmaps |
| Architecture Constraints | All modules (constraint compliance) | All development standards | Constraint relaxation ADRs | Budget, compliance milestones |
| Quality Attributes | All modules (quality targets) | Testing, performance standards | Quality trade-off ADRs | Quality milestone tracking |

---

## Cross-Reference Matrix

This table maps every architecture document to every other architecture document it directly relates to.

| Document | Overview | System | Principles | Tech Stack | Monolith | Module | Cross-Cut | Roadmap | Constraints | Quality |
|----------|----------|--------|-----------|-----------|----------|--------|----------|---------|------------|---------|
| Overview | — | Yes | Yes | No | Yes | Yes | No | Yes | No | No |
| System | Yes | — | Yes | Yes | Yes | Yes | Yes | Yes | No | No |
| Principles | Yes | Yes | — | No | Yes | Yes | Yes | No | Yes | Yes |
| Tech Stack | Yes | Yes | Yes | — | No | No | No | Yes | Yes | No |
| Monolith | Yes | Yes | Yes | No | — | Yes | No | Yes | Yes | No |
| Module | Yes | No | Yes | No | Yes | — | Yes | No | No | Yes |
| Cross-Cut | No | Yes | Yes | Yes | No | Yes | — | Yes | No | Yes |
| Roadmap | Yes | Yes | No | Yes | Yes | No | Yes | — | No | No |
| Constraints | No | No | Yes | Yes | Yes | No | No | No | — | Yes |
| Quality | No | No | Yes | No | No | Yes | Yes | No | Yes | — |

---

## Foundation → Architecture Traceability

| Foundation Document | Architecture Documents Influenced |
|--------------------|---------------------------------|
| [Vision](../01-Foundation/Vision.md) | Architecture Overview, Architecture Roadmap |
| [Mission](../01-Foundation/Mission.md) | Architecture Roadmap |
| [Core Principles](../01-Foundation/Core-Principles.md) | Architecture Principles, Module Architecture, Cross-Cutting Concerns, Architecture Constraints, Quality Attributes |
| [Product Philosophy](../01-Foundation/Product-Philosophy.md) | System Architecture, Architecture Principles, Architecture Constraints |
| [Technical Philosophy](../01-Foundation/Technical-Philosophy.md) | Architecture Overview, Architecture Principles, Technology Stack, Monolith Strategy, Module Architecture, Cross-Cutting Concerns |
| [Target Customer](../01-Foundation/Target-Customer.md) | Architecture Constraints |
| [Unique Selling Proposition](../01-Foundation/Unique-Selling-Proposition.md) | Architecture Constraints, Quality Attributes |
| [Company Values](../01-Foundation/Company-Values.md) | No direct architecture impact |
| [Success Metrics](../01-Foundation/Success-Metrics.md) | Architecture Constraints, Quality Attributes |

---

## Product → Architecture Traceability

| Product Document | Architecture Documents Influenced |
|-----------------|---------------------------------|
| [Kairo Platform](../02-Products/Kairo-Platform.md) | Architecture Overview, System Architecture, Technology Stack, Monolith Strategy, Cross-Cutting Concerns, Architecture Roadmap, Architecture Constraints, Quality Attributes |
| [Kairo Commerce](../02-Products/Kairo-Commerce.md) | System Architecture |
| [Product Ecosystem](../02-Products/Product-Ecosystem.md) | Architecture Overview, System Architecture, Monolith Strategy, Cross-Cutting Concerns, Architecture Roadmap |
| [Product Boundaries](../02-Products/Product-Boundaries.md) | Module Architecture, Monolith Strategy |
| [Domain Model](../02-Products/Domain-Model.md) | Module Architecture |
| [Commerce Domain](../02-Products/Commerce-Domain.md) | Module Architecture |
| [Future Products](../02-Products/Future-Products.md) | Architecture Roadmap |
| [Naming Conventions](../02-Products/Naming-Conventions.md) | Architecture Principles, Module Architecture |
| [Glossary](../02-Products/Glossary.md) | No direct architecture impact (terminology reference) |

---

## Capability → Architecture Traceability

| Capability Document | Architecture Documents Influenced |
|--------------------|---------------------------------|
| [Capability Map](../03-Business-Capabilities/Capability-Map.md) | Architecture Overview, System Architecture, Module Architecture |
| [Bounded Contexts](../03-Business-Capabilities/Bounded-Contexts.md) | Architecture Overview, Architecture Principles, Module Architecture |
| [Context Relationships](../03-Business-Capabilities/Context-Relationships.md) | Architecture Overview, Architecture Principles, Module Architecture |
| [Capability Dependencies](../03-Business-Capabilities/Capability-Dependencies.md) | System Architecture, Monolith Strategy, Cross-Cutting Concerns, Architecture Roadmap |
| [Shared Capabilities](../03-Business-Capabilities/Shared-Capabilities.md) | System Architecture, Cross-Cutting Concerns, Quality Attributes |
| [Capability Lifecycle](../03-Business-Capabilities/Capability-Lifecycle.md) | Monolith Strategy |
| [Capability Maturity](../03-Business-Capabilities/Capability-Maturity.md) | Architecture Roadmap, Architecture Constraints, Quality Attributes |
| [Capability Roadmap](../03-Business-Capabilities/Capability-Roadmap.md) | Architecture Roadmap |

---

## Architecture Impact

This matrix is itself an architectural artifact. Its impact:

- Changes to foundation, product, or capability documents trigger a review of the architecture documents identified in the upstream traceability tables.
- Changes to architecture documents trigger a review of the downstream artifacts identified in the downstream impact matrix.
- The cross-reference matrix identifies which architecture documents must be reviewed together when a change affects multiple concerns.

---

## Version Gate

| Version | Traceability Expectation |
|---------|------------------------|
| V1 | All architecture documents trace to their upstream foundation, product, and capability documents. Downstream module specifications reference the architecture documents they depend on. |
| V2 | ADRs reference the architecture documents they affect. Architecture document changes trigger documented impact assessments. |
| V3 | Cross-product traceability is maintained. Architecture changes affecting multiple products are assessed through this matrix. |

---

## Out of Scope

This document does not:

- Define the content of referenced documents — it maps relationships only.
- Replace the Related Documents field in individual document metadata — it provides a system-wide view.
- Track implementation-level dependencies — it operates at the document level.
- Track external dependencies (third-party documentation, specifications) — it covers internal repository documents only.

---

## Future Considerations

- As modules are created in `06-Modules/`, each module specification should be added to the downstream impact matrix.
- As ADRs are recorded in `08-ADR/`, each ADR should reference the architecture documents it affects.
- As the document count grows, consider tooling to validate traceability links automatically.
- Cross-product traceability (when Payments, POS, and ERP have their own architecture documents) will require expanding this matrix.

---

## Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 0.1 | 2026-07-15 | Chief Software Architect | Initial draft |
