# Data Ownership

## Metadata

| Field | Value |
|-------|-------|
| Title | Kairo Data Ownership and Boundaries |
| Document ID | KAI-DATA-002 |
| Status | Draft |
| Version | 0.1 |
| Target Release | V1 |
| Owner | Domain Data Ownership Architect |
| Created | 2026-07-20 |
| Last Updated | 2026-07-20 |
| Reviewers | TODO |
| Related Documents | [Data Architecture](./Data-Architecture.md), [Product Boundaries](../../02-Products/Product-Boundaries.md), [Domain Model](../../02-Products/Domain-Model.md), [Bounded Contexts](../../03-Business-Capabilities/Bounded-Contexts.md), [Module Architecture](../Module-Architecture.md), [Tenant Hierarchy](../Multi-Tenancy/Tenant-Hierarchy.md), [Data Isolation Strategy](../Multi-Tenancy/Data-Isolation-Strategy.md), [Data Protection](../Security/Data-Protection.md) |
| Dependencies | [Data Architecture](./Data-Architecture.md), [Module Architecture](../Module-Architecture.md) |

---

## Purpose

This document defines who owns what data in the Kairo platform. It establishes the rules for data creation, mutation, consumption, and lifecycle across all modules and services.

Ownership clarity prevents the most common data architecture failures: ambiguous accountability, coupling through shared data, inconsistent mutations from multiple sources, and data that nobody maintains. When every dataset has one owner, responsibility is clear and disputes are resolvable.

---

## Scope

This document covers:

- Ownership definitions and rules for all data categories.
- Mutation and consumption rights per data category.
- Ownership of derived, imported, and shared data.
- Rules for creation, update, deletion, export, and retention.
- Ownership transfer and dispute resolution.
- Ownership matrix mapping data to modules.

This document does not cover:

- Database schemas or table definitions — defined in module specifications.
- Data access layer implementation — defined in development standards.
- Tenant isolation enforcement — defined in [Data Isolation Strategy](../Multi-Tenancy/Data-Isolation-Strategy.md).
- Data encryption and classification — defined in [Data Protection](../Security/Data-Protection.md).

---

## 1. Data Ownership Definition

Data ownership means a single module is **accountable** for:

- Defining the data structure.
- Validating data on creation and update.
- Ensuring data correctness and consistency.
- Publishing changes through events.
- Managing the data lifecycle (retention, archival, deletion).
- Providing access to other modules through defined contracts.

Ownership is not about which table stores the data. It is about which module is responsible for it. A single database hosting all modules' data does not make all modules co-owners.

---

## 2. Domain Ownership

Data ownership follows business domain boundaries as defined in [Product Boundaries](../../02-Products/Product-Boundaries.md) and [Bounded Contexts](../../03-Business-Capabilities/Bounded-Contexts.md):

| Business Domain | Data Owned |
|----------------|-----------|
| Catalog | Products, variants, categories, attributes, product types, product relationships |
| Pricing | Price lists, prices, price rules, price schedules |
| Inventory | Stock levels, reservations, stock movements, locations |
| Orders | Orders, line items, order status history, returns |
| Cart | Cart state, cart items, cart calculations |
| Customers | Customer profiles, addresses, customer groups |
| Promotions | Discount rules, coupons, campaigns, usage records |
| Tax | Tax zones, tax rates, tax categories, tax exemptions |
| Fulfillment | Shipments, tracking references, shipping methods |
| Channels | Channel configurations, visibility rules |
| Identity | Users, credentials, sessions, roles, permissions, API keys |
| Configuration | Settings, overrides, feature flags |
| Audit | Audit entries, audit metadata |
| Events | Event log, delivery state, subscription records |
| Integration | Integration configurations, webhook registrations |
| Media | File metadata, storage references |
| Search | Index metadata, index state |

---

## 3. Module Ownership

Each module in the [Module Architecture](../Module-Architecture.md) is the exclusive owner of its data:

| Rule | Description |
|------|-------------|
| Single owner | Each dataset has exactly one owning module. No co-ownership. |
| Exclusive mutation | Only the owning module writes to its data. Other modules request changes through the owner's contracts. |
| Structure authority | The owning module defines and evolves its data structure. |
| Consistency authority | The owning module ensures its data is internally consistent. |
| Event authority | The owning module publishes events when its data changes. |
| Lifecycle authority | The owning module manages retention, archival, and deletion of its data. |
| Contract authority | The owning module defines how other modules access its data (query interfaces, DTOs). |

---

## 4. Platform-Owned Data

Data owned by the platform layer, not by any specific product or tenant:

| Data | Owner | Purpose |
|------|-------|---------|
| Platform configuration defaults | Configuration service | Global defaults for all tenants |
| Platform health metrics | Health service | Infrastructure monitoring |
| Platform aggregate usage | Platform analytics | Operational capacity planning |
| Encryption keys (platform-level) | Secret store | Cryptographic operations |
| Platform administrator accounts | Identity service | Platform administration |

### Rules

- Platform-owned data is not scoped to any tenant.
- Platform-owned data is invisible to tenants through business APIs.
- **Tenant-owned data cannot become platform-owned merely because it is aggregated.** Aggregated analytics derived from tenant data remains governed by tenant data rules (anonymization, consent, purpose limitation).

---

## 5. Organization-Owned Data

Data scoped to a specific organization (tenant):

| Data | Owner Module | Tenant Context |
|------|-------------|---------------|
| Users and memberships | Identity | Organization |
| API keys | Identity | Organization |
| Integration credentials | Integration | Organization |
| Webhook registrations | Integration | Organization |
| Organization configuration | Configuration | Organization |
| Audit trail | Audit | Organization |
| All commerce data (products, orders, etc.) | Respective commerce modules | Organization + Store |

### Rules

- All organization-owned data includes the organization identifier.
- Organization-owned data is invisible to other organizations.
- Deletion of an organization removes all its owned data (per [Tenant Lifecycle](../Multi-Tenancy/Tenant-Lifecycle.md)).

---

## 6. Store-Scoped Data

Commerce data further scoped to a specific store within an organization:

| Data | Owner Module | Scope |
|------|-------------|-------|
| Products and catalog | Catalog | Organization + Store |
| Prices and price lists | Pricing | Organization + Store |
| Inventory and stock | Inventory | Organization + Store |
| Orders | Orders | Organization + Store |
| Promotions | Promotions | Organization + Store |
| Tax configuration | Tax | Organization + Store |
| Fulfillment configuration | Fulfillment | Organization + Store |
| Channels | Channels | Organization + Store |

### Rules

- Store-scoped data includes both organization ID and store ID.
- Store-scoped data is accessible to users with org-level access (cross-store visibility within the org).
- Store-scoped data is deleted when the store is deactivated or the organization is deleted.

---

## 7. Customer-Owned Data

Data that belongs to a commerce customer within an organization:

| Data | Owner Module | Customer Access |
|------|-------------|----------------|
| Customer profile | Customers | Read + update own |
| Customer addresses | Customers | Full CRUD own |
| Order history | Orders | Read own |
| Cart state | Cart | Full CRUD own |

### Rules

- Customer data is organization-scoped (the customer belongs to one organization).
- Customers can view and manage their own data through self-service APIs.
- Customer data deletion follows privacy requirements (right of erasure).
- Customer-owned data is not accessible to other customers.

---

## 8. Application-Owned Data

Data generated by or specific to a client application:

| Data | Owner | Scope |
|------|-------|-------|
| Application session state | Platform (sessions) | Per application instance |
| Client-side preferences | Not stored on platform | N/A (client-managed) |

### Rules

- Applications do not own persistent business data on the platform. They consume and produce data through APIs.
- Application state (if any) is ephemeral and session-scoped.
- API keys that identify applications are owned by the Identity module, not by the application.

---

## 9. Shared Reference Data

Data that multiple modules need to reference but that has a single owner:

| Data | Owner | Consumers |
|------|-------|-----------|
| Products/variants | Catalog | Pricing, Inventory, Cart, Orders, Promotions, Fulfillment |
| Customers | Customers | Orders, Cart, Pricing, Promotions |
| Price lists | Pricing | Cart, Orders |
| Tax zones | Tax | Cart, Orders |
| Shipping methods | Fulfillment | Cart, Orders |

### Rules

- Shared reference data has **one owner** and **many consumers**.
- Consumers access shared data through the owning module's query contract.
- Consumers receive DTOs (projections), not internal entities.
- Consumers cache reference data locally (with TTL) for performance. The authoritative source remains the owner.
- **Shared database access does not grant shared ownership.** Being in the same database does not make another module's reference data yours.

---

## 10. Derived Data

Data produced by transforming or projecting authoritative data:

| Derived Data | Source | Owner |
|-------------|--------|-------|
| Search indexes | Module authoritative data | Search service (platform) |
| Cached values | Module authoritative data | Cache service (platform) |
| Aggregate metrics | Transactional data | Analytics service (future) |
| Webhook payloads | Event data | Event/webhook service (platform) |
| Export files | Module authoritative data | Export service |

### Rules

- **Derived data must remain traceable to authoritative data.** Every derived dataset can identify its source.
- Derived data can be rebuilt from the authoritative source at any time.
- Derived data does not modify the authoritative source.
- **Search indexes do not become authoritative.** If the index disagrees with the database, the database wins.
- **Reports do not become owners of transactional data.** A report reads from the authoritative source. It does not become the source.

---

## 11. Imported Data

Data brought into the platform from external sources:

| Import Type | Owner After Import | Source Tracking |
|------------|-------------------|----------------|
| Bulk product import | Catalog module | Import job ID, source reference |
| Customer import | Customers module | Import job ID, source reference |
| Inventory sync | Inventory module | Sync job ID, external reference |
| Price list import | Pricing module | Import job ID, source reference |

### Rules

- Once imported, data is owned by the receiving module. The import source does not retain ownership within the platform.
- **Integration copies must track their source and synchronization state.** Imported data records where it came from and when it was last synchronized.
- Import validation is the receiving module's responsibility. Invalid data is rejected, not stored.
- Import does not bypass business rules. Imported data goes through the same validation as API-created data.

---

## 12. Integration-Owned Data

Data that exists primarily to support external integrations:

| Data | Owner | Purpose |
|------|-------|---------|
| Integration configuration | Integration service | How to connect to external services |
| Webhook registrations | Integration service | Where to deliver events |
| Sync state | Integration service | Track synchronization progress with external systems |
| External reference mappings | Integration service | Map platform IDs to external system IDs |

### Rules

- Integration configuration and state is owned by the Integration service.
- External reference mappings enable correlation between platform data and external systems.
- Integration data is organization-scoped.
- Integration credentials are not "owned" by the Integration module — they are managed by the secret store.

---

## 13. Audit-Owned Data

| Data | Owner | Purpose |
|------|-------|---------|
| Audit entries | Audit service | Accountability record |
| Audit metadata | Audit service | Retention, indexing, delivery state |

### Rules

- Audit records are immutable once written.
- The Audit service is the sole owner. No other module modifies audit data.
- Audit data outlives the business data it describes (different retention).
- Audit data references business entities by ID. It does not embed copies of business data.

---

## 14. Analytical Data (Future)

| Data | Owner | Purpose |
|------|-------|---------|
| Aggregated metrics | Analytics service (future) | Business intelligence |
| Trend data | Analytics service (future) | Pattern detection |
| Benchmarks | Analytics service (future) | Comparative analysis |

### Rules

- Analytical data is derived from authoritative transactional data.
- Analytical data does not modify or replace authoritative data.
- Analytical data may be de-identified or aggregated for privacy.
- Analytical data is a V2+ concern. V1 does not implement an analytics pipeline.

---

## 15. Media Ownership

| Data | Owner | Purpose |
|------|-------|---------|
| File storage metadata | Media service | Track what files exist and where |
| File binary content | Media service (object storage) | Actual asset storage |
| File references (which entity uses which file) | Respective business module | Business association |

### Rules

- The Media service owns file storage and retrieval.
- Business modules own the association between their entities and media files (e.g., "Product X uses Image Y").
- Deleting a business entity should clean up its media references. Orphan detection handles missed cleanup.

---

## 16. Configuration Ownership

| Data | Owner | Purpose |
|------|-------|---------|
| Platform defaults | Configuration service | Global behavior defaults |
| Organization overrides | Configuration service (on behalf of org admin) | Tenant customization |
| Store overrides | Configuration service (on behalf of store admin) | Store customization |
| Feature flag state | Configuration service | Feature availability |

### Rules

- The Configuration service owns all configuration data.
- Business modules define what is configurable (keys, types, defaults). The Configuration service stores and resolves values.
- Configuration ownership follows the hierarchy defined in [Tenant Configuration](../Multi-Tenancy/Tenant-Configuration.md).

---

## 17. Search-Index Ownership

| Data | Owner | Purpose |
|------|-------|---------|
| Index structure | Search service | Query optimization |
| Indexed documents | Search service (derived from modules) | Searchable projection |

### Rules

- The Search service owns indexes as infrastructure.
- Indexed content is derived from module authoritative data.
- **Search indexes do not become authoritative.** Indexes can be rebuilt from the authoritative source.
- Modules inform the Search service what to index. The Search service does not reach into modules.

---

## 18. Cache Ownership

| Data | Owner | Purpose |
|------|-------|---------|
| Cached values | Cache service (platform) | Performance optimization |
| Cache invalidation state | Cache service (platform) | Freshness management |

### Rules

- Cache is infrastructure, not a data owner.
- **Caches and indexes are not authoritative sources of truth** (per [Data Architecture](./Data-Architecture.md)).
- Cache ownership means the Cache service manages storage and eviction. It does not own the data's meaning or correctness.
- Modules use the Cache service through the platform interface. They do not manage cache infrastructure directly.

---

## 19. Data Stewardship

Stewardship is operational responsibility for data quality and lifecycle:

| Role | Stewardship Responsibilities |
|------|------------------------------|
| Module team | Data quality, validation rules, consistency within their domain |
| Platform team | Infrastructure reliability, backup, tenant isolation enforcement |
| Organization admin | Business data accuracy within their organization (configuration, reference data) |
| Data architect | Cross-module data governance, ownership disputes, architecture evolution |

---

## 20. Ownership Transfer

When data ownership must change (rare, architecturally significant):

| Step | Action |
|------|--------|
| 1 | Identify the data category and its current owner |
| 2 | Identify the proposed new owner and the justification |
| 3 | Create an ADR documenting the transfer, rationale, and migration plan |
| 4 | Design the migration (data movement, contract changes, consumer updates) |
| 5 | Execute the migration with both old and new owners operating concurrently |
| 6 | Verify the new owner is authoritative and consumers have migrated |
| 7 | Decommission the old owner's responsibility for that data |

### Rules

- **Ownership changes require architecture review and potentially an ADR.**
- Ownership transfer is a deliberate, planned operation. It never happens accidentally.
- During transfer, there is still exactly one authoritative owner at any point in time. Concurrent dual ownership is a transition state, not a permanent state.

---

## 21. Ownership Disputes

When two modules believe they should own the same data:

| Step | Action |
|------|--------|
| 1 | Identify the data and the competing claims |
| 2 | Determine which module's bounded context most naturally owns the concept (per [Bounded Contexts](../../03-Business-Capabilities/Bounded-Contexts.md)) |
| 3 | If the data maps to one domain clearly, assign ownership to that domain |
| 4 | If the data spans domains, determine whether it should be split into domain-specific representations (each module owns its view) |
| 5 | Record the resolution in an ADR |

### Resolution Principles

- The module whose business domain the data serves most directly is the owner.
- If data serves multiple domains equally, it may belong to a shared platform service.
- Duplication (each module has its own copy) is acceptable when the data means different things in each context.
- The Data Architect has final authority on ownership disputes.

---

## 22. Future Product Ownership

When future products (Payments, POS, ERP) are introduced:

| Rule | Description |
|------|-------------|
| Products own their domain data | Payments owns payment transaction data. POS owns register session data. ERP owns accounting data. |
| Product boundaries are respected | Commerce does not own payment data even if Commerce initiates payments. |
| Cross-product data references use IDs | Products reference each other's entities by ID, not by embedding data. |
| Cross-product data access uses events or APIs | No direct cross-product database access. |
| Product data follows the same tenancy rules | All product data is organization-scoped. |

---

## Data Lifecycle Rules

### Creation

| Rule | Description |
|------|-------------|
| Single source of creation | Data is created through the owning module's defined creation path (API, import, event handler). |
| Validation at creation | The owning module validates all business rules before persisting. |
| Ownership assignment at creation | Organization ID and store ID (if applicable) are set at creation time. |
| Event publication at creation | Significant creations publish domain events for consumers. |

### Update

| Rule | Description |
|------|-------------|
| Only the owner mutates | Other modules request changes through the owner's command interface. |
| Validation at update | Business rules are re-validated on every update. |
| Event publication at update | Significant updates publish domain events for consumers (cache invalidation, index updates). |
| History preservation | For entities requiring history (orders), the update is recorded. |

### Validation

| Rule | Description |
|------|-------------|
| Owner validates | The owning module is responsible for validation. |
| Consumers do not validate on behalf of owners | A consuming module does not enforce the owning module's business rules. |
| Validation includes referential integrity | Cross-module references (IDs) are validated against the referenced module's contracts at write time. |

### Deletion

| Rule | Description |
|------|-------------|
| Owner deletes | Only the owning module can delete its data. |
| Cascade within module | Deletion cascades within the module (child entities deleted with parent). |
| Cross-module references | Cross-module references (IDs held by other modules) are handled through events or reconciliation. |
| Tenant deletion | Organization deletion coordinates across all modules (distributed operation). |

### Export

| Rule | Description |
|------|-------------|
| Owner provides export | The owning module defines what is exportable and in what format. |
| Tenant-scoped | Exports are always scoped to the authenticated organization. |
| No internal structure exposed | Export formats use documented, external representations (not internal schemas). |

### Retention

| Rule | Description |
|------|-------------|
| Owner manages retention | The owning module defines retention policies for its data. |
| Per [Data Protection](../Security/Data-Protection.md) | Retention follows data classification and purpose. |
| Deletion is audited | Retention expiration and deletion are recorded in the audit trail. |

### Replication

| Rule | Description |
|------|-------------|
| Derived copies track source | Any copy of data (cache, index, analytics) tracks its authoritative source. |
| Copies are rebuildable | Derived copies can be rebuilt from the authoritative source at any time. |
| Copies do not diverge permanently | Bounded staleness ensures copies converge with the source within a defined window. |

### Publication

| Rule | Description |
|------|-------------|
| Owner publishes events | Only the owning module publishes events about its data changes. |
| Events are facts | Events describe what happened (past tense). They are not commands to other modules. |
| Events do not replace ownership | Receiving an event does not make the consumer an owner of the data described in the event. |

### Consumption

| Rule | Description |
|------|-------------|
| Through contracts only | Consumers access data through the owning module's query contracts. |
| Read-only by default | Consumers have read access. Mutation requires going through the owner's command contracts. |
| Caching is permitted | Consumers may cache data locally (tenant-scoped, TTL-bounded). |
| **Other modules consume data through approved contracts.** | No direct database access to another module's tables. |

### Correction

| Rule | Description |
|------|-------------|
| Owner corrects | Data corrections are made through the owning module's update path. |
| Corrections are audited | Data corrections are recorded in the audit trail (before/after). |
| Derived data is refreshed | After correction, derived data (cache, indexes) is invalidated and rebuilt. |

---

## Data Ownership Matrix

| Data Category | Owning Capability | Owning Module | Tenant Scope | Authoritative Source | Allowed Consumers | Mutation Rights | Retention Owner | V1 Status |
|--------------|------------------|---------------|-------------|---------------------|------------------|----------------|----------------|:---------:|
| Products & variants | Catalog | Catalog | Org + Store | Catalog module DB | Pricing, Inventory, Cart, Orders, Promotions, Search | Catalog only | Catalog | V1 |
| Categories & attributes | Catalog | Catalog | Org + Store | Catalog module DB | Search, Promotions | Catalog only | Catalog | V1 |
| Price lists & prices | Pricing | Pricing | Org + Store | Pricing module DB | Cart, Orders | Pricing only | Pricing | V1 |
| Stock levels & reservations | Inventory | Inventory | Org + Store | Inventory module DB | Cart, Orders, Fulfillment | Inventory only | Inventory | V1 |
| Orders & line items | Orders | Orders | Org + Store | Orders module DB | Fulfillment, Reporting, Customers | Orders only | Orders | V1 |
| Cart state | Cart | Cart | Org + Store | Cart module DB | Orders (at checkout conversion) | Cart only | Cart | V1 |
| Customer profiles | Customers | Customers | Org | Customers module DB | Orders, Cart, Pricing, Promotions | Customers only | Customers | V1 |
| Discount rules & coupons | Promotions | Promotions | Org + Store | Promotions module DB | Cart, Orders | Promotions only | Promotions | V1 |
| Tax zones & rates | Tax | Tax | Org + Store | Tax module DB | Cart, Orders | Tax only | Tax | V1 |
| Shipments & tracking | Fulfillment | Fulfillment | Org + Store | Fulfillment module DB | Orders, Notifications | Fulfillment only | Fulfillment | V1 |
| Channel configuration | Channels | Channels | Org + Store | Channels module DB | Catalog, Pricing, Inventory, Promotions | Channels only | Channels | V1 |
| Users & credentials | Identity | Identity | Org (membership) | Identity service DB | All modules (authenticated context) | Identity only | Identity | V1 |
| Roles & permissions | Identity | Identity | Org | Identity service DB | Authorization pipeline | Identity only | Identity | V1 |
| API keys | Identity | Identity | Org | Identity service DB | Gateway | Identity only | Identity | V1 |
| Configuration settings | Configuration | Configuration | Hierarchy | Configuration service DB | All modules | Config service only | Configuration | V1 |
| Feature flags | Configuration | Configuration | Hierarchy | Configuration service DB | All modules | Config service only | Configuration | V1 |
| Audit entries | Audit | Audit | Org | Audit service DB | Compliance, Admins (read-only) | Audit service only | Audit | V1 |
| Event log | Events | Event service | Org | Event service DB | All subscribers | Event service only | Event service | V1 |
| Integration config | Integrations | Integration service | Org | Integration service DB | Platform services | Integration service only | Integration | V1 |
| Webhook registrations | Integrations | Integration service | Org | Integration service DB | Event/webhook delivery | Integration service only | Integration | V1 |
| Media files | Media | Media service | Org | Object storage | All modules (read) | Media service only | Media | V1 |
| Search indexes | Search | Search service | Org | Search infrastructure | All modules (query) | Search service only | Search | V1 |
| Cache entries | Cache | Cache service | Org (key prefix) | Cache infrastructure | All modules (read) | Cache service only | Cache (TTL) | V1 |
| Payment transactions | Payments | Payment service | Org | Payment service DB | Orders, Accounting | Payment service only | Payment | Future |
| Accounting entries | ERP | Accounting service | Org | Accounting service DB | Reporting | Accounting only | Accounting | Future |
| Register sessions | POS | POS service | Org + Store | POS service DB | Reporting, Accounting | POS only | POS | Future |

---

## Explicit Prohibitions

| Prohibition | Rationale |
|------------|-----------|
| **Shared database access does not grant shared ownership** | Being in the same database is an infrastructure decision. Ownership is determined by domain, not by physical location. |
| **Reports do not become owners of transactional data** | Reporting reads from authoritative sources. It does not become the source regardless of how much processing it does. |
| **Search indexes do not become authoritative** | Indexes are derived. If they disagree with the source, the source wins. Indexes are rebuildable. |
| **Integration copies must track their source and synchronization state** | Untracked copies become stale without anyone knowing. Source tracking enables reconciliation. |
| **Tenant-owned data cannot become platform-owned merely because it is aggregated** | Aggregation does not change ownership. Aggregated tenant data is still governed by tenant data rules. |
| **Ownership changes require architecture review and potentially an ADR** | Ownership defines who is accountable. Changing accountability is a significant architectural decision. |

---

## Version Gate

| Version | Data Ownership Gate |
|---------|-------------------|
| V1 | Every V1 data category has a defined owner. Module boundaries are enforced (no cross-module direct access). Tenant scoping is applied to all organization-owned data. Derived data tracks its authoritative source. Data lifecycle rules (creation, deletion, retention) are operational per category. |
| V2 | Future product ownership boundaries are formalized when products enter development. Analytical data ownership is established when the analytics pipeline is built. Integration sync tracking is automated. |
| V3 | Cross-product ownership boundaries are proven under multi-product operation. Ownership governance handles disputes across product teams. Data stewardship is formalized. |

---

## Decision Summary

| Decision | Rationale |
|----------|-----------|
| One owner per dataset | Ambiguous ownership leads to inconsistency. Clear ownership creates accountability. |
| Consumption through contracts only | Direct access creates coupling. Contracts enable independent evolution. |
| Derived data is explicitly not authoritative | Prevents the common failure where a cache or report becomes the de facto source of truth. |
| Imports are validated by the receiving module | The receiving module's business rules apply regardless of data source. Import does not bypass validation. |
| Platform aggregation does not change ownership | Privacy and tenant isolation apply to aggregated data. Aggregation is a transformation, not a transfer of ownership. |
| Ownership transfer requires ADR | Ownership determines system structure. Casual transfers create structural instability. |

---

## Alternatives Considered

| Alternative | Rejected Because |
|------------|-----------------|
| Shared ownership (multiple owners per dataset) | Creates ambiguity about who validates, who mutates, and who manages lifecycle. Leads to inconsistency. |
| Database-level ownership enforcement | V1 uses a shared database. Database-level separation (separate users per module) is future complexity. Application-level enforcement is sufficient for V1. |
| No formal ownership (implicit through usage) | Implicit ownership leads to "everyone owns it, no one maintains it." Data quality degrades without explicit accountability. |
| Import bypasses validation | Allows invalid data into the system. Invalid data causes downstream failures. All data enters through business rules. |
| Analytics owns a copy of transactional data | Creates a competing source of truth. Analytics reads from the authoritative source. It does not maintain its own copy in V1. |

---

## Trade-offs

| Trade-off | Accepted Because |
|-----------|-----------------|
| Cross-module access through contracts adds latency vs. direct queries | Contracts prevent coupling. The latency (milliseconds) is acceptable. Caching mitigates repeated access. |
| Single ownership means bottleneck if the owner is slow | The owning module is responsible for performance of its data access. Caching by consumers mitigates read bottlenecks. |
| No direct reporting queries across modules | Reports must use module contracts or event-derived projections. This prevents ad-hoc cross-module coupling but adds reporting complexity. |
| Import validation may reject data from trusted sources | Even trusted external systems may have data that violates platform business rules. Rejecting ensures consistency. |
| Derived data may be temporarily stale | Bounded staleness is acceptable for caches and indexes. The authoritative source is always available for critical decisions. |

---

## Architecture Impact

| Concern | Impact |
|---------|--------|
| Module design | Each module defines its data structure, exposes query contracts, and manages data lifecycle. |
| Data access layer | Enforces module boundaries. Prevents cross-module direct access. Routes through contracts. |
| Events | Modules publish events on data changes. Derived data consumers update from events. |
| Caching | Consumers cache reference data (tenant-scoped, TTL). Invalidation from owner's events. |
| Search | Search indexes are built from module data via event-driven updates. Indexes are not authoritative. |
| Reporting | Reports consume data through module contracts or event-derived projections. Reports do not own data. |
| Imports | Import paths go through the owning module's validation. No direct data injection. |
| Testing | Tests verify ownership enforcement (no cross-module direct access, single writer verification). |

---

## Implementation Impact

| Area | Impact |
|------|--------|
| Modules | Must define data structures. Must expose query and command contracts. Must publish events on changes. Must manage retention. Must not access other modules' data directly. |
| Data access layer | Must enforce single-owner writes. Must prevent cross-module queries. Must route through contracts. |
| Events | Must support event-driven derived data updates. Must carry ownership context (which module published). |
| Imports | Must route through owning module's validation. Must record source and sync state. |
| Exports | Must use module-defined export formats. Must scope to tenant. Must not expose internal structure. |
| Search/Cache | Must track authoritative source. Must support rebuild. Must scope to tenant. |
| Testing | Must verify no cross-module direct data access. Must verify single-writer principle. Must test ownership boundaries. |

---

## Security Responsibilities

| Role | Data Ownership Responsibilities |
|------|-------------------------------|
| Data Ownership Architect | Defines ownership rules. Resolves disputes. Reviews ownership changes. |
| Module Teams | Own their data. Maintain quality. Expose contracts. Manage lifecycle. |
| Platform Team | Enforces boundaries. Manages derived data infrastructure. Supports ownership model. |
| Data Architect | Governs cross-module data architecture. Maintains ownership matrix. |
| Security Team | Validates that ownership boundaries align with access control. Reviews data flows. |

---

## Multi-Tenancy Responsibilities

| Responsibility | Owner |
|---------------|-------|
| Organization ID on all tenant-owned data | Module teams (at data creation) |
| Tenant filter enforcement | Platform data access layer |
| Per-tenant data lifecycle | Module teams (retention, deletion) |
| Tenant-scoped derived data | Platform services (cache, search, events) |

---

## Out of Scope

This document does not define:

- Database table names or schema definitions — defined in module specifications.
- Data access layer implementation — defined in development standards.
- Specific data validation rules per module — defined in module specifications.
- Tenant isolation enforcement mechanisms — defined in [Data Isolation Strategy](../Multi-Tenancy/Data-Isolation-Strategy.md).
- Data classification and encryption — defined in [Data Protection](../Security/Data-Protection.md).

---

## Future Considerations

- **Data ownership visualization** — Tooling that displays the current ownership map and highlights violations.
- **Automated ownership validation** — CI/CD checks that verify no module accesses another module's data directly.
- **Data lineage tracking** — Automated tracking of where data flows from source through transformations.
- **Cross-product data governance** — When multiple products exist, governance ensures boundaries are respected across products.
- **Data ownership SLAs** — Formal commitments from module owners about data quality, availability, and freshness.

---

## Future Refactoring Triggers

This document should be revisited when:

- A new product is added (new domain with new data ownership).
- A module is extracted to an independent service (ownership becomes physically enforced).
- An ownership dispute cannot be resolved with existing rules.
- Derived data (cache, index, analytics) is being treated as authoritative.
- Cross-module direct data access is discovered (violation of ownership rules).
- Analytical data platform is built (new ownership category).
- Data mesh concepts are adopted (decentralized ownership formalization).

---

## Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 0.1 | 2026-07-20 | Domain Data Ownership Architect | Initial draft |
