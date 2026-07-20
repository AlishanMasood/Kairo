# Data Architecture

## Purpose

This folder contains the data architecture documentation for the Kairo platform. It defines how data is owned, organized, stored, and governed across the platform — from domain-level ownership through physical storage, consistency, lifecycle, and evolution.

Data architecture bridges business capabilities and implementation. It establishes the rules that ensure data is correct, consistent, secure, and maintainable without prescribing specific database schemas or ORM configurations.

## What Belongs Here

- Data architecture principles and philosophy
- Data ownership and boundary definitions
- Data classification and categorization
- Data consistency and transactional architecture
- Data lifecycle (creation, retention, archival, deletion)
- Data flow architecture (authoritative → derived → analytical)
- Storage strategy and placement direction
- Data quality and integrity rules
- Data governance and stewardship
- Backup and recovery architecture
- Data evolution and migration strategy

## What Does NOT Belong Here

- Database table schemas or SQL DDL (use module specifications)
- ORM entity definitions or mapping configuration (use development standards)
- API response models or DTO definitions (use module API specifications)
- Cloud-provider-specific database configuration (use infrastructure documentation)
- Data protection and encryption details (use `04-Architecture/Security/Data-Protection.md`)
- Tenant data isolation enforcement (use `04-Architecture/Multi-Tenancy/`)
- Individual module data models (use `06-Modules/`)

## Recommended Reading Order

| Order | Document | Purpose |
|-------|----------|---------|
| 1 | [Data Architecture](./Data-Architecture.md) | Foundation — data ownership, principles, categories, and evolution || 2 | [Data Ownership](./Data-Ownership.md) | Authoritative ownership rules, boundaries, and lifecycle |
| 3 | [Data Classification and Sensitivity](./Data-Classification-and-Sensitivity.md) | Classification model, handling rules, and per-surface responsibilities |
| 4 | [Data Modeling Principles](./Data-Modeling-Principles.md) | Logical modeling principles, domain patterns, and consistency rules |
| 5 | [Identifier Strategy](./Identifier-Strategy.md) | Resource identifiers, generation approach, and mapping rules |
| 6 | [Transaction and Consistency](./Transaction-and-Consistency-Architecture.md) | Transactional boundaries, consistency models, and failure handling |
| 7 | [Data Access and Persistence](./Data-Access-and-Persistence.md) | Persistence patterns, ORM/SQL usage, and query safety |
| 8 | [Schema Evolution and Migrations](./Schema-Evolution-and-Migrations.md) | Migration categories, compatibility, and zero-downtime strategy |
## Prerequisites

Before reading this folder, the following documents should be understood:

- [Module Architecture](../Module-Architecture.md)
- [Multi-Tenancy Architecture](../Multi-Tenancy/Multi-Tenancy-Architecture.md)
- [Data Isolation Strategy](../Multi-Tenancy/Data-Isolation-Strategy.md)
- [Data Protection](../Security/Data-Protection.md)
- [Monolith Strategy](../Monolith-Strategy.md)

## Contents

- [Data-Architecture.md](./Data-Architecture.md) — Data architecture foundation
- [Data-Ownership.md](./Data-Ownership.md) — Data ownership and boundaries
- [Data-Classification-and-Sensitivity.md](./Data-Classification-and-Sensitivity.md) — Data classification and sensitivity model
- [Data-Modeling-Principles.md](./Data-Modeling-Principles.md) — Data modeling principles
- [Identifier-Strategy.md](./Identifier-Strategy.md) — Platform identifier strategy
- [Transaction-and-Consistency-Architecture.md](./Transaction-and-Consistency-Architecture.md) — Transaction and consistency architecture
- [Data-Access-and-Persistence.md](./Data-Access-and-Persistence.md) — Data access and persistence architecture
- [Schema-Evolution-and-Migrations.md](./Schema-Evolution-and-Migrations.md) — Schema evolution and migration architecture

## Lifecycle Status

Status: Draft

## Phase Completion Criteria

This phase is complete when:

- Data ownership rules are unambiguously defined per domain/module.
- Logical and physical data boundaries are clearly separated.
- Authoritative, derived, and analytical data are distinguished.
- Tenant data principles are consistent with Multi-Tenancy architecture.
- V1 data architecture aligns with the modular monolith strategy.
- No implementation schemas have been introduced.
- Future data platform evolution is identified without entering V1.
