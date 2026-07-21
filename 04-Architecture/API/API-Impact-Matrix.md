# API Architecture Impact Matrix

## Metadata

| Field | Value |
|-------|-------|
| Title | Kairo API Architecture Traceability and Impact Matrix |
| Document ID | KAI-API-014 |
| Status | Draft |
| Version | 0.1 |
| Target Release | V1 |
| Owner | Chief API Architect |
| Created | 2026-07-21 |
| Last Updated | 2026-07-21 |
| Reviewers | TODO |
| Related Documents | All Phase 9 (API Architecture) documents, [Architecture Impact Matrix](../Architecture-Impact-Matrix.md), [Platform Impact Matrix](../Platform-Impact-Matrix.md), [Security Impact Matrix](../Security/Security-Impact-Matrix.md), [Multi-Tenancy Impact Matrix](../Multi-Tenancy/Multi-Tenancy-Impact-Matrix.md), [Data Impact Matrix](../Data/Data-Impact-Matrix.md) |
| Dependencies | All Phase 9 documents must be complete before this matrix is finalized |

---

## Applicable Version

V1 — This matrix applies to the initial release. Future capabilities are identified but not required for V1 delivery.

---

## Purpose

This document is the master traceability and impact matrix for the Kairo API Architecture phase. It maps every API architecture decision to its upstream dependencies, downstream consumers, implementation surfaces, and governance responsibilities.

This matrix serves three audiences:

1. **Implementers** — Know which API requirements apply to their component.
2. **Future architects** — Know which API decisions they must consume when building Event, Infrastructure, or Module architecture.
3. **Reviewers** — Verify completeness, consistency, and traceability across the entire API architecture phase.

---

## Scope

This matrix covers:

- All 13 Phase 9 (API Architecture) documents and their cross-dependencies.
- Traceability to upstream Foundation, Product, Business Capability, Security, Multi-Tenancy, and Data documents.
- Forward dependencies on future architecture phases (Event, Infrastructure, Development Standards).
- Implementation impact across all platform surfaces.
- Governance, testing, and audit responsibilities.

This matrix does not cover:

- Implementation details (no endpoints, no code).
- Specific module API contracts.
- Vendor or product selection.

---

## 1. API Document Dependencies

### Phase 9 Internal Dependencies

| Document | Depends On (within API phase) |
|----------|-------------------------------|
| [API Architecture](./API-Architecture.md) | — (foundational) |
| [API Surfaces and Boundaries](./API-Surfaces-and-Boundaries.md) | [API Architecture](./API-Architecture.md) |
| [API Contract Standards](./API-Contract-Standards.md) | [API Architecture](./API-Architecture.md) |
| [Resource and Operation Modeling](./Resource-and-Operation-Modeling.md) | [API Architecture](./API-Architecture.md), [API Contract Standards](./API-Contract-Standards.md) |
| [Request and Response Standards](./Request-and-Response-Standards.md) | [API Architecture](./API-Architecture.md), [API Contract Standards](./API-Contract-Standards.md) |
| [Error Architecture](./Error-Architecture.md) | [Request and Response Standards](./Request-and-Response-Standards.md) |
| [API Versioning and Compatibility](./API-Versioning-and-Compatibility.md) | [API Architecture](./API-Architecture.md), [API Contract Standards](./API-Contract-Standards.md) |
| [Pagination, Filtering, Sorting, and Search](./Pagination-Filtering-Sorting-and-Search.md) | [Request and Response Standards](./Request-and-Response-Standards.md) |
| [Idempotency, Concurrency, and Retries](./Idempotency-Concurrency-and-Retries.md) | [API Architecture](./API-Architecture.md), [Error Architecture](./Error-Architecture.md) |
| [Bulk and Asynchronous Operations](./Bulk-and-Asynchronous-Operations.md) | [Idempotency, Concurrency, and Retries](./Idempotency-Concurrency-and-Retries.md), [Request and Response Standards](./Request-and-Response-Standards.md) |
| [Webhook Architecture](./Webhook-Architecture.md) | [API Surfaces and Boundaries](./API-Surfaces-and-Boundaries.md), [Idempotency, Concurrency, and Retries](./Idempotency-Concurrency-and-Retries.md) |
| [API Documentation and SDK Architecture](./API-Documentation-and-SDK-Architecture.md) | [API Contract Standards](./API-Contract-Standards.md), [API Versioning and Compatibility](./API-Versioning-and-Compatibility.md) |
| [API Governance and Lifecycle](./API-Governance-and-Lifecycle.md) | All Phase 9 documents |

### Upstream Dependencies (documents API Architecture consumes)

| Source Phase | Document | API Architecture Documents That Consume It |
|-------------|----------|---------------------------------------------|
| Foundation | [Core Principles](../../01-Foundation/Core-Principles.md) | API Architecture |
| Foundation | [Technical Philosophy](../../01-Foundation/Technical-Philosophy.md) | API Architecture |
| Products | [Kairo Platform](../../02-Products/Kairo-Platform.md) | API Surfaces, API Architecture |
| Products | [Kairo Commerce](../../02-Products/Kairo-Commerce.md) | API Surfaces, Resource Modeling |
| Products | [Product Boundaries](../../02-Products/Product-Boundaries.md) | API Architecture, API Surfaces |
| Products | [Domain Model](../../02-Products/Domain-Model.md) | Resource Modeling, Contract Standards |
| Products | [Glossary](../../02-Products/Glossary.md) | Contract Standards |
| Business Capabilities | [Capability Map](../../03-Business-Capabilities/Capability-Map.md) | API Surfaces, Resource Modeling |
| Business Capabilities | [Bounded Contexts](../../03-Business-Capabilities/Bounded-Contexts.md) | Resource Modeling, API Governance |
| Architecture | [Module Architecture](../Module-Architecture.md) | API Architecture, Resource Modeling, Governance |
| Architecture | [Monolith Strategy](../Monolith-Strategy.md) | API Architecture (single deployment) |
| Security | [Security Architecture](../Security/Security-Architecture.md) | API Architecture, API Surfaces |
| Security | [API Security](../Security/API-Security.md) | All API phase documents |
| Security | [Identity and Authentication](../Security/Identity-and-Authentication.md) | API Surfaces, Webhook Architecture |
| Security | [Authorization Architecture](../Security/Authorization-Architecture.md) | API Surfaces, Resource Modeling, Pagination |
| Security | [Secrets and Key Management](../Security/Secrets-and-Key-Management.md) | Webhook Architecture |
| Security | [Audit and Security Monitoring](../Security/Audit-and-Security-Monitoring.md) | Error Architecture, Bulk Operations, Webhook Architecture |
| Security | [Secure Development Lifecycle](../Security/Secure-Development-Lifecycle.md) | API Governance, API Versioning |
| Multi-Tenancy | [Multi-Tenancy Architecture](../Multi-Tenancy/Multi-Tenancy-Architecture.md) | API Architecture, API Surfaces |
| Multi-Tenancy | [Tenant Resolution](../Multi-Tenancy/Tenant-Resolution.md) | API Surfaces, Webhook Architecture |
| Multi-Tenancy | [Tenant Isolation](../Multi-Tenancy/Tenant-Isolation.md) | Pagination, Bulk Operations |
| Multi-Tenancy | [Cross-Tenant Operations](../Multi-Tenancy/Cross-Tenant-Operations.md) | API Surfaces |
| Data | [Data Architecture](../Data/Data-Architecture.md) | API Architecture |
| Data | [Data Ownership](../Data/Data-Ownership.md) | Resource Modeling, API Governance |
| Data | [Identifier Strategy](../Data/Identifier-Strategy.md) | Contract Standards, Resource Modeling |
| Data | [Data Classification and Sensitivity](../Data/Data-Classification-and-Sensitivity.md) | Contract Standards, Error Architecture, Webhook Architecture |
| Data | [Transaction and Consistency Architecture](../Data/Transaction-and-Consistency-Architecture.md) | Idempotency, Resource Modeling |
| Data | [Data Access and Persistence](../Data/Data-Access-and-Persistence.md) | Pagination |
| Data | [Schema Evolution and Migrations](../Data/Schema-Evolution-and-Migrations.md) | API Versioning |
| Data | [Data Lifecycle and Retention](../Data/Data-Lifecycle-and-Retention.md) | Bulk Operations |
| Data | [Reporting and Analytics Architecture](../Data/Reporting-and-Analytics-Architecture.md) | Pagination |

### Downstream Dependencies (future phases that must consume API Architecture)

| Future Phase | Must Consume | Reason |
|-------------|--------------|--------|
| Event Architecture | Webhook Architecture, Idempotency, Resource Modeling | Events trigger webhooks. Event consumers need idempotency. Event payloads follow resource patterns. |
| Infrastructure Architecture | API Surfaces, Pagination, Bulk Operations, Webhook Architecture | Infrastructure hosts API routing, rate limiting, queue processing, webhook delivery. |
| Development Standards | All API phase documents | Development standards implement API patterns (controllers, middleware, validation, testing). |
| Module Specifications | All API phase documents | Each module defines its endpoints following these architectural standards. |
| ADRs | API Governance (triggers) | Significant API deviations require ADRs. |
| Roadmap | API Architecture (V1/future split) | Roadmap reflects API capability delivery timeline. |

---

## 2. API Surface Ownership

| Surface | Owner | Scope | Key Document |
|---------|-------|-------|-------------|
| Public Storefront API | Commerce module teams (Catalog, Cart, Checkout) | Shopping experience | [API Surfaces](./API-Surfaces-and-Boundaries.md) |
| Customer Account API | Customer module team | Customer self-service | [API Surfaces](./API-Surfaces-and-Boundaries.md) |
| Administrative API | Per-module (each module owns its admin endpoints) | Organization management | [API Surfaces](./API-Surfaces-and-Boundaries.md) |
| Partner/Integration API | Per-module + platform (integration framework) | System-to-system exchange | [API Surfaces](./API-Surfaces-and-Boundaries.md) |
| Developer Management API | Platform team | API key/webhook management | [API Surfaces](./API-Surfaces-and-Boundaries.md) |
| Platform Administration API | Platform team | Platform operations | [API Surfaces](./API-Surfaces-and-Boundaries.md) |
| Internal Module Contracts | Per-module (provider module owns contract) | Cross-module communication | [API Surfaces](./API-Surfaces-and-Boundaries.md) |
| Webhook (outbound) | Per-module (event source) + platform (delivery) | Event notification | [Webhook Architecture](./Webhook-Architecture.md) |
| Webhook (inbound) | Platform + per-integration | Provider callbacks | [Webhook Architecture](./Webhook-Architecture.md) |

---

## 3. Consumer-to-Surface Mapping

| Consumer | Storefront | Customer | Admin | Integration | Dev Mgmt | Platform | Internal | Webhooks Out | Webhooks In |
|----------|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| Storefront application | **Yes** | **Yes** | — | — | — | — | — | — | — |
| Mobile application | **Yes** | **Yes** | — | — | — | — | — | — | — |
| Admin portal | — | — | **Yes** | — | **Yes** | — | — | — | — |
| External ERP/system | — | — | — | **Yes** | — | — | — | — | — |
| Payment provider | — | — | — | — | — | — | — | — | **Yes** |
| Shipping carrier | — | — | — | — | — | — | — | — | **Yes** |
| Platform operator | — | — | — | — | — | **Yes** | — | — | — |
| Internal modules | — | — | — | — | — | — | **Yes** | — | — |
| Custom webhook receiver | — | — | — | — | — | — | — | **Yes** | — |

Source: [API Surfaces and Boundaries](./API-Surfaces-and-Boundaries.md)

---

## 4. Authentication Requirements

| Surface | Auth Method | Required | Document |
|---------|------------|:---:|----------|
| Storefront (browsing) | None or session | Optional | [API Surfaces](./API-Surfaces-and-Boundaries.md) |
| Storefront (checkout) | Customer JWT | Yes | [API Surfaces](./API-Surfaces-and-Boundaries.md) |
| Customer Account | Customer JWT | Yes | [API Surfaces](./API-Surfaces-and-Boundaries.md) |
| Administrative | Organization member JWT | Yes | [API Surfaces](./API-Surfaces-and-Boundaries.md) |
| Integration | API key / OAuth2 client credentials | Yes | [API Surfaces](./API-Surfaces-and-Boundaries.md) |
| Developer Management | Organization member JWT | Yes | [API Surfaces](./API-Surfaces-and-Boundaries.md) |
| Platform Admin | Platform operator credentials | Yes | [API Surfaces](./API-Surfaces-and-Boundaries.md) |
| Internal (V1 in-process) | N/A (same process) | Inherited | [API Surfaces](./API-Surfaces-and-Boundaries.md) |
| Webhook inbound | Signature verification | Yes | [Webhook Architecture](./Webhook-Architecture.md) |

---

## 5. Authorization Requirements

| Surface | Authorization Model | Document |
|---------|-------------------|----------|
| Storefront | Anonymous read / customer-scoped write | [API Surfaces](./API-Surfaces-and-Boundaries.md) |
| Customer Account | Own-data only | [API Surfaces](./API-Surfaces-and-Boundaries.md) |
| Administrative | Role-based within organization | [API Surfaces](./API-Surfaces-and-Boundaries.md) |
| Integration | Scoped capabilities per integration | [API Surfaces](./API-Surfaces-and-Boundaries.md) |
| Platform Admin | Platform-level privileges | [API Surfaces](./API-Surfaces-and-Boundaries.md) |
| Bulk operations | Per-item authorization | [Bulk and Async Operations](./Bulk-and-Asynchronous-Operations.md) |
| Webhook outbound | Subscription owner access | [Webhook Architecture](./Webhook-Architecture.md) |
| Webhook inbound | Scoped to integration's registered purpose | [Webhook Architecture](./Webhook-Architecture.md) |

---

## 6. Tenant-Resolution Requirements

| Surface | Resolution Method | Document |
|---------|------------------|----------|
| Storefront | Store context (request origin or store identifier) | [API Surfaces](./API-Surfaces-and-Boundaries.md) |
| Customer Account | Organization from authenticated customer | [API Surfaces](./API-Surfaces-and-Boundaries.md) |
| Administrative | Organization from authenticated member | [API Surfaces](./API-Surfaces-and-Boundaries.md) |
| Integration | Organization from integration credentials | [API Surfaces](./API-Surfaces-and-Boundaries.md) |
| Platform Admin | Explicit context selection (platform operator) | [API Surfaces](./API-Surfaces-and-Boundaries.md) |
| Internal (V1) | Inherited from originating request | [API Surfaces](./API-Surfaces-and-Boundaries.md) |
| Webhook inbound | Provider-account mapping to organization | [Webhook Architecture](./Webhook-Architecture.md) |
| Webhook outbound | Subscription owner's organization | [Webhook Architecture](./Webhook-Architecture.md) |

---

## 7. Data-Ownership Requirements

| API Concern | Data Rule | Document |
|-------------|-----------|----------|
| Resource identity | Public IDs only (no internal keys) | [API Contract Standards](./API-Contract-Standards.md) |
| Response fields | Contract types (never persistence entities) | [API Contract Standards](./API-Contract-Standards.md) |
| Cross-module references | By public ID through published contracts | [Resource and Operation Modeling](./Resource-and-Operation-Modeling.md) |
| Module boundary | Module owns its API contract for its data | [API Architecture](./API-Architecture.md) |
| Sensitive fields | Classification-driven inclusion/exclusion | [API Contract Standards](./API-Contract-Standards.md) |
| Financial data | Minimal in webhooks; full via API with auth | [Webhook Architecture](./Webhook-Architecture.md) |
| Export data | Classification inherited from content | [Bulk and Async Operations](./Bulk-and-Asynchronous-Operations.md) |

---

## 8. Contract Standards

| Standard | Applies To | Document |
|----------|-----------|----------|
| camelCase field naming | All JSON responses | [API Contract Standards](./API-Contract-Standards.md) |
| Money as object (amount + currency) | All monetary fields | [API Contract Standards](./API-Contract-Standards.md) |
| ISO 8601 UTC dates | All timestamp fields | [API Contract Standards](./API-Contract-Standards.md) |
| String identifiers | All resource IDs | [API Contract Standards](./API-Contract-Standards.md) |
| String enumerations | All enum values | [API Contract Standards](./API-Contract-Standards.md) |
| Nullable vs absent semantics | All optional fields | [API Contract Standards](./API-Contract-Standards.md) |
| Additive-compatible responses | All response evolution | [API Contract Standards](./API-Contract-Standards.md) |
| Approved envelope (isSuccess/data/message/errors) | All responses | [Request and Response Standards](./Request-and-Response-Standards.md) |

---

## 9. Error Standards

| Standard | Applies To | Document |
|----------|-----------|----------|
| Structured error objects (code/field/message/meta) | All error responses | [Error Architecture](./Error-Architecture.md) |
| Stable error codes (UPPER_SNAKE_CASE) | All errors | [Error Architecture](./Error-Architecture.md) |
| No internal details exposed | All 5xx responses | [Error Architecture](./Error-Architecture.md) |
| 404 for both missing and unauthorized | Resource access | [Error Architecture](./Error-Architecture.md) |
| Provider errors normalized | External provider failures | [Error Architecture](./Error-Architecture.md) |
| Retryability indicated | All error responses | [Error Architecture](./Error-Architecture.md) |
| Financial error state clarity | Payment/refund failures | [Error Architecture](./Error-Architecture.md) |
| Partial-success reporting | Bulk operations | [Error Architecture](./Error-Architecture.md) |

---

## 10. Versioning Requirements

| Requirement | Applies To | Document |
|-------------|-----------|----------|
| URL path versioning (`/v1/...`) | All public endpoints | [API Versioning](./API-Versioning-and-Compatibility.md) |
| 12-month minimum deprecation | Version retirement | [API Versioning](./API-Versioning-and-Compatibility.md) |
| Deprecation headers | Deprecated versions | [API Versioning](./API-Versioning-and-Compatibility.md) |
| Breaking changes require new version | Contract-breaking modifications | [API Versioning](./API-Versioning-and-Compatibility.md) |
| Additive changes are non-breaking | New optional fields, endpoints | [API Versioning](./API-Versioning-and-Compatibility.md) |
| SDK major version aligns with API version | SDK releases | [API Versioning](./API-Versioning-and-Compatibility.md) |
| Webhook payload versioning | Outbound webhooks | [Webhook Architecture](./Webhook-Architecture.md) |

---

## 11. Pagination and Query Requirements

| Requirement | Applies To | Document |
|-------------|-----------|----------|
| All collections paginated | Every collection endpoint | [Pagination](./Pagination-Filtering-Sorting-and-Search.md) |
| Cursor pagination (default) | Large/frequently-changing collections | [Pagination](./Pagination-Filtering-Sorting-and-Search.md) |
| Maximum page size enforced | All collections | [Pagination](./Pagination-Filtering-Sorting-and-Search.md) |
| Stable ordering with tiebreaker | All paginated collections | [Pagination](./Pagination-Filtering-Sorting-and-Search.md) |
| Filterable fields must be indexed | All filter parameters | [Pagination](./Pagination-Filtering-Sorting-and-Search.md) |
| Unsupported filters return 400 | All collection endpoints | [Pagination](./Pagination-Filtering-Sorting-and-Search.md) |
| Tenant filter cannot be removed | All queries | [Pagination](./Pagination-Filtering-Sorting-and-Search.md) |
| Search separate from filtering | Full-text vs transactional | [Pagination](./Pagination-Filtering-Sorting-and-Search.md) |

---

## 12. Idempotency Requirements

| Operation | Idempotency Key | Provider Key | Document |
|-----------|:---:|:---:|----------|
| Order creation | Required | — | [Idempotency](./Idempotency-Concurrency-and-Retries.md) |
| Payment authorization | Required | Required | [Idempotency](./Idempotency-Concurrency-and-Retries.md) |
| Payment capture | Required | Required | [Idempotency](./Idempotency-Concurrency-and-Retries.md) |
| Refund | Required | Required | [Idempotency](./Idempotency-Concurrency-and-Retries.md) |
| Cancellation (with compensation) | Required | Conditional | [Idempotency](./Idempotency-Concurrency-and-Retries.md) |
| Inventory adjustment | Required | — | [Idempotency](./Idempotency-Concurrency-and-Retries.md) |
| Inventory reservation | Required | — | [Idempotency](./Idempotency-Concurrency-and-Retries.md) |
| Subscription creation | Required | Conditional | [Idempotency](./Idempotency-Concurrency-and-Retries.md) |
| Webhook-triggered mutation | Required (event ID) | — | [Idempotency](./Idempotency-Concurrency-and-Retries.md) |
| Bulk mutation submission | Required (batch ID) | — | [Idempotency](./Idempotency-Concurrency-and-Retries.md) |

---

## 13. Concurrency Requirements

| Requirement | Mechanism | Document |
|-------------|-----------|----------|
| Resource updates | Optimistic concurrency (version/ETag) | [Idempotency](./Idempotency-Concurrency-and-Retries.md) |
| Conditional requests | If-Match / If-None-Match headers | [Idempotency](./Idempotency-Concurrency-and-Retries.md) |
| Lost-update prevention | Version check on write | [Idempotency](./Idempotency-Concurrency-and-Retries.md) |
| Conflict response | 409 with current version | [Idempotency](./Idempotency-Concurrency-and-Retries.md) |
| Bulk item concurrency | Per-item optimistic concurrency | [Bulk and Async Operations](./Bulk-and-Asynchronous-Operations.md) |

---

## 14. Bulk-Operation Requirements

| Requirement | Detail | Document |
|-------------|--------|----------|
| Synchronous limit | Small batches only (10-50 items) | [Bulk and Async Operations](./Bulk-and-Asynchronous-Operations.md) |
| Async for large | Operations exceeding sync limit are async (202 + polling) | [Bulk and Async Operations](./Bulk-and-Asynchronous-Operations.md) |
| Per-item validation | Same validation as single-item | [Bulk and Async Operations](./Bulk-and-Asynchronous-Operations.md) |
| Per-item authorization | Individual items authorized | [Bulk and Async Operations](./Bulk-and-Asynchronous-Operations.md) |
| Partial success default | Partial processing with per-item results | [Bulk and Async Operations](./Bulk-and-Asynchronous-Operations.md) |
| Export URL security | Signed, time-limited, tenant-scoped | [Bulk and Async Operations](./Bulk-and-Asynchronous-Operations.md) |
| Result retention | TTL-based, auto-deleted | [Bulk and Async Operations](./Bulk-and-Asynchronous-Operations.md) |

---

## 15. Webhook Requirements

| Requirement | Direction | Document |
|-------------|-----------|----------|
| Outbound: HMAC signing | Per-subscription secret | [Webhook Architecture](./Webhook-Architecture.md) |
| Outbound: at-least-once delivery | Exponential retry | [Webhook Architecture](./Webhook-Architecture.md) |
| Outbound: automatic suspension | After sustained failure | [Webhook Architecture](./Webhook-Architecture.md) |
| Outbound: decoupled from transactions | Async queue | [Webhook Architecture](./Webhook-Architecture.md) |
| Inbound: signature verification first | Before any processing | [Webhook Architecture](./Webhook-Architecture.md) |
| Inbound: tenant from config | Not from payload | [Webhook Architecture](./Webhook-Architecture.md) |
| Inbound: async processing | Acknowledge then process | [Webhook Architecture](./Webhook-Architecture.md) |
| Inbound: event deduplication | Provider event ID | [Webhook Architecture](./Webhook-Architecture.md) |

---

## 16. Documentation Requirements

| Requirement | Applies To | Document |
|-------------|-----------|----------|
| OpenAPI 3.1 specification | All public endpoints | [API Documentation and SDK](./API-Documentation-and-SDK-Architecture.md) |
| Code-first generation | Specification source | [API Documentation and SDK](./API-Documentation-and-SDK-Architecture.md) |
| Reference docs generated | All endpoints | [API Documentation and SDK](./API-Documentation-and-SDK-Architecture.md) |
| Conceptual guides hand-written | Auth, pagination, errors, idempotency, webhooks | [API Documentation and SDK](./API-Documentation-and-SDK-Architecture.md) |
| Quick-start guide | Developer onboarding | [API Documentation and SDK](./API-Documentation-and-SDK-Architecture.md) |
| Drift detection in CI | Spec vs implementation | [API Documentation and SDK](./API-Documentation-and-SDK-Architecture.md) |
| Fictional data only in examples | All documentation | [API Documentation and SDK](./API-Documentation-and-SDK-Architecture.md) |
| Deprecated APIs marked | Docs + headers + spec | [API Documentation and SDK](./API-Documentation-and-SDK-Architecture.md) |

---

## 17. SDK Requirements

| Requirement | Detail | Document |
|-------------|--------|----------|
| .NET SDK (V1 primary) | Full handwritten experience layer | [API Documentation and SDK](./API-Documentation-and-SDK-Architecture.md) |
| TypeScript SDK (V1 secondary) | Generated + light wrapping | [API Documentation and SDK](./API-Documentation-and-SDK-Architecture.md) |
| Safe retry behavior | Only auto-retry known-safe errors | [API Documentation and SDK](./API-Documentation-and-SDK-Architecture.md) |
| Idempotency helpers | Auto key generation and injection | [API Documentation and SDK](./API-Documentation-and-SDK-Architecture.md) |
| Pagination iterators | Auto-paginate through collections | [API Documentation and SDK](./API-Documentation-and-SDK-Architecture.md) |
| Typed errors | Language-idiomatic error types | [API Documentation and SDK](./API-Documentation-and-SDK-Architecture.md) |
| Major version = API version | SDK 1.x → API v1 | [API Documentation and SDK](./API-Documentation-and-SDK-Architecture.md) |

---

## 18. Audit Requirements

| Event | Audit Required | Document |
|-------|:-:|----------|
| Authentication failure | Yes | [Error Architecture](./Error-Architecture.md) |
| Authorization failure | Yes | [Error Architecture](./Error-Architecture.md) |
| Financial operation (all) | Yes | [Idempotency](./Idempotency-Concurrency-and-Retries.md) |
| Bulk operation submission | Yes | [Bulk and Async Operations](./Bulk-and-Asynchronous-Operations.md) |
| Data export | Yes | [Bulk and Async Operations](./Bulk-and-Asynchronous-Operations.md) |
| Webhook inbound receipt | Yes | [Webhook Architecture](./Webhook-Architecture.md) |
| Webhook outbound delivery | Yes (metadata) | [Webhook Architecture](./Webhook-Architecture.md) |
| API key creation/revocation | Yes | [API Surfaces](./API-Surfaces-and-Boundaries.md) |
| Breaking change deployment | Yes | [API Governance](./API-Governance-and-Lifecycle.md) |

---

## 19. Security-Testing Requirements

| Test Category | Scope | Document |
|--------------|-------|----------|
| Authentication bypass | All surfaces | [API Surfaces](./API-Surfaces-and-Boundaries.md) |
| Authorization escalation | All surfaces | [API Surfaces](./API-Surfaces-and-Boundaries.md) |
| Tenant isolation (cross-tenant access) | All data-accessing endpoints | [Pagination](./Pagination-Filtering-Sorting-and-Search.md) |
| Information leakage in errors | All error responses | [Error Architecture](./Error-Architecture.md) |
| Rate-limit effectiveness | All public surfaces | [API Surfaces](./API-Surfaces-and-Boundaries.md) |
| Webhook signature verification | All inbound endpoints | [Webhook Architecture](./Webhook-Architecture.md) |
| Export URL access control | All export operations | [Bulk and Async Operations](./Bulk-and-Asynchronous-Operations.md) |
| Idempotency key abuse | Financial operations | [Idempotency](./Idempotency-Concurrency-and-Retries.md) |
| Input validation completeness | All mutation endpoints | [API Contract Standards](./API-Contract-Standards.md) |

---

## 20. Contract-Testing Requirements

| Test Category | Scope | Document |
|--------------|-------|----------|
| Response envelope compliance | All endpoints | [Request and Response Standards](./Request-and-Response-Standards.md) |
| Error code stability | All error responses | [Error Architecture](./Error-Architecture.md) |
| Field naming compliance | All responses | [API Contract Standards](./API-Contract-Standards.md) |
| Pagination structure | All collections | [Pagination](./Pagination-Filtering-Sorting-and-Search.md) |
| Versioning compatibility | Between versions | [API Versioning](./API-Versioning-and-Compatibility.md) |
| Webhook payload contract | Outbound deliveries | [Webhook Architecture](./Webhook-Architecture.md) |
| Spec-implementation alignment | All endpoints | [API Documentation and SDK](./API-Documentation-and-SDK-Architecture.md) |
| Breaking-change detection | CI pipeline | [API Versioning](./API-Versioning-and-Compatibility.md) |

---

## 21. V1 versus Future Capabilities

| Capability | V1 | V2 | V3+ | Document |
|-----------|:---:|:---:|:---:|----------|
| REST/JSON APIs | Yes | Yes | Yes | [API Architecture](./API-Architecture.md) |
| URL path versioning | Yes | Yes | Yes | [API Versioning](./API-Versioning-and-Compatibility.md) |
| Approved response envelope | Yes | Yes | Yes | [Request and Response Standards](./Request-and-Response-Standards.md) |
| Cursor pagination | Yes | Yes | Yes | [Pagination](./Pagination-Filtering-Sorting-and-Search.md) |
| Idempotency-Key for financial ops | Yes | Yes | Yes | [Idempotency](./Idempotency-Concurrency-and-Retries.md) |
| Optimistic concurrency | Yes | Yes | Yes | [Idempotency](./Idempotency-Concurrency-and-Retries.md) |
| Outbound webhooks (HMAC signed) | Yes | Yes | Yes | [Webhook Architecture](./Webhook-Architecture.md) |
| Inbound webhooks (verified) | Yes | Yes | Yes | [Webhook Architecture](./Webhook-Architecture.md) |
| .NET SDK | Yes | Yes | Yes | [API Documentation and SDK](./API-Documentation-and-SDK-Architecture.md) |
| TypeScript SDK | Yes | Yes | Yes | [API Documentation and SDK](./API-Documentation-and-SDK-Architecture.md) |
| OpenAPI specification generated | Yes | Yes | Yes | [API Documentation and SDK](./API-Documentation-and-SDK-Architecture.md) |
| API gateway separation | — | Yes | Yes | [API Architecture](./API-Architecture.md) |
| GraphQL surface | — | Eval | If adopted | [API Architecture](./API-Architecture.md) |
| Event streaming API | — | Yes | Yes | [API Architecture](./API-Architecture.md) |
| Field selection (sparse fieldsets) | — | Yes | Yes | [Pagination](./Pagination-Filtering-Sorting-and-Search.md) |
| Webhook management API (self-service) | — | Yes | Yes | [Webhook Architecture](./Webhook-Architecture.md) |
| PHP/Python SDKs | — | Yes | Yes | [API Documentation and SDK](./API-Documentation-and-SDK-Architecture.md) |
| Interactive API explorer | — | Yes | Yes | [API Documentation and SDK](./API-Documentation-and-SDK-Architecture.md) |
| gRPC for internal | — | — | If extracted | [API Architecture](./API-Architecture.md) |
| Date-based versioning | — | — | Eval | [API Versioning](./API-Versioning-and-Compatibility.md) |

---

## 22. Required ADR Triggers

| Trigger Condition | ADR Required For | Related Document |
|-------------------|-----------------|-----------------|
| New API surface type proposed | Establishes new long-term category | [API Governance](./API-Governance-and-Lifecycle.md) |
| Authentication mechanism change | Affects all consumers | [API Governance](./API-Governance-and-Lifecycle.md) |
| Response envelope modification | Affects all consumers and SDKs | [Request and Response Standards](./Request-and-Response-Standards.md) |
| Versioning strategy change | Affects compatibility promises | [API Versioning](./API-Versioning-and-Compatibility.md) |
| Cross-tenant API capability | Governance and security precedent | [API Surfaces](./API-Surfaces-and-Boundaries.md) |
| New protocol (WebSocket, gRPC, GraphQL) | Technology commitment | [API Architecture](./API-Architecture.md) |
| Exception to governance rules | Documents constraints and conditions | [API Governance](./API-Governance-and-Lifecycle.md) |
| Permanent removal of published capability | Irreversible consumer impact | [API Governance](./API-Governance-and-Lifecycle.md) |

---

## 23. Future Refactoring Triggers

| Trigger | Impact | Source Document |
|---------|--------|----------------|
| Single deployment bottleneck | API gateway separation | [API Architecture](./API-Architecture.md) |
| Frontend query complexity exceeds REST | GraphQL evaluation | [API Architecture](./API-Architecture.md) |
| Module extraction to services | gRPC for internal, distributed idempotency | [API Architecture](./API-Architecture.md), [Idempotency](./Idempotency-Concurrency-and-Retries.md) |
| Query complexity exceeds parameters | Advanced query language | [Pagination](./Pagination-Filtering-Sorting-and-Search.md) |
| Polling load on status endpoints | Webhook completion notifications | [Bulk and Async Operations](./Bulk-and-Asynchronous-Operations.md) |
| Webhook volume exceeds queue capacity | Fan-out architecture | [Webhook Architecture](./Webhook-Architecture.md) |
| SDK demand in unlisted languages | Additional SDK development | [API Documentation and SDK](./API-Documentation-and-SDK-Architecture.md) |
| Review cycle times bottleneck delivery | Self-service governance | [API Governance](./API-Governance-and-Lifecycle.md) |

---

## 24. Documentation Gaps

### Identified Unresolved Dependencies

| Gap | Required By | Resolution Phase | Notes |
|-----|------------|-----------------|-------|
| Event Architecture | Webhook Architecture, Resource Modeling | Future phase | Internal events trigger outbound webhooks. Event payload contracts needed. |
| Infrastructure Architecture | API Surfaces, Pagination, Webhooks | Future phase | API gateway, rate-limit infrastructure, queue processing, webhook delivery infrastructure. |
| Development Standards | All API documents | Future phase | Controller patterns, middleware, validation framework, testing patterns. |
| Module Specifications | All API documents | Future phase | Each module defines its endpoints following these standards. |
| Glossary update | API Contract Standards | Pending | API-specific terms (idempotency key, cursor, envelope, webhook, SDK) should be in glossary. |
| Architecture Roadmap | V1/Future split | Pending | Roadmap should reflect API capability delivery timeline. |

### Phase 9 Completeness Check

| Document | Created | Linked in README | Linked in MASTER_INDEX | Internal Links Valid |
|----------|:---:|:---:|:---:|:---:|
| API Architecture | Yes | Yes | Yes | Yes |
| API Surfaces and Boundaries | Yes | Yes | Yes | Yes |
| API Contract Standards | Yes | Yes | Yes | Yes |
| Resource and Operation Modeling | Yes | Yes | Yes | Yes |
| Request and Response Standards | Yes | Yes | Yes | Yes |
| Error Architecture | Yes | Yes | Yes | Yes |
| API Versioning and Compatibility | Yes | Yes | Yes | Yes |
| Pagination, Filtering, Sorting, and Search | Yes | Yes | Yes | Yes |
| Idempotency, Concurrency, and Retries | Yes | Yes | Yes | Yes |
| Bulk and Asynchronous Operations | Yes | Yes | Yes | Yes |
| Webhook Architecture | Yes | Yes | Yes | Yes |
| API Documentation and SDK Architecture | Yes | Yes | Yes | Yes |
| API Governance and Lifecycle | Yes | Yes | Yes | Yes |
| API Impact Matrix (this document) | Yes | Yes | Yes | Yes |

---

## Implementation Impact Across Surfaces

| Surface | API Requirements | Key Documents |
|---------|------------------|---------------|
| **Backend** | All API standards. Module-scoped endpoints. Domain validation. Tenant filtering. Idempotency. Concurrency. Envelope. Error codes. | All Phase 9 documents |
| **Frontend** | Consumes APIs through approved envelope. Handles errors by code. Implements pagination. Manages auth tokens. | [Request and Response Standards](./Request-and-Response-Standards.md), [Error Architecture](./Error-Architecture.md), [Pagination](./Pagination-Filtering-Sorting-and-Search.md) |
| **SDKs** | Generated from OpenAPI. Typed errors. Retry logic. Pagination helpers. Idempotency helpers. Version-aligned. | [API Documentation and SDK](./API-Documentation-and-SDK-Architecture.md) |
| **Administrative Portal** | Admin surface API consumption. Bulk operations. Export/import management. Webhook subscription management. | [API Surfaces](./API-Surfaces-and-Boundaries.md), [Bulk and Async Operations](./Bulk-and-Asynchronous-Operations.md), [Webhook Architecture](./Webhook-Architecture.md) |
| **Storefront** | Storefront surface. Anonymous browsing. Cursor pagination. Cart/checkout with idempotency. | [API Surfaces](./API-Surfaces-and-Boundaries.md), [Pagination](./Pagination-Filtering-Sorting-and-Search.md), [Idempotency](./Idempotency-Concurrency-and-Retries.md) |
| **Customer Portal** | Customer surface. Own-data access. Order history pagination. | [API Surfaces](./API-Surfaces-and-Boundaries.md), [Pagination](./Pagination-Filtering-Sorting-and-Search.md) |
| **Developer Portal** | API documentation hosting. OpenAPI specs. SDK downloads. Changelog. Key management. | [API Documentation and SDK](./API-Documentation-and-SDK-Architecture.md) |
| **Background Workers** | Async operation processing. Webhook delivery. Import processing. Export generation. | [Bulk and Async Operations](./Bulk-and-Asynchronous-Operations.md), [Webhook Architecture](./Webhook-Architecture.md) |
| **Webhooks** | Outbound delivery infrastructure. Inbound receiver endpoints. Signing. Verification. Retry. | [Webhook Architecture](./Webhook-Architecture.md) |
| **Events and Queues** | Webhook delivery queue. Async operation queue. Event-triggered webhook selection. | [Webhook Architecture](./Webhook-Architecture.md), [Bulk and Async Operations](./Bulk-and-Asynchronous-Operations.md) |
| **Search** | Search endpoint (separate from transactional filtering). Eventually consistent index. | [Pagination](./Pagination-Filtering-Sorting-and-Search.md) |
| **Reporting** | Reporting queries use same pagination/filtering patterns. Freshness communicated. | [Pagination](./Pagination-Filtering-Sorting-and-Search.md) |
| **Imports and Exports** | Async processing. File validation. Result retention. Download security. | [Bulk and Async Operations](./Bulk-and-Asynchronous-Operations.md) |
| **Integrations** | Integration surface. Scoped credentials. Idempotency. Webhook inbound. | [API Surfaces](./API-Surfaces-and-Boundaries.md), [Idempotency](./Idempotency-Concurrency-and-Retries.md), [Webhook Architecture](./Webhook-Architecture.md) |
| **Infrastructure** | Rate limiting. TLS termination. Correlation ID injection. API gateway (future). | [API Architecture](./API-Architecture.md), [API Surfaces](./API-Surfaces-and-Boundaries.md) |
| **CI/CD** | Contract test execution. Spec validation. Breaking-change detection. Documentation generation. | [API Documentation and SDK](./API-Documentation-and-SDK-Architecture.md), [API Governance](./API-Governance-and-Lifecycle.md) |
| **Support Tooling** | Correlation ID lookup. Delivery log access. Operation status visibility. | [Error Architecture](./Error-Architecture.md), [Webhook Architecture](./Webhook-Architecture.md) |
| **Public Documentation** | Developer portal content. API reference. Guides. Changelog. SDK docs. | [API Documentation and SDK](./API-Documentation-and-SDK-Architecture.md) |

---

## Version Gate

| Version | API Architecture Gate |
|---------|---------------------|
| V1 | All 14 Phase 9 documents complete. REST/JSON with approved envelope. URL path versioning. All surfaces operational. Authentication and authorization on all endpoints. Tenant resolution per-surface. Idempotency for financial/inventory. Optimistic concurrency. Cursor pagination. Structured errors. Webhooks (inbound/outbound). .NET and TypeScript SDKs. OpenAPI specification generated. Contract tests. API ownership assigned. Governance process operational. |
| V2 | API gateway separation. Enhanced SDKs (PHP, Python). Interactive developer portal. Webhook management API. Event streaming evaluated. Field selection. Automated breaking-change detection. |
| V3 | GraphQL evaluated. gRPC for extracted services. Multi-region routing. Date-based versioning evaluated. API marketplace. |

---

## Decision Summary

| Decision | Rationale |
|----------|-----------|
| Single impact matrix for API phase | Distributed traceability is hard to validate. Consolidated matrix ensures completeness. |
| Every matrix entry links to authoritative document | Traceability requires source attribution. Claims without links are unverifiable. |
| Documentation gaps explicitly identified | Honest gap identification enables future phases to know what they must address. |
| Implementation surfaces mapped to documents | Implementers need a lookup from their component to relevant API requirements. |
| ADR triggers listed proactively | Teams know when they must write an ADR. |

---

## Architecture Impact

| Concern | Impact |
|---------|--------|
| Phase completeness | This matrix validates that Phase 9 is complete and internally consistent |
| Cross-phase traceability | Future phases can identify which API requirements they must consume |
| Implementation guidance | Every platform surface knows which API documents apply to it |
| Governance | ADR triggers and refactoring triggers provide governance guardrails |
| Gap visibility | Identified gaps become work items for future phases |

---

## Implementation Impact

| Area | Impact |
|------|--------|
| Module teams | Use this matrix to identify all API responsibilities for their domain |
| Platform team | Use implementation surface matrix to ensure infrastructure meets all API requirements |
| Security team | Use security-testing matrix to validate coverage |
| Operations | Use webhook, bulk, and monitoring matrices for operational planning |
| Architecture | Use gap analysis and ADR triggers for future phase planning |

---

## Security Responsibilities

| Role | Matrix Responsibilities |
|------|----------------------|
| Chief API Architect | Maintains this matrix. Validates phase completeness. Identifies gaps. |
| Module Teams | Validate their surface entries are accurate. Flag missing responsibilities. |
| Security Team | Validate security-testing matrix completeness. Ensure no gaps in auth/authz coverage. |
| Operations | Validate webhook, bulk, and monitoring entries match operational reality. |
| Platform Team | Validate infrastructure-related entries are implementable. |

---

## Multi-Tenancy Responsibilities

| Responsibility | Matrix Coverage |
|---------------|----------------|
| Tenant resolution per surface | Matrix 6 (Tenant-Resolution Requirements) |
| Authorization per surface | Matrix 5 (Authorization Requirements) |
| Tenant isolation in queries | Matrix 11 (Pagination and Query Requirements) |
| Tenant isolation in bulk ops | Matrix 14 (Bulk-Operation Requirements) |
| Webhook tenant resolution | Matrix 15 (Webhook Requirements) |

---

## Out of Scope

This matrix does not:

- Define new architecture (it traces existing decisions).
- Override individual document decisions (it references them).
- Provide implementation guidance beyond document references.
- Prescribe tooling for traceability management.
- Define module-level API contracts.

---

## Future Considerations

- **Automated traceability** — Tooling that validates links and completeness automatically.
- **Living matrix** — Auto-generated from document metadata as the repository grows.
- **Module-level matrices** — Each module specification includes its own API impact subset.
- **Cross-phase matrices** — Similar matrices for Event and Infrastructure phases.
- **Compliance traceability** — Regulatory requirements mapped to API controls.

---

## Future Refactoring Triggers

This document should be revisited when:

- New API architecture documents are added (extend all matrices).
- Module specifications are written (validate surface ownership).
- Event Architecture phase begins (validate webhook/event alignment).
- Infrastructure Architecture phase begins (validate hosting requirements).
- Development Standards phase begins (validate implementation patterns).
- ADRs create exceptions to API architecture rules (update affected matrices).

---

## Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 0.1 | 2026-07-21 | Chief API Architect | Initial draft — Phase 9 completion matrix |
