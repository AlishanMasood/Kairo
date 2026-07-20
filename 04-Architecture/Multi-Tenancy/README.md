# Multi-Tenancy

## Purpose

This folder contains the multi-tenancy architecture documentation for the Kairo platform. It defines how the platform isolates tenants, propagates tenant context, and ensures that one tenant's data, configuration, and operations are never visible to another.

Multi-tenancy is a foundational architectural concern. It determines how data is scoped, how access is controlled, and how the platform serves many independent businesses from shared infrastructure without compromise.

## What Belongs Here

- Multi-tenancy architecture and philosophy
- Tenant boundary definitions
- Tenant context propagation architecture
- Tenant isolation enforcement patterns
- Tenant lifecycle architecture
- Tenant-aware service design patterns
- Multi-tenancy impact on data, events, caching, and background processing

## What Does NOT Belong Here

- Database schemas or ORM filter implementations (use module specifications)
- Middleware code or request pipeline implementation (use development standards)
- API endpoint contracts (use module API specifications)
- Cloud-provider-specific deployment configuration (use infrastructure documentation)
- Security architecture (use `04-Architecture/Security/` — multi-tenancy consumes security; it does not define it)
- Organization or store business logic (use `05-Platform-Core/` or `06-Modules/`)

## Recommended Reading Order

| Order | Document | Purpose |
|-------|----------|---------|
| 1 | [Multi-Tenancy Architecture](./Multi-Tenancy-Architecture.md) | Foundation — tenant boundaries, isolation, and context propagation || 2 | [Tenant Hierarchy](./Tenant-Hierarchy.md) | Scope model, resource ownership, and hierarchy rules |
| 3 | [Tenant Resolution](./Tenant-Resolution.md) | Context resolution, validation, propagation, and failure behavior |
| 4 | [Tenant Isolation](./Tenant-Isolation.md) | Defence-in-depth isolation across all platform layers |
| 5 | [Data Isolation Strategy](./Data-Isolation-Strategy.md) | Storage models, V1 decision, and evolution path |
| 6 | [Tenant-Aware Authorization](./Tenant-Aware-Authorization.md) | Authorization rules across tenant boundaries |
| 7 | [Tenant Configuration](./Tenant-Configuration.md) | Configuration inheritance, overrides, and tenant safety |
| 8 | [Tenant Lifecycle](./Tenant-Lifecycle.md) | Lifecycle stages, transitions, suspension, deletion, and restoration |
| 9 | [Cross-Tenant Operations](./Cross-Tenant-Operations.md) | When cross-tenant access is allowed, restricted, or prohibited |
| 10 | [Tenant Scaling and Placement](./Tenant-Scaling-and-Placement.md) | Scaling strategy, placement evolution, and noisy-neighbor mitigation |
## Prerequisites

Before reading this folder, the following documents should be understood:

- [Platform Hierarchy](../../05-Platform-Core/Platform-Hierarchy.md)
- [Organization Model](../../05-Platform-Core/Organization-Model.md)
- [Store Model](../../05-Platform-Core/Store-Model.md)
- [Security Architecture](../Security/Security-Architecture.md)
- [Authorization Architecture](../Security/Authorization-Architecture.md)

## Contents

- [Multi-Tenancy-Architecture.md](./Multi-Tenancy-Architecture.md) — Multi-tenancy architecture foundation
- [Tenant-Hierarchy.md](./Tenant-Hierarchy.md) — Tenant hierarchy and scope model
- [Tenant-Resolution.md](./Tenant-Resolution.md) — Tenant resolution architecture
- [Tenant-Isolation.md](./Tenant-Isolation.md) — Tenant isolation architecture
- [Data-Isolation-Strategy.md](./Data-Isolation-Strategy.md) — Tenant data isolation strategy
- [Tenant-Aware-Authorization.md](./Tenant-Aware-Authorization.md) — Tenant-aware authorization
- [Tenant-Configuration.md](./Tenant-Configuration.md) — Tenant configuration architecture
- [Tenant-Lifecycle.md](./Tenant-Lifecycle.md) — Tenant lifecycle architecture
- [Cross-Tenant-Operations.md](./Cross-Tenant-Operations.md) — Cross-tenant operation controls
- [Tenant-Scaling-and-Placement.md](./Tenant-Scaling-and-Placement.md) — Tenant scaling and placement strategy

## Lifecycle Status

Status: Draft

## Phase Completion Criteria

This phase is complete when:

- The primary tenant boundary is unambiguously defined.
- Tenant context propagation is architecturally specified.
- Isolation requirements cover all platform layers (data, events, cache, background processing, observability).
- V1 requirements are separated from future enterprise tenancy models.
- Security integration is documented (links to Security Architecture and Authorization Architecture).
- No implementation details have been introduced.
