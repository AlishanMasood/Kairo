# Event Contract Standards

## Metadata

| Field | Value |
|-------|-------|
| Title | Kairo Event Contract and Schema Governance Standards |
| Document ID | KAI-EVT-003 |
| Status | Draft |
| Version | 0.1 |
| Target Release | V1 |
| Owner | Event Contract and Schema Governance Architect |
| Created | 2026-07-21 |
| Last Updated | 2026-07-21 |
| Reviewers | TODO |
| Related Documents | [Event Architecture](./Event-Architecture.md), [Event Taxonomy and Ownership](./Event-Taxonomy-and-Ownership.md), [API Contract Standards](../API/API-Contract-Standards.md), [Identifier Strategy](../Data/Identifier-Strategy.md), [Data Classification and Sensitivity](../Data/Data-Classification-and-Sensitivity.md), [Transaction and Consistency Architecture](../Data/Transaction-and-Consistency-Architecture.md) |
| Dependencies | [Event Architecture](./Event-Architecture.md), [Event Taxonomy and Ownership](./Event-Taxonomy-and-Ownership.md), [Identifier Strategy](../Data/Identifier-Strategy.md) |

---

## Applicable Version

This document defines V1 event contract standards. All integration events published across module boundaries must conform to these conventions. The standards ensure consistent structure, safe evolution, and predictable consumption across the platform.

---

## Purpose

This document defines the rules for event contract design — how events are identified, structured, versioned, and documented. Contract consistency ensures that any consumer can reliably process any event from any module without module-specific parsing logic or undocumented assumptions.

Without contract standards, each module invents its own event structure, identifier conventions, timestamp formats, and metadata placement. The result is an event ecosystem that requires per-module consumer logic, creates parsing fragility, and resists evolution.

---

## Scope

This document covers:

- Event envelope structure (identity, type, timestamps, context, metadata).
- Payload conventions (naming, types, nullability, monetary values, identifiers).
- Context fields (tenant, correlation, causation, trace).
- Payload style evaluation (full-state, change, reference, hybrid).
- Schema evolution and compatibility rules.
- Contract testing and documentation requirements.

This document does not cover:

- Specific event type definitions per module (module specifications).
- Serialization library or format selection (development standards).
- Event broker configuration (infrastructure documentation).
- Outbox table schema (module implementation).
- Consumer implementation patterns (development standards).

---

## Mandatory Principles

| # | Principle |
|---|-----------|
| 1 | Event contracts are stable integration contracts when published beyond an internal boundary |
| 2 | Event payloads include only data needed by intended consumers |
| 3 | Events must not expose persistence entities directly |
| 4 | Tenant context must be explicit where applicable |
| 5 | Correlation and causation identifiers serve different purposes |
| 6 | Event occurrence time and publication time are different concepts |
| 7 | Monetary values require currency context |
| 8 | Sensitive data must be minimized |
| 9 | Consumers must not depend on undocumented fields |
| 10 | Schema evolution must preserve compatibility |
| 11 | An event identifier is not the same as the business resource identifier |
| 12 | Metadata must not become an uncontrolled dumping ground |

---

## Event Envelope Structure

Every integration event consists of an **envelope** (standard metadata) and a **payload** (domain-specific content). These are structurally separated.

### Conceptual Envelope (non-binding)

```json
{
  "id": "evt_01H5AAABBBCCC",
  "type": "order.placed",
  "version": "1",
  "occurredAt": "2026-07-21T10:30:00.000Z",
  "publishedAt": "2026-07-21T10:30:00.250Z",
  "producer": "order-module",
  "tenantId": "org_01H3DDDEEEFFF",
  "storeId": "store_01H4GGGHHHJJJ",
  "correlationId": "corr_01H5KKKLLMM",
  "causationId": "evt_01H5AAAPREVIOUS",
  "traceId": "trace-abc-123-def",
  "subject": {
    "type": "order",
    "id": "order_01H5NNNPPPQQQ"
  },
  "data": {
    "orderId": "order_01H5NNNPPPQQQ",
    "customerId": "cust_01H2RRSSSTTT",
    "status": "pending",
    "totalAmount": { "amount": "129.99", "currency": "USD" },
    "itemCount": 3
  },
  "metadata": {
    "schemaVersion": "1.0",
    "dataClassification": "confidential"
  }
}
```

---

## Envelope Fields

### 1. Event Identity

**An event identifier is not the same as the business resource identifier.**

| Rule | Detail |
|------|--------|
| Field | `id` |
| Purpose | Uniquely identifies this specific event instance |
| Format | Platform identifier format (ULID-style string, prefixed `evt_`) per [Identifier Strategy](../Data/Identifier-Strategy.md) |
| Uniqueness | Globally unique across all events, all tenants, all time |
| Immutable | Never changes after publication |
| Deduplication | Consumers use this ID for deduplication (detect duplicate deliveries) |
| Not the resource ID | The event ID identifies the event. The resource ID (in payload or subject) identifies the business entity. They are different. |

---

### 2. Event Type

| Rule | Detail |
|------|--------|
| Field | `type` |
| Purpose | Identifies what kind of event this is — routes to correct consumers |
| Format | Lowercase dot-notation: `{domain}.{action_past_tense}` (e.g., `order.placed`, `payment.captured`, `inventory.adjusted`) |
| Stable | Event type strings are stable contracts. Renaming is breaking. |
| Routing key | Consumers subscribe by type. Infrastructure uses type for routing. |
| Documented | Every event type is documented in the event catalog |

---

### 3. Event Name

| Rule | Detail |
|------|--------|
| Convention | Past tense (events are facts that have occurred) |
| Domain prefix | Domain name comes first: `product.activated`, `customer.registered` |
| Specificity | Specific enough to route correctly. `order.placed` not `order.changed`. |
| No implementation detail | `order.placed` not `order.inserted_into_database` |
| Action clarity | The name communicates what business action completed |

---

### 4. Event Version

| Rule | Detail |
|------|--------|
| Field | `version` |
| Purpose | Identifies the schema version of this event's payload |
| Format | Simple integer string: `"1"`, `"2"`, `"3"` |
| Per event type | Each event type has its own version sequence |
| Required from V1 | Even when only one version exists, version field is present (enables future evolution) |
| Consumer handling | Consumers declare which version(s) they support |

---

### 5. Occurrence Time

**Event occurrence time and publication time are different concepts.**

| Rule | Detail |
|------|--------|
| Field | `occurredAt` |
| Purpose | When the business event actually happened (state change committed) |
| Format | ISO 8601 UTC with millisecond precision: `"2026-07-21T10:30:00.000Z"` |
| Source | Set by the producing module at the moment the state change is committed |
| Immutable | Never changes after creation |
| Business meaning | Represents the real-world moment the fact became true |

---

### 6. Publication Time

| Rule | Detail |
|------|--------|
| Field | `publishedAt` |
| Purpose | When the event was published to event infrastructure (delivered from outbox) |
| Format | ISO 8601 UTC with millisecond precision |
| Source | Set by the outbox processor at delivery time |
| Difference from occurrence | Publication may lag occurrence (outbox processing delay) |
| Consumer use | Consumers use `occurredAt` for business logic. `publishedAt` is for operational monitoring (lag detection). |

---

### 7. Producer

| Rule | Detail |
|------|--------|
| Field | `producer` |
| Purpose | Identifies which module published this event |
| Format | Module identifier string (e.g., `"order-module"`, `"payment-module"`) |
| Routing | May be used for diagnostic routing or filtering |
| Ownership | Confirms the module that owns this event type |
| Not consumer-actionable | Consumers should not branch logic based on producer (couple to type, not producer) |

---

### 8. Tenant Context

**Tenant context must be explicit where applicable.**

| Rule | Detail |
|------|--------|
| Field | `tenantId` |
| Purpose | Identifies which organization this event belongs to |
| Required | Mandatory for all tenant-scoped events (which is nearly all business events) |
| Format | Organization public identifier |
| Consumer scoping | Consumers process within this tenant context. Must not affect other tenants. |
| Platform events | Platform-level events (not tenant-specific) may have `tenantId: null` — explicitly documented |

---

### 9. Organization Context

| Rule | Detail |
|------|--------|
| Relationship | `tenantId` IS the organization identifier in Kairo's model |
| Clarity | Tenant = Organization as the primary boundary |
| Always present | For business events involving organization-owned data |

---

### 10. Store Context

| Rule | Detail |
|------|--------|
| Field | `storeId` |
| Purpose | Identifies which store within the organization (when applicable) |
| Optional | Not all events are store-scoped (organization-level events may omit) |
| Format | Store public identifier or `null` |
| Consumer use | Consumers may filter or route by store when relevant |

---

### 11. Correlation Identifiers

**Correlation and causation identifiers serve different purposes.**

| Rule | Detail |
|------|--------|
| Field | `correlationId` |
| Purpose | Links this event to the originating user request or business operation |
| Propagation | The same correlation ID flows through all events and operations triggered by a single user action |
| Format | String (same as the API request's `X-Correlation-Id`) |
| Tracing | Enables end-to-end tracing from user request through all resulting events and reactions |
| Example | User places order → API request has correlation ID → `order.placed` event carries it → payment, inventory, notification events all carry the same correlation ID |

---

### 12. Causation Identifiers

| Rule | Detail |
|------|--------|
| Field | `causationId` |
| Purpose | Identifies the specific event or command that directly caused this event |
| Relationship | Correlation = "what user operation started this chain." Causation = "what specific predecessor produced this event." |
| Format | Event ID of the causing event, or command ID of the causing command |
| Ancestry | Enables event chain reconstruction (A caused B, B caused C) |
| Example | `order.placed` → causes → `payment.authorized` (causationId = order.placed event ID) → causes → `payment.captured` (causationId = payment.authorized event ID) |

---

### 13. Trace Context

| Rule | Detail |
|------|--------|
| Field | `traceId` |
| Purpose | Distributed tracing identifier for operational observability |
| Format | Standard trace format (W3C Trace Context compatible direction) |
| Propagation | Flows from API request through event publication and consumption |
| Operational | Used by monitoring/tracing infrastructure, not by business logic |
| Separate from correlation | Trace is operational. Correlation is business-level. They may coincide but serve different purposes. |

---

### 14. Subject or Resource Reference

| Rule | Detail |
|------|--------|
| Field | `subject` (object with `type` and `id`) |
| Purpose | Identifies the primary business resource this event is about |
| Type | The resource type: `"order"`, `"product"`, `"customer"`, `"payment"` |
| ID | The resource's public identifier |
| Routing | Enables per-resource routing, ordering, and consumer filtering |
| Distinct from event ID | Event ID identifies the event. Subject ID identifies the business entity. |

---

### 15. Payload

**Event payloads include only data needed by intended consumers.**
**Events must not expose persistence entities directly.**

| Rule | Detail |
|------|--------|
| Field | `data` |
| Purpose | Domain-specific content meaningful to consumers |
| Consumer-oriented | Designed for what consumers need to react, not for what the database stores |
| Minimal | Include only what consumers reasonably need. Not full entity dumps. |
| Typed | Each event type defines its specific `data` shape |
| No internal fields | No database IDs, no audit columns, no ORM metadata, no internal flags |
| References over copies | Where full data is not needed, include resource ID (consumer fetches via API if needed) |

---

### 16. Metadata

**Metadata must not become an uncontrolled dumping ground.**

| Rule | Detail |
|------|--------|
| Field | `metadata` |
| Purpose | Operational and governance information about the event itself (not about the business entity) |
| Controlled | Only platform-defined metadata fields. Modules cannot arbitrarily extend metadata. |
| Fields | `schemaVersion`, `dataClassification`, and platform-defined operational fields |
| Not a bag | Metadata is not a generic key-value store for arbitrary data |
| Consumer-irrelevant | Consumers should not build business logic from metadata fields (use `data` for business content) |

---

## Payload Conventions

### 17. Data Classification

**Sensitive data must be minimized.**

| Rule | Detail |
|------|--------|
| Classification declared | `metadata.dataClassification` indicates the highest classification of data in the payload |
| Minimization | Payloads carry the minimum sensitive data needed for consumers to react |
| No secrets | Events never contain passwords, tokens, API keys, or credentials |
| PII minimized | Customer IDs preferred over full names/emails. Full PII only when consumers genuinely need it. |
| Financial minimized | Order total may be included. Full payment card details never. |
| Follows rules | Per [Data Classification and Sensitivity](../Data/Data-Classification-and-Sensitivity.md) |

---

### 18. Serialization Direction

| Rule | Detail |
|------|--------|
| Format | JSON (UTF-8) for V1 |
| Compatibility | JSON is human-readable, widely supported, and sufficient for V1 event sizes |
| Future | Binary formats (Avro, Protobuf) evaluated for V2+ if event volume requires it |
| Encoding | Standard JSON serialization rules. No custom binary encoding in V1. |
| Schema | Event schemas are documented and testable. Formal schema language (JSON Schema) direction for V2+. |

---

### 19. Nullability

| Rule | Detail |
|------|--------|
| Null = explicitly absent | A field with `null` means "this value is intentionally empty for this event" |
| Absent = not applicable | A field not present means "not relevant for this event instance" |
| Required envelope fields | Envelope fields (`id`, `type`, `version`, `occurredAt`, `producer`, `tenantId`) are never null |
| Optional envelope fields | `storeId`, `causationId` may be null where not applicable |
| Payload nullability | Defined per event type schema. Required payload fields must have values. |
| Empty collections | Empty arrays `[]` not `null` for collection fields with no items |

---

### 20. Enumerations

| Rule | Detail |
|------|--------|
| Representation | String values (lowercase_snake_case) |
| Extensible | Consumers must handle unknown enum values gracefully |
| Adding values | Non-breaking (consumers ignore unknown) |
| Removing values | Breaking (requires schema version bump) |
| Documentation | All valid values documented per field |
| Consistent with API | Same enum conventions as [API Contract Standards](../API/API-Contract-Standards.md) |

---

### 21. Monetary Values

**Monetary values require currency context.**

| Rule | Detail |
|------|--------|
| Structure | `{ "amount": "29.99", "currency": "USD" }` |
| Amount type | String (preserves precision) |
| Currency | ISO 4217 three-letter code |
| Never bare | A monetary value without currency is never valid |
| Consistent with API | Same conventions as [API Contract Standards](../API/API-Contract-Standards.md) |

---

### 22. Dates and Time Zones

| Rule | Detail |
|------|--------|
| Format | ISO 8601: `"2026-07-21T10:30:00.000Z"` |
| Timezone | UTC always (suffix `Z`) |
| Precision | Milliseconds for event timestamps. Seconds acceptable in payload business fields. |
| Field naming | `*At` suffix for instants: `placedAt`, `capturedAt` |
| Consistent with API | Same conventions as [API Contract Standards](../API/API-Contract-Standards.md) |

---

### 23. Identifiers

| Rule | Detail |
|------|--------|
| Format | String. Public identifiers per [Identifier Strategy](../Data/Identifier-Strategy.md). |
| Prefix | Resource type prefix where applicable: `order_`, `prod_`, `cust_` |
| No internal IDs | Database surrogate keys never appear in event payloads |
| Consistent | Same identifier format as API responses (consumer can use the ID to call the API) |
| Event ID is distinct | The event's own ID (`evt_...`) is separate from any business resource ID in the payload |

---

### 24. Optional Fields

| Rule | Detail |
|------|--------|
| Documented | Per event type schema, each field is documented as required or optional |
| Consumer resilience | Consumers must handle optional fields being absent gracefully |
| No silent semantics | An optional field's absence must not carry implicit business meaning (if it does, it is not truly optional) |
| Adding optional fields | Non-breaking change (consumers ignore unknown fields) |

---

### 25. Deprecated Fields

| Rule | Detail |
|------|--------|
| Marked | Deprecated fields are marked in schema documentation |
| Continued inclusion | Deprecated fields continue to be included during deprecation period |
| Migration guidance | Documentation indicates what replaces the deprecated field |
| Removal | Field removal only in a new schema version |
| Consumer notice | Consumers are notified of deprecation through event catalog / changelog |

---

### 26. Unknown Fields

**Consumers must not depend on undocumented fields.**

| Rule | Detail |
|------|--------|
| Consumer tolerance | Consumers must ignore unknown fields in payloads (forward compatibility) |
| No undocumented dependence | A field that is not in the schema documentation is not a contract. Consumers that depend on it do so at their own risk. |
| Schema is contract | Only documented schema fields are guaranteed to exist and maintain meaning |
| Producer freedom | Producers may add new fields without consumer breakage (consumers ignore them until documented) |

---

### 27. Contract Validation

| Rule | Detail |
|------|--------|
| Producer validates | Events are validated against their schema before publication |
| Required fields | Missing required fields prevent publication (programming error, not runtime flexibility) |
| Type correctness | Field types match schema (string for money, ISO format for dates) |
| Structural correctness | Event envelope structure is validated by infrastructure |
| Payload validation | Payload structure is validated by the producing module |

---

### 28. Schema Documentation

| Rule | Detail |
|------|--------|
| Every event type documented | Type name, schema version, all fields, field types, required/optional, description |
| Event catalog | All published integration events are listed in an event catalog |
| Consumer guidance | Documentation includes which consumers are expected and what reactions are typical |
| Change history | Schema changes are documented (same as API changelog) |
| V1 direction | Documentation in architecture repository (markdown). Formal schema registry V2+. |

---

### 29. Contract Testing

| Rule | Detail |
|------|--------|
| Producer tests | Tests verify that published events match the documented schema |
| Consumer tests | Tests verify that consumers handle events correctly (including unknown fields, deprecated fields) |
| Compatibility tests | Tests verify that new schema versions remain compatible with documented consumers |
| CI integration | Contract tests run in CI. Schema violations fail the build. |
| Forward compatibility | Tests verify consumers do not break when new fields are added |

---

## Payload Style Evaluation

| Style | Description | When Appropriate | When Not Appropriate |
|-------|-------------|-----------------|---------------------|
| **Full-state** | Payload contains the complete current state of the resource | Resource is small. Most consumers need most fields. Avoids API callback. | Resource is large. Creates sensitivity issues. Creates tight coupling to producer's model. |
| **Change event** | Payload contains only what changed (before/after or delta) | Consumers need to know specifically what changed (audit, differential sync). | Most consumers need current state regardless of what changed. Complex to process. |
| **Reference-only** | Payload contains only the resource ID and event type. Consumer fetches data via API. | Resource is large or sensitive. Consumer needs latest state (not state at event time). | Consumer needs point-in-time data. API call adds latency and coupling. |
| **Hybrid** | Payload contains key summary fields + resource ID for full fetch if needed. | Balance between self-sufficiency and minimalism. Most common. | When full state or reference-only is clearly more appropriate. |

### V1 Recommendation

| Direction | Detail |
|-----------|--------|
| **Hybrid as default** | Include key routing/decision fields + resource ID. Consumer has enough to decide whether to fetch more. |
| Full-state for small resources | Where the resource is small and most consumers need most fields (e.g., inventory level change). |
| Reference-only for sensitive | Where the payload would contain extensive PII or financial data. Consumer fetches via authorized API. |
| Change events for audit | Audit events may use change format (before/after values for compliance records). Not integration events. |

### Conceptual Examples (non-binding)

**Hybrid (recommended default):**
```json
{
  "data": {
    "orderId": "order_01H5NNN",
    "customerId": "cust_01H2RRR",
    "status": "pending",
    "totalAmount": { "amount": "129.99", "currency": "USD" },
    "itemCount": 3
  }
}
```
Consumer knows: which order, which customer, status, total. Can decide whether to fetch line items via API.

**Full-state (small resource):**
```json
{
  "data": {
    "sku": "WDG-001",
    "productId": "prod_01H5PPP",
    "locationId": "loc_01H4QQQ",
    "previousQuantity": 50,
    "newQuantity": 45,
    "adjustmentDelta": -5,
    "reason": "sale"
  }
}
```
Consumer has everything needed. Resource is small. No API callback necessary.

**Reference-only (sensitive):**
```json
{
  "data": {
    "customerId": "cust_01H2RRR",
    "registeredAt": "2026-07-21T10:30:00Z"
  }
}
```
Consumer knows a customer registered. Must call Customer API with authorization to get profile details (PII).

---

## Version Gate

| Version | Event Contract Standards Gate |
|---------|------------------------------|
| V1 | All integration events use standard envelope (id, type, version, occurredAt, publishedAt, producer, tenantId, correlationId, subject, data, metadata). Hybrid payload style as default. JSON serialization. UTC timestamps. String identifiers. Money as object. Schema version present on all events. Consumers ignore unknown fields. Contract tests in CI. Event types documented. |
| V2 | Formal JSON Schema for all integration events. Schema registry operational. Automated compatibility checking. Change events for select use cases. Event catalog published to developers. |
| V3 | Binary serialization option (Avro/Protobuf) for high-volume events. Consumer-driven contract tests. Schema evolution automation. |

---

## Decision Summary

| Decision | Rationale |
|----------|-----------|
| Hybrid payload as default style | Balances self-sufficiency (consumer has enough to decide) with minimalism (not full entity dump). Practical for most use cases. |
| Separate envelope from payload | Envelope is standard (infrastructure can process without understanding business). Payload is domain-specific (varies per event type). |
| Event ID distinct from resource ID | An order may generate many events (placed, paid, shipped). Each event needs its own identity. The order has one identity. |
| Occurrence time vs publication time | Business logic cares about when the fact happened. Operations cares about delivery lag. Different purposes. |
| Correlation vs causation | Correlation traces the entire user operation chain. Causation traces the immediate predecessor. Both are needed for different debugging scenarios. |
| JSON for V1 | Human-readable, widely tooled, sufficient for V1 event volumes. Binary is premature optimization. |
| Schema version from day one | Adding versioning later requires coordinating all consumers. Including it from V1 (even with one version) enables seamless future evolution. |
| Metadata is controlled | Uncontrolled metadata becomes a dumping ground for undocumented fields that consumers secretly depend on. Controlled set prevents this. |

---

## Alternatives Considered

| Alternative | Rejected Because |
|------------|-----------------|
| Full-state as default | Creates large events with sensitive data. Couples consumers to producer's full model. Updates to model break all events. |
| Reference-only as default | Forces every consumer to make API calls. Adds latency. Creates back-pressure on API. Loses point-in-time context. |
| No envelope (flat structure) | Infrastructure cannot process events generically (routing, filtering, tracing) without understanding each event type. Envelope enables generic handling. |
| Auto-incrementing event IDs | Not globally unique. Requires coordination. ULID-style is coordination-free and unique. |
| Local timestamps (not UTC) | Ambiguous across time zones. UTC eliminates confusion. Consumers convert for display. |
| Metadata as extensible key-value | Becomes uncontrolled. Consumers depend on undocumented fields. Governance impossible. Controlled set is safer. |
| Binary serialization in V1 | Premature. Adds complexity (schema registry required, debugging harder). JSON is sufficient for V1 volumes. |
| No schema version field | Migration later requires coordinating all producers and consumers. Including version from day one is trivial and enables future evolution. |

---

## Architecture Impact

| Concern | Impact |
|---------|--------|
| Module design | Modules must define their integration event schemas (envelope + payload). Must validate events before publication. Must document all event types. |
| Event infrastructure | Must understand envelope structure for routing (type field), tracing (correlation/trace IDs), and monitoring (timestamps). |
| Consumer design | Consumers handle events by type. Must ignore unknown fields. Must use event ID for deduplication. |
| Observability | Correlation and trace IDs enable end-to-end tracing from API request through all resulting events. |
| Testing | Contract tests verify schema compliance. Forward-compatibility tests verify consumers handle evolution. |
| SDK/tooling direction | Event envelope structure enables generic producer/consumer libraries that handle envelope, letting modules focus on payload. |

---

## Implementation Impact

| Area | Impact |
|------|--------|
| Modules | Must produce events conforming to the standard envelope. Must define typed payloads per event type. Must validate before publication. Must document schemas. |
| Platform | Must provide envelope construction utilities. Must validate envelope structure. Must route by type. Must propagate correlation/trace context. |
| Consumers | Must parse envelope generically. Must handle payload per type. Must ignore unknown fields. Must deduplicate by event ID. |
| Testing | Must test envelope compliance (all required fields present, correct formats). Must test payload per type. Must test forward compatibility. |
| Documentation | Every integration event type must be documented: type name, version, all fields, required/optional, description, example. |

---

## Security Responsibilities

| Role | Contract Standards Responsibilities |
|------|-----------------------------------|
| Event Contract Architect | Defines and evolves contract standards. Reviews new event schemas. Validates classification compliance. |
| Module Teams | Design event payloads following standards. Validate before publication. Document schemas. Minimize sensitive data. |
| Platform Team | Provides envelope utilities. Validates envelope structure. Routes by type. Propagates context identifiers. |
| Security Team | Reviews event payloads for sensitive data leakage. Validates classification metadata accuracy. Ensures no credentials in events. |

---

## Multi-Tenancy Responsibilities

| Responsibility | Detail |
|---------------|--------|
| Explicit tenant ID | `tenantId` field required in envelope for all tenant-scoped events |
| Consumer scoping | Consumers process within the event's declared tenant context |
| No cross-tenant confusion | Event infrastructure ensures events are not misrouted to wrong tenant's consumers |
| Platform events marked | Events without tenant context (platform-level) are explicitly documented as such |

---

## Out of Scope

This document does not define:

- Specific event type definitions per module (module specifications).
- Serialization library selection (development standards).
- JSON Schema definitions for specific events (module specifications, V2+).
- Event broker message format (infrastructure documentation).
- Consumer deserialization implementation (development standards).
- Outbox storage format (module implementation detail).

---

## Future Considerations

- **Formal JSON Schema** — Machine-readable schema definitions for all integration events (V2).
- **Schema registry** — Centralized registry with compatibility validation.
- **Binary serialization** — Avro or Protobuf for high-volume event types.
- **Consumer-driven contract tests** — Consumers define their expectations, producers verify.
- **Event catalog UI** — Searchable catalog of all event types with schemas and documentation.
- **Schema evolution automation** — Tooling that validates compatibility on schema changes.
- **Event versioning migration** — Infrastructure support for multi-version events during transitions.

---

## Future Refactoring Triggers

This document should be revisited when:

- Event volume makes JSON serialization a performance bottleneck (trigger for binary format evaluation).
- Schema registry is deployed (trigger for formal schema language adoption).
- New payload style needs emerge (trigger for style guidance expansion).
- Consumer-driven testing is adopted (trigger for contract testing update).
- Cross-service events require explicit security context in envelope (trigger for envelope extension).
- Event types exceed manageable documentation scope (trigger for catalog tooling).

---

## Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 0.1 | 2026-07-21 | Event Contract and Schema Governance Architect | Initial draft — event contract standards |
