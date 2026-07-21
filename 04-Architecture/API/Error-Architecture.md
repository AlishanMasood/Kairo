# Error Architecture

## Metadata

| Field | Value |
|-------|-------|
| Title | Kairo API Error and Failure Semantics Architecture |
| Document ID | KAI-API-006 |
| Status | Draft |
| Version | 0.1 |
| Target Release | V1 |
| Owner | API Error and Failure Semantics Architect |
| Created | 2026-07-21 |
| Last Updated | 2026-07-21 |
| Reviewers | TODO |
| Related Documents | [Request and Response Standards](./Request-and-Response-Standards.md), [API Security](../Security/API-Security.md), [Incident Response](../Security/Incident-Response.md), [Audit and Security Monitoring](../Security/Audit-and-Security-Monitoring.md), [API Architecture](./API-Architecture.md), [API Contract Standards](./API-Contract-Standards.md), [Transaction and Consistency Architecture](../Data/Transaction-and-Consistency-Architecture.md) |
| Dependencies | [Request and Response Standards](./Request-and-Response-Standards.md), [API Security](../Security/API-Security.md) |

---

## Applicable Version

This document defines V1 error semantics. All modules must produce errors conforming to these conventions. Error codes defined here are stable contracts — they may be extended but not changed or removed without versioning.

---

## Purpose

This document defines how the Kairo platform communicates failures to API consumers. It establishes error categories, machine-readable codes, disclosure rules, retryability, and the relationship between API errors and internal logging, audit, and security monitoring.

Errors are a critical part of the API contract. A developer who receives an error must know: what went wrong, whether it is their fault or the platform's, whether they can retry, and what action to take. Simultaneously, error responses must not leak internal implementation details, sensitive resource existence, or security-relevant information.

---

## Scope

This document covers:

- Error categories and their semantic meaning.
- Machine-readable error codes and their stability guarantees.
- Error disclosure rules (what is safe to communicate externally).
- Retryability classification.
- Field-level error identification.
- Provider-error normalization.
- Partial-operation and batch-error behavior.
- Relationship between API errors and logging/audit/security systems.

This document does not cover:

- Internal exception class hierarchies (development standards).
- Specific module error code registries (module specifications).
- Logging infrastructure or log levels (operations documentation).
- Incident response procedures (see [Incident Response](../Security/Incident-Response.md)).
- Retry policy configuration values (deployment configuration).

---

## Mandatory Principles

| # | Principle |
|---|-----------|
| 1 | Error codes are stable contracts |
| 2 | Human-readable messages are not stable parsing contracts |
| 3 | Internal exceptions are never exposed directly |
| 4 | Authorization failures must not leak sensitive resource existence |
| 5 | Tenant failures fail safely |
| 6 | Provider-specific failures are normalized while preserving internal diagnostic traceability |
| 7 | Retryable and non-retryable failures must be distinguishable |
| 8 | Validation errors identify fields when safe |
| 9 | Financial failures require clear state and reconciliation direction |
| 10 | Partial success must never be ambiguous |

---

## Error Envelope

All errors use the approved response envelope (see [Request and Response Standards](./Request-and-Response-Standards.md)):

```
{
  "isSuccess": false,
  "data": null,
  "message": "<human-readable summary>",
  "errors": [
    {
      "code": "<STABLE_ERROR_CODE>",
      "field": "<field_name or null>",
      "message": "<human-readable detail>",
      "meta": { <additional machine-readable context> }
    }
  ]
}
```

| Field | Stability | Purpose |
|-------|-----------|---------|
| `code` | **Stable** — machine contract. Never changes meaning. | Programmatic error handling |
| `field` | Stable per validation context | Identifies which input field caused the error |
| `message` | **Unstable** — may be reworded, localized, or clarified at any time | Human readability |
| `meta` | Semi-stable — keys are stable, values may vary | Additional machine-readable context |

---

## Error Categories

### Authentication Errors

| Aspect | Detail |
|--------|--------|
| Cause | Missing, expired, malformed, or revoked credentials |
| HTTP Status | 401 Unauthorized |
| Retryable | Yes — with fresh credentials |
| Disclosure | Safe to indicate authentication is required. Do not specify which credential component failed. |
| Codes | `AUTH_REQUIRED`, `AUTH_TOKEN_EXPIRED`, `AUTH_TOKEN_INVALID`, `AUTH_TOKEN_REVOKED` |

### Authorization Errors

| Aspect | Detail |
|--------|--------|
| Cause | Authenticated caller lacks permission for the requested operation or resource |
| HTTP Status | 403 Forbidden |
| Retryable | No — unless permissions change |
| Disclosure | Indicate insufficient permission. **Never reveal whether the resource exists if the caller lacks access.** |
| Codes | `FORBIDDEN`, `INSUFFICIENT_PERMISSIONS`, `OPERATION_NOT_ALLOWED` |

### Tenant-Context Errors

| Aspect | Detail |
|--------|--------|
| Cause | Tenant context cannot be resolved, or caller is not a member of the required tenant |
| HTTP Status | 403 Forbidden |
| Retryable | No — unless tenant membership changes |
| Disclosure | Indicate context failure. Do not reveal valid tenant identifiers. **Tenant failures fail safely** — ambiguous tenant context is denied, not guessed. |
| Codes | `TENANT_CONTEXT_REQUIRED`, `TENANT_ACCESS_DENIED`, `TENANT_NOT_ACTIVE` |

### Validation Errors

| Aspect | Detail |
|--------|--------|
| Cause | Request data does not meet format, type, or constraint requirements |
| HTTP Status | 400 Bad Request |
| Retryable | Yes — with corrected input |
| Disclosure | **Validation errors identify fields when safe.** Report all field violations in a single response. |
| Codes | `VALIDATION_FAILED`, `FIELD_REQUIRED`, `FIELD_INVALID_FORMAT`, `FIELD_TOO_SHORT`, `FIELD_TOO_LONG`, `FIELD_OUT_OF_RANGE`, `FIELD_INVALID_VALUE` |
| Field identification | Each error object includes the `field` path (e.g., `"price.amount"`, `"lineItems[0].quantity"`) |

### Resource-Not-Found Errors

| Aspect | Detail |
|--------|--------|
| Cause | The requested resource does not exist or is not accessible to the authenticated caller/tenant |
| HTTP Status | 404 Not Found |
| Retryable | No (unless the resource is expected to be created soon) |
| Disclosure | **Do not distinguish between "does not exist" and "exists but you cannot access it."** Both return 404. This prevents enumeration attacks. |
| Codes | `RESOURCE_NOT_FOUND` |

### Conflict Errors

| Aspect | Detail |
|--------|--------|
| Cause | The request conflicts with current resource state (duplicate, invalid transition) |
| HTTP Status | 409 Conflict |
| Retryable | Depends on conflict type (duplicate: no; transient state: possibly) |
| Disclosure | Safe to indicate the nature of the conflict (duplicate SKU, invalid status transition). Include current state where helpful. |
| Codes | `CONFLICT_DUPLICATE`, `CONFLICT_STATE_INVALID`, `CONFLICT_BUSINESS_RULE` |

### Concurrency Errors

| Aspect | Detail |
|--------|--------|
| Cause | Optimistic concurrency conflict — resource was modified between read and write |
| HTTP Status | 409 Conflict |
| Retryable | Yes — re-read current state and retry |
| Disclosure | Indicate concurrency conflict. Include current version/ETag for re-read. |
| Codes | `CONCURRENCY_CONFLICT` |
| Meta | `{ "currentVersion": "..." }` |

### Idempotency Conflicts

| Aspect | Detail |
|--------|--------|
| Cause | Idempotency key matches a previous request but with different parameters |
| HTTP Status | 409 Conflict |
| Retryable | No — the key is already associated with a different operation |
| Disclosure | Indicate key conflict. Do not expose details of the original request. |
| Codes | `IDEMPOTENCY_KEY_CONFLICT` |

### State-Transition Errors

| Aspect | Detail |
|--------|--------|
| Cause | Requested operation is not valid for the resource's current state |
| HTTP Status | 409 Conflict |
| Retryable | No — unless the state changes externally |
| Disclosure | Safe to indicate current state and valid transitions from that state |
| Codes | `STATE_TRANSITION_INVALID` |
| Meta | `{ "currentState": "...", "validTransitions": ["..."] }` |

### Rate-Limit Errors

| Aspect | Detail |
|--------|--------|
| Cause | Consumer has exceeded their allowed request rate |
| HTTP Status | 429 Too Many Requests |
| Retryable | Yes — after the indicated wait period |
| Disclosure | Indicate rate limit exceeded. Provide retry-after guidance. |
| Codes | `RATE_LIMIT_EXCEEDED` |
| Headers | `Retry-After: <seconds>` |
| Meta | `{ "retryAfter": <seconds>, "limit": <requests>, "window": "<duration>" }` |

### Quota Errors

| Aspect | Detail |
|--------|--------|
| Cause | Consumer has exceeded a resource or usage quota (not rate, but total allocation) |
| HTTP Status | 403 Forbidden or 429 (context-dependent) |
| Retryable | No — unless quota is increased |
| Disclosure | Indicate quota exceeded. Identify which quota. |
| Codes | `QUOTA_EXCEEDED` |
| Meta | `{ "quota": "<name>", "limit": <value>, "current": <value> }` |

### Dependency Errors

| Aspect | Detail |
|--------|--------|
| Cause | An internal dependency (another module, database) failed |
| HTTP Status | 503 Service Unavailable |
| Retryable | Yes — dependency may recover |
| Disclosure | Indicate temporary failure. **Never identify the specific internal dependency.** |
| Codes | `SERVICE_TEMPORARILY_UNAVAILABLE` |
| Headers | `Retry-After: <seconds>` (when estimatable) |

### Provider Errors

| Aspect | Detail |
|--------|--------|
| Cause | An external provider (payment gateway, shipping carrier) failed or rejected the request |
| HTTP Status | 502 Bad Gateway (provider unavailable) or 422 (provider rejected) |
| Retryable | Depends on provider failure type |
| Disclosure | **Provider-specific failures are normalized.** Indicate external provider failure without exposing provider identity, provider error codes, or provider-specific messages directly. Internal logs retain full provider detail. |
| Codes | `PROVIDER_UNAVAILABLE`, `PROVIDER_REJECTED`, `PROVIDER_TIMEOUT` |

### Timeout Errors

| Aspect | Detail |
|--------|--------|
| Cause | Operation exceeded its time limit |
| HTTP Status | 504 Gateway Timeout (external) or 408 Request Timeout |
| Retryable | Yes — with caution (check idempotency for state-changing operations) |
| Disclosure | Indicate timeout. Warn about potential partial completion for non-idempotent operations. |
| Codes | `OPERATION_TIMEOUT` |

### Temporary Availability Errors

| Aspect | Detail |
|--------|--------|
| Cause | Platform is temporarily unable to serve requests (maintenance, overload, circuit breaker open) |
| HTTP Status | 503 Service Unavailable |
| Retryable | Yes — after indicated period |
| Disclosure | Indicate temporary unavailability. Provide retry guidance. Do not expose internal reason. |
| Codes | `SERVICE_TEMPORARILY_UNAVAILABLE`, `MAINTENANCE_IN_PROGRESS` |
| Headers | `Retry-After: <seconds>` |

### Internal Errors

| Aspect | Detail |
|--------|--------|
| Cause | Unexpected failure within the platform (unhandled exception, data corruption, programming error) |
| HTTP Status | 500 Internal Server Error |
| Retryable | Possibly — depends on root cause (unknown to consumer) |
| Disclosure | **Internal exceptions are never exposed directly.** Return generic message. Full details in internal logs only. Include correlation ID for support reference. |
| Codes | `INTERNAL_ERROR` |

### Partial-Operation Errors

| Aspect | Detail |
|--------|--------|
| Cause | Bulk operation where some items succeeded and some failed |
| HTTP Status | 207 Multi-Status or 200 with partial result |
| Retryable | Failed items may be retried individually |
| Disclosure | **Partial success must never be ambiguous.** Per-item results clearly identify successes and failures. |
| Codes | `PARTIAL_SUCCESS` |
| Structure | See batch error behavior section |

### Asynchronous-Operation Errors

| Aspect | Detail |
|--------|--------|
| Cause | A previously accepted async operation failed during processing |
| HTTP Status | 200 (on the polling response — the operation status itself is the resource being returned) |
| Retryable | Depends on failure type (available in the operation status) |
| Disclosure | Operation status includes error details following the same disclosure rules as synchronous errors |
| Codes | Failure codes specific to the operation type |

---

## Error Taxonomy Table

| Category | HTTP Status | Retryable | Consumer Action | Audit Level |
|----------|:-----------:|:---------:|----------------|-------------|
| Authentication | 401 | With fresh credentials | Re-authenticate | Security event |
| Authorization | 403 | No | Request permission escalation | Security event |
| Tenant context | 403 | No | Verify tenant membership | Security event |
| Validation | 400 | With corrected input | Fix request per field errors | None |
| Not found | 404 | No | Verify resource ID | None (unless high frequency → security) |
| Conflict (business) | 409 | Depends | Resolve conflict per guidance | Business event |
| Concurrency | 409 | Yes (re-read + retry) | Re-fetch and resubmit | None |
| Idempotency conflict | 409 | No | Use new idempotency key | None |
| State transition | 409 | No (until state changes) | Check current state, use valid transition | Business event |
| Rate limit | 429 | Yes (after delay) | Respect Retry-After | Monitoring |
| Quota | 403/429 | No (until quota increases) | Request quota increase | Monitoring |
| Dependency | 503 | Yes (with backoff) | Retry with exponential backoff | Operational alert |
| Provider | 502/422 | Depends | Retry or resolve with provider | Operational event |
| Timeout | 504/408 | With caution | Check operation status before retry | Operational alert |
| Temporary | 503 | Yes (after delay) | Respect Retry-After | Monitoring |
| Internal | 500 | Possibly | Report to support with correlation ID | Incident trigger |
| Partial | 207/200 | Failed items only | Process successes, retry/fix failures | Business event |
| Async failure | 200 (polling) | Depends | Check operation status for guidance | Business event |

---

## Retryability Matrix

| Category | Retryable | Retry Strategy | Consumer Guidance |
|----------|:---------:|----------------|-------------------|
| Authentication (expired) | Yes | Immediate with refresh token | Refresh credentials, retry once |
| Authentication (invalid) | No | — | Obtain new credentials |
| Authorization | No | — | Insufficient permissions. Escalate. |
| Validation | Yes | After correction | Fix input per error details |
| Not found | No | — | Resource does not exist |
| Concurrency conflict | Yes | Re-read + retry | Fetch latest version, resubmit |
| Rate limit | Yes | Wait for Retry-After | Respect delay, then retry |
| Dependency / temporary | Yes | Exponential backoff | Wait, retry with backoff (max 3-5 attempts) |
| Provider unavailable | Yes | Exponential backoff | Wait, retry with backoff |
| Provider rejected | No | — | Resolve with provider (payment declined, address invalid) |
| Timeout | Cautious | Check state first | Verify operation status (may have partially completed) |
| Internal | Unknown | Single retry acceptable | If persists, report to support |
| Idempotency conflict | No | — | Use a new idempotency key |
| State transition | No | — | Resource state has changed; reassess |

### Retryability Signal

Retryability is communicated through:

| Signal | Location | Detail |
|--------|----------|--------|
| `Retry-After` header | Response headers | Seconds to wait before retrying (rate limit, 503) |
| `meta.retryable` | Error meta object | Boolean indicating if this specific error is retryable |
| HTTP status category | Status code | 5xx generally retryable; 4xx generally not (with exceptions above) |
| Error code documentation | Developer docs | Each error code documents its retryability |

---

## Disclosure Matrix

| Information | Disclosed to Consumer | Internal Only |
|------------|:---:|:---:|
| Error code | Yes | — |
| Human-readable message | Yes | — |
| Field name (validation) | Yes | — |
| Current resource state | Yes (for state errors) | — |
| Valid transitions | Yes (for state errors) | — |
| Rate limit/quota details | Yes | — |
| Retry-After timing | Yes | — |
| Correlation ID | Yes (in header) | — |
| Stack trace | — | Yes |
| Internal class/method names | — | Yes |
| Database error messages | — | Yes |
| Internal service names | — | Yes |
| Provider identity | — | Yes |
| Provider error codes | — | Yes |
| Provider raw response | — | Yes |
| Other tenant identifiers | — | Yes |
| Internal resource existence (authz failure) | — | Yes |
| Server hostname/IP | — | Yes |
| Internal queue/topic names | — | Yes |

---

## 1. Machine-Readable Error Codes

**Error codes are stable contracts.**

| Rule | Detail |
|------|--------|
| Format | UPPER_SNAKE_CASE string |
| Stability | Once published, a code's meaning never changes. Codes are never removed from a version. |
| Namespacing | Platform codes: `AUTH_*`, `VALIDATION_*`, `RESOURCE_*`, `RATE_*`, `SERVICE_*`, `INTERNAL_*`. Module codes: `{MODULE}_*` (e.g., `ORDER_CANNOT_CANCEL`, `PAYMENT_DECLINED`). |
| New codes | New codes may be added without breaking changes (clients must handle unknown codes gracefully) |
| Documentation | Every code is documented with meaning, retryability, and recommended action |
| Parsing target | Consumers match on `code`, never on `message` text |

---

## 2. Human-Readable Messages

**Human-readable messages are not stable parsing contracts.**

| Rule | Detail |
|------|--------|
| Purpose | Developer understanding. Not for programmatic matching. |
| Volatility | Messages may be reworded, clarified, or localized at any time without notice |
| Localization | May be localized based on `Accept-Language` header |
| Content | Actionable where possible ("Product name is required" not just "validation error") |
| Safety | Never contains internal details (class names, SQL, stack traces) |

---

## 3. Field-Level Errors

**Validation errors identify fields when safe.**

| Rule | Detail |
|------|--------|
| Path notation | Dot-notation for nested fields: `"price.amount"`, `"address.postalCode"` |
| Array notation | Bracket notation for array items: `"lineItems[0].quantity"` |
| All violations reported | Return all field errors in a single response (not one at a time) |
| Non-field errors | `field` is `null` for errors not tied to a specific field (e.g., cross-field validation) |
| Security | Do not identify fields that reveal internal structure. Standard request fields only. |

---

## 4. Error Details (Meta)

| Rule | Detail |
|------|--------|
| Purpose | Machine-readable context enabling programmatic resolution |
| Examples | `{ "min": 1, "max": 100 }` for range errors. `{ "currentState": "shipped" }` for transition errors. `{ "retryAfter": 30 }` for rate limits. |
| Optional | `meta` is null when no additional context is useful |
| Stability | Meta keys are semi-stable. Consumers should not break if a new key appears. |
| Security | Never contains internal details. Only consumer-actionable context. |

---

## 5. Correlation Identifiers

| Rule | Detail |
|------|--------|
| Header | `X-Correlation-Id` returned on all error responses |
| Purpose | Enables consumer to reference specific failures when contacting support |
| Internal | Maps to the full internal trace (logs, spans, events) |
| Disclosure | The correlation ID itself is safe to expose. The internal data it references is not. |
| Consumer action | "Include this ID when contacting support" |

---

## 6. Retryability

**Retryable and non-retryable failures must be distinguishable.**

| Mechanism | Detail |
|-----------|--------|
| Status code | Primary signal (5xx generally retryable, 4xx generally not) |
| `Retry-After` header | Present when retry timing is known |
| `meta.retryable` | Explicit boolean when status code alone is ambiguous |
| Documentation | Each error code documents its retryability in the developer portal |

---

## 7. Safe Disclosure

**Internal exceptions are never exposed directly.**

| Rule | Detail |
|------|--------|
| No stack traces | Never in API responses. Only in internal logs. |
| No internal names | No class names, method names, queue names, or database table names |
| No provider details | External provider names, error codes, and responses are normalized |
| Generic internal errors | 500 responses use `INTERNAL_ERROR` with a generic message. Correlation ID is the bridge to detailed logs. |
| Not-found is not-found | Both "does not exist" and "exists but unauthorized" return 404. No distinction. |
| Tenant safety | Errors never reference other tenants. No indication that data "belongs to another organization." |

---

## 8. Localization

| Rule | Detail |
|------|--------|
| `message` field | May be localized based on `Accept-Language` |
| `code` field | Never localized. Always English UPPER_SNAKE_CASE. |
| `meta` values | Not localized (machine-readable) |
| V1 | English messages. Localization infrastructure prepared for V2. |
| Fallback | If requested language unavailable, fall back to English |

---

## 9. Error Stability

| Aspect | Stability |
|--------|-----------|
| Error codes | Stable. Never change meaning. Never removed from a version. |
| Error messages | Unstable. May be reworded at any time. |
| Meta keys | Semi-stable. Existing keys maintain meaning. New keys may appear. |
| HTTP status for a code | Stable. A given error code always returns the same HTTP status. |
| Error object structure | Stable. The four fields (code, field, message, meta) do not change. |

---

## 10. Deprecation

| Rule | Detail |
|------|--------|
| Error codes are not deprecated | They remain valid. Conditions that produce them may become impossible in new versions. |
| New codes added freely | Adding new error codes is non-breaking (consumers handle unknowns gracefully) |
| Renamed codes | A code is never renamed. If semantics change significantly, a new code is created. |
| Version removal | A code may be absent from a new API version if the condition is impossible in that version |

---

## 11. Logging Relationship

| Rule | Detail |
|------|--------|
| All errors logged | Every error response is logged with full internal context |
| Log level | 4xx → Warning. 5xx → Error. Authentication failures → Warning + security event. |
| Correlation | Log entries include the correlation ID from the response |
| Internal detail | Logs contain full stack traces, internal service names, provider responses — everything excluded from the API response |
| Retention | Error logs follow operational log retention policies |

---

## 12. Audit Relationship

| Error Category | Audit Event | Detail |
|---------------|:-----------:|--------|
| Authentication failure | Yes | Security-relevant. Tracked for brute-force detection. |
| Authorization failure | Yes | Security-relevant. May indicate escalation attempt. |
| Tenant-context failure | Yes | Security-relevant. May indicate cross-tenant probe. |
| Validation failure | No | Normal operation. Not security-relevant unless patterns emerge. |
| Not found (high frequency) | Conditional | Elevated frequency from single caller may indicate enumeration. |
| Financial operation failure | Yes | Business-critical. Reconciliation relevance. |
| State-transition failure | Conditional | May indicate workflow issues or integration problems. |
| Internal errors | Yes | Operational accountability. Incident correlation. |

---

## 13. Security-Event Relationship

| Error Category | Security Event | Response |
|---------------|:-----------:|---------|
| Repeated auth failures | Yes | May trigger account lockout, IP blocking |
| Authorization probing (many 403s) | Yes | May trigger rate limiting, investigation |
| Tenant boundary probing | Yes | May trigger IP blocking, security review |
| High-volume 404s from single source | Yes | May indicate enumeration attempt |
| Provider credential failures | Yes | May indicate compromised integration credentials |

See [Audit and Security Monitoring](../Security/Audit-and-Security-Monitoring.md) for security event handling.

---

## 14. Provider-Error Normalization

**Provider-specific failures are normalized while preserving internal diagnostic traceability.**

| Rule | Detail |
|------|--------|
| External provider identity hidden | API consumer does not learn which payment provider, which carrier, or which service failed |
| Normalized codes | `PROVIDER_UNAVAILABLE`, `PROVIDER_REJECTED`, `PROVIDER_TIMEOUT` — not provider-specific codes |
| Consumer message | Generic: "External service temporarily unavailable" or "Operation rejected by external provider" |
| Internal log | Full provider response, provider error code, provider request ID — all in internal logs with correlation |
| Reconciliation | For financial providers, internal records maintain provider reference for reconciliation |
| Retryability | Normalized: provider timeout → retryable. Provider rejection → not retryable. Provider unavailable → retryable. |

---

## 15. Batch Error Behavior

**Partial success must never be ambiguous.**

| Rule | Detail |
|------|--------|
| Per-item results | Every item in a batch has an explicit success or failure indication |
| No silent failures | If an item failed, it is reported. Items are never silently skipped. |
| No silent successes | Successfully processed items are confirmed. |
| Overall status | `isSuccess: true` only if ALL items succeeded. `isSuccess: false` with `PARTIAL_SUCCESS` code if any failed. |
| HTTP status | 200 if all succeed. 200 with `PARTIAL_SUCCESS` error code for partial. Standard error for total failure. |

### Batch Response Structure

```
{
  "isSuccess": false,
  "data": {
    "totalCount": 5,
    "successCount": 3,
    "failureCount": 2,
    "results": [
      { "index": 0, "success": true, "data": { "id": "..." } },
      { "index": 1, "success": true, "data": { "id": "..." } },
      { "index": 2, "success": false, "error": { "code": "VALIDATION_FAILED", "field": "sku", "message": "...", "meta": null } },
      { "index": 3, "success": true, "data": { "id": "..." } },
      { "index": 4, "success": false, "error": { "code": "CONFLICT_DUPLICATE", "field": null, "message": "...", "meta": null } }
    ]
  },
  "message": "3 of 5 items processed successfully",
  "errors": [
    { "code": "PARTIAL_SUCCESS", "field": null, "message": "2 items failed processing", "meta": { "successCount": 3, "failureCount": 2 } }
  ]
}
```

---

## Financial Failure Clarity

**Financial failures require clear state and reconciliation direction.**

| Rule | Detail |
|------|--------|
| Clear outcome | After a financial error, the consumer must know: did the money move or not? |
| Unknown state | If the platform cannot determine outcome (timeout to provider), explicitly state "outcome unknown — check before retrying" |
| Reconciliation | Financial errors include guidance: "safe to retry" vs. "check status before retrying" vs. "do not retry — contact support" |
| Idempotency | Financial operations always support idempotency keys. Retrying with the same key is always safe. |
| Provider failure | If provider reports failure, indicate clearly that no charge was made |
| Provider timeout | If provider did not respond, indicate uncertainty and recommend checking operation status |

### Financial Error Meta

Financial error `meta` includes:

| Key | Purpose |
|-----|---------|
| `chargeAttempted` | Whether a charge was attempted with the provider |
| `chargeConfirmed` | Whether the charge was confirmed (null if unknown) |
| `safeToRetry` | Whether retrying with the same idempotency key is safe |
| `checkStatusFirst` | Whether consumer should check operation status before any action |

---

## Conceptual Error-Envelope Examples (Non-Binding)

### Validation Error

```json
{
  "isSuccess": false,
  "data": null,
  "message": "Request validation failed",
  "errors": [
    { "code": "FIELD_REQUIRED", "field": "name", "message": "Product name is required", "meta": null },
    { "code": "FIELD_OUT_OF_RANGE", "field": "price.amount", "message": "Price must be greater than zero", "meta": { "min": "0.01" } }
  ]
}
```

### Authorization Error (No Existence Leakage)

```json
{
  "isSuccess": false,
  "data": null,
  "message": "Resource not found",
  "errors": [
    { "code": "RESOURCE_NOT_FOUND", "field": null, "message": "The requested resource was not found", "meta": null }
  ]
}
```

Note: This response is identical whether the resource does not exist OR exists but belongs to another tenant.

### State-Transition Error

```json
{
  "isSuccess": false,
  "data": null,
  "message": "Operation not valid for current resource state",
  "errors": [
    { "code": "STATE_TRANSITION_INVALID", "field": null, "message": "Cannot cancel an order that has already been shipped", "meta": { "currentState": "shipped", "validTransitions": ["deliver", "return_request"] } }
  ]
}
```

### Rate-Limit Error

```json
{
  "isSuccess": false,
  "data": null,
  "message": "Rate limit exceeded",
  "errors": [
    { "code": "RATE_LIMIT_EXCEEDED", "field": null, "message": "Too many requests. Please retry after 30 seconds.", "meta": { "retryAfter": 30, "limit": 100, "window": "60s", "retryable": true } }
  ]
}
```

### Internal Error (Safe Disclosure)

```json
{
  "isSuccess": false,
  "data": null,
  "message": "An unexpected error occurred. Please try again or contact support with the correlation ID.",
  "errors": [
    { "code": "INTERNAL_ERROR", "field": null, "message": "Internal server error", "meta": null }
  ]
}
```

### Financial Provider Timeout (Unknown State)

```json
{
  "isSuccess": false,
  "data": null,
  "message": "Payment processing timed out. Please check payment status before retrying.",
  "errors": [
    { "code": "PROVIDER_TIMEOUT", "field": null, "message": "External payment processing did not respond in time", "meta": { "chargeAttempted": true, "chargeConfirmed": null, "safeToRetry": false, "checkStatusFirst": true, "retryable": false } }
  ]
}
```

---

## Version Gate

| Version | Error Architecture Gate |
|---------|------------------------|
| V1 | All error responses use approved envelope. Stable error codes per category. Validation errors include field identification. Correlation IDs on all errors. Retryability documented per code. Safe disclosure enforced (no internal details). Provider errors normalized. 404 for both missing and unauthorized resources. Financial errors include state clarity. Batch operations report per-item results. |
| V2 | Localized error messages. Enhanced retry guidance (dynamic Retry-After based on load). Error analytics dashboard. Consumer-specific error documentation in developer portal. |
| V3 | Machine-learned anomaly detection on error patterns. Self-healing retry suggestions. Error budget tracking per consumer. |

---

## Decision Summary

| Decision | Rationale |
|----------|-----------|
| 404 for both missing and unauthorized resources | Prevents enumeration. Attacker cannot distinguish "resource exists" from "resource does not exist" by trying IDs. |
| Stable error codes, unstable messages | Machines need stability. Humans need clarity. Messages can improve without breaking integrations. |
| Provider normalization (hide identity) | Provider is an implementation detail. Consumers should not build logic around which provider is used. Provider may change without API change. |
| Per-item results for batch operations | Partial success must be unambiguous. Silent failure or all-or-nothing are both worse than clear per-item reporting. |
| Financial error meta with charge state | Money operations have real-world consequences. Ambiguity about "did the charge go through?" causes customer support burden and potential double-charges. |
| Correlation ID as the bridge | Detailed diagnostics stay internal. The correlation ID connects consumer support requests to internal investigation without exposing details. |
| Generic 500 for all internal failures | Any specificity in 500 responses risks information leakage. Internal details are for logs, not for API consumers. |

---

## Alternatives Considered

| Alternative | Rejected Because |
|------------|-----------------|
| Distinguish 403 from 404 for existing but unauthorized resources | Enables enumeration attacks. Attacker learns resource existence by comparing 403 vs 404. |
| Provider-specific error codes in API | Couples consumers to specific provider. Provider change becomes a breaking API change. |
| Messages as parsing contracts | Messages change for clarity, localization, or correction. Depending on message text breaks with every improvement. |
| Single generic error per response | Validation requires reporting all failures at once. Single error forces fix-one-retry-fix-another cycle. |
| All-or-nothing for batch operations | One bad item failing an entire 1000-item import is impractical. Partial success with clear reporting is better. |
| Expose Retry-After on all retryable errors | Some retryable errors (concurrency conflict) should be retried immediately. Retry-After only when a specific delay is needed. |
| Separate error endpoint for detailed error lookup | Over-engineering. Correlation ID + internal logs serve the same purpose without API surface overhead. |

---

## Architecture Impact

| Concern | Impact |
|---------|--------|
| Module design | Every module must produce errors conforming to this architecture. Domain exceptions map to standard error codes. |
| Middleware | Platform provides exception-to-error-response mapping. Unhandled exceptions produce safe INTERNAL_ERROR responses. |
| Security | Error responses are reviewed for information leakage. 404 ≡ 403 for resource access. No internal detail exposure. |
| Monitoring | Error codes enable platform-wide alerting (spike in 500s, spike in 401s from single source). |
| Documentation | Every error code is documented with meaning, retryability, and consumer action. |
| Testing | Error response tests verify: no internal leakage, correct codes, correct HTTP status, proper disclosure. |

---

## Implementation Impact

| Area | Impact |
|------|--------|
| Modules | Must map domain exceptions to standard error codes. Must provide field paths for validation errors. Must produce financial error meta where applicable. Must handle batch per-item reporting. |
| Platform | Must provide exception-handling middleware that catches all unhandled errors and produces safe responses. Must inject correlation IDs. Must normalize provider errors. |
| Frontend | Must handle errors by `code` (not by message text). Must display user-friendly messages. Must handle batch partial-success UI. |
| SDKs | Must expose structured error handling. Must provide access to error codes, fields, and meta. Must handle retry logic based on retryability signals. |
| Operations | Must monitor error rates by category. Must investigate 500 error spikes. Must manage rate-limit configuration. |
| Support | Must use correlation IDs to locate full error context in internal logs. |

---

## Security Responsibilities

| Role | Error Architecture Responsibilities |
|------|-----------------------------------|
| API Error Architect | Defines error categories and disclosure rules. Reviews new error codes for safety. |
| Module Teams | Map domain exceptions to standard codes. Ensure no internal leakage in error messages. Implement financial error meta. |
| Platform Team | Provides exception middleware. Normalizes provider errors. Enforces safe disclosure. Implements rate-limit error responses. |
| Security Team | Reviews error responses for information leakage. Validates 404/403 equivalence. Reviews provider normalization. Monitors security-relevant error patterns. |
| Operations | Monitors error rates. Investigates 500 spikes. Manages rate-limit thresholds. |

---

## Multi-Tenancy Responsibilities

| Responsibility | Detail |
|---------------|--------|
| No cross-tenant information | Error messages never reference other tenants or indicate data belongs to another organization |
| Tenant context failures are safe | Ambiguous tenant context is denied, not guessed. No helpful hints about valid tenants. |
| Per-tenant rate limits | Rate limiting is per-tenant. One tenant's rate-limit error does not affect others. |
| Correlation isolation | Correlation IDs from one tenant's errors cannot be used to access another tenant's diagnostic data |

---

## Out of Scope

This document does not define:

- Internal exception class hierarchies (development standards).
- Specific module error code registries (module specifications).
- Rate-limit threshold values (deployment configuration).
- Log storage infrastructure (operations documentation).
- Incident escalation procedures (see [Incident Response](../Security/Incident-Response.md)).
- Error page UI design (frontend specifications).

---

## Future Considerations

- **Error analytics** — Dashboard showing error distribution, trends, and anomalies.
- **Consumer-specific error docs** — Developer portal showing errors a consumer has received with resolution guidance.
- **Machine-learned retry guidance** — Dynamic retry timing based on current system load.
- **Error budget tracking** — Per-consumer error budgets for SLA management.
- **Webhook for error alerting** — Notify consumers when their integration is producing high error rates.
- **Structured error catalog API** — Machine-readable endpoint listing all possible error codes with documentation.

---

## Future Refactoring Triggers

This document should be revisited when:

- Error categories do not cover new failure modes (trigger for taxonomy expansion).
- Localization infrastructure is ready (trigger for message localization implementation).
- Security audit identifies information leakage in errors (trigger for disclosure rule tightening).
- Consumer feedback indicates error messages are not actionable (trigger for message improvement).
- New provider types introduce unhandled failure patterns (trigger for normalization expansion).
- Batch operation complexity exceeds current partial-success model (trigger for enhanced batch error reporting).

---

## Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 0.1 | 2026-07-21 | API Error and Failure Semantics Architect | Initial draft — error architecture |
