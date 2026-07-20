# Data Modeling Principles

## Metadata

| Field | Value |
|-------|-------|
| Title | Kairo Data Modeling Principles |
| Document ID | KAI-DATA-004 |
| Status | Draft |
| Version | 0.1 |
| Target Release | V1 |
| Owner | Enterprise Data Modeling Architect |
| Created | 2026-07-20 |
| Last Updated | 2026-07-20 |
| Reviewers | TODO |
| Related Documents | [Data Architecture](./Data-Architecture.md), [Data Ownership](./Data-Ownership.md), [Bounded Contexts](../../03-Business-Capabilities/Bounded-Contexts.md), [Tenant Hierarchy](../Multi-Tenancy/Tenant-Hierarchy.md), [Glossary](../../02-Products/Glossary.md), [Module Architecture](../Module-Architecture.md), [Commerce Domain](../../02-Products/Commerce-Domain.md) |
| Dependencies | [Data Architecture](./Data-Architecture.md), [Data Ownership](./Data-Ownership.md) |

---

## Purpose

This document defines the principles for conceptual and logical data modeling across the Kairo platform. It establishes how business concepts are represented in data, how relationships are expressed, and how common modeling challenges (money, time, status, extensibility) are addressed consistently.

Data models are the most durable artifacts in a system. APIs change. UIs are rebuilt. But the underlying data model persists for years. Getting the model right — modeling the business domain, not the UI or the current implementation convenience — creates a foundation that supports evolution.

---

## Scope

This document covers:

- Modeling philosophy and abstraction levels.
- Entity, value object, and aggregate principles.
- Domain-specific modeling patterns (money, time, status, addresses).
- Extensibility, denormalization, and flexibility boundaries.
- Tenant and module scoping in models.

This document does not cover:

- Concrete table definitions, column types, or SQL — defined in module specifications.
- ORM entity classes or mapping — defined in development standards.
- Physical database optimization (indexes, partitioning) — defined in implementation.
- Specific module data models — defined in each module's specification.

---

## Model Abstraction Levels

### 1. Conceptual Model

The business view. Describes what exists in the domain using business language.

| Aspect | Detail |
|--------|--------|
| Audience | Business stakeholders, architects, product owners |
| Content | Business entities, relationships, high-level rules |
| Detail level | No attributes. No types. No storage concerns. |
| Example | "A Product has Variants. Variants have Prices. Prices belong to Price Lists." |
| Purpose | Align on what the domain contains before deciding how to model it |

### 2. Logical Model

The design view. Describes entities, attributes, relationships, and constraints without physical storage decisions.

| Aspect | Detail |
|--------|--------|
| Audience | Architects, developers |
| Content | Entities with attributes, relationships, cardinality, business rules, constraints |
| Detail level | Named fields with domain types. No database-specific types. No indexes. |
| Example | "Product: id (UUID), name (string), status (ProductStatus), created_at (timestamp)" |
| Purpose | Define the data structure that module code will work with |

### 3. Physical Model

The implementation view. Describes how the logical model maps to storage.

| Aspect | Detail |
|--------|--------|
| Audience | Developers, DBAs |
| Content | Tables, columns, data types, indexes, constraints, partitioning |
| Detail level | Database-specific. Implementation-complete. |
| Purpose | Guide implementation. Defined in module specifications, not here. |

**This document addresses conceptual and logical modeling principles only.** Physical modeling decisions are implementation concerns documented elsewhere.

---

## 2. Aggregate Boundaries

An aggregate is a cluster of entities that are always consistent together and are treated as a unit for data access.

### Principles

- An aggregate has one root entity. External references point to the root, not to internal entities.
- Consistency within an aggregate is strong (transactional).
- Consistency across aggregates is eventual (event-driven).
- Aggregate boundaries align with module boundaries in most cases.
- An aggregate is loaded and saved as a unit. Partial updates to internal entities go through the root.

### Conceptual Example

```
Order (aggregate root)
├── LineItem (internal entity)
├── LineItem (internal entity)
└── OrderStatusHistory (internal entity)
```

External modules reference the Order by its ID. They do not reference LineItems directly.

### Rules

- Keep aggregates small. Large aggregates create contention and performance issues.
- Cross-aggregate references use IDs, not embedded objects.
- Cross-module relationships are always cross-aggregate (by ID reference).

---

## 3. Entity Identity

Every entity in the system has a unique identity that persists throughout its lifecycle.

### Principles

- Entities are identified by a platform-generated UUID.
- Business identifiers (SKU, order number) are attributes, not primary identifiers.
- IDs are globally unique within the platform (not just within a tenant).
- IDs are assigned at creation time and never change.
- IDs do not encode tenant, module, or type information (opaque).

### Tenant-Scoped vs. Platform-Wide Uniqueness

- **Platform-wide:** Entity IDs (UUIDs) are unique across all tenants. No two entities share an ID regardless of tenant.
- **Tenant-scoped:** Business identifiers (SKU "ABC-123") are unique within a tenant. Different tenants may have the same SKU.

**Tenant-scoped uniqueness must be distinguished from platform-wide uniqueness.** Uniqueness constraints on business identifiers must include the tenant scope (organization_id + business identifier).

---

## 4. Value Objects

A value object represents a concept with no identity — defined entirely by its attributes.

### Principles

- Value objects are immutable. Changing a value means replacing it, not mutating it.
- Value objects are compared by value, not by reference.
- Value objects often represent measurements, quantities, or composite values.

### Conceptual Examples

- **Money:** amount + currency (e.g., 29.99 EUR)
- **Address:** street + city + region + postal code + country
- **Weight:** value + unit (e.g., 2.5 kg)
- **DateRange:** start + end

### Rules

- Value objects do not have their own identity or lifecycle independent of their owning entity.
- Value objects are stored as part of their parent entity (embedded, not in separate tables with their own IDs).

---

## 5. Relationships

### Principles

- Relationships within an aggregate use direct object references.
- Relationships across aggregates (or modules) use ID references only.
- Relationships across modules are documented in the owning module's specification.
- **Cross-module relationships must respect module boundaries.** A module holds another module's entity ID. It does not hold a copy of that entity's data.

### Cardinality Documentation

Every relationship documents its cardinality:

| Notation | Meaning |
|----------|---------|
| 1:1 | Exactly one on each side |
| 1:N | One to many |
| N:M | Many to many (typically through a junction concept) |
| 0..1 | Optional on one side |
| 0..N | Optional, many possible |

---

## 6. Ownership

**Use explicit ownership.**

| Principle | Detail |
|-----------|--------|
| Every entity has an owner | The owning module is responsible for the entity's lifecycle |
| Ownership is declared | Module specifications declare which entities they own |
| No orphans | Entities cannot exist without a parent context (organization, store, owning entity) |
| Tenant context mandatory | Every tenant-owned entity includes organization_id |
| Store context where applicable | Commerce entities include store_id |

---

## 7. Reference Data

Reference data defines the stable entities against which transactional data operates.

### Principles

- Reference data changes infrequently (administrative operations).
- Reference data is read frequently (every transaction references it).
- Reference data has a defined lifecycle (active, inactive/archived).
- Changes to reference data publish events so consumers can refresh caches.
- Historical transactions reference the reference data as it was at transaction time (snapshot, not live reference).

### Conceptual Examples

- Product catalog (products, categories, attributes)
- Tax zones and rates
- Shipping methods
- Price lists

---

## 8. Enumerations

Enumerations represent a fixed set of allowed values for a field.

### Principles

- Enumerations are defined in the domain model and code, not only in the database.
- New values require a code change and deployment (they represent new business concepts).
- Enumerations are named with domain terminology (not generic codes).
- Serialization uses the name (string), not a numeric ID (for readability and resilience).

### Conceptual Examples

- OrderStatus: Draft, Confirmed, Processing, Shipped, Delivered, Cancelled, Returned
- PaymentStatus: Pending, Authorized, Captured, Failed, Refunded
- ProductStatus: Draft, Active, Archived

---

## 9. Status Modeling

**Status fields require documented transition rules.**

### Principles

- Status is not a free-form string. It is an enumeration with defined values.
- Not all transitions are valid. A Cancelled order cannot become Shipped.
- Transitions are documented (state machine diagram or transition table).
- Invalid transitions are rejected by the domain model.
- Status changes are events (published for consumers).

### Conceptual Example

```
Order Status Transitions:
  Draft → Confirmed
  Confirmed → Processing
  Processing → Shipped
  Shipped → Delivered
  Confirmed → Cancelled
  Processing → Cancelled
  (Delivered → Returned is a separate lifecycle)
```

### Rules

- Every status enumeration has a documented transition table.
- The domain model enforces valid transitions (not just the database).
- Status history is preserved for entities that require audit trail.

---

## 10. Temporal Data

Data that is meaningful only within a time context.

### Principles

- **Time values require timezone semantics.** Every timestamp declares whether it is UTC, local time, or timezone-aware.
- Platform-internal timestamps are stored in UTC.
- Display times are converted to the user's/tenant's configured timezone at the presentation layer.
- Time precision is documented (second, millisecond, microsecond).

---

## 11. Effective Dates

Data that is valid within a defined time range.

### Principles

- Effective dates define when a value becomes active and when it expires.
- Effective-dated data has explicit start and optional end dates.
- Queries for "current" data filter by effective date range.
- Overlapping effective periods for the same entity are either prohibited (by constraint) or have priority rules.

### Conceptual Examples

- Price effective date: "This price is valid from 2026-08-01 to 2026-08-31."
- Promotion schedule: "This discount is active from July 1 to July 15."

---

## 12. Soft Deletion

**Soft deletion must not be used as a universal substitute for lifecycle design.**

### Principles

- Soft deletion (marking a record as deleted instead of physically removing it) is appropriate for some entities, not all.
- Entities with soft deletion must document why they require it (audit trail, undo capability, referential integrity).
- Soft-deleted records must be excluded from all business queries by default.
- Soft deletion does not satisfy privacy deletion requirements (right of erasure requires actual removal or anonymization).
- Prefer explicit lifecycle states (Active, Archived, Deactivated) over a generic "deleted" flag when the business domain has meaningful non-active states.

### When Soft Deletion Is Appropriate

- Records referenced by historical transactions (products that have been ordered).
- Records needed for short-term undo/recovery.
- Records with compliance retention requirements.

### When Soft Deletion Is NOT Appropriate

- As a universal default for all entities.
- As a substitute for proper lifecycle design.
- For personal data subject to deletion requests (must actually delete or anonymize).

---

## 13. Archival

Moving data from active storage to long-term storage when it is no longer operationally needed.

### Principles

- Archived data is not deleted. It is moved to a less accessible, lower-cost tier.
- Archived data may have different query performance characteristics.
- Archival criteria are documented per entity (e.g., "orders older than 2 years").
- Archived data can be recalled if needed (with potentially higher latency).

---

## 14. Historical Records

Records that capture a point-in-time snapshot.

### Principles

- When a transaction references data that may change (prices, addresses), the transaction captures a snapshot of the referenced value at transaction time.
- Historical snapshots are immutable. Changing the source data does not change the snapshot.
- Snapshots are clearly distinguished from live references.

### Conceptual Examples

- Order line items capture the resolved price at order time (not a live reference to the price list).
- Shipping address on an order is a snapshot (customer may update their address later; the order retains the original).

---

## 15. Immutable Records

Records that cannot be changed after creation.

### Principles

- Audit entries are immutable.
- Published events are immutable.
- Completed order snapshots are immutable (status changes are new records, not mutations to the snapshot).
- Immutable records may be deleted (per retention policy) but never updated.

---

## 16. Monetary Values

**Monetary values require currency context.**

### Principles

- Money is represented as an amount plus a currency code (ISO 4217).
- Amount is stored as an integer in minor units (cents, pence) to avoid floating-point errors.
- Currency is never implied. Every monetary field explicitly includes its currency.
- Arithmetic on money validates that currencies match (cannot add EUR to USD).
- Display formatting is a presentation concern, not a storage concern.

### Conceptual Representation

```
Money:
  amount: integer (minor units, e.g., 2999 = $29.99)
  currency: string (ISO 4217, e.g., "USD", "EUR")
```

---

## 17. Quantities and Units

**Quantities require units where relevant.**

### Principles

- Quantities that have units (weight, dimensions, volume) store both the value and the unit.
- Unit conversion is a presentation or calculation concern, not a storage concern.
- Unit-less quantities (count of items) do not require a unit field.
- Negative quantities are valid only where the business domain permits them (e.g., inventory adjustment can be negative).

### Conceptual Representation

```
Weight:
  value: decimal
  unit: string ("kg", "lb", "g", "oz")
```

---

## 18. Time Zones

**Time values require timezone semantics.**

### Principles

- All platform-internal timestamps are stored in UTC.
- UTC timestamps are converted to local time at the presentation layer using the tenant's configured timezone.
- "Date-only" values (e.g., promotion start date) are stored as dates without time, interpreted in the tenant's timezone.
- Scheduled operations use the tenant's timezone for scheduling, converted to UTC for execution.
- Timezone is never guessed. It comes from tenant configuration or explicit request context.

---

## 19. Localization Data

Data that varies by locale (language, region).

### Principles

- Localizable fields (product names, descriptions) may have multiple locale-specific values.
- The default locale value is always present. Additional locales are optional.
- Locale resolution follows: requested locale → fallback locale → default locale.
- Localization data is stored alongside the entity it describes (not in a separate "translations" table for a different module).
- Localization is a data concern (what content exists) and a presentation concern (which to show). This document addresses the data aspect.

---

## 20. Address Data

Addresses represent physical or postal locations.

### Principles

- Addresses are value objects (no independent identity).
- Address structure accommodates international formats (not all countries have states/provinces, zip codes vary in format).
- Address fields are strings with minimal platform-enforced format constraints (validation beyond basic format is business-specific or provider-delegated).
- Country uses ISO 3166-1 alpha-2 codes.
- Addresses stored on transactions (orders, shipments) are snapshots (not live references to the customer's current address).

---

## 21. Metadata

Data about data.

### Principles

- Every entity carries creation and modification metadata (created_at, updated_at, created_by, updated_by).
- Metadata fields are set by the platform/module automatically, not by external callers.
- Metadata is not business data — it supports operational and audit concerns.
- Metadata follows consistent naming across all modules.

---

## 22. Extensibility

Allowing tenants to extend the data model without schema changes.

### Principles

- The platform supports custom fields (defined per tenant, per entity type).
- Custom fields store data alongside standard entity data.
- Custom fields have defined types (string, number, boolean, date, reference).
- Custom fields are validated at write time per their type definition.
- Custom fields are accessible through APIs (queryable, filterable where supported).
- **Flexible fields must not replace proper domain modeling.** If a concept is known and common, it should be a first-class modeled field, not a custom field.

---

## 23. Custom Fields

### Principles

- Custom field definitions are tenant-scoped (each organization defines their own).
- Custom field values are stored per entity instance.
- Custom fields do not participate in core business logic (they are data extensions, not logic extensions).
- Custom fields have defined constraints (required/optional, max length, allowed values).
- Adding a custom field does not require platform deployment or schema migration.

### Boundaries

- Custom fields cannot replace module-owned fields. If the platform should model a concept natively, it should be a schema field.
- Custom fields cannot create relationships between entities (no foreign keys in custom fields).
- Custom field names follow defined conventions (no reserved names, no collision with platform fields).

---

## 24. Schema Flexibility

### Trade-offs

| Approach | When Appropriate | When NOT Appropriate |
|----------|-----------------|---------------------|
| Strict schema (defined columns) | Known, stable domain concepts | Highly variable data that differs per tenant |
| JSON/flexible fields | Tenant-specific extensions, metadata, integration-specific data | Core domain concepts that every tenant shares |
| EAV (Entity-Attribute-Value) | Highly dynamic attribute sets (product attributes) | Core transactional data with known structure |

### Rules

- Core domain data uses strict schema (products, orders, customers have defined fields).
- Tenant extensions use the custom fields mechanism.
- Integration metadata (external system references, sync state) may use flexible storage.
- **Flexible fields must not replace proper domain modeling.** The temptation to make everything flexible creates unmaintainable, unqueryable data.

---

## 25. Denormalization

**Denormalization must have a documented reason and authoritative source.**

### Principles

- The default is normalized data. Denormalization is an optimization applied to specific, documented cases.
- Every denormalized field documents its authoritative source and refresh mechanism.
- Denormalized data is treated as derived data (not authoritative).
- Denormalization is acceptable for read performance optimization when the authoritative source is identified and the staleness window is bounded.

### Appropriate Denormalization

- Order line items embedding the product name at order time (historical snapshot, not a live reference).
- Search indexes containing denormalized product data (derived, rebuildable).
- Cached aggregate counts (derived, eventually consistent).

### Inappropriate Denormalization

- Copying another module's data into your tables because "it's easier than calling their API."
- Storing customer email on every order record for "convenience" (creates data synchronization problems).

---

## 26. Derived Fields

Fields whose values are calculated from other data.

### Principles

- Derived fields document their calculation source and formula.
- Derived fields are either computed on read (no storage) or materialized (stored for performance).
- Materialized derived fields have a documented refresh mechanism.
- Derived fields do not become authoritative sources for the data they are derived from.

---

## 27. Audit Fields

Standard metadata fields present on entities that require change tracking.

### Principles

- Standard audit fields: created_at, created_by, updated_at, updated_by.
- These fields are set by the platform automatically. They are not client-settable through APIs.
- Additional audit (full change history) is provided by the Audit service, not by per-entity history tables (unless the domain specifically requires inline history, e.g., order status transitions).

---

## 28. Tenant Ownership Fields

### Principles

- Every tenant-owned entity includes `organization_id` (non-nullable, set at creation).
- Commerce entities additionally include `store_id` (non-nullable for store-scoped entities).
- Tenant ownership fields are set at creation and never change (resources do not move between tenants).
- These fields participate in uniqueness constraints where business identifiers are tenant-scoped.

---

## 29. Store Scope Fields

### Principles

- Entities scoped to a store include both `organization_id` and `store_id`.
- Store scope determines visibility and access control (per [Tenant Hierarchy](../Multi-Tenancy/Tenant-Hierarchy.md)).
- Entities shared across stores within an organization (customers, users) include `organization_id` but not `store_id`.
- The presence or absence of `store_id` communicates the entity's scope level.

---

## 30. Future Service Extraction

### Principles

- Data models are designed so that a module's data can be physically separated (moved to its own database) without changing the logical model.
- Cross-module references use IDs (not foreign keys that would break with separation).
- No cross-module joins exist in the logical model (even if the physical database currently allows them).
- Aggregate boundaries are module-aligned, enabling clean extraction.

---

## Mandatory Principles Summary

| Principle | Rationale |
|-----------|-----------|
| **Model business concepts, not UI screens** | UIs change frequently. Domain concepts are stable. Models survive UI rewrites. |
| **Preserve domain terminology** | Use terms from the [Glossary](../../02-Products/Glossary.md). "Order" not "TransactionRecord." "Product" not "Item." |
| **Avoid generic entities when a domain concept is known** | "Entity" with a "type" field is lazy modeling. If it's a Product, model a Product. |
| **Use explicit ownership** | Every entity knows who owns it (module, tenant, store). No orphans. |
| **Monetary values require currency context** | Money without currency is meaningless. $100 and €100 are different values. |
| **Quantities require units where relevant** | A weight of "5" is meaningless. 5kg and 5lb are different values. |
| **Time values require timezone semantics** | Store in UTC. Display in local time. Never ambiguous. |
| **Status fields require documented transition rules** | Status without transitions is a free-form string. Transitions make the lifecycle explicit. |
| **Soft deletion must not be used as a universal substitute for lifecycle design** | Not everything needs soft delete. Use proper lifecycle states. Respect privacy deletion. |
| **Flexible fields must not replace proper domain modeling** | Custom fields are for tenant extensions. Known domain concepts get proper schema. |
| **Denormalization must have a documented reason and authoritative source** | Undocumented denormalization creates inconsistency. Document the source and refresh mechanism. |
| **Cross-module relationships must respect module boundaries** | Reference by ID. No embedded foreign objects. No cross-module joins. |
| **Tenant-scoped uniqueness must be distinguished from platform-wide uniqueness** | SKU uniqueness is per-tenant. Entity ID uniqueness is platform-wide. Different constraints. |

---

## Trade-offs

### Normalization vs. Denormalization

| Factor | Normalization | Denormalization |
|--------|--------------|----------------|
| Data integrity | Higher (single source) | Lower (must sync copies) |
| Write performance | Better (update one place) | Worse (update multiple places) |
| Read performance | May require joins | Better for read-heavy queries |
| Complexity | Simpler writes, complex reads | Complex writes, simpler reads |
| When to use | Default for transactional data | Read-optimized projections, historical snapshots |

### JSON/Flexible Fields vs. Strict Schema

| Factor | Strict Schema | JSON/Flexible |
|--------|---------------|---------------|
| Query performance | Better (indexed columns) | Varies (JSON indexing support varies) |
| Validation | Database-enforced | Application-enforced |
| Evolution | Requires migration | Schema-less evolution |
| When to use | Known, stable domain concepts | Tenant-specific extensions, variable attributes |

### Inheritance vs. Composition

| Factor | Inheritance (type hierarchy) | Composition (separate entities) |
|--------|-----------------------------|---------------------------------|
| Query simplicity | Single table, polymorphic | Separate tables, explicit joins |
| Schema clarity | May have many null columns | Each type has its own clean schema |
| When to use | Few types, much shared structure | Many types, different structures |

### Generic Reference Model vs. Explicit Entities

| Factor | Generic (one table for all reference data) | Explicit (dedicated entity per concept) |
|--------|---------------------------------------------|----------------------------------------|
| Development speed | Faster initial (one pattern) | Slower initial (per concept) |
| Maintainability | Low (unclear what the data means) | High (each concept is modeled) |
| Query clarity | Low (generic columns, type field) | High (named fields with meaning) |
| When to use | Never for core domain | Always for core domain |

### Read Models vs. Single Model

| Factor | Single Model | Dedicated Read Model |
|--------|-------------|---------------------|
| Complexity | Lower | Higher (separate projection) |
| Read performance | May be limited by write-optimized schema | Optimized for query patterns |
| Consistency | Immediate | Eventually consistent |
| When to use | Most entities in V1 | High-read-volume projections (search, reporting) |

### Historical Snapshots vs. Live References

| Factor | Snapshot at Transaction Time | Live Reference |
|--------|-----------------------------|--------------------|
| Data integrity | Preserves what was true at the time | Shows current truth |
| Storage cost | Higher (copies embedded) | Lower (reference only) |
| When to use | Transactions (order prices, shipping addresses) | Active references (current catalog, active price list) |

---

## Version Gate

| Version | Data Modeling Gate |
|---------|------------------|
| V1 | All V1 module data models follow these principles. Money includes currency. Time is UTC. Status has transition rules. Custom fields mechanism is defined. Module boundaries are respected in data relationships. Tenant and store scope fields are present on all relevant entities. |
| V2 | Effective-dated data patterns are proven for pricing and promotions. Archival policies are implemented for high-volume entities. Historical snapshot patterns are standardized across modules. |
| V3 | Complex localization patterns are formalized. Advanced custom field types are evaluated. Cross-module read models are architecturally defined for reporting. |

---

## Decision Summary

| Decision | Rationale |
|----------|-----------|
| UUIDs for entity identity | Globally unique. No coordination needed for ID generation. Support distributed systems future. |
| Money as integer minor units + currency | Avoids floating-point errors. Makes arithmetic safe. Currency context prevents calculation errors. |
| UTC for all internal timestamps | Eliminates timezone ambiguity in storage. Presentation layer handles conversion. |
| Aggregate boundaries align with modules | Enables clean service extraction. Prevents cross-module transactional coupling. |
| Custom fields for tenant extensions | Extensibility without schema migration. Tenant-specific without cluttering the core model. |
| Snapshots for transactional references | Preserves historical accuracy. Price at order time should not change when the price list is updated. |
| Status as enumeration with transitions | Prevents invalid state combinations. Makes lifecycle explicit. Enables event-driven status changes. |

---

## Alternatives Considered

| Alternative | Rejected Because |
|------------|-----------------|
| Floating-point for money | Floating-point arithmetic produces rounding errors. Integer minor units are precise. |
| Local time storage | Ambiguous for multi-timezone operations. UTC is unambiguous. Local conversion at display. |
| Auto-increment integer IDs | Not globally unique. Creates collision risk on service extraction. UUIDs are inherently unique. |
| Generic entity model ("everything is a row in one table") | Unmaintainable. Unclear semantics. No type safety. Terrible query performance. |
| Custom fields for everything (no strict schema) | Custom fields are untyped in business logic. Core concepts need proper modeling for validation, querying, and business rule enforcement. |
| Soft delete for all entities | Creates confusion about what is "really there." Breaks privacy deletion requirements. Proper lifecycle states are clearer. |

---

## Architecture Impact

| Concern | Impact |
|---------|--------|
| Module design | Each module models its domain using these principles. Models are logical (not physical) until implementation. |
| API design | API response models align with logical model concepts (entities become resources, value objects become embedded structures). |
| Events | Event payloads use the same domain terminology and structures as the logical model. |
| Database | Physical models implement the logical model. Implementation choices (JSON columns, table inheritance) are per-module decisions within these principles. |
| Testing | Tests validate business rules, status transitions, and constraint enforcement defined in the logical model. |
| Documentation | Module specifications include a logical data model section following these principles. |

---

## Implementation Impact

| Area | Impact |
|------|--------|
| Modules | Must model their domain following these principles. Must document entities, relationships, value objects, status transitions, and ownership. |
| Money handling | Must use integer minor units + currency everywhere. Must never use floating-point for financial values. |
| Time handling | Must store UTC. Must convert at presentation layer. Must document "date-only" vs. "timestamp" semantics. |
| Custom fields | Must use the platform custom fields mechanism. Must not add arbitrary JSON columns to bypass proper modeling. |
| Cross-module references | Must use ID references only. Must not embed or join across module boundaries. |
| Status fields | Must document valid transitions. Must enforce in domain logic. Must publish status change events. |

---

## Security Responsibilities

| Role | Modeling Responsibilities |
|------|-------------------------|
| Data Modeling Architect | Defines modeling principles. Reviews module data models. Maintains modeling standards. |
| Module Teams | Model their domain following these principles. Document logical models in specifications. |
| Platform Team | Provides custom fields mechanism, audit fields automation, and tenant context enforcement. |
| Data Architect | Reviews cross-module relationships. Validates domain alignment. |

---

## Multi-Tenancy Responsibilities

| Responsibility | Detail |
|---------------|--------|
| organization_id on all tenant entities | Non-nullable. Set at creation. Never changes. |
| store_id on commerce entities | Non-nullable for store-scoped entities. |
| Tenant-scoped uniqueness | Business identifiers (SKU) unique within tenant, not platform-wide. |
| Cross-tenant references prohibited | No entity references an entity in another organization. |

---

## Out of Scope

This document does not define:

- Specific module data models (entities, fields, types) — defined in module specifications.
- Database table definitions or SQL — defined in implementation.
- ORM configuration — defined in development standards.
- Physical optimization (indexing, partitioning) — defined in implementation.
- Data migration scripts — defined in operational documentation.

---

## Future Considerations

- **Domain event sourcing** — Some modules may benefit from modeling state as a sequence of events rather than a current snapshot.
- **Bi-temporal modeling** — Tracking both "when it happened" and "when we recorded it" for complex auditing.
- **Graph relationships** — If product recommendations or customer relationships become complex, graph modeling may supplement relational models.
- **Versioned entities** — Formal entity versioning for entities that require tracked evolution (beyond simple updated_at).
- **Modeling automation** — Code generation from logical model specifications to reduce manual translation.

---

## Future Refactoring Triggers

This document should be revisited when:

- A module's domain requires modeling patterns not covered here.
- Service extraction exposes cross-module data dependencies that violate these principles.
- The custom fields mechanism needs expansion beyond current constraints.
- Bi-temporal or event-sourced patterns are adopted for specific modules.
- A new product introduces domain concepts that challenge existing modeling patterns.
- Localization requirements exceed the current simple model.

---

## Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 0.1 | 2026-07-20 | Enterprise Data Modeling Architect | Initial draft |
