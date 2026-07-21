# Event Taxonomy and Ownership

## Metadata

| Field | Value |
|-------|-------|
| Title | Kairo Event Taxonomy, Categories, and Ownership Model |
| Document ID | KAI-EVT-002 |
| Status | Draft |
| Version | 0.1 |
| Target Release | V1 |
| Owner | Domain Event Taxonomy and Ownership Architect |
| Created | 2026-07-21 |
| Last Updated | 2026-07-21 |
| Reviewers | TODO |
| Related Documents | [Event Architecture](./Event-Architecture.md), [Bounded Contexts](../../03-Business-Capabilities/Bounded-Contexts.md), [Data Ownership](../Data/Data-Ownership.md), [Audit and Security Monitoring](../Security/Audit-and-Security-Monitoring.md), [Webhook Architecture](../API/Webhook-Architecture.md), [Module Architecture](../Module-Architecture.md), [Data Classification and Sensitivity](../Data/Data-Classification-and-Sensitivity.md) |
| Dependencies | [Event Architecture](./Event-Architecture.md), [Bounded Contexts](../../03-Business-Capabilities/Bounded-Contexts.md), [Data Ownership](../Data/Data-Ownership.md) |

---

## Applicable Version

This document defines the V1 event taxonomy. It establishes categories, ownership rules, and usage boundaries for all event types in the platform. V1 implements domain events and integration events. Other categories are identified for V2+ or exist as infrastructure concerns separate from domain event architecture.

---

## Purpose

This document defines what kinds of events exist in the Kairo platform, who owns them, who may consume them, and what rules govern each category. It establishes the taxonomy that prevents conflation between business facts, operational telemetry, audit records, and external notifications.

Without a clear taxonomy, platforms evolve toward a single undifferentiated "event" concept that serves every purpose poorly. Business events become polluted with operational noise. Audit records become treated as integration events. Analytics consumers become accidental sources of truth. This document prevents all of these.

---

## Scope

This document covers:

- Event category definitions and their distinct purposes.
- Ownership model per category.
- 10 aspects documented per category (purpose, owner, consumers, scope, durability, compatibility, tenant context, sensitive data, audit, V1 status).
- Relationship between commands, events, requests, and responses.
- Conceptual examples from core business domains.
- Matrices mapping categories to governance dimensions.

This document does not cover:

- Specific event names or schemas per module (module specifications).
- Event infrastructure technology (infrastructure documentation).
- Event handler implementation (development standards).
- Webhook payload formats (see [Webhook Architecture](../API/Webhook-Architecture.md)).
- Audit event schema (see [Audit and Security Monitoring](../Security/Audit-and-Security-Monitoring.md)).

---

## Mandatory Principles

| # | Principle |
|---|-----------|
| 1 | Commands request actions; events describe completed facts |
| 2 | Domain events primarily model behavior inside an owned domain boundary |
| 3 | Integration events communicate approved facts across boundaries |
| 4 | Business audit events are not automatically integration events |
| 5 | Operational telemetry is not a business event |
| 6 | Analytics events must not become sources of truth |
| 7 | Webhooks are externally delivered integration notifications, not internal domain ownership |
| 8 | Event ownership belongs to the capability that owns the fact |
| 9 | Consumers do not gain mutation rights over producer-owned data |
| 10 | Shared infrastructure does not own business-event meaning |

---

## Fundamental Distinctions

### Commands versus Events versus Requests versus Responses

| Concept | Direction | Timing | Semantics | Example |
|---------|-----------|--------|-----------|---------|
| **Command** | Caller → handler | Before action | "Please do this" | `PlaceOrder`, `CapturePayment` |
| **Event** | Producer → subscribers | After action | "This happened" | `OrderPlaced`, `PaymentCaptured` |
| **Request** | Client → server | Before processing | "I need this data/action" | HTTP GET /products, HTTP POST /orders |
| **Response** | Server → client | After processing | "Here is the result" | 200 OK + product data, 201 Created + order |

**Commands request actions; events describe completed facts.**

| Commands | Events |
|----------|--------|
| Imperative ("Do X") | Declarative ("X happened") |
| May be rejected | Cannot be rejected (already occurred) |
| Addressed to specific handler | Published to all interested subscribers |
| Expects a result | Does not expect a response |
| Coupling: caller knows handler | Decoupling: producer does not know consumers |
| One handler processes | Many consumers may react |

---

## Event Categories

### 1. Domain Events

| Aspect | Detail |
|--------|--------|
| **Purpose** | Model business-significant state changes within a bounded context |
| **Owner** | The module that owns the business capability and the state change |
| **Intended consumers** | Same module's internal handlers (projections, side effects within the domain) |
| **Allowed scope** | Within the owning module boundary. Not directly consumed across modules. |
| **Durability expectations** | Transient — processed in-memory or through outbox. Not permanently stored (unless event-sourced). |
| **Compatibility expectations** | Internal — may change freely as the module evolves (no external consumers) |
| **Tenant-context requirements** | Required for tenant-scoped entities |
| **Sensitive-data restrictions** | May include internal detail (stays within module boundary) |
| **Audit requirements** | Not directly audited (the state change they represent is audited) |
| **V1 or future status** | **V1** |

**Domain events primarily model behavior inside an owned domain boundary.**

---

### 2. Integration Events

| Aspect | Detail |
|--------|--------|
| **Purpose** | Communicate approved, significant facts across module boundaries |
| **Owner** | The producing module (owns the fact being communicated) |
| **Intended consumers** | Other modules that need to react to cross-module state changes |
| **Allowed scope** | Cross-module within the platform. Published through event infrastructure. |
| **Durability expectations** | Durable — persisted in outbox, delivered at-least-once, dead-lettered on failure |
| **Compatibility expectations** | Stable — additive compatible. Schema versioned. Breaking changes require governance. |
| **Tenant-context requirements** | Required (explicit tenant ID in event envelope) |
| **Sensitive-data restrictions** | Minimized — resource IDs preferred over full data. No unnecessary PII. Classification rules apply. |
| **Audit requirements** | Publication is traced. Consumption is traced. Failures are investigated. |
| **V1 or future status** | **V1** |

**Integration events communicate approved facts across boundaries.**

---

### 3. Application Events

| Aspect | Detail |
|--------|--------|
| **Purpose** | Coordinate application-level workflow within a module (not domain-meaningful, but operationally necessary) |
| **Owner** | The module's application layer |
| **Intended consumers** | Same module's application services (workflow coordination, process managers) |
| **Allowed scope** | Within the owning module |
| **Durability expectations** | Process-level — may be lost on process restart without business harm (retry-safe) |
| **Compatibility expectations** | Internal — evolves freely |
| **Tenant-context requirements** | Inherited from the originating operation |
| **Sensitive-data restrictions** | Internal handling — same as domain events |
| **Audit requirements** | Not individually audited |
| **V1 or future status** | **V1** (implicit in module design, not a distinct infrastructure concern) |

---

### 4. Internal Notifications

| Aspect | Detail |
|--------|--------|
| **Purpose** | Trigger side effects within the platform (send email, update search index, invalidate cache) |
| **Owner** | The module or platform service that handles the notification |
| **Intended consumers** | Notification service, search indexer, cache service, analytics |
| **Allowed scope** | Platform-internal. Triggered by integration events or domain events. |
| **Durability expectations** | At-least-once delivery (same as integration events) |
| **Compatibility expectations** | Moderate — consumers are internal, coordinated deployment |
| **Tenant-context requirements** | Required (notification is for a specific tenant's data) |
| **Sensitive-data restrictions** | Minimal — notification carries intent + reference, not full content |
| **Audit requirements** | Notification delivery may be audited (email sent, SMS sent) |
| **V1 or future status** | **V1** (subset handled as integration event consumers) |

---

### 5. Business Audit Events

| Aspect | Detail |
|--------|--------|
| **Purpose** | Record who did what, when, and in what context for compliance and accountability |
| **Owner** | Audit service (infrastructure-owned capture, but meaning belongs to the originating domain) |
| **Intended consumers** | Audit storage, compliance systems, security monitoring |
| **Allowed scope** | Platform audit infrastructure. Not consumed as integration events by business modules. |
| **Durability expectations** | Permanent — retained per compliance requirements. Never lost. |
| **Compatibility expectations** | Stable — audit format must be consistent for reporting and compliance |
| **Tenant-context requirements** | Required (who acted within which organization) |
| **Sensitive-data restrictions** | Records what changed (before/after) but follows classification for PII masking in audit views |
| **Audit requirements** | Self-auditing (the audit system's integrity is separately assured) |
| **V1 or future status** | **V1** |

**Business audit events are not automatically integration events.** An audit record of "product price changed" is for compliance. The integration event "ProductPriceUpdated" is for business coordination. They have different consumers, retention, and purposes.

---

### 6. Security Events

| Aspect | Detail |
|--------|--------|
| **Purpose** | Record security-significant occurrences for monitoring, alerting, and investigation |
| **Owner** | Security infrastructure and monitoring systems |
| **Intended consumers** | Security monitoring, SIEM, incident response, alerting systems |
| **Allowed scope** | Security infrastructure. Not consumed by business modules. |
| **Durability expectations** | Long-term retention for investigation and compliance |
| **Compatibility expectations** | Stable — security tools depend on consistent format |
| **Tenant-context requirements** | When applicable (authentication failures for a specific tenant vs. platform-level attacks) |
| **Sensitive-data restrictions** | Contains security context but not credentials, tokens, or passwords |
| **Audit requirements** | Security events are audit events (subset of audit with security focus) |
| **V1 or future status** | **V1** (via [Audit and Security Monitoring](../Security/Audit-and-Security-Monitoring.md)) |

---

### 7. Operational Events

| Aspect | Detail |
|--------|--------|
| **Purpose** | Infrastructure health, performance, and operational visibility |
| **Owner** | Platform operations infrastructure |
| **Intended consumers** | Monitoring systems, alerting, operations dashboards |
| **Allowed scope** | Operations infrastructure. Not consumed by business modules. |
| **Durability expectations** | Short to medium retention (days to weeks for most metrics) |
| **Compatibility expectations** | Low — operational telemetry evolves with infrastructure changes |
| **Tenant-context requirements** | May be tenant-attributed (per-tenant resource usage) or platform-wide |
| **Sensitive-data restrictions** | No business data. No PII. Operational metrics only. |
| **Audit requirements** | Not audited (they are the operational record, not business decisions) |
| **V1 or future status** | **V1** (infrastructure concern, separate from domain event architecture) |

**Operational telemetry is not a business event.**

---

### 8. Analytics Events

| Aspect | Detail |
|--------|--------|
| **Purpose** | Capture behavioral data for business intelligence and reporting |
| **Owner** | Analytics infrastructure (meaning defined by product/business teams) |
| **Intended consumers** | Analytics pipelines, reporting systems, data warehouse (future) |
| **Allowed scope** | Analytics infrastructure. Read-only consumption of business signals. |
| **Durability expectations** | Medium to long retention depending on analytical value |
| **Compatibility expectations** | Moderate — analytics schemas evolve with business questions |
| **Tenant-context requirements** | Required for per-tenant analytics. De-identified for platform-wide analytics. |
| **Sensitive-data restrictions** | Must follow data classification. PII minimized or de-identified. |
| **Audit requirements** | Not individually audited |
| **V1 or future status** | **Future** (V2+ — V1 uses direct queries for analytics per [Reporting and Analytics Architecture](../Data/Reporting-and-Analytics-Architecture.md)) |

**Analytics events must not become sources of truth.** Analytics consumes data. It does not own it. If an analytics pipeline disagrees with the authoritative module, the module is correct.

---

### 9. Webhook Events

| Aspect | Detail |
|--------|--------|
| **Purpose** | Deliver platform events to external consumer endpoints |
| **Owner** | Platform webhook infrastructure (delivery). Module (event source and meaning). |
| **Intended consumers** | External applications, partner systems, customer integrations |
| **Allowed scope** | External — crosses the platform boundary to registered external endpoints |
| **Durability expectations** | At-least-once delivery attempt with retry and suspension |
| **Compatibility expectations** | High — external consumers cannot easily update. Versioned. Long deprecation. |
| **Tenant-context requirements** | Delivered to subscription owner's organization. Payload includes org/store context. |
| **Sensitive-data restrictions** | Minimized — summary information. Full data available via API. No secrets. |
| **Audit requirements** | Delivery logged. Receipt acknowledged. |
| **V1 or future status** | **V1** (via [Webhook Architecture](../API/Webhook-Architecture.md)) |

**Webhooks are externally delivered integration notifications, not internal domain ownership.** Webhook delivery is a notification mechanism. It does not transfer data ownership to the external consumer.

---

### 10–12. Commands, Requests, Responses

These are NOT events but are included for clarity of distinction:

| Category | Purpose | Relationship to Events |
|----------|---------|----------------------|
| **Commands** | Request that an action be performed | Command execution MAY produce events (if the action succeeds and is business-significant) |
| **Requests** | API-level interaction requesting data or action | A successful request MAY trigger event publication (the API handler processes → state changes → event published) |
| **Responses** | API-level reply to a request | Responses are synchronous feedback. Not events. Not published to subscribers. |

---

## Ownership Rules

**Event ownership belongs to the capability that owns the fact.**

| Rule | Detail |
|------|--------|
| Producer owns meaning | The module that produces the event defines what it means |
| Producer owns schema | The producing module defines and evolves the event schema |
| Producer owns lifecycle | The producer decides when to deprecate or retire an event type |
| **Consumers do not gain mutation rights over producer-owned data** | Consuming an event does not grant write access to the producer's data |
| **Shared infrastructure does not own business-event meaning** | The event bus, message broker, or outbox infrastructure does not define what an event means. It delivers; the domain owns meaning. |
| Consumer reacts independently | Each consumer decides its own reaction. The producer does not dictate consumer behavior. |
| No ownership transfer | After publication, the event is a notification. The source data remains owned by the producer. |

---

## Conceptual Examples

### Product Activation

| Aspect | Example |
|--------|---------|
| Command | `ActivateProduct` — request to make a draft product active |
| Domain event | Product state changes within Catalog module (internal, completeness validated) |
| Integration event | "Product became active" — published for cross-module consumers |
| Consumers | Search indexer (add to search), storefront cache (make available), analytics |
| Audit | "User X activated product Y at time T" |
| Webhook | `product.activated` delivered to external subscribers |

### Inventory Adjustment

| Aspect | Example |
|--------|---------|
| Command | `AdjustInventory` — request to adjust stock by a delta with reason |
| Domain event | Stock level changed within Inventory module (movement recorded) |
| Integration event | "Inventory adjusted" — published with SKU, delta, and new level |
| Consumers | Storefront availability (update display), alerts (low-stock notification) |
| Audit | "User X adjusted SKU Y by -5 (reason: damaged) at time T" |
| Webhook | `inventory.adjusted` delivered to external subscribers |

### Order Creation

| Aspect | Example |
|--------|---------|
| Command | `PlaceOrder` — customer completes checkout |
| Domain event | Order created within Order module (line items, totals, status: pending) |
| Integration event | "Order placed" — published with order ID, customer ID, total, item count |
| Consumers | Payment (initiate authorization), Inventory (reserve stock), Notification (confirmation email), Fulfillment (prepare) |
| Audit | "Customer X placed order Y for $Z at time T" |
| Webhook | `order.created` delivered to external subscribers |

### Payment Capture

| Aspect | Example |
|--------|---------|
| Command | `CapturePayment` — request to capture authorized funds |
| Domain event | Payment captured within Payment module (provider confirmed, amount recorded) |
| Integration event | "Payment captured" — published with payment ID, order ID, amount, provider reference |
| Consumers | Order (update status to paid), Finance (record revenue), Notification (receipt) |
| Audit | "System captured payment X for order Y, amount $Z at time T" |
| Webhook | `payment.captured` delivered to external subscribers |

### Refund Completion

| Aspect | Example |
|--------|---------|
| Command | `ProcessRefund` — initiate refund for a payment |
| Domain event | Refund completed within Payment module (provider confirmed return) |
| Integration event | "Refund completed" — published with refund ID, original payment ID, amount |
| Consumers | Order (update to refunded), Finance (record adjustment), Inventory (return stock if applicable), Notification (refund confirmation) |
| Audit | "User X initiated refund Y for payment Z, amount $A at time T" |
| Webhook | `refund.completed` delivered to external subscribers |

### Customer Registration

| Aspect | Example |
|--------|---------|
| Command | `RegisterCustomer` — customer creates an account |
| Domain event | Customer created within Customer module (profile established) |
| Integration event | "Customer registered" — published with customer ID, organization |
| Consumers | Notification (welcome email), Analytics (acquisition tracking) |
| Audit | "Customer X registered in organization Y at time T" |
| Webhook | `customer.registered` delivered to external subscribers |

### Tenant Suspension

| Aspect | Example |
|--------|---------|
| Command | `SuspendTenant` — platform operator suspends an organization |
| Domain event | Organization suspended within Tenant module (status changed, reason recorded) |
| Integration event | "Tenant suspended" — published with organization ID and reason |
| Consumers | All modules (cease active operations for this tenant), Notification (alert org admins) |
| Audit | "Platform operator X suspended organization Y for reason Z at time T" |
| Webhook | Not delivered (tenant is suspended — webhook subscriptions paused) |

---

## Event Category Matrix

| Category | Owner | Producer | Primary Consumers | Scope | Delivery | Compatibility | Classification | V1 |
|----------|-------|----------|-------------------|-------|----------|--------------|----------------|:---:|
| Domain events | Module (domain layer) | Module domain logic | Same module handlers | Within module | In-process | Internal (free change) | Internal | Yes |
| Integration events | Module (boundary) | Module outbox | Other modules | Cross-module | At-least-once (outbox) | Stable (versioned) | Minimized | Yes |
| Application events | Module (app layer) | Module app services | Same module workflows | Within module | Process-level | Internal | Internal | Yes |
| Internal notifications | Platform/module | Integration event consumers | Platform services | Platform-internal | At-least-once | Moderate | Minimized | Yes |
| Business audit events | Audit service | All modules (emit audit) | Audit storage, compliance | Audit infra | Permanent, guaranteed | Stable | Per-classification | Yes |
| Security events | Security infra | Platform security layer | SIEM, monitoring, alerting | Security infra | Guaranteed, long-term | Stable | No credentials | Yes |
| Operational events | Operations infra | Platform infrastructure | Monitoring, dashboards | Operations | Best-effort, short-term | Low | No business data | Yes |
| Analytics events | Analytics infra | Business modules | Analytics pipeline | Analytics infra | At-least-once | Moderate | De-identified where needed | Future |
| Webhook events | Platform + module | Webhook delivery infra | External applications | External | At-least-once (HMAC signed) | High (versioned, long deprecation) | Public-safe, minimized | Yes |

---

## Security Classification Matrix

| Category | May Contain PII | May Contain Financial | May Contain Credentials | Data Minimization Required |
|----------|:---:|:---:|:---:|:---:|
| Domain events | Yes (within module) | Yes (within module) | No | No (internal) |
| Integration events | Minimal | Reference only | No | **Yes** |
| Application events | Yes (within module) | Yes (within module) | No | No (internal) |
| Internal notifications | Minimal | No | No | **Yes** |
| Business audit events | Yes (who + what changed) | Yes (what amounts changed) | No | Classification-driven |
| Security events | No | No | **Never** | Yes |
| Operational events | No | No | No | Yes |
| Analytics events | De-identified | Aggregated | No | **Yes** |
| Webhook events | Minimal | Summary only | **Never** | **Yes** |

---

## Delivery Expectations Matrix

| Category | Delivery Guarantee | Retry on Failure | Dead-Letter | Ordering | Max Latency |
|----------|-------------------|:---:|:---:|----------|-------------|
| Domain events | In-process delivery | N/A (synchronous) | N/A | Per-aggregate | Immediate |
| Integration events | At-least-once | Yes | Yes | Per-aggregate (best-effort) | Seconds |
| Application events | Process-level | Limited | No | Within process | Immediate |
| Internal notifications | At-least-once | Yes | Yes | Not guaranteed | Seconds-minutes |
| Business audit events | Guaranteed (never lost) | Yes (indefinite) | Separate investigation | Chronological | Seconds |
| Security events | Guaranteed | Yes | Alert on failure | Chronological | Seconds |
| Operational events | Best-effort | No | No | Not guaranteed | Real-time |
| Analytics events | At-least-once | Yes | Skip on failure | Not required | Minutes-hours |
| Webhook events | At-least-once | Yes (exponential backoff) | Suspension | Best-effort chronological | Seconds-minutes |

---

## Version Gate

| Version | Event Taxonomy Gate |
|---------|---------------------|
| V1 | Domain events within modules. Integration events across modules (outbox + in-process delivery). Business audit events captured. Security events emitted. Operational metrics via standard infrastructure (Prometheus). Webhook events delivered via webhook infrastructure. Event categories clearly separated. Ownership assigned per module. Integration events have versioned schemas. Consumers are idempotent. |
| V2 | Analytics event pipeline operational. Event catalog published. Schema registry evaluated. Enhanced dead-letter management. Event replay for new consumers. |
| V3 | Event streaming for external consumers. CQRS projections at scale. Saga orchestration through events. Cross-service event delivery (extracted modules). |

---

## Decision Summary

| Decision | Rationale |
|----------|-----------|
| Distinct taxonomy (not one generic "event" type) | Different categories have different owners, consumers, durability, and sensitivity. Treating them uniformly leads to incorrect handling. |
| Domain events internal to module | Module autonomy. Internal domain modeling evolves freely without affecting external consumers. |
| Integration events as the cross-module boundary | Clean separation: modules share facts through governed, versioned integration events — not through internal domain events. |
| Audit events separate from integration events | Different retention (permanent vs. TTL), different consumers (compliance vs. business logic), different sensitivity. Conflating them creates problems in both directions. |
| Analytics events deferred to V2+ | V1 uses direct queries for analytics. Event-driven analytics pipeline adds infrastructure complexity not needed at V1 scale. |
| Webhooks as delivery mechanism, not ownership | An external consumer receiving a webhook notification does not own the data. The producing module remains authoritative. |
| Producer owns meaning (not infrastructure) | The event bus routes messages. The domain defines what "OrderPlaced" means. Infrastructure does not assign business semantics. |

---

## Alternatives Considered

| Alternative | Rejected Because |
|------------|-----------------|
| Single undifferentiated "event" type | Different categories have fundamentally different requirements (retention, sensitivity, consumers, compatibility). Treating all events the same creates incorrect handling. |
| Domain events exposed directly across modules | Couples consumers to internal module structure. Any internal refactoring breaks external consumers. Integration events are a stable boundary. |
| Audit events as a subset of integration events | Audit has permanent retention, compliance requirements, and restricted access. Integration events have TTL and business consumers. Different lifecycle. |
| Analytics events as source of truth | Analytics consumes and transforms. If it disagrees with the authoritative module, the module is correct. Analytics cannot own data. |
| Consumer-defined event contracts | Producer must be free to evolve internally. Consumer-defined contracts create reverse coupling (producer must satisfy all consumers). |
| Infrastructure owns event semantics | A message broker routes. It does not define "what is an order" or "what does payment captured mean." Business ownership prevents drift. |
| CDC (database change capture) as integration events | Not all DB changes are business events. CDC couples to schema. Domain-driven integration events are intentional and meaningful. |

---

## Architecture Impact

| Concern | Impact |
|---------|--------|
| Module design | Modules must distinguish internal domain events from published integration events. Must define integration event schemas. |
| Event infrastructure | Must support both in-process (domain) and outbox-based (integration) event patterns. Must route to correct consumer category. |
| Audit infrastructure | Must capture business audit events separately from integration events. Different retention and access. |
| Security | Security events feed monitoring. Business events carry tenant context. Neither carries credentials. |
| Observability | Each category has appropriate observability (business tracing for integration, metrics for operational, alerting for security). |
| Webhook integration | Integration events trigger webhook evaluation. Webhook events are derived from integration events, not identical to them. |

---

## Implementation Impact

| Area | Impact |
|------|--------|
| Modules | Must define domain events (internal). Must define integration events (published). Must emit audit events. Must implement idempotent consumers for integration events they subscribe to. |
| Platform | Must provide event infrastructure supporting both internal and outbox-based patterns. Must route audit events to audit infrastructure. Must route security events to monitoring. |
| Audit Service | Must capture, store, and serve business audit events. Must handle classification for viewing. |
| Security Monitoring | Must consume security events. Must alert on anomalies. Must retain for investigation. |
| Operations | Must capture operational metrics. Must not conflate operational telemetry with business events. |
| Webhooks | Must evaluate integration events against webhook subscriptions. Must deliver matching events to external endpoints. |

---

## Security Responsibilities

| Role | Taxonomy Responsibilities |
|------|--------------------------|
| Event Taxonomy Architect | Defines categories and ownership rules. Reviews new event types for correct categorization. Validates boundary compliance. |
| Module Teams | Categorize their events correctly (domain vs integration). Ensure integration events follow data minimization. Emit audit events for significant actions. |
| Platform Team | Provides infrastructure for each category (bus, outbox, audit capture, security event routing). Enforces category separation. |
| Security Team | Reviews integration event payloads for sensitive data leakage. Validates security events are comprehensive. Reviews webhook payload safety. |
| Operations | Manages operational telemetry separately from business events. Monitors event health per category. |

---

## Multi-Tenancy Responsibilities

| Responsibility | Detail |
|---------------|--------|
| Tenant context in integration events | Explicit organization ID in envelope for all tenant-scoped events |
| Tenant context in audit events | Records which tenant the action was performed within |
| No cross-tenant event leakage | Integration events for Tenant A are never delivered to Tenant B's consumers |
| Platform events distinguished | Events that are platform-wide (not tenant-specific) are clearly marked |
| Webhook tenant scoping | Webhook events delivered only to the owning tenant's subscriptions |

---

## Out of Scope

This document does not define:

- Specific event type names or schemas (module specifications).
- Event broker or queue technology (infrastructure documentation).
- Event handler implementation code (development standards).
- Audit event schema format (see [Audit and Security Monitoring](../Security/Audit-and-Security-Monitoring.md)).
- Webhook payload format (see [Webhook Architecture](../API/Webhook-Architecture.md)).
- Analytics pipeline technology (future infrastructure decision).
- Security monitoring tools or SIEM configuration (security operations).

---

## Future Considerations

- **Event catalog** — Published registry of all integration event types with schemas and consumer documentation.
- **Schema registry** — Centralized compatibility validation for event schemas.
- **Analytics event pipeline** — Dedicated infrastructure for behavioral analytics events (V2+).
- **Cross-service event security** — Authentication and authorization for events crossing service boundaries.
- **Event lineage** — Tracking how events flow from producer through infrastructure to all consumers.
- **Event replay** — Ability to replay historical events for new consumers or recovery.
- **Complex event processing** — Detecting patterns across multiple events (fraud detection, anomaly detection).

---

## Future Refactoring Triggers

This document should be revisited when:

- Module extraction creates service boundaries that events must cross (trigger for cross-service event security).
- Analytics pipeline is designed (trigger for analytics event specification).
- Event volume requires category-specific infrastructure scaling (trigger for infrastructure separation).
- Event catalog tooling is implemented (trigger for formal schema registry).
- Complex event processing needs emerge (trigger for CEP architecture).
- New event categories are proposed that do not fit existing taxonomy (trigger for taxonomy extension).

---

## Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 0.1 | 2026-07-21 | Domain Event Taxonomy and Ownership Architect | Initial draft — event taxonomy and ownership |
