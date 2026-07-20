# Data Architecture Impact Matrix

## Metadata

| Field | Value |
|-------|-------|
| Title | Kairo Data Architecture Traceability and Impact Matrix |
| Document ID | KAI-DATA-013 |
| Status | Draft |
| Version | 0.1 |
| Target Release | V1 |
| Owner | Chief Data Architect |
| Created | 2026-07-20 |
| Last Updated | 2026-07-20 |
| Reviewers | TODO |
| Related Documents | All Phase 8 (Data Architecture) documents, [Architecture Impact Matrix](../Architecture-Impact-Matrix.md), [Platform Impact Matrix](../Platform-Impact-Matrix.md), [Security Impact Matrix](../Security/Security-Impact-Matrix.md), [Multi-Tenancy Impact Matrix](../Multi-Tenancy/Multi-Tenancy-Impact-Matrix.md) |
| Dependencies | All Phase 8 documents must be complete before this matrix is finalized |

---

## Applicable Version

V1 — This matrix applies to the initial release. Future capabilities are identified but not required for V1 delivery.

---

## Purpose

This document is the master traceability and impact matrix for the Kairo Data Architecture phase. It maps every data architecture decision to its upstream dependencies, downstream consumers, implementation surfaces, and governance responsibilities.

This matrix serves three audiences:

1. **Implementers** — Know which data requirements apply to their component.
2. **Future architects** — Know which data decisions they must consume when building API, Event, or Infrastructure architecture.
3. **Reviewers** — Verify completeness, consistency, and traceability across the entire data architecture phase.

---

## Scope

This matrix covers:

- All 12 Phase 8 (Data Architecture) documents and their cross-dependencies.
- Traceability to upstream Foundation, Product, Business Capability, Platform Core, Security, and Multi-Tenancy documents.
- Forward dependencies on future architecture phases (API, Event, Infrastructure, Development Standards).
- Implementation impact across all platform surfaces.
- Governance, testing, and audit responsibilities.

This matrix does not cover:

- Implementation details (no schemas, no code).
- Specific module data models.
- Vendor or product selection.

---

## 1. Data-Document Dependencies

### Phase 8 Internal Dependencies

| Document | Depends On (within Data phase) |
|----------|-------------------------------|
| [Data Architecture](./Data-Architecture.md) | — (foundational) |
| [Data Ownership](./Data-Ownership.md) | [Data Architecture](./Data-Architecture.md) |
| [Data Classification](./Data-Classification-and-Sensitivity.md) | [Data Architecture](./Data-Architecture.md), [Data Ownership](./Data-Ownership.md) |
| [Data Modeling](./Data-Modeling-Principles.md) | [Data Ownership](./Data-Ownership.md) |
| [Identifier Strategy](./Identifier-Strategy.md) | [Data Modeling](./Data-Modeling-Principles.md) |
| [Transaction and Consistency](./Transaction-and-Consistency-Architecture.md) | [Data Ownership](./Data-Ownership.md), [Data Modeling](./Data-Modeling-Principles.md) |
| [Data Access](./Data-Access-and-Persistence.md) | [Transaction and Consistency](./Transaction-and-Consistency-Architecture.md), [Identifier Strategy](./Identifier-Strategy.md) |
| [Schema Evolution](./Schema-Evolution-and-Migrations.md) | [Data Access](./Data-Access-and-Persistence.md), [Data Modeling](./Data-Modeling-Principles.md) |
| [Data Lifecycle](./Data-Lifecycle-and-Retention.md) | [Data Ownership](./Data-Ownership.md), [Data Classification](./Data-Classification-and-Sensitivity.md) |
| [Backup and Recovery](./Backup-Restore-and-Disaster-Recovery.md) | [Data Lifecycle](./Data-Lifecycle-and-Retention.md), [Data Access](./Data-Access-and-Persistence.md) |
| [Reporting and Analytics](./Reporting-and-Analytics-Architecture.md) | [Data Ownership](./Data-Ownership.md), [Transaction and Consistency](./Transaction-and-Consistency-Architecture.md), [Data Classification](./Data-Classification-and-Sensitivity.md) |
| [Data Quality](./Data-Quality-and-Governance.md) | [Data Ownership](./Data-Ownership.md), [Data Modeling](./Data-Modeling-Principles.md), [Reporting and Analytics](./Reporting-and-Analytics-Architecture.md) |

### Upstream Dependencies (documents Data Architecture consumes)

| Source Phase | Document | Data Architecture Documents That Consume It |
|-------------|----------|---------------------------------------------|
| Foundation | [Core Principles](../../01-Foundation/Core-Principles.md) | Data Architecture, Data Modeling |
| Foundation | [Technical Philosophy](../../01-Foundation/Technical-Philosophy.md) | Data Architecture, Data Access |
| Products | [Kairo Platform](../../02-Products/Kairo-Platform.md) | Data Architecture, Data Ownership |
| Products | [Kairo Commerce](../../02-Products/Kairo-Commerce.md) | Data Ownership, Data Classification |
| Products | [Domain Model](../../02-Products/Domain-Model.md) | Data Ownership, Data Modeling, Identifier Strategy |
| Business Capabilities | [Capability Map](../../03-Business-Capabilities/Capability-Map.md) | Data Ownership |
| Business Capabilities | [Bounded Contexts](../../03-Business-Capabilities/Bounded-Contexts.md) | Data Ownership, Data Modeling, Transaction and Consistency |
| Architecture | [Module Architecture](../Module-Architecture.md) | Data Ownership, Data Access, Transaction and Consistency |
| Architecture | [Monolith Strategy](../Monolith-Strategy.md) | Transaction and Consistency, Data Access |
| Architecture | [System Architecture](../System-Architecture.md) | Data Architecture, Backup and Recovery |
| Architecture | [Technology Stack](../Technology-Stack.md) | Data Access, Schema Evolution |
| Security | [Security Architecture](../Security/Security-Architecture.md) | Data Classification, Data Quality |
| Security | [Data Protection](../Security/Data-Protection.md) | Data Classification, Data Lifecycle |
| Security | [Authorization Architecture](../Security/Authorization-Architecture.md) | Data Access, Reporting and Analytics |
| Security | [Audit and Security Monitoring](../Security/Audit-and-Security-Monitoring.md) | Data Quality, Data Lifecycle |
| Security | [Secure Development Lifecycle](../Security/Secure-Development-Lifecycle.md) | Data Quality, Schema Evolution |
| Multi-Tenancy | [Multi-Tenancy Architecture](../Multi-Tenancy/Multi-Tenancy-Architecture.md) | Data Architecture, Data Ownership |
| Multi-Tenancy | [Data Isolation Strategy](../Multi-Tenancy/Data-Isolation-Strategy.md) | Data Access, Data Lifecycle, Backup and Recovery |
| Multi-Tenancy | [Tenant Isolation](../Multi-Tenancy/Tenant-Isolation.md) | Data Access, Reporting and Analytics |
| Multi-Tenancy | [Cross-Tenant Operations](../Multi-Tenancy/Cross-Tenant-Operations.md) | Reporting and Analytics |
| Multi-Tenancy | [Tenant Lifecycle](../Multi-Tenancy/Tenant-Lifecycle.md) | Data Lifecycle |

### Downstream Dependencies (future phases that must consume Data Architecture)

| Future Phase | Must Consume | Reason |
|-------------|--------------|--------|
| API Architecture | Data Ownership, Identifier Strategy, Data Classification, Data Access | APIs expose data — must respect ownership, use correct IDs, apply classification |
| Event Architecture | Data Ownership, Transaction and Consistency, Identifier Strategy | Events carry data — must respect boundaries, consistency, identifiers |
| Infrastructure Architecture | Data Access, Backup and Recovery, Schema Evolution | Infrastructure hosts data — must implement persistence, backup, migration |
| Development Standards | Data Access, Data Modeling, Schema Evolution, Data Quality | Development implements data — must follow patterns, quality, migration rules |
| Module Specifications | All Data Architecture documents | Modules own data — must implement all data responsibilities |
| ADRs | All Data Architecture documents (as triggers) | Significant deviations from data architecture require ADRs |
| Roadmap | Data Architecture (V1/Future split) | Roadmap must reflect data capability delivery timeline |

---

## 2. Domain Data Ownership

| Domain | Authoritative Module | Data Scope | Ownership Document |
|--------|---------------------|-----------|-------------------|
| Catalog | Catalog Module | Products, categories, attributes, variants | [Data Ownership](./Data-Ownership.md) |
| Pricing | Pricing Module | Prices, price lists, discount rules | [Data Ownership](./Data-Ownership.md) |
| Inventory | Inventory Module | Stock levels, reservations, movements | [Data Ownership](./Data-Ownership.md) |
| Customers | Customer Module | Customer profiles, addresses, segments | [Data Ownership](./Data-Ownership.md) |
| Orders | Order Module | Orders, line items, order status | [Data Ownership](./Data-Ownership.md) |
| Payments | Payment Module | Payment records, refunds, reconciliation | [Data Ownership](./Data-Ownership.md) |
| Fulfillment | Fulfillment Module | Shipments, tracking, delivery | [Data Ownership](./Data-Ownership.md) |
| Identity | Identity Module | Users, credentials, sessions | [Data Ownership](./Data-Ownership.md) |
| Organizations | Tenant Module | Organizations, stores, configuration | [Data Ownership](./Data-Ownership.md) |
| Audit | Audit Service | Audit events, compliance records | [Data Ownership](./Data-Ownership.md) |
| Notifications | Notification Module | Templates, delivery records | [Data Ownership](./Data-Ownership.md) |
| Search | Search Module | Indexes (derived, not authoritative) | [Data Ownership](./Data-Ownership.md) |

---

## 3. Tenant Data Ownership

| Data Category | Tenant Owns | Platform Owns | Shared | Governance |
|---------------|:-:|:-:|:-:|------------|
| Business data (products, orders, customers) | Yes | — | — | [Data Isolation Strategy](../Multi-Tenancy/Data-Isolation-Strategy.md) |
| Tenant configuration | Yes | — | — | [Tenant Configuration](../Multi-Tenancy/Tenant-Configuration.md) |
| User credentials | — | Yes | — | [Identity and Authentication](../Security/Identity-and-Authentication.md) |
| Audit records | — | — | Yes | [Audit and Security Monitoring](../Security/Audit-and-Security-Monitoring.md) |
| Platform reference data | — | Yes | — | [Data Quality](./Data-Quality-and-Governance.md) |
| Infrastructure metrics | — | Yes | — | [Reporting and Analytics](./Reporting-and-Analytics-Architecture.md) |
| Aggregate platform analytics | — | Yes | — | [Reporting and Analytics](./Reporting-and-Analytics-Architecture.md) |

---

## 4. Classification and Sensitivity

| Data Category | Classification | Handling Rules | Document |
|--------------|---------------|----------------|----------|
| Customer PII (name, email, address) | Confidential | Encrypted at rest, masked in logs, retention-limited | [Data Classification](./Data-Classification-and-Sensitivity.md) |
| Payment data | Restricted | PCI-scoped, tokenized, minimal retention | [Data Classification](./Data-Classification-and-Sensitivity.md) |
| Authentication credentials | Restricted | Hashed, never logged, never exported | [Data Classification](./Data-Classification-and-Sensitivity.md) |
| Order data | Confidential | Tenant-isolated, audit-logged access | [Data Classification](./Data-Classification-and-Sensitivity.md) |
| Product catalog data | Internal | Tenant-scoped, publishable to storefront | [Data Classification](./Data-Classification-and-Sensitivity.md) |
| Audit records | Confidential | Immutable, long retention, restricted access | [Data Classification](./Data-Classification-and-Sensitivity.md) |
| Platform configuration | Internal | Version-controlled, restricted modification | [Data Classification](./Data-Classification-and-Sensitivity.md) |
| Analytics (aggregated) | Internal | De-identified, platform-team access | [Reporting and Analytics](./Reporting-and-Analytics-Architecture.md) |
| Exported data | Inherits source | Classification travels with data | [Data Classification](./Data-Classification-and-Sensitivity.md) |

---

## 5. Authoritative versus Derived Data

| Data Type | Authoritative Source | Derived Consumers | Consistency Model | Document |
|-----------|---------------------|-------------------|-------------------|----------|
| Product definitions | Catalog Module | Search index, storefront cache | Eventually consistent | [Data Ownership](./Data-Ownership.md) |
| Pricing | Pricing Module | Cart calculations, order totals | Point-in-time capture | [Data Ownership](./Data-Ownership.md) |
| Stock levels | Inventory Module | Product availability display | Eventually consistent | [Data Ownership](./Data-Ownership.md) |
| Customer profiles | Customer Module | Order records (snapshot), notifications | Snapshot at use | [Data Ownership](./Data-Ownership.md) |
| Order records | Order Module | Fulfillment, payment, analytics | Event-driven propagation | [Data Ownership](./Data-Ownership.md) |
| Payment status | Payment Module | Order status updates | Event-driven | [Data Ownership](./Data-Ownership.md) |
| Search indexes | — (derived) | Storefront search, admin search | Rebuilt from authoritative | [Data Ownership](./Data-Ownership.md) |
| Report aggregates | — (derived) | Dashboards, exports | Periodically computed | [Reporting and Analytics](./Reporting-and-Analytics-Architecture.md) |

---

## 6. Identifier Responsibilities

| Responsibility | Owner | Rule | Document |
|---------------|-------|------|----------|
| ID generation approach | Platform infrastructure | ULID-style, time-sortable, coordination-free | [Identifier Strategy](./Identifier-Strategy.md) |
| ID assignment | Owning module | Generated at creation, immutable thereafter | [Identifier Strategy](./Identifier-Strategy.md) |
| External ID exposure | API layer | Public IDs only. No internal surrogate exposure. | [Identifier Strategy](./Identifier-Strategy.md) |
| ID-to-resource mapping | Owning module | Module resolves its own IDs | [Identifier Strategy](./Identifier-Strategy.md) |
| Cross-module references | Referencing module | Store the foreign resource's public ID | [Identifier Strategy](./Identifier-Strategy.md) |
| Idempotency keys | API consumer / integration | Client-provided for safe retries | [Identifier Strategy](./Identifier-Strategy.md) |

---

## 7. Transaction and Consistency Requirements

| Boundary | Consistency Model | Guarantee | Document |
|----------|------------------|-----------|----------|
| Within a module (single operation) | ACID | Full transactional consistency | [Transaction and Consistency](./Transaction-and-Consistency-Architecture.md) |
| Within a module (multi-step) | ACID (single DB transaction) | All-or-nothing within the module | [Transaction and Consistency](./Transaction-and-Consistency-Architecture.md) |
| Across modules | Eventual consistency | Transactional outbox + domain events | [Transaction and Consistency](./Transaction-and-Consistency-Architecture.md) |
| Read models / projections | Eventually consistent | Bounded lag, freshness visible | [Reporting and Analytics](./Reporting-and-Analytics-Architecture.md) |
| Search indexes | Eventually consistent | Rebuilt from authoritative source | [Data Architecture](./Data-Architecture.md) |
| Cache | Eventually consistent | TTL-based invalidation | [Data Access](./Data-Access-and-Persistence.md) |

---

## 8. Data-Access Responsibilities

| Responsibility | Owner | Pattern | Document |
|---------------|-------|---------|----------|
| ORM/query framework | Module team | EF Core, module-scoped DbContext | [Data Access](./Data-Access-and-Persistence.md) |
| Tenant filtering | Platform infrastructure | Automatic query filter on all tenant data | [Data Access](./Data-Access-and-Persistence.md), [Data Isolation Strategy](../Multi-Tenancy/Data-Isolation-Strategy.md) |
| Connection management | Platform infrastructure | Pooled, module-isolated | [Data Access](./Data-Access-and-Persistence.md) |
| Query performance | Module team | Indexes, pagination, query limits | [Data Access](./Data-Access-and-Persistence.md) |
| Read/write separation | Platform (future) | Single DB in V1; read replicas future | [Data Access](./Data-Access-and-Persistence.md) |
| Cache strategy | Module team + Platform | Module decides what to cache; platform provides infrastructure | [Data Access](./Data-Access-and-Persistence.md) |

---

## 9. Migration Responsibilities

| Responsibility | Owner | Rule | Document |
|---------------|-------|------|----------|
| Schema migration authoring | Module team | Module owns its schema evolution | [Schema Evolution](./Schema-Evolution-and-Migrations.md) |
| Migration review | Architecture + module team | Breaking changes require review | [Schema Evolution](./Schema-Evolution-and-Migrations.md) |
| Zero-downtime enforcement | Platform team | Migrations must not require downtime | [Schema Evolution](./Schema-Evolution-and-Migrations.md) |
| Backward compatibility | Module team | Old code must work with new schema during rollout | [Schema Evolution](./Schema-Evolution-and-Migrations.md) |
| Migration testing | Module team | Migrations tested against production-like data | [Schema Evolution](./Schema-Evolution-and-Migrations.md) |
| Rollback strategy | Module team + Ops | Every migration has a rollback plan | [Schema Evolution](./Schema-Evolution-and-Migrations.md) |
| Data migration (transforms) | Module team | Separate from schema migration; auditable | [Schema Evolution](./Schema-Evolution-and-Migrations.md) |

---

## 10. Retention Responsibilities

| Data Category | Retention Owner | Retention Period | Deletion Trigger | Document |
|--------------|----------------|-----------------|-----------------|----------|
| Business data (orders, payments) | Domain module | Per regulatory + business need | Tenant deletion or retention expiry | [Data Lifecycle](./Data-Lifecycle-and-Retention.md) |
| Customer PII | Customer module | Retention-limited (GDPR-aware) | Deletion request or retention expiry | [Data Lifecycle](./Data-Lifecycle-and-Retention.md) |
| Audit records | Audit service | Long retention (compliance-driven) | Retention expiry only | [Data Lifecycle](./Data-Lifecycle-and-Retention.md) |
| Session data | Identity module | Short-lived (hours/days) | Session expiry | [Data Lifecycle](./Data-Lifecycle-and-Retention.md) |
| Cache data | Platform infrastructure | TTL-based | TTL expiry or invalidation | [Data Lifecycle](./Data-Lifecycle-and-Retention.md) |
| Search indexes | Search service | Rebuilt (no independent retention) | Rebuilt on source change | [Data Lifecycle](./Data-Lifecycle-and-Retention.md) |
| Backups | Operations | Defined backup retention schedule | Rotation policy | [Backup and Recovery](./Backup-Restore-and-Disaster-Recovery.md) |
| Exported files | Platform infrastructure | Short TTL (download window) | Auto-deleted after TTL | [Reporting and Analytics](./Reporting-and-Analytics-Architecture.md) |

---

## 11. Backup and Recovery Responsibilities

| Responsibility | Owner | Scope | Document |
|---------------|-------|-------|----------|
| Backup configuration | Operations | All persistent stores | [Backup and Recovery](./Backup-Restore-and-Disaster-Recovery.md) |
| Backup verification | Operations | Regular restore testing | [Backup and Recovery](./Backup-Restore-and-Disaster-Recovery.md) |
| RPO/RTO definition | Architecture + Business | Per data category | [Backup and Recovery](./Backup-Restore-and-Disaster-Recovery.md) |
| Point-in-time recovery | Operations | Database-level | [Backup and Recovery](./Backup-Restore-and-Disaster-Recovery.md) |
| Tenant-level recovery | Operations + Module team | Tenant data restoration | [Backup and Recovery](./Backup-Restore-and-Disaster-Recovery.md) |
| Disaster recovery plan | Operations + Architecture | Full platform recovery | [Backup and Recovery](./Backup-Restore-and-Disaster-Recovery.md) |
| Backup encryption | Security + Operations | Backups follow data classification | [Backup and Recovery](./Backup-Restore-and-Disaster-Recovery.md) |

---

## 12. Reporting Responsibilities

| Responsibility | Owner | Rule | Document |
|---------------|-------|------|----------|
| Operational reporting | Module team | Module exposes query endpoints for current-state reports | [Reporting and Analytics](./Reporting-and-Analytics-Architecture.md) |
| Analytical reporting | Module team + Analytics | Read models for heavy queries; freshness visible | [Reporting and Analytics](./Reporting-and-Analytics-Architecture.md) |
| Financial reporting | Module team (Orders, Payments) | Must reconcile with authoritative transactions | [Reporting and Analytics](./Reporting-and-Analytics-Architecture.md) |
| Export | Module team + Platform | Tenant-scoped, authorized, audit-logged | [Reporting and Analytics](./Reporting-and-Analytics-Architecture.md) |
| Platform analytics | Platform team | De-identified, infrastructure metrics only (V1) | [Reporting and Analytics](./Reporting-and-Analytics-Architecture.md) |
| Tenant analytics | Module team | Tenant-isolated, authorized | [Reporting and Analytics](./Reporting-and-Analytics-Architecture.md) |
| Data freshness communication | Module team | Freshness visible on all non-real-time reports | [Reporting and Analytics](./Reporting-and-Analytics-Architecture.md) |

---

## 13. Data-Quality Ownership

| Domain | Quality Accountable | Validation Owner | Correction Owner | Document |
|--------|--------------------|--------------------|-----------------|----------|
| Products | Catalog team | Catalog domain layer | Catalog team + Tenant admin | [Data Quality](./Data-Quality-and-Governance.md) |
| Pricing | Pricing team | Pricing domain layer | Pricing team + Tenant admin | [Data Quality](./Data-Quality-and-Governance.md) |
| Inventory | Inventory team | Inventory domain layer | Inventory team + Tenant admin | [Data Quality](./Data-Quality-and-Governance.md) |
| Customers | Customer team | Customer domain layer | Customer team + Tenant admin | [Data Quality](./Data-Quality-and-Governance.md) |
| Orders | Order team | Order domain layer | Order team (corrections require governance) | [Data Quality](./Data-Quality-and-Governance.md) |
| Payments | Payment team | Payment domain layer | Payment team (restricted) | [Data Quality](./Data-Quality-and-Governance.md) |
| Imports | Integration owner | Import validation pipeline | Integration owner + Module team | [Data Quality](./Data-Quality-and-Governance.md) |
| Reference data | Platform team | Platform validation | Platform team | [Data Quality](./Data-Quality-and-Governance.md) |

---

## 14. Security Controls

| Control | Applied To | Owner | Document |
|---------|-----------|-------|----------|
| Encryption at rest | All persistent data | Operations + Security | [Data Protection](../Security/Data-Protection.md) |
| Encryption in transit | All data movement | Platform infrastructure | [Data Protection](../Security/Data-Protection.md) |
| Tenant isolation (query filter) | All tenant data access | Platform infrastructure | [Data Isolation Strategy](../Multi-Tenancy/Data-Isolation-Strategy.md) |
| Classification-based handling | All data by category | Module teams | [Data Classification](./Data-Classification-and-Sensitivity.md) |
| Access authorization | All data operations | Platform + Module | [Authorization Architecture](../Security/Authorization-Architecture.md) |
| Audit logging | All mutations to business data | Platform infrastructure | [Audit and Security Monitoring](../Security/Audit-and-Security-Monitoring.md) |
| Secret protection | Credentials, keys, tokens | Platform + Security | [Secrets and Key Management](../Security/Secrets-and-Key-Management.md) |
| Data masking in logs | PII and sensitive data | Platform infrastructure | [Data Classification](./Data-Classification-and-Sensitivity.md) |
| Export classification | All exported data | Platform + Module | [Data Classification](./Data-Classification-and-Sensitivity.md) |
| Backup encryption | All backup data | Operations | [Backup and Recovery](./Backup-Restore-and-Disaster-Recovery.md) |

---

## 15. Required Audit Events

| Event Category | Trigger | Data Captured | Document |
|---------------|---------|---------------|----------|
| Data creation | New business entity created | Entity type, ID, actor, timestamp | [Audit and Security Monitoring](../Security/Audit-and-Security-Monitoring.md) |
| Data modification | Business entity updated | Entity, field changes, actor, timestamp | [Audit and Security Monitoring](../Security/Audit-and-Security-Monitoring.md) |
| Data deletion | Business entity soft/hard deleted | Entity, actor, timestamp, reason | [Data Lifecycle](./Data-Lifecycle-and-Retention.md) |
| Data correction | Historical data corrected | Original value, new value, actor, reason | [Data Quality](./Data-Quality-and-Governance.md) |
| Data export | Data exported from platform | Scope, classification, actor, timestamp | [Reporting and Analytics](./Reporting-and-Analytics-Architecture.md) |
| Data import | Bulk data imported | Source, count, accepted/rejected, actor | [Data Quality](./Data-Quality-and-Governance.md) |
| Access to restricted data | Restricted-classified data accessed | Entity, accessor, purpose, timestamp | [Data Classification](./Data-Classification-and-Sensitivity.md) |
| Schema migration | Database schema changed | Migration ID, scope, actor, timestamp | [Schema Evolution](./Schema-Evolution-and-Migrations.md) |
| Backup/restore | Backup or restore operation | Scope, type, actor, timestamp | [Backup and Recovery](./Backup-Restore-and-Disaster-Recovery.md) |
| Tenant data deletion | Tenant offboarding data removal | Tenant ID, scope, actor, verification | [Data Lifecycle](./Data-Lifecycle-and-Retention.md) |

---

## 16. Required Testing

| Test Category | Scope | Owner | Document |
|--------------|-------|-------|----------|
| Validation rules | Every domain validation rule produces correct accept/reject | Module team | [Data Quality](./Data-Quality-and-Governance.md) |
| Tenant isolation | No cross-tenant data leakage in queries | Module team + Security | [Data Isolation Strategy](../Multi-Tenancy/Data-Isolation-Strategy.md) |
| Referential integrity | Cross-module references resolve correctly | Module team | [Data Modeling](./Data-Modeling-Principles.md) |
| Transaction boundaries | Operations are atomic within module | Module team | [Transaction and Consistency](./Transaction-and-Consistency-Architecture.md) |
| Eventually consistent flows | Cross-module events propagate and converge | Module team + Integration | [Transaction and Consistency](./Transaction-and-Consistency-Architecture.md) |
| Schema migrations | Migrations apply/rollback cleanly on production-like data | Module team + Ops | [Schema Evolution](./Schema-Evolution-and-Migrations.md) |
| Data lifecycle | Retention expiry triggers correct deletion/archival | Module team | [Data Lifecycle](./Data-Lifecycle-and-Retention.md) |
| Backup and restore | Backups are restorable and complete | Operations | [Backup and Recovery](./Backup-Restore-and-Disaster-Recovery.md) |
| Import validation | Invalid imports are rejected with proper errors | Module team | [Data Quality](./Data-Quality-and-Governance.md) |
| Export authorization | Exports respect tenant scope and authorization | Module team + Security | [Reporting and Analytics](./Reporting-and-Analytics-Architecture.md) |
| ID generation | IDs are unique, time-sortable, correct format | Platform team | [Identifier Strategy](./Identifier-Strategy.md) |
| Classification handling | Each classification level produces correct handling | Module team + Security | [Data Classification](./Data-Classification-and-Sensitivity.md) |

---

## 17. V1 versus Future Capabilities

| Capability | V1 | V2 | V3+ | Document |
|-----------|:---:|:---:|:---:|----------|
| Module-local ACID transactions | Yes | Yes | Yes | [Transaction and Consistency](./Transaction-and-Consistency-Architecture.md) |
| Cross-module eventual consistency (outbox) | Yes | Yes | Yes | [Transaction and Consistency](./Transaction-and-Consistency-Architecture.md) |
| Backend domain-layer validation | Yes | Yes | Yes | [Data Quality](./Data-Quality-and-Governance.md) |
| Tenant query filtering | Yes | Yes | Yes | [Data Isolation Strategy](../Multi-Tenancy/Data-Isolation-Strategy.md) |
| ULID-style identifiers | Yes | Yes | Yes | [Identifier Strategy](./Identifier-Strategy.md) |
| Single shared database | Yes | Evaluate | Possible split | [Data Access](./Data-Access-and-Persistence.md) |
| EF Core with module-scoped DbContext | Yes | Yes | Yes | [Data Access](./Data-Access-and-Persistence.md) |
| Zero-downtime migrations | Yes | Yes | Yes | [Schema Evolution](./Schema-Evolution-and-Migrations.md) |
| Data classification enforcement | Yes | Yes | Yes | [Data Classification](./Data-Classification-and-Sensitivity.md) |
| Manual reconciliation | Yes | Automated (financial) | Full automation | [Reporting and Analytics](./Reporting-and-Analytics-Architecture.md) |
| Direct-query reporting | Yes | Yes (+ read models) | Yes (+ warehouse) | [Reporting and Analytics](./Reporting-and-Analytics-Architecture.md) |
| Import with validation/quarantine | Yes | Yes | Yes | [Data Quality](./Data-Quality-and-Governance.md) |
| Read replicas | — | Yes | Yes | [Data Access](./Data-Access-and-Persistence.md) |
| Dedicated analytical store | — | — | Yes | [Reporting and Analytics](./Reporting-and-Analytics-Architecture.md) |
| Event-driven analytics pipeline | — | — | Yes | [Reporting and Analytics](./Reporting-and-Analytics-Architecture.md) |
| Data catalog tooling | — | — | Yes | [Data Quality](./Data-Quality-and-Governance.md) |
| Automated lineage tracking | — | — | Yes | [Data Quality](./Data-Quality-and-Governance.md) |
| AI/ML data consumption | — | — | Future | [Reporting and Analytics](./Reporting-and-Analytics-Architecture.md) |
| Multi-region data replication | — | — | Future | [Backup and Recovery](./Backup-Restore-and-Disaster-Recovery.md) |

---

## 18. Required ADR Triggers

| Trigger Condition | ADR Required For | Related Document |
|-------------------|-----------------|-----------------|
| Module needs to own data currently assigned to another module | Data ownership reassignment | [Data Ownership](./Data-Ownership.md) |
| Cross-module synchronous data access required | Deviation from eventual consistency model | [Transaction and Consistency](./Transaction-and-Consistency-Architecture.md) |
| New data classification level needed | Classification model extension | [Data Classification](./Data-Classification-and-Sensitivity.md) |
| Schema migration requires downtime | Exception to zero-downtime rule | [Schema Evolution](./Schema-Evolution-and-Migrations.md) |
| Retention period shorter than regulatory minimum | Compliance risk acceptance | [Data Lifecycle](./Data-Lifecycle-and-Retention.md) |
| Automatic duplicate merging proposed | Exception to manual-resolution rule | [Data Quality](./Data-Quality-and-Governance.md) |
| Shared mutable data between modules | Exception to single-owner rule | [Data Ownership](./Data-Ownership.md) |
| Non-ULID identifier format | Exception to identifier strategy | [Identifier Strategy](./Identifier-Strategy.md) |
| Direct database access bypassing module contracts | Exception to module data encapsulation | [Data Access](./Data-Access-and-Persistence.md) |
| Platform analytics using tenant business data | Governance exception for cross-tenant access | [Reporting and Analytics](./Reporting-and-Analytics-Architecture.md) |
| Database-per-module or database split | Data access architecture change | [Data Access](./Data-Access-and-Persistence.md) |

---

## 19. Future Refactoring Triggers

| Trigger | Impact | Source Document |
|---------|--------|----------------|
| Reporting queries impact transactional performance | Introduce read replicas or dedicated analytical store | [Reporting and Analytics](./Reporting-and-Analytics-Architecture.md) |
| Module data size exceeds single-database capacity | Evaluate database-per-module split | [Data Access](./Data-Access-and-Persistence.md) |
| Migration complexity exceeds manual review capacity | Introduce migration automation tooling | [Schema Evolution](./Schema-Evolution-and-Migrations.md) |
| Data quality incidents exceed manual triage capacity | Introduce quality monitoring tooling | [Data Quality](./Data-Quality-and-Governance.md) |
| Cross-module analytics required | Introduce event-driven projections | [Reporting and Analytics](./Reporting-and-Analytics-Architecture.md) |
| Financial reconciliation volume exceeds manual capacity | Automate reconciliation | [Reporting and Analytics](./Reporting-and-Analytics-Architecture.md) |
| Tenant data volume requires per-tenant isolation at storage level | Re-evaluate data isolation strategy | [Data Isolation Strategy](../Multi-Tenancy/Data-Isolation-Strategy.md) |
| Team size requires dedicated data steward roles | Formalize governance organization | [Data Quality](./Data-Quality-and-Governance.md) |
| Regulatory audit requires formal lineage evidence | Introduce lineage tooling | [Data Quality](./Data-Quality-and-Governance.md) |
| AI/ML product enters development | Introduce governed data consumption pipeline | [Reporting and Analytics](./Reporting-and-Analytics-Architecture.md) |

---

## 20. Documentation Gaps

### Identified Unresolved Dependencies

| Gap | Required By | Resolution Phase | Notes |
|-----|------------|-----------------|-------|
| API Architecture | Data Access, Identifier Strategy, Data Classification | Future phase | API layer must implement ID exposure rules, classification-based filtering |
| Event Architecture | Transaction and Consistency, Data Ownership | Future phase | Event contracts must carry correct data, respect boundaries |
| Infrastructure Architecture | Data Access, Backup and Recovery, Schema Evolution | Future phase | Physical infrastructure must support persistence, backup, migration patterns |
| Development Standards | Data Access, Data Modeling, Schema Evolution, Data Quality | Future phase | Coding standards must enforce data patterns (EF Core usage, validation, testing) |
| Module Specifications | All data documents | Future phase | Each module spec must declare its data ownership, validation, lifecycle rules |
| Glossary update | Data Architecture | Pending | Glossary should include data-specific terms |
| Roadmap data milestones | V1/Future capability split | Pending | Roadmap should reflect data capability delivery timeline |

### Phase 8 Completeness Check

| Document | Created | Linked in README | Linked in MASTER_INDEX | Internal Links Valid |
|----------|:---:|:---:|:---:|:---:|
| Data Architecture | Yes | Yes | Yes | Yes |
| Data Ownership | Yes | Yes | Yes | Yes |
| Data Classification and Sensitivity | Yes | Yes | Yes | Yes |
| Data Modeling Principles | Yes | Yes | Yes | Yes |
| Identifier Strategy | Yes | Yes | Yes | Yes |
| Transaction and Consistency | Yes | Yes | Yes | Yes |
| Data Access and Persistence | Yes | Yes | Yes | Yes |
| Schema Evolution and Migrations | Yes | Yes | Yes | Yes |
| Data Lifecycle and Retention | Yes | Yes | Yes | Yes |
| Backup, Restore, and Disaster Recovery | Yes | Yes | Yes | Yes |
| Reporting and Analytics Architecture | Yes | Yes | Yes | Yes |
| Data Quality and Governance | Yes | Yes | Yes | Yes |
| Data Impact Matrix (this document) | Yes | Yes | Yes | Yes |

---

## Implementation Impact Across Surfaces

| Surface | Data Requirements | Key Documents |
|---------|------------------|---------------|
| **Backend** | All data architecture applies. Module-scoped persistence. Domain validation. Tenant filtering. Transactional boundaries. Event publishing. | All Phase 8 documents |
| **Frontend** | No direct data access. Consumes API. UI validation is convenience only. | [Data Quality](./Data-Quality-and-Governance.md) |
| **SDKs** | Expose public IDs only. Respect classification on responses. Rate-limited data access. | [Identifier Strategy](./Identifier-Strategy.md), [Data Classification](./Data-Classification-and-Sensitivity.md) |
| **Administrative Portal** | Tenant-scoped data views. Export capability. Correction workflows. Audit visibility. | [Reporting and Analytics](./Reporting-and-Analytics-Architecture.md), [Data Quality](./Data-Quality-and-Governance.md) |
| **Storefront** | Read-only data consumption. Eventually consistent product/price/inventory. Public IDs only. | [Data Ownership](./Data-Ownership.md), [Identifier Strategy](./Identifier-Strategy.md) |
| **Developer Portal** | API documentation reflects data contracts. ID format documentation. Classification guidance. | [Identifier Strategy](./Identifier-Strategy.md), [Data Classification](./Data-Classification-and-Sensitivity.md) |
| **Background Workers** | Event processing. Eventual consistency propagation. Outbox processing. Lifecycle jobs. | [Transaction and Consistency](./Transaction-and-Consistency-Architecture.md), [Data Lifecycle](./Data-Lifecycle-and-Retention.md) |
| **Events and Queues** | Domain event publishing. Outbox pattern. Event schema evolution. | [Transaction and Consistency](./Transaction-and-Consistency-Architecture.md), [Schema Evolution](./Schema-Evolution-and-Migrations.md) |
| **Search** | Derived indexes from authoritative sources. Rebuild capability. Eventually consistent. | [Data Ownership](./Data-Ownership.md), [Data Access](./Data-Access-and-Persistence.md) |
| **Cache** | TTL-based. Derived from authoritative source. Invalidation on change. | [Data Access](./Data-Access-and-Persistence.md) |
| **Media** | Tenant-scoped storage. Classification-aware. Lifecycle managed. | [Data Classification](./Data-Classification-and-Sensitivity.md), [Data Lifecycle](./Data-Lifecycle-and-Retention.md) |
| **Reports** | Tenant-isolated. Freshness visible. Financial reconciled. No authoritative ownership. | [Reporting and Analytics](./Reporting-and-Analytics-Architecture.md) |
| **Exports and Imports** | Validation pipeline. Quarantine. Classification carries. Authorization. Audit. | [Data Quality](./Data-Quality-and-Governance.md), [Reporting and Analytics](./Reporting-and-Analytics-Architecture.md) |
| **Integrations** | External data untrusted. Same validation as user input. Source tracking. | [Data Quality](./Data-Quality-and-Governance.md) |
| **Infrastructure** | Encryption at rest/transit. Backup. Recovery. Connection pooling. | [Backup and Recovery](./Backup-Restore-and-Disaster-Recovery.md), [Data Access](./Data-Access-and-Persistence.md) |
| **CI/CD** | Migration execution. Migration testing. Schema validation. | [Schema Evolution](./Schema-Evolution-and-Migrations.md) |
| **Support Tooling** | Tenant data visibility (authorized). Correction workflows. Quality incident management. | [Data Quality](./Data-Quality-and-Governance.md) |
| **Public Documentation** | ID format. Data contracts. Classification guidance. API data models. | [Identifier Strategy](./Identifier-Strategy.md), [Data Classification](./Data-Classification-and-Sensitivity.md) |

---

## Version Gate

| Version | Data Architecture Gate |
|---------|----------------------|
| V1 | All 13 Phase 8 documents complete. Domain data ownership assigned. Tenant isolation enforced. Backend validation authoritative. Module-scoped persistence operational. ULID identifiers implemented. Zero-downtime migrations enforced. Data classification applied. Audit events captured. Import validation with quarantine. Manual reconciliation for financial data. Backup and recovery operational. |
| V2 | Read replicas evaluated. Automated financial reconciliation. Quality metrics dashboarded. Cross-module projections for reporting. Data contract monitoring. |
| V3 | Analytical data store operational. Event-driven analytics pipeline. Data catalog tooling. Automated lineage. Quality SLAs. AI/ML consumption pipeline (governed). |

---

## Decision Summary

| Decision | Rationale |
|----------|-----------|
| Single impact matrix consolidating all data architecture traceability | Distributed traceability is hard to validate. A single matrix ensures completeness and consistency review. |
| Every matrix entry links to authoritative document | Traceability requires source attribution. Claims without links are unverifiable. |
| Documentation gaps explicitly identified | Honest gap identification enables future phases to know what they must address. |
| Implementation surfaces mapped to documents | Implementers need a lookup from their component to relevant data requirements. |
| ADR triggers listed proactively | Teams know when they must write an ADR rather than discovering the need during review. |

---

## Architecture Impact

| Concern | Impact |
|---------|--------|
| Phase completeness | This matrix validates that Phase 8 is complete and internally consistent |
| Cross-phase traceability | Future architecture phases can identify which data requirements they must consume |
| Implementation guidance | Every platform surface knows which data documents apply to it |
| Governance | ADR triggers and refactoring triggers provide governance guardrails |
| Gap visibility | Identified gaps become work items for future phases |

---

## Implementation Impact

| Area | Impact |
|------|--------|
| Module teams | Use this matrix to identify all data responsibilities for their domain |
| Platform team | Use implementation surface matrix to ensure infrastructure meets all data requirements |
| Security team | Use security controls matrix to validate coverage |
| Operations | Use backup, retention, and migration matrices for operational planning |
| Architecture | Use gap analysis and ADR triggers for future phase planning |

---

## Security Responsibilities

| Role | Matrix Responsibilities |
|------|----------------------|
| Chief Data Architect | Maintains this matrix. Validates phase completeness. Identifies gaps. |
| Module Teams | Validate their domain's entries are accurate. Flag missing responsibilities. |
| Security Team | Validate security controls matrix completeness. Ensure no gaps in classification coverage. |
| Operations | Validate backup, recovery, and retention entries match operational reality. |
| Platform Team | Validate infrastructure-related entries are implementable. |

---

## Multi-Tenancy Responsibilities

| Responsibility | Matrix Coverage |
|---------------|----------------|
| Tenant data ownership visible | Matrix 3 (Tenant Data Ownership) |
| Tenant isolation in all data paths | Matrix 14 (Security Controls) |
| Tenant-scoped reporting | Matrix 12 (Reporting Responsibilities) |
| Tenant lifecycle data handling | Matrix 10 (Retention Responsibilities) |
| Cross-tenant access governance | Matrix 3, Matrix 14, Matrix 15 |

---

## Out of Scope

This matrix does not:

- Define new architecture (it traces existing decisions).
- Override individual document decisions (it references them).
- Provide implementation guidance beyond document references.
- Prescribe tooling for traceability management.
- Define module-level data models.

---

## Future Considerations

- **Automated traceability** — Tooling that validates links and completeness automatically.
- **Living matrix** — Auto-generated from document metadata as the repository grows.
- **Module-level matrices** — Each module specification includes its own data impact subset.
- **Cross-phase matrices** — Similar matrices for API, Event, and Infrastructure phases.
- **Compliance traceability** — Regulatory requirements mapped to data controls.

---

## Future Refactoring Triggers

This document should be revisited when:

- New data architecture documents are added (extend all matrices).
- Module specifications are written (validate ownership assignments).
- API Architecture phase begins (validate downstream dependency consumption).
- Event Architecture phase begins (validate event data requirements).
- Infrastructure Architecture phase begins (validate operational requirements).
- ADRs create exceptions to data architecture rules (update affected matrices).
- Regulatory requirements change data governance needs.

---

## Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 0.1 | 2026-07-20 | Chief Data Architect | Initial draft — Phase 8 completion matrix |
