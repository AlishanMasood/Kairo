# Identifier Strategy

## Metadata

| Field | Value |
|-------|-------|
| Title | Kairo Platform Identifier Strategy |
| Document ID | KAI-DATA-005 |
| Status | Draft |
| Version | 0.1 |
| Target Release | V1 |
| Owner | Platform Identifier and Resource Addressing Architect |
| Created | 2026-07-20 |
| Last Updated | 2026-07-20 |
| Reviewers | TODO |
| Related Documents | [Data Modeling Principles](./Data-Modeling-Principles.md), [Data Architecture](./Data-Architecture.md), [API Security](../Security/API-Security.md), [Tenant Resolution](../Multi-Tenancy/Tenant-Resolution.md), [Tenant Hierarchy](../Multi-Tenancy/Tenant-Hierarchy.md), [Glossary](../../02-Products/Glossary.md) |
| Dependencies | [Data Modeling Principles](./Data-Modeling-Principles.md), [Data Architecture](./Data-Architecture.md) |

---

## Purpose

This document defines the identifier strategy for the Kairo platform — how resources are identified internally, exposed externally, correlated across operations, and mapped to external systems.

Identifiers are foundational. They are referenced in every API response, every event, every log entry, every database query, and every external integration. Changing an identifier strategy after launch is one of the most disruptive changes possible. Getting it right from V1 prevents years of migration pain.

---

## Scope

This document covers:

- Identifier categories and their distinct purposes.
- Properties required per identifier type (uniqueness, stability, predictability).
- Architectural direction for identifier generation.
- External system mapping and correlation.
- Human-readable references vs. internal identifiers.
- V1 decisions and future considerations.

This document does not cover:

- Specific library or package selection for ID generation.
- Database column types or index configuration.
- API URL path format (defined in API architecture).
- Concrete ID format specifications (e.g., exact UUID version or ULID implementation).

---

## Identifier Categories

### Internal Identifiers

The primary identity of every entity within the platform.

| Aspect | Detail |
|--------|--------|
| Purpose | Uniquely identify every resource instance across the entire platform |
| Scope | Platform-wide (globally unique across all tenants) |
| Visibility | Exposed through APIs as the canonical resource identifier |
| Stability | Immutable. Never changes after creation. |
| Format direction | UUID or ULID-style (see evaluation below) |
| Generation | Platform-generated at resource creation time |

### Public Identifiers

Identifiers exposed to external consumers through APIs and events.

| Aspect | Detail |
|--------|--------|
| Purpose | Identify resources in API requests, responses, events, and webhooks |
| Relationship to internal ID | In V1, the public identifier IS the internal identifier. No separate public/internal mapping. |
| Stability | Immutable. Once exposed, it cannot change. |
| Format | Same as internal identifier (UUID/ULID-style) |

### External Provider Identifiers

Identifiers assigned by third-party systems (payment providers, shipping carriers, tax services).

| Aspect | Detail |
|--------|--------|
| Purpose | Reference a resource in an external system |
| Ownership | The external system generates and owns this identifier |
| Storage | Stored as a mapped attribute on the Kairo entity, not as the primary ID |
| Stability | Depends on the external provider. May change if the provider migrates. |
| Relationship | Mapped to the Kairo internal ID. One Kairo entity may have multiple external references (one per integration). |

### Human-Readable References

Sequential or formatted identifiers meaningful to humans (order numbers, invoice numbers).

| Aspect | Detail |
|--------|--------|
| Purpose | Provide a reference that humans can communicate (phone, email, print) |
| Scope | Tenant-scoped. An order number is unique within an organization, not platform-wide. |
| Format | Shorter than UUIDs. May include prefixes, sequences, or formatted patterns. |
| Stability | Immutable once assigned |
| Relationship | An additional attribute on the entity. Not the primary database identity. |
| Generation | Platform-generated per tenant using the tenant's configured sequence or pattern. |

### Tenant Identifiers

| Type | Scope | Purpose |
|------|-------|---------|
| Organization ID | Platform-wide | Identify the tenant boundary |
| Store ID | Platform-wide | Identify a store within an organization |

### Customer References

| Type | Scope | Purpose |
|------|-------|---------|
| Customer ID (internal) | Platform-wide | Internal identity of the customer entity |
| Customer number (human-readable) | Tenant-scoped | Human-communicable customer reference |

### Order References

| Type | Scope | Purpose |
|------|-------|---------|
| Order ID (internal) | Platform-wide | Internal identity of the order entity |
| Order number (human-readable) | Tenant-scoped | Human-communicable order reference (e.g., "ORD-10042") |

### Payment References

| Type | Scope | Purpose |
|------|-------|---------|
| Payment ID (internal) | Platform-wide | Internal identity of the payment record |
| Provider transaction ID | External (provider) | Reference in the payment provider's system |
| Payment reference (human-readable) | Tenant-scoped | Human-communicable payment reference |

### Event Identifiers

| Type | Scope | Purpose |
|------|-------|---------|
| Event ID | Platform-wide | Unique identity of a single event instance |
| Event type | Platform-wide (defined set) | Identifies what kind of event occurred |

### Correlation Identifiers

| Type | Scope | Purpose |
|------|-------|---------|
| Request ID | Per request | Traces a single API request through all processing |
| Trace ID | Per operation chain | Traces a distributed operation across services |
| Session ID | Per user session | Identifies an authenticated session |

### Idempotency Identifiers

| Type | Scope | Purpose |
|------|-------|---------|
| Idempotency key | Tenant-scoped, time-limited | Ensures a specific operation executes only once |

### Import Identifiers

| Type | Scope | Purpose |
|------|-------|---------|
| Import job ID | Platform-wide | Identifies a specific bulk import operation |
| Source reference | External (per record) | Identifies the record in the source system |

### Integration Mapping Identifiers

| Type | Scope | Purpose |
|------|-------|---------|
| External system ID | Per integration, per entity | Maps a Kairo entity to its counterpart in an external system |
| Sync reference | Per integration | Tracks synchronization state |

---

## Identifier Properties

### 1. Purpose

Every identifier type has one defined purpose. An identifier should not serve dual purposes (e.g., a correlation ID is not a business reference number).

**Correlation IDs and business identifiers are different concepts.** A request ID traces processing. An order number identifies a business entity. They must not be conflated.

---

### 2. Global Uniqueness

| Identifier Type | Global Uniqueness | Rationale |
|----------------|:-:|--------------|
| Internal resource IDs | Yes | No two entities share an ID regardless of tenant |
| Organization IDs | Yes | Tenants are unique platform-wide |
| Event IDs | Yes | Events are unique across all tenants |
| Correlation IDs (request, trace) | Yes | Must be traceable without tenant context |
| Human-readable references | No (tenant-scoped) | "ORD-1001" in Org A is independent of "ORD-1001" in Org B |
| Idempotency keys | No (tenant-scoped) | Keys are unique within a tenant + time window |
| External provider IDs | No (external scope) | Uniqueness is the provider's responsibility |

---

### 3. Tenant-Scoped Uniqueness

**Tenant-scoped references must include tenant context during resolution.**

A human-readable reference like "ORD-1001" requires the organization context to resolve to a specific resource. Without tenant context, the reference is ambiguous.

| Identifier | Resolution Requires Tenant Context |
|-----------|:--:|
| Internal ID (UUID) | No — globally unique, but tenant is still validated for authorization |
| Human-readable reference | Yes — must be resolved within a specific organization |
| External provider ID | Yes — must be resolved within a specific integration within an organization |
| Idempotency key | Yes — scoped to the tenant that submitted it |

---

### 4. Stability

All identifiers are stable once assigned. They do not change over the resource's lifetime.

| Rule | Description |
|------|-------------|
| IDs are assigned at creation | No temporary IDs that are later replaced |
| IDs survive updates | Updating an entity does not change its ID |
| IDs survive status changes | Moving an order to "Cancelled" does not change its ID |
| IDs do not encode mutable state | An ID does not contain the entity's status, type, or other changeable attribute |

---

### 5. Immutability

**Identifier reuse is prohibited unless explicitly designed.**

| Rule | Description |
|------|-------------|
| No ID reuse | A deleted resource's ID is retired permanently |
| No reassignment | An ID never points to a different resource |
| Organization IDs never reused | Prevents stale references from associating with a new tenant |
| Human-readable sequences never reset | An organization's order number sequence does not restart |

---

### 6. Predictability

**Unpredictable identifiers do not replace authorization.**

| Rule | Description |
|------|-------------|
| IDs should not be easily guessable | Sequential integers expose total count and enable enumeration |
| Unpredictability is not security | Even unpredictable UUIDs require authorization validation on every access |
| Authorization is independent of ID format | Whether the ID is a UUID or an integer, the authorization check is identical |

A caller who knows a valid ID from another tenant is still denied access. The authorization system does not rely on ID obscurity.

---

### 7. Sortability

| Need | Identifier Property |
|------|-------------------|
| Chronological ordering | Time-sortable IDs (ULID-style) enable natural chronological ordering without an additional column |
| Index efficiency | Monotonically increasing IDs produce better index locality than random UUIDs |
| Query patterns | "Most recent" queries benefit from time-sortable IDs |

---

### 8. Generation Ownership

| Identifier Type | Generator | Why |
|----------------|-----------|-----|
| Internal resource IDs | The platform (module creating the entity) | Centralized format consistency |
| Human-readable references | The platform (per-tenant sequence) | Must be unique within tenant |
| Correlation IDs | API gateway (request) or platform (trace, session) | Set at the entry point |
| Idempotency keys | Client (submitted with request) | Client controls idempotency |
| Event IDs | Platform event service | Unique per event instance |
| External provider IDs | External system | Platform does not control external ID generation |

---

### 9. Collision Handling

| Identifier Type | Collision Risk | Handling |
|----------------|---------------|----------|
| UUID v4 (random) | Negligible (2^122 space) | No collision handling needed for practical purposes |
| ULID | Negligible (128-bit with timestamp) | Same as UUID — practical collision is impossible |
| Human-readable sequences | None (sequential within tenant) | Database-enforced uniqueness constraint |
| Idempotency keys (client-generated) | Possible (client may reuse) | Handled by design — reuse returns cached result |

---

### 10. Public Exposure

| Identifier Type | Exposed in API | Exposed in Events | Exposed in Webhooks | Exposed in Logs |
|----------------|:-:|:-:|:-:|:-:|
| Internal resource IDs | Yes | Yes | Yes | Yes |
| Human-readable references | Yes (as attribute) | Yes (as attribute) | Yes (as attribute) | Yes |
| Correlation IDs | Yes (headers) | Yes (envelope) | Yes (header) | Yes |
| External provider IDs | Limited (specific APIs) | Limited | Limited | Yes |
| Idempotency keys | No (request-only) | No | No | Yes (hashed or truncated) |
| Database internal sequences | Never | Never | Never | Never |

---

### 11. Enumeration Risk

| Risk | Mitigation |
|------|-----------|
| Sequential IDs reveal resource count | Use non-sequential IDs for internal identifiers (UUID/ULID) |
| Sequential order numbers reveal volume | Acceptable — order numbers are tenant-scoped and visible only within the organization |
| ID patterns reveal creation time | Acceptable for ULID-style (timestamp is non-sensitive operational metadata) |
| Cross-tenant enumeration | Authorization prevents access regardless of ID format. Not an ID-level concern. |

---

### 12. Migration

| Scenario | Approach |
|----------|----------|
| Moving from external system to Kairo | External IDs stored as mapped references. Kairo generates its own internal IDs. Both coexist. |
| Changing ID format in future | Internal IDs are immutable. Format changes apply to new entities only. Old entities retain old format. Both formats are accepted. |
| Merging tenants (future) | IDs are globally unique. Merged data retains original IDs without collision. |

---

### 13. External System Mapping

**External provider IDs must not become Kairo's primary identity by default.**

| Rule | Description |
|------|-------------|
| Kairo generates its own ID | Every entity has a Kairo-generated primary ID regardless of external system involvement |
| External IDs are attributes | Stored as additional reference attributes, not as the primary key |
| Multiple external references | One entity may have references in multiple external systems (payment provider ID + ERP ID + shipping carrier ID) |
| External ID format is unconstrained | Kairo does not dictate external ID format. It stores whatever the provider returns as a string. |
| Lookup by external ID | APIs support finding a Kairo entity by its external reference (within tenant scope) |

---

### 14. Human-Readable Numbers

**Human-readable order numbers and internal resource IDs may serve different purposes.**

| Aspect | Internal ID | Human-Readable Reference |
|--------|-------------|--------------------------|
| Purpose | System identity | Human communication |
| Format | UUID/ULID (long, opaque) | Short, memorable, communicable (e.g., "ORD-10042") |
| Uniqueness | Platform-wide | Tenant-scoped |
| API usage | Primary lookup key | Secondary lookup (within tenant) |
| Displayed to customers | Sometimes (if no human-readable alternative) | Always (receipts, emails, support) |

### Human-Readable Rules

- Human-readable references are tenant-scoped (organization's sequence).
- They are monotonically increasing within the tenant (no gaps expected, but gaps acceptable).
- Format may be configurable per organization (prefix, padding, starting number) — future capability.
- They are assigned at entity creation and never change.
- They supplement (not replace) the internal ID.

---

### 15. Composite Identifiers

Identifiers formed from multiple components.

| When Appropriate | When NOT Appropriate |
|-----------------|---------------------|
| Junction/relationship records (product-category assignment) | Primary entity identity (use a single-value ID) |
| Tenant-scoped uniqueness constraints (org_id + SKU) | API resource addressing (use the single internal ID) |
| External system mapping (integration_id + external_id) | Cross-module references (use the single internal ID) |

### Rules

- Composite keys are used for uniqueness constraints and relationship records.
- Primary entity identity uses a single-value generated ID.
- APIs address resources by their single internal ID, not by composite keys.

---

### 16. Environment Separation

| Rule | Description |
|------|-------------|
| Test and production IDs are independent | IDs generated in test environments do not conflict with production |
| No cross-environment ID resolution | A test ID cannot be used to query production (or vice versa) |
| ID format does not differ by environment | The same generation approach is used in all environments |

---

### 17. Test vs. Production Identifiers

| Aspect | Test | Production |
|--------|------|-----------|
| Generation | Same algorithm | Same algorithm |
| Namespace | Separate database (isolated) | Separate database (isolated) |
| Collision risk across environments | None (separate storage) | None (separate storage) |
| Distinguishable? | Not by format. By environment context only. | Same format as test. |

**Idempotency keys are not permanent resource identities.** They are transient, time-limited, and tenant-scoped. They identify a specific attempt at an operation, not the resulting resource.

---

### 18. Future Distributed Generation

| Consideration | Direction |
|--------------|-----------|
| Multiple application instances | ID generation must not require coordination between instances |
| Service extraction | IDs generated by an extracted service must remain globally unique and format-compatible |
| Multi-region | IDs generated in different regions must not collide |
| High throughput | ID generation must not be a bottleneck under high-volume operations |

These requirements favor UUID/ULID-style generation (client-side, no coordination) over database sequences (centralized, coordination-required) for internal resource IDs.

---

## Identifier Approach Evaluation

### Integer Sequences

| Factor | Assessment |
|--------|-----------|
| Global uniqueness | No (per-database only) |
| Coordination-free | No (database generates) |
| Sortable | Yes (natural ordering) |
| Predictable | Yes (enumerable) |
| Human-readable | Moderate (short, but meaningless) |
| Distributed generation | Poor (requires centralized sequence) |
| V1 suitability for internal IDs | **Not recommended** — fails distributed generation and enumeration criteria |
| V1 suitability for human-readable refs | **Suitable** — sequential, tenant-scoped, simple |

### UUID v4 (Random)

| Factor | Assessment |
|--------|-----------|
| Global uniqueness | Yes (probabilistically) |
| Coordination-free | Yes (generated anywhere) |
| Sortable | No (random order) |
| Predictable | No (128-bit random) |
| Human-readable | No (36 characters) |
| Distributed generation | Excellent (no coordination) |
| Index performance | Poor (random distribution fragments B-tree indexes) |
| V1 suitability for internal IDs | **Suitable** — meets uniqueness and distribution requirements. Index fragmentation is a consideration. |

### ULID-Style (Timestamp + Random)

| Factor | Assessment |
|--------|-----------|
| Global uniqueness | Yes (timestamp + random component) |
| Coordination-free | Yes (generated anywhere) |
| Sortable | Yes (chronological by generation time) |
| Predictable | Partially (timestamp component is knowable; random component is not) |
| Human-readable | No (26 characters in Crockford base32) |
| Distributed generation | Excellent (no coordination) |
| Index performance | Good (monotonically increasing improves B-tree locality) |
| V1 suitability for internal IDs | **Recommended direction** — combines uniqueness, sortability, and distributed generation |

### Composite Business Keys

| Factor | Assessment |
|--------|-----------|
| Global uniqueness | Depends on composition (tenant + key is unique) |
| Coordination-free | Depends (some components may need coordination) |
| Sortable | Depends on composition |
| Human-readable | Often (e.g., "PROD-SKU-12345") |
| Distributed generation | Moderate (some components are coordinated) |
| V1 suitability for internal IDs | **Not recommended** — too coupled to business values that may change |
| V1 suitability for business references | **Suitable** — for tenant-scoped human-readable references |

---

## V1 Architectural Direction

| Identifier Type | V1 Direction | Rationale |
|----------------|-------------|-----------|
| Internal resource IDs | ULID-style (time-sortable, globally unique, coordination-free) | Best combination of properties for a platform that will scale |
| Human-readable references | Tenant-scoped sequential numbers with configurable prefix | Simple, meaningful to humans, no collision across tenants |
| Organization/Store IDs | Same as internal IDs (ULID-style) | Consistent approach |
| Correlation IDs | UUID v4 (randomness is the priority, not sortability) | Random for security. Traced through request lifecycle. |
| Event IDs | ULID-style (time-sortable for event ordering) | Chronological ordering is valuable for events |
| Idempotency keys | Client-provided (any string, tenant-scoped) | Client controls; platform stores and matches |
| External references | Stored as-is (string attribute) | Platform does not constrain external ID format |

---

## Decision Matrix

| Criterion | Integer Seq | UUID v4 | ULID-style | Composite |
|-----------|:-:|:-:|:-:|:-:|
| Global uniqueness | — | ★★★ | ★★★ | ★★ |
| Coordination-free | — | ★★★ | ★★★ | ★★ |
| Time-sortable | ★★★ | — | ★★★ | ★ |
| Index performance | ★★★ | ★ | ★★★ | ★★ |
| Distributed ready | — | ★★★ | ★★★ | ★★ |
| Non-enumerable | — | ★★★ | ★★ | ★ |
| Human-readable | ★★ | — | — | ★★★ |

★★★ = Excellent, ★★ = Good, ★ = Limited, — = Not suitable

**Recommended for internal resource IDs: ULID-style (time-sortable, globally unique, coordination-free, good index performance).**

**Recommended for human-readable references: Tenant-scoped sequential integers with prefix formatting.**

---

## Identifier Lifecycle

| Stage | Rules |
|-------|-------|
| Generation | Generated at entity creation. Immutable once assigned. |
| Storage | Stored as the primary key. Indexed for lookup. |
| Exposure | Included in API responses, events, and webhooks as the canonical reference. |
| Reference | Other entities and modules reference by this ID. |
| Correlation | Correlation IDs (request, trace) link related operations to this entity. |
| External mapping | External provider IDs are stored as mapped attributes alongside the internal ID. |
| Deletion | On entity deletion, the ID is retired. Never reused. Never reassigned. |
| Migration | On system migration, IDs travel with their entities unchanged. |

---

## Version Gate

| Version | Identifier Strategy Gate |
|---------|------------------------|
| V1 | Internal IDs use ULID-style generation. Human-readable references use tenant-scoped sequences. External IDs are stored as mapped attributes. Correlation IDs are generated at the gateway. Idempotency keys are tenant-scoped and time-limited. No ID reuse. Authorization does not depend on ID obscurity. |
| V2 | Configurable human-readable formats per organization. Multi-region ID generation validated. External ID lookup APIs operational for all major integrations. |
| V3 | Distributed generation proven at scale. ID format migration tooling (if needed). Cross-product ID consistency validated. |

---

## Decision Summary

| Decision | Rationale |
|----------|-----------|
| ULID-style for internal IDs | Combines global uniqueness, time-sortability (index performance), and coordination-free generation. Best fit for a platform that will scale and potentially distribute. |
| Separate human-readable references | Humans cannot communicate ULIDs. Order numbers, customer numbers, and invoice numbers serve a distinct human communication purpose. |
| External IDs as mapped attributes | Kairo must own its identity. External systems may change, be replaced, or have conflicting ID formats. Mapping preserves Kairo's independence. |
| No ID reuse | Reuse creates ambiguity and potential security/data-integrity issues. Retired IDs remain retired. |
| Authorization independent of ID format | ID obscurity provides no security guarantee. Authorization is validated on every access regardless of how the ID was obtained. |
| Tenant-scoped human references include tenant context for resolution | "ORD-1001" is ambiguous without knowing which organization. Resolution requires context. |

---

## Alternatives Considered

| Alternative | Rejected Because |
|------------|-----------------|
| Integer sequences for internal IDs | Not globally unique. Require database coordination. Enable enumeration. Poor for distributed generation. |
| UUID v4 for everything | Random distribution fragments B-tree indexes. No time-sortability. ULID provides the same uniqueness with better index behavior. |
| Composite business keys as primary identity | Business values may change (SKUs renamed, categories restructured). Primary identity must be independent of business attributes. |
| External provider ID as primary key | Creates dependency on external system. Provider migration requires changing primary keys (catastrophic). |
| Sequential human-readable IDs as primary key | Not globally unique. Tenant-scoped only. Cannot serve as cross-module reference. |
| Separate public/internal ID mapping in V1 | Adds complexity without V1 benefit. Internal ID can be exposed directly. Mapping is a future option if needed. |

---

## Trade-offs

| Trade-off | Accepted Because |
|-----------|-----------------|
| ULID exposes approximate creation time in the ID | Creation timestamp is non-sensitive operational metadata. The security benefit of sortability (index performance, natural ordering) outweighs the minimal information exposure. |
| Human-readable references add a second identifier to track | Humans cannot use ULIDs in conversation. The operational benefit of short, memorable references justifies the additional field. |
| External ID mapping adds storage per integration | Alternatives (using external IDs as primary) create catastrophic coupling. Mapping storage cost is negligible. |
| Client-generated idempotency keys may have poor entropy | The platform does not rely on idempotency keys for security. Their purpose is deduplication, not authentication. Format is the client's choice. |
| ULID is less universally supported than UUID | Most languages have ULID libraries. Where unavailable, UUID v7 (timestamp-based) provides similar properties. The architectural direction (time-sortable, globally unique) is more important than the specific format. |

---

## Architecture Impact

| Concern | Impact |
|---------|--------|
| Data model | Every entity has a ULID-style primary ID. Human-readable references are additional indexed attributes. External references are mapped attributes. |
| API design | Resources are addressed by internal ID in URL paths. Human-readable references are queryable through filter parameters. |
| Events | Events carry internal IDs for referenced entities. Event envelopes carry event IDs and correlation IDs. |
| Logging | Log entries include request ID, entity IDs, and tenant context. Never include idempotency key values in full. |
| Database | Primary keys are ULID-style (stored as appropriate column type). Human-readable sequences use database sequences per tenant. |
| Integrations | External reference mapping table per integration. Bidirectional lookup (Kairo ID → external, external → Kairo ID). |
| Caching | Cache keys use internal IDs (prefixed with tenant context). |
| Testing | Test environments generate IDs with the same algorithm. No cross-environment ID significance. |

---

## Implementation Impact

| Area | Impact |
|------|--------|
| Modules | Must use platform ID generation for all entity creation. Must not generate custom ID formats. Must store external references as mapped attributes. |
| APIs | Must accept internal IDs for resource addressing. Must support lookup by human-readable reference (within tenant). Must support lookup by external reference (within tenant + integration). |
| Events | Must include internal IDs in payloads. Must include event ID in envelope. Must include correlation IDs. |
| Gateway | Must generate request ID and trace ID on entry. Must propagate through all processing. |
| Integrations | Must store external system IDs as mapped attributes. Must support bidirectional lookup. Must never adopt external ID as primary. |
| Sequences | Must generate human-readable sequences per tenant. Must ensure uniqueness within tenant. Must never reset or reuse. |

---

## Security Responsibilities

| Role | Identifier Responsibilities |
|------|---------------------------|
| Identifier Architect | Defines ID strategy. Reviews ID usage patterns. Maintains format consistency. |
| Platform Team | Implements ID generation, sequence management, and correlation propagation. |
| Module Teams | Use platform ID generation. Store external references as mappings. Include IDs in events. |
| Security Team | Validates that authorization does not depend on ID obscurity. Reviews enumeration risk. |

---

## Multi-Tenancy Responsibilities

| Responsibility | Detail |
|---------------|--------|
| Internal IDs are globally unique (no tenant-scoped collision) | Two entities in different tenants never share an ID |
| Human-readable references are tenant-scoped | Resolution requires tenant context |
| Idempotency keys are tenant-scoped | Keys from Org A do not collide with keys from Org B |
| External references are tenant-scoped | Lookup by external ID requires specifying the organization and integration |
| Organization IDs are never reused | Prevents stale reference association after tenant deletion |

---

## Out of Scope

This document does not define:

- Specific ULID or UUID library selection — implementation decision.
- Database column types for storing IDs — defined in module specifications.
- API URL path format — defined in API architecture.
- Human-readable reference format patterns per entity type — defined in module specifications.
- Sequence generation implementation — defined in development standards.

---

## Future Considerations

- **UUID v7 adoption** — As UUID v7 (timestamp-ordered) gains broader support, it may become the standard format replacing ULID in library choices while preserving the same architectural properties.
- **Configurable reference formats** — Allow organizations to configure their human-readable reference format (prefix, length, starting number).
- **Vanity identifiers** — Allow organizations to assign custom slugs or identifiers for specific resources (URL-friendly product identifiers).
- **ID federation** — If Kairo operates in a marketplace model, IDs from one organization may need to be referenced by another (with controlled visibility).
- **Short URLs** — Generated short identifiers for sharing (tracking links, QR codes) that resolve to internal IDs.

---

## Future Refactoring Triggers

This document should be revisited when:

- The platform exceeds the scale where single-database sequence generation is practical (distributed sequence needed).
- Multi-region deployment requires validating cross-region ID uniqueness.
- A migration from ULID to another format is considered (backward compatibility planning).
- External systems require Kairo to accept their IDs as primary (architecture must resist this unless an ADR justifies it).
- Human-readable reference format customization becomes a customer requirement.
- UUID v7 becomes the ecosystem standard and migration from ULID is evaluated.

---

## Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 0.1 | 2026-07-20 | Platform Identifier and Resource Addressing Architect | Initial draft |
