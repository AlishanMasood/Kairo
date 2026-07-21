# Request and Response Standards

## Metadata

| Field | Value |
|-------|-------|
| Title | Kairo API Request and Response Standards |
| Document ID | KAI-API-005 |
| Status | Draft |
| Version | 0.1 |
| Target Release | V1 |
| Owner | API Consistency and Client Experience Architect |
| Created | 2026-07-21 |
| Last Updated | 2026-07-21 |
| Reviewers | TODO |
| Related Documents | [API Architecture](./API-Architecture.md), [API Contract Standards](./API-Contract-Standards.md), [API Surfaces and Boundaries](./API-Surfaces-and-Boundaries.md), [Data Classification and Sensitivity](../Data/Data-Classification-and-Sensitivity.md), [Transaction and Consistency Architecture](../Data/Transaction-and-Consistency-Architecture.md) |
| Dependencies | [API Architecture](./API-Architecture.md), [API Contract Standards](./API-Contract-Standards.md) |
| Forward References | Error Architecture (future document in this phase) |

---

## Applicable Version

This document defines V1 request and response standards. All modules must produce responses conforming to these conventions. The standards ensure that API consumers experience a single, consistent platform regardless of which module serves a request.

---

## Purpose

This document defines how requests are structured, how responses are formatted, how success and failure are communicated, and how metadata, correlation, and asynchronous operations are represented across all Kairo API surfaces.

Predictability is the foundation of developer experience. A developer who has integrated with one Kairo module should immediately understand responses from every other module. This document ensures that predictability by defining the response envelope, status conventions, error formatting, and metadata placement that all modules follow.

---

## Scope

This document covers:

- Request and response structure conventions.
- The approved response envelope (isSuccess, data, message, errors).
- Success, failure, collection, empty, mutation, and async response patterns.
- Metadata, correlation, idempotency, and content conventions.
- Validation, redaction, filtering, and expansion direction.
- HTTP semantic and status-category alignment.

This document does not cover:

- Concrete endpoint definitions or payloads (module specifications).
- Detailed error taxonomy or error codes (Error Architecture — forward reference).
- Pagination algorithm implementation (API patterns document).
- Rate-limiting response headers (infrastructure configuration).
- OpenAPI generation tooling (development standards).

---

## Explicit Prohibitions

| # | Prohibition | Rationale |
|---|-------------|-----------|
| 1 | **Returning internal exception details** | Stack traces, internal class names, and database error messages expose implementation details and create security risks. |
| 2 | **Inconsistent success flags and HTTP outcomes** | If `isSuccess` is `true`, the HTTP status must be 2xx. If `isSuccess` is `false`, the HTTP status must be 4xx or 5xx. Never mix. |
| 3 | **Returning persistence entities directly** | Persistence models contain internal fields, audit metadata, and structural details that do not belong in API responses. |
| 4 | **Returning sensitive fields by default** | Personal data, financial details, and security-related fields require explicit inclusion decisions per [Data Classification and Sensitivity](../Data/Data-Classification-and-Sensitivity.md). |
| 5 | **Using success responses for failed business operations** | A rejected order is not 200 OK. A failed payment is not a successful response with an error message in the body. HTTP status reflects the outcome. |
| 6 | **Using human-readable messages as the only machine-readable error information** | Messages are for humans. Machines need stable error codes/types that do not change when messages are reworded or localized. |

---

## Approved Response Envelope

All API responses use this envelope structure:

| Field | Type | Present | Purpose |
|-------|------|---------|---------|
| `isSuccess` | boolean | Always | Indicates whether the operation succeeded |
| `data` | object or array or null | Always | The response payload (resource, collection, or null) |
| `message` | string or null | Always | Human-readable summary (success confirmation or error summary) |
| `errors` | array or null | Always | Machine-readable error details (null when successful) |

### Envelope Rules

| Rule | Detail |
|------|--------|
| Always present | All four fields are present in every response. No conditional field omission. |
| Consistent typing | `isSuccess` is always boolean. `data` is always object/array/null. `message` is always string/null. `errors` is always array/null. |
| HTTP alignment | `isSuccess: true` ↔ HTTP 2xx. `isSuccess: false` ↔ HTTP 4xx/5xx. No exceptions. |
| Null conventions | `data` is null on error responses. `errors` is null on success responses. |

---

## 1. Request Structure

| Convention | Detail |
|-----------|--------|
| Body format | JSON (UTF-8) for all request bodies |
| Content-Type header | `application/json` required when body is present |
| Query parameters | Used for filtering, sorting, pagination, and expansion on GET requests |
| Path parameters | Used for resource identification |
| Headers | Used for metadata (correlation, idempotency, content negotiation, authentication) |
| No body on GET/DELETE | GET and DELETE requests must not include a request body |
| Size limits | Request bodies have documented maximum sizes (module-specific). Exceeding returns 413. |

---

## 2. Response Structure

All responses follow the approved envelope regardless of outcome:

| Scenario | `isSuccess` | `data` | `message` | `errors` | HTTP Status |
|----------|:-----------:|--------|-----------|----------|:-----------:|
| Successful resource retrieval | `true` | Resource object | null or confirmation | `null` | 200 |
| Successful creation | `true` | Created resource | null or confirmation | `null` | 201 |
| Successful deletion | `true` | `null` | Confirmation message | `null` | 200 |
| Successful async acceptance | `true` | Operation status object | Acceptance message | `null` | 202 |
| Validation failure | `false` | `null` | Summary of failure | Error array | 400 |
| Authentication failure | `false` | `null` | "Authentication required" | Error array | 401 |
| Authorization failure | `false` | `null` | "Insufficient permissions" | Error array | 403 |
| Resource not found | `false` | `null` | "Resource not found" | Error array | 404 |
| Business rule violation | `false` | `null` | Summary of violation | Error array | 409 or 422 |
| Internal error | `false` | `null` | Generic error message | Error array (no internals) | 500 |

---

## 3. Success Envelope

When `isSuccess` is `true`:

| Field | Value |
|-------|-------|
| `isSuccess` | `true` |
| `data` | The response payload (resource object, collection, or null for 204-equivalent) |
| `message` | Optional human-readable confirmation (may be `null` for routine operations) |
| `errors` | `null` |

### Conceptual Example (non-binding)

```json
{
  "isSuccess": true,
  "data": {
    "id": "prod_01H5EXAMPLE",
    "name": "Example Widget",
    "status": "active",
    "price": { "amount": "29.99", "currency": "USD" },
    "createdAt": "2026-07-21T10:00:00Z"
  },
  "message": null,
  "errors": null
}
```

---

## 4. Failure Envelope

When `isSuccess` is `false`:

| Field | Value |
|-------|-------|
| `isSuccess` | `false` |
| `data` | `null` |
| `message` | Human-readable error summary |
| `errors` | Array of structured error objects |

### Error Object Structure

Each error in the `errors` array:

| Field | Type | Purpose |
|-------|------|---------|
| `code` | string | Stable, machine-readable error identifier (e.g., `"VALIDATION_FAILED"`, `"INSUFFICIENT_STOCK"`) |
| `field` | string or null | The request field this error relates to (null for non-field errors) |
| `message` | string | Human-readable description of this specific error |
| `meta` | object or null | Additional machine-readable context (e.g., `{ "min": 1, "max": 100 }`) |

### Conceptual Example (non-binding)

```json
{
  "isSuccess": false,
  "data": null,
  "message": "Validation failed",
  "errors": [
    {
      "code": "FIELD_REQUIRED",
      "field": "name",
      "message": "Product name is required",
      "meta": null
    },
    {
      "code": "INVALID_FORMAT",
      "field": "price.amount",
      "message": "Amount must be a valid decimal string",
      "meta": { "example": "29.99" }
    }
  ]
}
```

---

## 5. Single-Resource Responses

| Convention | Detail |
|-----------|--------|
| `data` contains | The resource object directly |
| Shape | Same resource shape regardless of which endpoint returns it |
| Self-reference | Resource includes its own `id` |
| Timestamps | `createdAt` and `updatedAt` included on all resources |
| Relationships | Referenced by ID. Expanded only when explicitly requested. |

---

## 6. Collection Responses

| Convention | Detail |
|-----------|--------|
| `data` contains | An object with `items` array and `pagination` metadata |
| Items | Array of resources (same shape as single-resource responses) |
| Empty collection | `items` is an empty array `[]`, never `null` |
| Pagination | Always present. Includes cursor, page size, and navigation info. |

### Collection Data Structure

| Field | Type | Purpose |
|-------|------|---------|
| `items` | array | The collection of resources |
| `pagination` | object | Pagination metadata |
| `pagination.cursor` | string or null | Cursor for the next page (null if no more pages) |
| `pagination.pageSize` | integer | Number of items per page |
| `pagination.hasMore` | boolean | Whether more items exist beyond this page |
| `pagination.totalCount` | integer or null | Total count (provided only when feasible and requested) |

### Conceptual Example (non-binding)

```json
{
  "isSuccess": true,
  "data": {
    "items": [
      { "id": "prod_01H5AAA", "name": "Widget A", "status": "active" },
      { "id": "prod_01H5BBB", "name": "Widget B", "status": "active" }
    ],
    "pagination": {
      "cursor": "eyJpZCI6InByb2RfMDFINUJCQiJ9",
      "pageSize": 20,
      "hasMore": true,
      "totalCount": null
    }
  },
  "message": null,
  "errors": null
}
```

---

## 7. Empty Responses

| Scenario | HTTP Status | `data` | Notes |
|----------|:-----------:|--------|-------|
| Successful deletion | 200 | `null` | Envelope with `isSuccess: true`, `data: null` |
| Empty collection | 200 | `{ "items": [], "pagination": {...} }` | Collection structure with empty items array |
| No content needed | 200 | `null` | Used when the operation succeeded but has no meaningful return |

Note: The platform uses 200 with the standard envelope for deletion (not 204), because the envelope provides consistent structure for all responses.

---

## 8. Mutation Responses

| Convention | Detail |
|-----------|--------|
| Creation | Returns the created resource in `data`. HTTP 201. |
| Update | Returns the updated resource in `data`. HTTP 200. |
| Business action | Returns the resulting resource state in `data`. HTTP 200. |
| Deletion | Returns `data: null` with confirmation message. HTTP 200. |
| Always current state | Mutation responses reflect the resource state after the operation, not before. |

---

## 9. Asynchronous Operation Responses

**Asynchronous acceptance and completion are distinct.**

| Phase | HTTP Status | `data` Content |
|-------|:-----------:|----------------|
| Acceptance | 202 | Operation status object (pending) |
| Polling (in progress) | 200 | Operation status object (processing) |
| Polling (completed) | 200 | Operation status object (completed + result) |
| Polling (failed) | 200 | Operation status object (failed + errors) |

### Operation Status Object

| Field | Type | Purpose |
|-------|------|---------|
| `operationId` | string | Unique identifier for this operation |
| `status` | string | `"pending"`, `"processing"`, `"completed"`, `"failed"` |
| `progress` | integer or null | Percentage (0-100) if measurable |
| `result` | object or null | The result when completed |
| `errors` | array or null | Errors when failed |
| `createdAt` | string | When the operation was initiated |
| `updatedAt` | string | Last status change |
| `statusUrl` | string | URL to poll for updates |

### Conceptual Example — Acceptance (non-binding)

```json
{
  "isSuccess": true,
  "data": {
    "operationId": "op_01H5EXAMPLE",
    "status": "pending",
    "progress": null,
    "result": null,
    "errors": null,
    "createdAt": "2026-07-21T10:00:00Z",
    "updatedAt": "2026-07-21T10:00:00Z",
    "statusUrl": "/v1/operations/op_01H5EXAMPLE"
  },
  "message": "Import accepted for processing",
  "errors": null
}
```

---

## 10. Metadata

| Category | Location | Examples |
|----------|----------|---------|
| Request metadata | Headers | Correlation ID, Idempotency-Key, Content-Type, Accept |
| Response metadata | Headers | Correlation ID, Rate-Limit info, Cache-Control |
| Collection metadata | Within `data.pagination` | Cursor, page size, hasMore, totalCount |
| Resource metadata | Within resource fields | `createdAt`, `updatedAt`, `version` |

Metadata is never mixed into the top-level envelope fields. It lives in headers or structurally separated within `data`.

---

## 11. Correlation Identifiers

| Aspect | Detail |
|--------|--------|
| Header | `X-Correlation-Id` |
| Generation | Platform generates if not provided by client |
| Propagation | Flows through all internal processing, events, and logs |
| Response | Always returned in the response header |
| Purpose | Trace a request through the entire system for debugging and support |
| Format | String. Opaque to consumers. Platform generates as ULID or UUID. |

---

## 12. Request Identifiers

| Aspect | Detail |
|--------|--------|
| Header | `X-Request-Id` |
| Generation | Platform always generates (unique per request) |
| Response | Always returned in the response header |
| Purpose | Identify a specific request instance (distinct from correlation which may span multiple requests) |
| Logging | All log entries for this request include the request ID |

---

## 13. Idempotency Identifiers

| Aspect | Detail |
|--------|--------|
| Header | `Idempotency-Key` |
| When required | Financial mutations, inventory adjustments, order placement — any operation where duplicate execution is dangerous |
| Generation | Client-provided. Unique per intended operation. |
| Behavior | First request processes normally. Subsequent requests with same key return the stored result without re-executing. |
| TTL | Idempotency keys expire after a defined period (platform-configured) |
| Scope | Tenant-scoped. Same key from different tenants are independent. |
| Response | Same response as the original request (including status code) |

---

## 14. Content Types

| Direction | Header | Value |
|-----------|--------|-------|
| Request body | `Content-Type` | `application/json` |
| Response body | `Content-Type` | `application/json; charset=utf-8` |
| File upload | `Content-Type` | `multipart/form-data` |
| No body | — | Header omitted |

No other content types are supported for standard API operations in V1.

---

## 15. Character Encoding

| Rule | Detail |
|------|--------|
| Encoding | UTF-8 for all request and response bodies |
| BOM | No byte-order mark |
| Escaping | Standard JSON escaping rules apply |
| Emoji and Unicode | Fully supported. No ASCII-only restriction. |

---

## 16. Date and Time Representation

| Convention | Detail |
|-----------|--------|
| Format | ISO 8601: `"2026-07-21T10:30:00Z"` |
| Timezone | UTC always (suffix `Z`). See [API Contract Standards](./API-Contract-Standards.md). |
| Date-only | `"2026-07-21"` where only date is meaningful |
| Field naming | `*At` suffix for instants, `*Date` suffix for date-only |
| Precision | Seconds for business events. Milliseconds where needed. |

---

## 17. Monetary Representation

| Convention | Detail |
|-----------|--------|
| Structure | `{ "amount": "29.99", "currency": "USD" }` |
| Amount type | String (preserves precision) |
| Currency | ISO 4217 three-letter code |
| Never bare | Money is never a bare number without currency context |

See [API Contract Standards](./API-Contract-Standards.md) for full monetary conventions.

---

## 18. Boolean Representation

| Rule | Detail |
|------|--------|
| Values | `true` or `false` (JSON native booleans) |
| Never strings | Not `"true"` / `"false"`. Not `"1"` / `"0"`. |
| Never integers | Not `1` / `0`. |
| Naming | Positive assertion: `isActive`, `hasShipped`, `canCancel` |

---

## 19. Null and Missing Fields

| Scenario | Representation | Meaning |
|----------|---------------|---------|
| Value is intentionally empty | `"field": null` | Field exists, value is explicitly absent |
| Field is not applicable | Field omitted from response | Not relevant in this context |
| Empty collection | `"items": []` | Collection exists but has no members (never `null`) |
| Empty string | `"field": ""` | Value is empty string (distinct from null) |

| Rule | Detail |
|------|--------|
| Required response fields | Never null. Always have a value. |
| Optional response fields | May be null or omitted (document per field) |
| `errors` on success | `null` (not empty array) |
| `data` on failure | `null` (not empty object) |

---

## 20. Validation

| Convention | Detail |
|-----------|--------|
| Server-side authoritative | Server validates all input. Client validation is convenience only. |
| Early rejection | Invalid requests are rejected before business logic executes |
| Structured errors | Validation failures use the failure envelope with per-field error objects |
| All fields validated | All violations are reported in a single response (not one-at-a-time) |
| HTTP status | 400 Bad Request for request format/validation failures |
| Error code | Each validation error has a stable `code` (e.g., `"FIELD_REQUIRED"`, `"INVALID_FORMAT"`, `"VALUE_TOO_LONG"`) |

### Validation Error Conceptual Example (non-binding)

```json
{
  "isSuccess": false,
  "data": null,
  "message": "Request validation failed",
  "errors": [
    { "code": "FIELD_REQUIRED", "field": "name", "message": "Name is required", "meta": null },
    { "code": "VALUE_TOO_SHORT", "field": "sku", "message": "SKU must be at least 3 characters", "meta": { "min": 3 } }
  ]
}
```

---

## 21. Unknown Request Fields

| Rule | Detail |
|------|--------|
| Default behavior | Unknown fields in request bodies are ignored (forward compatibility) |
| Strict mode | Specific security-sensitive endpoints may reject unknown fields (documented per endpoint) |
| No silent processing | Unknown fields are never silently interpreted or stored |
| Logging | Unknown fields may be logged for monitoring (not in production response) |

---

## 22. Field Selection Direction

| Aspect | Detail |
|--------|--------|
| V1 direction | All resource fields are returned by default. No sparse fieldset support in V1. |
| Future | V2+ may support `fields` query parameter for response field selection |
| Rationale | V1 simplicity. Field selection adds implementation complexity. Standard responses are cacheable. |
| Sensitive fields | Sensitive fields are excluded by default regardless (not controlled by field selection) |

---

## 23. Expansion Direction

| Aspect | Detail |
|--------|--------|
| V1 direction | Limited expansion support. Specific expand options documented per endpoint. |
| Pattern | `?expand=customer,store` query parameter |
| Default | No expansion. References are IDs only by default. |
| Limits | Maximum expansion depth: 1 level. No recursive expansion. |
| Future | V2+ may support richer expansion with nested path notation |

---

## 24. Localization

| Aspect | Detail |
|--------|--------|
| Request | `Accept-Language` header for consumer language preference |
| Response messages | `message` field may be localized based on preference |
| Error codes | Error `code` values are never localized (machine-readable, stable) |
| Error messages | Error `message` values may be localized |
| Business data | Returned in the organization's configured language(s) |
| V1 | Single language per organization. Localized platform messages where available. |

---

## 25. Response Filtering

| Aspect | Detail |
|--------|--------|
| Collection filtering | Query parameters on collection endpoints: `?status=active&createdAfter=2026-01-01` |
| Consistent syntax | Same filter parameter conventions across all modules |
| Server-enforced | Filtering is server-side. Clients do not receive unfiltered data and filter locally. |
| Default filters | Active resources by default. Archived/deleted require explicit filter. |
| Documentation | Available filter parameters documented per collection endpoint |

---

## 26. Sensitive-Data Redaction

| Rule | Detail |
|------|--------|
| Not returned by default | Sensitive fields (full email, phone, payment details) are not in standard responses |
| Masked when included | Where partial visibility is appropriate: `"email": "j***@example.com"` |
| Explicit expansion | Full sensitive data requires explicit request and elevated authorization |
| Classification-driven | Redaction follows [Data Classification and Sensitivity](../Data/Data-Classification-and-Sensitivity.md) |
| Never in errors | Error responses never include sensitive data, even when describing the error context |
| Logs | Sensitive data is never included in response logging |

---

## 27. HTTP Semantic Direction

| HTTP Method | Semantics | Request Body | Idempotent |
|-------------|-----------|:---:|:---:|
| GET | Retrieve resource or collection | No | Yes |
| POST | Create resource or execute command | Yes | No (unless Idempotency-Key) |
| PUT | Full resource replacement (rare in Kairo) | Yes | Yes |
| PATCH | Partial resource update | Yes | No (unless Idempotency-Key) |
| DELETE | Remove resource | No | Yes |

| Rule | Detail |
|------|--------|
| GET is safe | Never produces side effects |
| POST for commands | Business actions use POST (even if they are conceptually idempotent — use Idempotency-Key header) |
| PUT is rare | Kairo prefers PATCH for updates and POST for business actions. Full replacement is rarely appropriate. |
| DELETE is logical | Soft-delete by default. Hard-delete only through lifecycle processes. |

---

## 28. Status-Category Direction

| Status Range | Category | Usage |
|-------------|----------|-------|
| 200 | Success | Successful retrieval, update, business action, deletion |
| 201 | Created | Successful resource creation |
| 202 | Accepted | Asynchronous operation accepted for processing |
| 400 | Bad Request | Request validation failure (malformed, missing fields, invalid format) |
| 401 | Unauthorized | Authentication missing or invalid |
| 403 | Forbidden | Authenticated but insufficient permissions or tenant mismatch |
| 404 | Not Found | Resource does not exist or not accessible to this tenant |
| 409 | Conflict | Business rule conflict (invalid state transition, concurrency conflict) |
| 413 | Payload Too Large | Request body exceeds size limit |
| 422 | Unprocessable Entity | Semantically invalid (valid format but business rules reject) |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Unexpected server failure (no implementation details exposed) |
| 503 | Service Unavailable | Temporary unavailability (maintenance, overload) |

### 409 versus 422 Guidance

| Status | Use When |
|--------|----------|
| 409 Conflict | The request conflicts with current resource state (invalid transition, concurrent modification, duplicate) |
| 422 Unprocessable Entity | The request is well-formed but violates business rules that are not state-conflict related |

---

## Warnings

Warnings communicate non-fatal issues that did not prevent the operation from succeeding.

| Aspect | Detail |
|--------|--------|
| Representation | Optional `warnings` field in the envelope (alongside `data`) |
| When present | Operation succeeded but there are conditions worth noting (deprecated field used, partial data, approaching limit) |
| Structure | Same format as `errors` array (code, field, message, meta) but does not indicate failure |
| `isSuccess` | Remains `true` — warnings do not make the response a failure |
| V1 scope | Minimal use in V1. Reserved for deprecation notices and approaching-limit warnings. |

### Conceptual Example (non-binding)

```json
{
  "isSuccess": true,
  "data": { "id": "prod_01H5EXAMPLE", "name": "Widget" },
  "message": null,
  "errors": null,
  "warnings": [
    { "code": "FIELD_DEPRECATED", "field": "legacySku", "message": "legacySku is deprecated. Use sku instead.", "meta": { "sunset": "2027-01-01" } }
  ]
}
```

---

## Version Gate

| Version | Request and Response Standards Gate |
|---------|-------------------------------------|
| V1 | Approved envelope on all responses. Consistent HTTP status usage. Structured error objects with stable codes. Correlation and request IDs on all requests. Idempotency-Key for financial/inventory mutations. UTC dates. Money as object. Sensitive data redacted by default. Collection pagination always present. Async operations use 202 + polling. |
| V2 | Field selection support. Enhanced expansion. Webhook completion notifications for async operations. Localized error messages. Warning framework expanded. |
| V3 | Streaming responses for large datasets. Event-driven response alternatives. GraphQL response conventions (if adopted). |

---

## Decision Summary

| Decision | Rationale |
|----------|-----------|
| Fixed envelope with isSuccess/data/message/errors | Predictable structure on every response. Consumers can write generic handling logic. No guessing where the payload or errors are. |
| isSuccess aligns with HTTP status | Eliminates the confusing anti-pattern of 200 OK with error body. Status code and envelope agree. |
| Structured error objects with stable codes | Machines need codes. Humans need messages. Both are provided. Codes are stable for programmatic handling; messages can be localized or reworded. |
| Collection as object with items + pagination | Consistent structure. Pagination metadata is always available. Empty collection is clearly distinct from error. |
| 200 for deletion (not 204) | Maintains consistent envelope. 204 has no body, which breaks the envelope pattern. 200 with null data is consistent. |
| Null for absent envelope fields | `errors: null` on success is clearer than omitting `errors`. Consumers always know the field exists. |
| Correlation ID in header (not body) | Infrastructure concern. Does not pollute business payload. Can be added/read without parsing JSON body. |
| Warnings as separate field (not in errors) | Warnings are informational, not failures. Mixing them in errors confuses consumers who handle errors programmatically. |

---

## Alternatives Considered

| Alternative | Rejected Because |
|------------|-----------------|
| No envelope (bare resource responses) | No consistent place for errors, metadata, or success indication. Every consumer must handle each endpoint differently. |
| Envelope with HTTP 200 for all responses | Hides failure from HTTP-aware infrastructure (proxies, CDNs, monitoring). Breaks standard HTTP semantics. |
| Errors as strings (not objects) | Not machine-parseable. Cannot identify which field failed. Cannot localize without parsing natural language. |
| 204 for deletion | No response body. Breaks envelope consistency. Consumers must special-case deletion responses. |
| Omit null fields from envelope | Consumers cannot distinguish "field is null" from "field was omitted due to an error in serialization." Always-present is predictable. |
| Error codes as integers | Not self-documenting. Require lookup tables. String codes are immediately meaningful. |
| Pagination in headers | Not discoverable. Awkward for nested objects. Pagination belongs with the collection data it describes. |
| Success flag embedded in data | Mixes meta-information with business data. The envelope separates concerns cleanly. |

---

## Architecture Impact

| Concern | Impact |
|---------|--------|
| Module design | Every module must produce responses conforming to the envelope. Response mapping is mandatory, not optional. |
| Error handling | All modules use the same error object structure. Error codes are module-namespaced but structurally consistent. |
| Middleware | Platform provides envelope wrapping, correlation ID injection, and error formatting as cross-cutting middleware. |
| Caching | Consistent structure enables generic cache key strategies. Collection pagination enables incremental caching. |
| SDK generation | Consistent envelope enables generic SDK response handling. One deserialization strategy across all endpoints. |
| Monitoring | Consistent `isSuccess` enables platform-wide success/failure monitoring without per-endpoint configuration. |

---

## Implementation Impact

| Area | Impact |
|------|--------|
| Modules | Must return domain results that the platform wraps in the envelope. Must use structured error codes. Must not throw unhandled exceptions that bypass the envelope. |
| Platform | Must provide envelope middleware. Must inject correlation/request IDs. Must format unhandled exceptions as safe error responses (no internals leaked). |
| Frontend | Can build a generic response handler that works for all endpoints. Handles errors consistently. Reads pagination uniformly. |
| SDKs | Generated SDKs expose a consistent response type with `isSuccess`, typed data, and error access. |
| Testing | Response structure tests verify envelope compliance per endpoint. Error format tests verify code stability. |
| Documentation | All endpoint docs show the envelope structure. Error codes are documented in a registry. |

---

## Security Responsibilities

| Role | Request/Response Responsibilities |
|------|----------------------------------|
| API Consistency Architect | Defines and evolves envelope and response conventions. Reviews compliance. |
| Module Teams | Produce responses conforming to standards. Use structured error codes. Handle sensitive data redaction. |
| Platform Team | Provides envelope middleware. Injects correlation IDs. Catches unhandled exceptions. Sanitizes error details. |
| Security Team | Validates no internal details leak in error responses. Reviews sensitive data redaction. Validates correlation ID propagation. |

---

## Multi-Tenancy Responsibilities

| Responsibility | Detail |
|---------------|--------|
| Envelope is tenant-unaware | The envelope structure is identical for all tenants |
| Data is tenant-scoped | `data` content is always scoped to the authenticated tenant |
| Errors do not leak tenant info | Error messages never reference other tenants or cross-tenant details |
| Correlation spans tenant context | Correlation IDs work within a single tenant's request chain |

---

## Out of Scope

This document does not define:

- Concrete endpoint payloads per module (module specifications).
- Complete error code taxonomy (Error Architecture document — forward reference).
- Pagination cursor format or algorithm (API patterns document).
- Rate-limit headers and retry guidance (infrastructure/patterns document).
- WebSocket or streaming response conventions (future — V2+).
- Specific middleware implementation (development standards).

---

## Future Considerations

- **Streaming responses** — For large datasets that exceed practical JSON body sizes.
- **Server-Sent Events** — Real-time response streaming for long-running operations.
- **Conditional requests** — ETag-based conditional GET/PUT for caching and concurrency.
- **Content negotiation** — Supporting additional response formats beyond JSON.
- **Response compression** — gzip/br compression for large response bodies.
- **Hypermedia** — HATEOAS-style links for discoverability (evaluated, not required V1).

---

## Future Refactoring Triggers

This document should be revisited when:

- Envelope structure creates performance issues for high-volume endpoints (trigger for lightweight response option).
- SDK generation reveals envelope handling pain points (trigger for envelope refinement).
- Streaming or real-time needs emerge (trigger for non-JSON response conventions).
- Error taxonomy grows beyond manageable size (trigger for error categorization refinement).
- Multi-language localization is needed for error messages (trigger for localization framework).
- GraphQL adoption changes response structure expectations (trigger for alternative envelope).

---

## Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 0.1 | 2026-07-21 | API Consistency and Client Experience Architect | Initial draft — request and response standards |
