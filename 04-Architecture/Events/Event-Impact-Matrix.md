# Event Architecture Impact Matrix

## Metadata

| Field | Value |
|-------|-------|
| Title | Kairo Event Architecture Traceability and Impact Matrix |
| Document ID | KAI-EVT-014 |
| Status | Draft |
| Version | 0.1 |
| Target Release | V1 |
| Owner | Chief Event Architect |
| Created | 2026-07-22 |
| Last Updated | 2026-07-22 |
| Reviewers | TODO |
| Related Documents | All Phase 10 (Event Architecture) documents, [Architecture Impact Matrix](../Architecture-Impact-Matrix.md), [Data Impact Matrix](../Data/Data-Impact-Matrix.md), [API Impact Matrix](../API/API-Impact-Matrix.md), [Security Impact Matrix](../Security/Security-Impact-Matrix.md), [Multi-Tenancy Impact Matrix](../Multi-Tenancy/Multi-Tenancy-Impact-Matrix.md) |
| Dependencies | All Phase 10 documents must be complete before this matrix is finalized |

---

## Applicable Version

V1 — This matrix applies to the initial release. Future capabilities are identified but not required for V1 delivery.

---

## Purpose

This document is the master traceability and impact matrix for the Kairo Event Architecture phase. It maps every event architecture decision to its upstream dependencies, downstream consumers, implementation surfaces, and governance responsibilities.

This matrix serves three audiences:

1. **Implementers** — Know which event requirements apply to their component.
2. **Future architects** — Know which event decisions they must consume when building Infrastructure, Development Standards, or Module specifications.
3. **Reviewers** — Verify completeness, consistency, and traceability across the entire event architecture phase.

---

## Scope

This matrix covers:

- All 13 Phase 10 (Event Architecture) documents and their cross-dependencies.
- Traceability to upstream Foundation, Product, Business Capability, Security, Multi-Tenancy, Data, and API documents.
- Forward dependencies on future architecture phases (Infrastructure, Development Standards).
- Implementation impact across all platform surfaces.
- Governance, testing, and audit responsibilities.

This matrix does not cover:

- Implementation details (no event names, no topics, no code).
- Specific module event type definitions.
- Vendor or product selection.

---

## 1. Event-Document Dependencies

### Phase 10 Internal Dependencies

| Document | Depends On (within Event phase) |
|----------|-------------------------------|
| [Event Architecture](./Event-Architecture.md) | — (foundational) |
| [Event Taxonomy and Ownership](./Event-Taxonomy-and-Ownership.md) | [Event Architecture](./Event-Architecture.md) |
| [Event Contract Standards](./Event-Contract-Standards.md) | [Event Architecture](./Event-Architecture.md), [Event Taxonomy](./Event-Taxonomy-and-Ownership.md) |
| [Domain Event Architecture](./Domain-Event-Architecture.md) | [Event Architecture](./Event-Architecture.md), [Event Contract Standards](./Event-Contract-Standards.md) |
| [Integration Event Architecture](./Integration-Event-Architecture.md) | [Event Architecture](./Event-Architecture.md), [Event Taxonomy](./Event-Taxonomy-and-Ownership.md), [Domain Event Architecture](./Domain-Event-Architecture.md) |
| [Event Publishing and Outbox](./Event-Publishing-and-Outbox.md) | [Event Architecture](./Event-Architecture.md), [Integration Event Architecture](./Integration-Event-Architecture.md) |
| [Event Consumption and Inbox](./Event-Consumption-and-Inbox.md) | [Event Architecture](./Event-Architecture.md), [Event Publishing and Outbox](./Event-Publishing-and-Outbox.md) |
| [Delivery, Ordering, and Consistency](./Delivery-Ordering-and-Consistency.md) | [Event Publishing and Outbox](./Event-Publishing-and-Outbox.md), [Event Consumption and Inbox](./Event-Consumption-and-Inbox.md) |
| [Retry, Dead-Letter, and Recovery](./Retry-Dead-Letter-and-Recovery.md) | [Event Consumption and Inbox](./Event-Consumption-and-Inbox.md), [Delivery, Ordering, and Consistency](./Delivery-Ordering-and-Consistency.md) |
| [Event Security and Tenant Context](./Event-Security-and-Tenant-Context.md) | [Event Architecture](./Event-Architecture.md), [Event Contract Standards](./Event-Contract-Standards.md) |
| [Event Versioning and Compatibility](./Event-Versioning-and-Compatibility.md) | [Event Contract Standards](./Event-Contract-Standards.md), [Integration Event Architecture](./Integration-Event-Architecture.md) |
| [Event Observability and Auditing](./Event-Observability-and-Auditing.md) | All preceding documents |
| [Event Governance and Lifecycle](./Event-Governance-and-Lifecycle.md) | All Phase 10 documents |

### Upstream Dependencies (documents Event Architecture consumes)

| Source Phase | Document | Event Architecture Documents That Consume It |
|-------------|----------|---------------------------------------------|
| Foundation | [Core Principles](../../01-Foundation/Core-Principles.md) | Event Architecture |
| Foundation | [Technical Philosophy](../../01-Foundation/Technical-Philosophy.md) | Event Architecture |
| Products | [Domain Model](../../02-Products/Domain-Model.md) | Domain Event Architecture, Event Taxonomy |
| Products | [Kairo Commerce](../../02-Products/Kairo-Commerce.md) | Event Taxonomy (business event examples) |
| Business Capabilities | [Bounded Contexts](../../03-Business-Capabilities/Bounded-Contexts.md) | Event Taxonomy, Domain Event Architecture |
| Architecture | [Module Architecture](../Module-Architecture.md) | Event Architecture, Domain Event Architecture, Integration Event Architecture |
| Architecture | [Monolith Strategy](../Monolith-Strategy.md) | Event Architecture (V1 in-process direction) |
| Security | [Security Architecture](../Security/Security-Architecture.md) | Event Security |
| Security | [Threat Model](../Security/Threat-Model.md) | Event Security |
| Security | [Audit and Security Monitoring](../Security/Audit-and-Security-Monitoring.md) | Event Observability, Event Security |
| Security | [Secrets and Key Management](../Security/Secrets-and-Key-Management.md) | Event Security |
| Security | [Secure Development Lifecycle](../Security/Secure-Development-Lifecycle.md) | Event Governance |
| Security | [Incident Response](../Security/Incident-Response.md) | Retry and Recovery, Event Observability |
| Multi-Tenancy | [Multi-Tenancy Architecture](../Multi-Tenancy/Multi-Tenancy-Architecture.md) | Event Architecture, Event Security |
| Multi-Tenancy | [Tenant Isolation](../Multi-Tenancy/Tenant-Isolation.md) | Event Security |
| Multi-Tenancy | [Tenant Resolution](../Multi-Tenancy/Tenant-Resolution.md) | Event Security, Event Consumption |
| Data | [Data Architecture](../Data/Data-Architecture.md) | Event Architecture |
| Data | [Data Ownership](../Data/Data-Ownership.md) | Event Taxonomy, Domain Event Architecture, Integration Event Architecture |
| Data | [Transaction and Consistency Architecture](../Data/Transaction-and-Consistency-Architecture.md) | Event Publishing and Outbox, Delivery and Consistency |
| Data | [Data Classification and Sensitivity](../Data/Data-Classification-and-Sensitivity.md) | Event Contract Standards, Event Security |
| Data | [Identifier Strategy](../Data/Identifier-Strategy.md) | Event Contract Standards |
| Data | [Schema Evolution and Migrations](../Data/Schema-Evolution-and-Migrations.md) | Event Versioning |
| Data | [Data Lifecycle and Retention](../Data/Data-Lifecycle-and-Retention.md) | Event Publishing and Outbox |
| Data | [Reporting and Analytics Architecture](../Data/Reporting-and-Analytics-Architecture.md) | Delivery and Consistency |
| Data | [Data Access and Persistence](../Data/Data-Access-and-Persistence.md) | Delivery and Consistency |
| API | [API Architecture](../API/API-Architecture.md) | Event Architecture (sync vs async) |
| API | [Webhook Architecture](../API/Webhook-Architecture.md) | Event Taxonomy, Integration Event Architecture |
| API | [Idempotency, Concurrency, and Retries](../API/Idempotency-Concurrency-and-Retries.md) | Event Consumption, Retry and Recovery |
| API | [API Versioning and Compatibility](../API/API-Versioning-and-Compatibility.md) | Event Versioning |
| API | [API Governance and Lifecycle](../API/API-Governance-and-Lifecycle.md) | Event Governance |

### Downstream Dependencies (future phases that must consume Event Architecture)

| Future Phase | Must Consume | Reason |
|-------------|--------------|--------|
| Infrastructure Architecture | Event Publishing, Delivery, Retry | Infrastructure hosts outbox processing, event bus/broker, dead-letter storage |
| Development Standards | All Event documents | Development standards implement event patterns (producer, consumer, outbox, inbox, handler) |
| Module Specifications | All Event documents | Each module defines its domain events, integration events, and consumer handlers |
| ADRs | Event Governance (triggers) | Significant event deviations require ADRs |
| Roadmap | Event Architecture (V1/future split) | Roadmap reflects event capability delivery timeline |

---

## 2. Event-Category Ownership

| Category | Owner | Scope | Governance | Document |
|----------|-------|-------|-----------|----------|
| Domain events | Module domain layer | Within module | Module team (no external governance) | [Event Taxonomy](./Event-Taxonomy-and-Ownership.md) |
| Integration events | Producing module | Cross-module | Architecture review required | [Event Taxonomy](./Event-Taxonomy-and-Ownership.md) |
| Webhook events | Module + platform delivery | External | Architecture + security review | [Event Taxonomy](./Event-Taxonomy-and-Ownership.md) |
| Business audit events | Audit service | Compliance | Permanent retention, restricted access | [Event Taxonomy](./Event-Taxonomy-and-Ownership.md) |
| Security events | Security infrastructure | Security monitoring | Security team ownership | [Event Taxonomy](./Event-Taxonomy-and-Ownership.md) |
| Operational events | Platform operations | Infrastructure monitoring | Operations team | [Event Taxonomy](./Event-Taxonomy-and-Ownership.md) |
| Analytics events | Analytics infrastructure | Business intelligence | Future (V2+) | [Event Taxonomy](./Event-Taxonomy-and-Ownership.md) |

---

## 3. Producer Responsibilities

| Responsibility | Detail | Document |
|---------------|--------|----------|
| Define event types | Choose what to publish and what it means | [Domain Event Architecture](./Domain-Event-Architecture.md) |
| Define contract schema | Document envelope + payload fields | [Event Contract Standards](./Event-Contract-Standards.md) |
| Transform domain → integration | Map internal event to published contract | [Domain Event Architecture](./Domain-Event-Architecture.md) |
| Write to outbox atomically | Same transaction as state change | [Event Publishing and Outbox](./Event-Publishing-and-Outbox.md) |
| Set tenant context | From authenticated request context | [Event Security and Tenant Context](./Event-Security-and-Tenant-Context.md) |
| Minimize sensitive data | Payloads carry only what consumers need | [Event Contract Standards](./Event-Contract-Standards.md) |
| Maintain compatibility | Additive within version, version bump for breaking | [Event Versioning and Compatibility](./Event-Versioning-and-Compatibility.md) |
| Provide reconciliation API | Consumers can rebuild state via API | [Integration Event Architecture](./Integration-Event-Architecture.md) |
| Not depend on consumer success | Transaction commits regardless of consumption | [Integration Event Architecture](./Integration-Event-Architecture.md) |

---

## 4. Consumer Responsibilities

| Responsibility | Detail | Document |
|---------------|--------|----------|
| Subscribe deliberately | Register for specific event types | [Event Consumption and Inbox](./Event-Consumption-and-Inbox.md) |
| Validate contract | Check type, version, required fields | [Event Consumption and Inbox](./Event-Consumption-and-Inbox.md) |
| Validate tenant | Verify tenant is valid and active | [Event Consumption and Inbox](./Event-Consumption-and-Inbox.md) |
| Deduplicate (inbox) | Prevent duplicate business effects | [Event Consumption and Inbox](./Event-Consumption-and-Inbox.md) |
| Process idempotently | Handle duplicate delivery safely | [Event Consumption and Inbox](./Event-Consumption-and-Inbox.md) |
| Handle unknown fields | Ignore new fields (forward compatibility) | [Event Contract Standards](./Event-Contract-Standards.md) |
| Handle unknown versions | Log and skip gracefully (not crash) | [Event Versioning and Compatibility](./Event-Versioning-and-Compatibility.md) |
| Reconcile after failure | Rebuild state from producer API if events missed | [Integration Event Architecture](./Integration-Event-Architecture.md) |
| Not mutate producer data | Consumer modifies its own state only | [Integration Event Architecture](./Integration-Event-Architecture.md) |

---

## 5. Contract Responsibilities

| Responsibility | Detail | Document |
|---------------|--------|----------|
| Standard envelope | id, type, version, occurredAt, publishedAt, producer, tenantId, correlationId, subject, data, metadata | [Event Contract Standards](./Event-Contract-Standards.md) |
| Hybrid payload default | Key fields + resource ID for full data fetch | [Event Contract Standards](./Event-Contract-Standards.md) |
| camelCase naming | Same conventions as API contracts | [Event Contract Standards](./Event-Contract-Standards.md) |
| Money as object | `{ amount: "29.99", currency: "USD" }` | [Event Contract Standards](./Event-Contract-Standards.md) |
| ISO 8601 UTC dates | All timestamps in UTC | [Event Contract Standards](./Event-Contract-Standards.md) |
| String identifiers | Public IDs, prefixed, opaque | [Event Contract Standards](./Event-Contract-Standards.md) |
| Version from day one | Version field present in all events | [Event Versioning and Compatibility](./Event-Versioning-and-Compatibility.md) |
| Schema documentation | Every event type documented | [Event Contract Standards](./Event-Contract-Standards.md) |
| Contract testing | Tests verify schema compliance | [Event Contract Standards](./Event-Contract-Standards.md) |

---

## 6. Tenant-Context Requirements

| Requirement | Detail | Document |
|-------------|--------|----------|
| Explicit in envelope | `tenantId` field required for tenant-scoped events | [Event Security and Tenant Context](./Event-Security-and-Tenant-Context.md) |
| Set from trusted context | Infrastructure sets from authenticated request, not arbitrary code | [Event Security and Tenant Context](./Event-Security-and-Tenant-Context.md) |
| Consumer validates | Tenant checked active/valid before processing | [Event Consumption and Inbox](./Event-Consumption-and-Inbox.md) |
| No cross-tenant effects | Processing one tenant's event never affects another tenant | [Event Security and Tenant Context](./Event-Security-and-Tenant-Context.md) |
| Platform events marked | Events without tenant context explicitly documented | [Event Security and Tenant Context](./Event-Security-and-Tenant-Context.md) |
| Consumer override prohibited | Consumers cannot replace event's tenant context | [Event Security and Tenant Context](./Event-Security-and-Tenant-Context.md) |

---

## 7. Security Requirements

| Requirement | Detail | Document |
|-------------|--------|----------|
| No secrets in events | Never passwords, tokens, keys, credentials | [Event Security and Tenant Context](./Event-Security-and-Tenant-Context.md) |
| Sensitive data minimized | IDs preferred over PII. Minimal payload. | [Event Security and Tenant Context](./Event-Security-and-Tenant-Context.md) |
| Classification in metadata | Data classification declared per event | [Event Security and Tenant Context](./Event-Security-and-Tenant-Context.md) |
| Encryption at rest | Outbox/inbox follow database encryption policy | [Event Security and Tenant Context](./Event-Security-and-Tenant-Context.md) |
| Logging restrictions | Payload never logged for Confidential/Restricted events | [Event Observability and Auditing](./Event-Observability-and-Auditing.md) |
| Subscription controlled | Not all consumers can subscribe to all event types | [Event Security and Tenant Context](./Event-Security-and-Tenant-Context.md) |
| Replay respects current rules | Replayed events subject to current authorization and tenant state | [Event Security and Tenant Context](./Event-Security-and-Tenant-Context.md) |
| Dead-letter classified | Dead-letter records inherit event classification | [Retry, Dead-Letter, and Recovery](./Retry-Dead-Letter-and-Recovery.md) |

---

## 8. Data-Classification Requirements

| Classification | Event Handling | Logging | Dead-Letter | Document |
|---------------|----------------|---------|-------------|----------|
| Public | Standard handling | Payload loggable (debug) | Standard access | [Event Security](./Event-Security-and-Tenant-Context.md) |
| Internal | Standard handling | Payload loggable (debug, non-prod) | Standard access | [Event Security](./Event-Security-and-Tenant-Context.md) |
| Confidential | Minimized payload | Metadata only | Restricted access | [Event Security](./Event-Security-and-Tenant-Context.md) |
| Restricted | Reference-only payload | Metadata only | Highly restricted access | [Event Security](./Event-Security-and-Tenant-Context.md) |

---

## 9. Publication and Outbox Requirements

| Requirement | Detail | Document |
|-------------|--------|----------|
| Transactional outbox | Event written in same transaction as state change | [Event Publishing and Outbox](./Event-Publishing-and-Outbox.md) |
| Atomic persistence | State + event or neither (no split-brain) | [Event Publishing and Outbox](./Event-Publishing-and-Outbox.md) |
| Publication worker | Background worker reads outbox and dispatches | [Event Publishing and Outbox](./Event-Publishing-and-Outbox.md) |
| Bounded retry | Finite retry count with backoff for dispatch failures | [Event Publishing and Outbox](./Event-Publishing-and-Outbox.md) |
| Duplicate publication possible | At-least-once publication, consumers handle | [Event Publishing and Outbox](./Event-Publishing-and-Outbox.md) |
| Outbox monitoring | Pending count, age, failure count metrics | [Event Publishing and Outbox](./Event-Publishing-and-Outbox.md) |
| Short-term retention | Published records cleaned up after retention period | [Event Publishing and Outbox](./Event-Publishing-and-Outbox.md) |
| Not audit storage | Outbox is delivery infrastructure, not permanent record | [Event Publishing and Outbox](./Event-Publishing-and-Outbox.md) |

---

## 10. Consumption and Inbox Requirements

| Requirement | Detail | Document |
|-------------|--------|----------|
| Contract validation | Type, version, required fields checked | [Event Consumption and Inbox](./Event-Consumption-and-Inbox.md) |
| Tenant validation | Tenant active/valid before processing | [Event Consumption and Inbox](./Event-Consumption-and-Inbox.md) |
| Inbox deduplication | For mutations — check event ID before processing | [Event Consumption and Inbox](./Event-Consumption-and-Inbox.md) |
| State-based idempotency | For projections — set-based operations naturally idempotent | [Event Consumption and Inbox](./Event-Consumption-and-Inbox.md) |
| Atomic processing | Handler + dedup record in same transaction | [Event Consumption and Inbox](./Event-Consumption-and-Inbox.md) |
| External calls after commit | APIs, notifications after transaction commits | [Event Consumption and Inbox](./Event-Consumption-and-Inbox.md) |
| Poison event isolation | Failing event does not block others | [Event Consumption and Inbox](./Event-Consumption-and-Inbox.md) |
| Consumer monitoring | Lag, failures, dead-letter per consumer | [Event Consumption and Inbox](./Event-Consumption-and-Inbox.md) |

---

## 11. Delivery Guarantees

| Category | Guarantee | Duplicates | Ordering | Consumer Idempotency | Document |
|----------|-----------|:---:|----------|:---:|----------|
| Integration events | At-least-once | Yes | Per-aggregate (best-effort) | **Required** | [Delivery and Consistency](./Delivery-Ordering-and-Consistency.md) |
| Domain events (internal) | In-process reliable | Rare | Per-aggregate | Recommended | [Delivery and Consistency](./Delivery-Ordering-and-Consistency.md) |
| Notifications (cache, hints) | At-most-once (best-effort) | No | None | Not required | [Delivery and Consistency](./Delivery-Ordering-and-Consistency.md) |
| Webhook delivery | At-least-once | Yes | Best-effort | **Required** (external) | [Delivery and Consistency](./Delivery-Ordering-and-Consistency.md) |
| Audit events | Guaranteed (never lost) | Rare | Chronological | Yes | [Delivery and Consistency](./Delivery-Ordering-and-Consistency.md) |

---

## 12. Ordering Requirements

| Scope | Guarantee | Mechanism | Document |
|-------|-----------|-----------|----------|
| Per-aggregate (same instance) | Best-effort (V1), guaranteed (future) | Single outbox worker / broker partitioning | [Delivery and Consistency](./Delivery-Ordering-and-Consistency.md) |
| Per-tenant (different aggregates) | None | Not provided | [Delivery and Consistency](./Delivery-Ordering-and-Consistency.md) |
| Cross-module | None | Not provided | [Delivery and Consistency](./Delivery-Ordering-and-Consistency.md) |
| Global | None | Not provided (not needed) | [Delivery and Consistency](./Delivery-Ordering-and-Consistency.md) |

---

## 13. Idempotency Requirements

| Consumer Type | Idempotency Strategy | Inbox Required | Document |
|--------------|---------------------|:---:|----------|
| Financial state changes (payment, refund) | Inbox deduplication | **Yes** | [Event Consumption and Inbox](./Event-Consumption-and-Inbox.md) |
| Inventory mutations | Inbox deduplication | **Yes** | [Event Consumption and Inbox](./Event-Consumption-and-Inbox.md) |
| Order status updates | Inbox deduplication | **Yes** | [Event Consumption and Inbox](./Event-Consumption-and-Inbox.md) |
| Notification sending | Inbox deduplication | **Yes** | [Event Consumption and Inbox](./Event-Consumption-and-Inbox.md) |
| Search index updates | State-based (re-index) | No | [Event Consumption and Inbox](./Event-Consumption-and-Inbox.md) |
| Cache invalidation | Natural (re-invalidate) | No | [Event Consumption and Inbox](./Event-Consumption-and-Inbox.md) |
| Read model projection | State-based (set value) | No | [Event Consumption and Inbox](./Event-Consumption-and-Inbox.md) |

---

## 14. Retry and Dead-Letter Requirements

| Failure Type | Retryable | Max Retries | Dead-Letter | Document |
|-------------|:---------:|:-----------:|:-----------:|----------|
| Transient (infrastructure) | Yes | 5-10 | After exhaustion | [Retry and Recovery](./Retry-Dead-Letter-and-Recovery.md) |
| Dependency (module unavailable) | Yes | 5-10 | After exhaustion | [Retry and Recovery](./Retry-Dead-Letter-and-Recovery.md) |
| Validation (malformed) | No | 0 | Immediate | [Retry and Recovery](./Retry-Dead-Letter-and-Recovery.md) |
| Compatibility (version mismatch) | No | 0 | Immediate | [Retry and Recovery](./Retry-Dead-Letter-and-Recovery.md) |
| Authorization | No | 0 | Immediate | [Retry and Recovery](./Retry-Dead-Letter-and-Recovery.md) |
| Tenant-context (invalid) | No | 0 | Immediate | [Retry and Recovery](./Retry-Dead-Letter-and-Recovery.md) |
| Consumer defect (code bug) | No | 0-1 | After detection | [Retry and Recovery](./Retry-Dead-Letter-and-Recovery.md) |
| Poison event | Limited | 3-5 | After exhaustion | [Retry and Recovery](./Retry-Dead-Letter-and-Recovery.md) |

---

## 15. Versioning Requirements

| Requirement | Detail | Document |
|-------------|--------|----------|
| Version field in envelope | Simple integer string, per event type | [Event Versioning and Compatibility](./Event-Versioning-and-Compatibility.md) |
| Present from V1 | Version included even when only one version exists | [Event Versioning and Compatibility](./Event-Versioning-and-Compatibility.md) |
| Additive is non-breaking | New optional fields, no version bump | [Event Versioning and Compatibility](./Event-Versioning-and-Compatibility.md) |
| Removal is breaking | Field removal requires version bump | [Event Versioning and Compatibility](./Event-Versioning-and-Compatibility.md) |
| Historical events immutable | Stored events never retroactively modified | [Event Versioning and Compatibility](./Event-Versioning-and-Compatibility.md) |
| Consumer forward-compatible | Ignores unknown fields, handles unknown versions gracefully | [Event Versioning and Compatibility](./Event-Versioning-and-Compatibility.md) |
| Independent from API/DB versions | Event versions evolve on their own timeline | [Event Versioning and Compatibility](./Event-Versioning-and-Compatibility.md) |

---

## 16. Observability Requirements

| Requirement | Signal Type | Document |
|-------------|------------|----------|
| Publication rate and lag | Metrics | [Event Observability](./Event-Observability-and-Auditing.md) |
| Consumer lag per consumer | Metrics | [Event Observability](./Event-Observability-and-Auditing.md) |
| Processing duration | Metrics | [Event Observability](./Event-Observability-and-Auditing.md) |
| Failure and retry rates | Metrics | [Event Observability](./Event-Observability-and-Auditing.md) |
| Dead-letter count and alerts | Metrics + Alerts | [Event Observability](./Event-Observability-and-Auditing.md) |
| Correlation propagation | Structured logs | [Event Observability](./Event-Observability-and-Auditing.md) |
| Sensitive payload not logged | Logging restriction | [Event Observability](./Event-Observability-and-Auditing.md) |
| Event lifecycle logging | Structured logs (metadata only) | [Event Observability](./Event-Observability-and-Auditing.md) |

---

## 17. Audit Requirements

| Event | Audit Record | Document |
|-------|-------------|----------|
| Business state change (via domain event) | Who, what, when, where | [Event Observability](./Event-Observability-and-Auditing.md) |
| Event dead-lettered | Event ID, consumer, reason | [Retry and Recovery](./Retry-Dead-Letter-and-Recovery.md) |
| Event replayed | Operator, scope, authorization | [Retry and Recovery](./Retry-Dead-Letter-and-Recovery.md) |
| Event skipped | Operator, event ID, reason, authorization | [Retry and Recovery](./Retry-Dead-Letter-and-Recovery.md) |
| Event corrected | Original + corrected IDs, operator, reason | [Retry and Recovery](./Retry-Dead-Letter-and-Recovery.md) |
| Subscription changed | Consumer, event type, operator | [Event Governance](./Event-Governance-and-Lifecycle.md) |
| Dead-letter accessed | Operator, scope | [Retry and Recovery](./Retry-Dead-Letter-and-Recovery.md) |

---

## 18. Testing Requirements

| Test Category | Scope | Document |
|--------------|-------|----------|
| Event production | Aggregate produces correct events for operations | [Domain Event Architecture](./Domain-Event-Architecture.md) |
| Contract compliance | Published events match documented schema | [Event Contract Standards](./Event-Contract-Standards.md) |
| Consumer idempotency | Duplicate processing produces no additional effects | [Event Consumption and Inbox](./Event-Consumption-and-Inbox.md) |
| Tenant isolation | Cross-tenant effects impossible via events | [Event Security](./Event-Security-and-Tenant-Context.md) |
| Forward compatibility | Consumers handle unknown fields | [Event Versioning](./Event-Versioning-and-Compatibility.md) |
| Poison event isolation | Failing event does not block others | [Retry and Recovery](./Retry-Dead-Letter-and-Recovery.md) |
| Outbox atomicity | Rollback loses both state and event record | [Event Publishing and Outbox](./Event-Publishing-and-Outbox.md) |
| Retry behavior | Transient failures retried, permanent failures dead-lettered | [Retry and Recovery](./Retry-Dead-Letter-and-Recovery.md) |
| Reconciliation | Consumer can rebuild state from producer API | [Integration Event Architecture](./Integration-Event-Architecture.md) |
| No secrets in payloads | Automated scanning for secret patterns | [Event Security](./Event-Security-and-Tenant-Context.md) |

---

## 19. Governance Requirements

| Requirement | Detail | Document |
|-------------|--------|----------|
| Business justification | Events exist for business need, not convenience | [Event Governance](./Event-Governance-and-Lifecycle.md) |
| Owner assigned | Every integration event has an owner | [Event Governance](./Event-Governance-and-Lifecycle.md) |
| Sync-vs-async decision | Documented choice between API and event | [Event Governance](./Event-Governance-and-Lifecycle.md) |
| Security review for sensitive | Elevated review for financial, cross-tenant, external | [Event Governance](./Event-Governance-and-Lifecycle.md) |
| Consumer registration | Consumers documented per event type | [Event Governance](./Event-Governance-and-Lifecycle.md) |
| Quarterly inventory review | Unused events identified and retired | [Event Governance](./Event-Governance-and-Lifecycle.md) |
| AI-generated same governance | No bypass for AI-authored code | [Event Governance](./Event-Governance-and-Lifecycle.md) |
| Replay requires authorization | Manual replay authorized and audited | [Event Governance](./Event-Governance-and-Lifecycle.md) |
| Breaking changes require approval | Architecture board approval | [Event Governance](./Event-Governance-and-Lifecycle.md) |

---

## 20. V1 versus Future Capabilities

| Capability | V1 | V2 | V3+ | Document |
|-----------|:---:|:---:|:---:|----------|
| In-process event delivery | Yes | Transitioning | — | [Event Architecture](./Event-Architecture.md) |
| Transactional outbox | Yes | Yes | Yes | [Event Publishing and Outbox](./Event-Publishing-and-Outbox.md) |
| Consumer inbox deduplication | Yes | Yes | Yes | [Event Consumption and Inbox](./Event-Consumption-and-Inbox.md) |
| At-least-once delivery | Yes | Yes | Yes | [Delivery and Consistency](./Delivery-Ordering-and-Consistency.md) |
| Per-aggregate ordering (best-effort) | Yes | Guaranteed (partitions) | Guaranteed | [Delivery and Consistency](./Delivery-Ordering-and-Consistency.md) |
| Bounded retry with backoff | Yes | Yes | Yes | [Retry and Recovery](./Retry-Dead-Letter-and-Recovery.md) |
| Dead-letter table | Yes | Enhanced UI | Self-service | [Retry and Recovery](./Retry-Dead-Letter-and-Recovery.md) |
| Event version in envelope | Yes | Yes | Yes | [Event Versioning](./Event-Versioning-and-Compatibility.md) |
| Correlation ID propagation | Yes | Yes | Yes | [Event Observability](./Event-Observability-and-Auditing.md) |
| Prometheus metrics | Yes | Yes (+ dashboards) | Yes | [Event Observability](./Event-Observability-and-Auditing.md) |
| External message broker | — | Yes | Yes | [Event Architecture](./Event-Architecture.md) |
| Competing consumers | — | Yes | Yes | [Delivery and Consistency](./Delivery-Ordering-and-Consistency.md) |
| Event replay automated | — | Yes | Yes | [Retry and Recovery](./Retry-Dead-Letter-and-Recovery.md) |
| Schema registry | — | Yes | Yes | [Event Versioning](./Event-Versioning-and-Compatibility.md) |
| Distributed tracing | — | Yes | Yes | [Event Observability](./Event-Observability-and-Auditing.md) |
| Event streaming for external | — | — | Evaluated | [Integration Event Architecture](./Integration-Event-Architecture.md) |
| Analytics event pipeline | — | Evaluated | Yes | [Event Taxonomy](./Event-Taxonomy-and-Ownership.md) |
| Cross-service events | — | — | Yes | [Event Architecture](./Event-Architecture.md) |

---

## 21. Required ADR Triggers

| Trigger Condition | ADR Required For | Related Document |
|-------------------|-----------------|-----------------|
| External message broker selection | Technology commitment | [Event Architecture](./Event-Architecture.md) |
| Event sourcing adopted for a module | Persistence pattern change | [Domain Event Architecture](./Domain-Event-Architecture.md) |
| Cross-service event delivery | Security and delivery pattern change | [Event Architecture](./Event-Architecture.md) |
| Saga/process manager pattern | Cross-module coordination approach | [Domain Event Architecture](./Domain-Event-Architecture.md) |
| Exception to event governance | Governance bypass | [Event Governance](./Event-Governance-and-Lifecycle.md) |
| New event category proposed | Taxonomy extension | [Event Taxonomy](./Event-Taxonomy-and-Ownership.md) |
| Binary serialization adopted | Contract format change | [Event Contract Standards](./Event-Contract-Standards.md) |
| Global ordering required for specific use case | Architecture exception | [Delivery and Consistency](./Delivery-Ordering-and-Consistency.md) |

---

## 22. Future Refactoring Triggers

| Trigger | Impact | Source Document |
|---------|--------|----------------|
| Event volume exceeds in-process capacity | Deploy external broker | [Event Architecture](./Event-Architecture.md) |
| Consumer lag unacceptable | Competing consumers or optimization | [Delivery and Consistency](./Delivery-Ordering-and-Consistency.md) |
| Module extraction to services | Distributed event delivery, service credentials | [Event Architecture](./Event-Architecture.md) |
| Dead-letter volume exceeds manual capacity | Self-service investigation UI | [Retry and Recovery](./Retry-Dead-Letter-and-Recovery.md) |
| Event types exceed documentation capacity | Schema registry and catalog tooling | [Event Versioning](./Event-Versioning-and-Compatibility.md) |
| Reconciliation needs exceed API-based approach | Event replay infrastructure | [Integration Event Architecture](./Integration-Event-Architecture.md) |
| Financial reconciliation automation needed | Automated gap detection | [Retry and Recovery](./Retry-Dead-Letter-and-Recovery.md) |
| Analytics pipeline needed | Analytics event category activation | [Event Taxonomy](./Event-Taxonomy-and-Ownership.md) |

---

## 23. Documentation Gaps

### Identified Unresolved Dependencies

| Gap | Required By | Resolution Phase | Notes |
|-----|------------|-----------------|-------|
| Infrastructure Architecture | Event Publishing, Delivery, Retry | Future phase | Hosts outbox processing, event bus/broker, dead-letter storage |
| Development Standards | All Event documents | Future phase | Implements event patterns (producer, consumer, outbox, inbox, handler code) |
| Module Specifications | All Event documents | Future phase | Each module defines its specific event types and consumer handlers |
| Glossary update | Event Contract Standards | Pending | Event-specific terms (outbox, inbox, dead-letter, integration event, correlation ID) |
| Architecture Roadmap | V1/Future split | Pending | Roadmap should reflect event capability delivery timeline |

### Phase 10 Completeness Check

| Document | Created | Linked in README | Linked in MASTER_INDEX | Internal Links Valid |
|----------|:---:|:---:|:---:|:---:|
| Event Architecture | Yes | Yes | Yes | Yes |
| Event Taxonomy and Ownership | Yes | Yes | Yes | Yes |
| Event Contract Standards | Yes | Yes | Yes | Yes |
| Domain Event Architecture | Yes | Yes | Yes | Yes |
| Integration Event Architecture | Yes | Yes | Yes | Yes |
| Event Publishing and Outbox | Yes | Yes | Yes | Yes |
| Event Consumption and Inbox | Yes | Yes | Yes | Yes |
| Delivery, Ordering, and Consistency | Yes | Yes | Yes | Yes |
| Retry, Dead-Letter, and Recovery | Yes | Yes | Yes | Yes |
| Event Security and Tenant Context | Yes | Yes | Yes | Yes |
| Event Versioning and Compatibility | Yes | Yes | Yes | Yes |
| Event Observability and Auditing | Yes | Yes | Yes | Yes |
| Event Governance and Lifecycle | Yes | Yes | Yes | Yes |
| Event Impact Matrix (this document) | Yes | Yes | Yes | Yes |

---

## Implementation Impact Across Surfaces

| Surface | Event Requirements | Key Documents |
|---------|-------------------|---------------|
| **Backend modules** | Produce domain events from aggregates. Transform to integration events. Write to outbox atomically. Implement idempotent consumers. Validate contracts and tenant. Support reconciliation APIs. | All Phase 10 documents |
| **Administrative portal** | Consumes events for real-time updates (via read models). May trigger events through admin operations. | [Domain Event Architecture](./Domain-Event-Architecture.md) |
| **Storefront** | Indirectly affected (storefront reads from read models updated by event consumers). Does not directly consume events. | [Integration Event Architecture](./Integration-Event-Architecture.md) |
| **Customer portal** | Indirectly affected (order status updated by event-driven consumers). | [Integration Event Architecture](./Integration-Event-Architecture.md) |
| **Developer portal** | Documents event types for external webhook subscription. Event catalog reference. | [Event Governance](./Event-Governance-and-Lifecycle.md) |
| **Background workers** | Outbox processor. Dead-letter processor. Lifecycle workers. Notification workers. | [Event Publishing and Outbox](./Event-Publishing-and-Outbox.md), [Retry and Recovery](./Retry-Dead-Letter-and-Recovery.md) |
| **Event publishers** | All modules that produce integration events. Outbox pattern mandatory. | [Event Publishing and Outbox](./Event-Publishing-and-Outbox.md) |
| **Event consumers** | All modules that react to integration events. Inbox/idempotency mandatory for mutations. | [Event Consumption and Inbox](./Event-Consumption-and-Inbox.md) |
| **Webhooks** | Integration events trigger outbound webhook evaluation and delivery. | [Integration Event Architecture](./Integration-Event-Architecture.md), [Webhook Architecture](../API/Webhook-Architecture.md) |
| **Integrations** | Inbound webhooks produce internal events. Integration events may trigger outbound sync. | [Event Taxonomy](./Event-Taxonomy-and-Ownership.md) |
| **Search** | Search indexer is an event consumer (updates index from product/catalog events). Naturally idempotent. | [Event Consumption and Inbox](./Event-Consumption-and-Inbox.md) |
| **Cache** | Cache invalidation triggered by events. Best-effort (at-most-once acceptable). | [Delivery and Consistency](./Delivery-Ordering-and-Consistency.md) |
| **Reporting** | Reporting read models updated by event consumers. Lag visible. | [Delivery and Consistency](./Delivery-Ordering-and-Consistency.md) |
| **Analytics** | Future: analytics pipeline consumes events. V1: direct queries. | [Event Taxonomy](./Event-Taxonomy-and-Ownership.md) |
| **Imports and exports** | Import completion produces events. Export may be triggered by events. | [Integration Event Architecture](./Integration-Event-Architecture.md) |
| **Infrastructure** | Hosts outbox processing, event bus (V1 in-process, future broker), dead-letter storage, retry infrastructure. | [Event Publishing and Outbox](./Event-Publishing-and-Outbox.md), [Retry and Recovery](./Retry-Dead-Letter-and-Recovery.md) |
| **CI/CD** | Contract tests in CI. Schema compliance validation. Breaking-change detection (future). | [Event Contract Standards](./Event-Contract-Standards.md), [Event Governance](./Event-Governance-and-Lifecycle.md) |
| **Support tooling** | Dead-letter investigation. Replay tooling. Correlation-based event tracing. Tenant-scoped views. | [Retry and Recovery](./Retry-Dead-Letter-and-Recovery.md), [Event Observability](./Event-Observability-and-Auditing.md) |
| **Security monitoring** | Security events from event infrastructure. Signature verification failures. Unauthorized subscription attempts. | [Event Security](./Event-Security-and-Tenant-Context.md) |
| **Public documentation** | Event catalog for external webhook consumers. Event contract documentation. | [Event Governance](./Event-Governance-and-Lifecycle.md) |

---

## Version Gate

| Version | Event Architecture Gate |
|---------|------------------------|
| V1 | All 14 Phase 10 documents complete. In-process event delivery via transactional outbox. Domain events from aggregates. Integration events for cross-module facts. Consumer idempotency (inbox for mutations, state-based for projections). Tenant context in all events (set by infrastructure). At-least-once delivery. Bounded retry with dead-letter. Event versioning from day one. Correlation propagation. Prometheus metrics (publication, lag, failures, dead-letter). Structured logging (metadata only for sensitive). Business audit from domain events. Governance (ownership, review, quarterly inventory). Contract tests in CI. |
| V2 | External broker deployed. Competing consumers. Event replay automated. Schema registry. Consumer lag dashboard. Enhanced dead-letter management. Tenant-facing webhook event status. Distributed tracing. |
| V3 | Cross-service events with service credentials. Event streaming for external. Saga orchestration. Analytics event pipeline. Event sourcing for select aggregates. Self-service governance for mature teams. |

---

## Decision Summary

| Decision | Rationale |
|----------|-----------|
| Single impact matrix for event phase | Consolidated traceability ensures completeness and consistency review. |
| Every entry links to authoritative document | Traceability requires source attribution. Claims without links are unverifiable. |
| Documentation gaps explicitly identified | Honest gap identification enables future phases to know what they must address. |
| Implementation surfaces mapped to documents | Implementers need a lookup from their component to relevant event requirements. |
| ADR triggers listed proactively | Teams know when they must write an ADR. |

---

## Architecture Impact

| Concern | Impact |
|---------|--------|
| Phase completeness | This matrix validates that Phase 10 is complete and internally consistent |
| Cross-phase traceability | Future phases can identify which event requirements they must consume |
| Implementation guidance | Every platform surface knows which event documents apply to it |
| Governance | ADR triggers and refactoring triggers provide governance guardrails |
| Gap visibility | Identified gaps become work items for future phases |

---

## Implementation Impact

| Area | Impact |
|------|--------|
| Module teams | Use this matrix to identify all event responsibilities for their domain |
| Platform team | Use implementation surface matrix to ensure infrastructure meets all event requirements |
| Security team | Use security and tenant matrices to validate coverage |
| Operations | Use delivery, retry, and observability matrices for operational planning |
| Architecture | Use gap analysis and ADR triggers for future phase planning |

---

## Security Responsibilities

| Role | Matrix Responsibilities |
|------|----------------------|
| Chief Event Architect | Maintains this matrix. Validates phase completeness. Identifies gaps. |
| Module Teams | Validate their entries are accurate. Flag missing responsibilities. |
| Security Team | Validate security matrix completeness. Ensure no gaps in tenant/classification coverage. |
| Operations | Validate delivery, retry, and observability entries match operational reality. |
| Platform Team | Validate infrastructure-related entries are implementable. |

---

## Multi-Tenancy Responsibilities

| Responsibility | Matrix Coverage |
|---------------|----------------|
| Tenant context in events | Matrix 6 (Tenant-Context Requirements) |
| Consumer tenant validation | Matrix 10 (Consumption Requirements) |
| Cross-tenant prevention | Matrix 7 (Security Requirements) |
| Tenant-scoped observability | Matrix 16 (Observability Requirements) |
| Dead-letter tenant scoping | Matrix 14 (Retry and Dead-Letter) |

---

## Out of Scope

This matrix does not:

- Define new architecture (it traces existing decisions).
- Override individual document decisions (it references them).
- Provide implementation guidance beyond document references.
- Prescribe tooling for traceability management.
- Define module-level event type definitions.

---

## Future Considerations

- **Automated traceability** — Tooling that validates links and completeness automatically.
- **Living matrix** — Auto-generated from document metadata as the repository grows.
- **Module-level matrices** — Each module specification includes its own event impact subset.
- **Cross-phase matrices** — Similar matrices for Infrastructure and Development Standards phases.
- **Compliance traceability** — Regulatory requirements mapped to event controls.

---

## Future Refactoring Triggers

This document should be revisited when:

- New event architecture documents are added (extend all matrices).
- Module specifications are written (validate event responsibilities).
- Infrastructure Architecture phase begins (validate hosting requirements).
- Development Standards phase begins (validate implementation patterns).
- ADRs create exceptions to event architecture rules (update affected matrices).
- External broker is deployed (update delivery and infrastructure references).

---

## Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 0.1 | 2026-07-22 | Chief Event Architect | Initial draft — Phase 10 completion matrix |
