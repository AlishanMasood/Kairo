# Pagination, Filtering, Sorting, and Search

## Metadata

| Field | Value |
|-------|-------|
| Title | Kairo API Pagination, Filtering, Sorting, and Search Architecture |
| Document ID | KAI-API-008 |
| Status | Draft |
| Version | 0.1 |
| Target Release | V1 |
| Owner | API Query and Collection Architecture Lead |
| Created | 2026-07-21 |
| Last Updated | 2026-07-21 |
| Reviewers | TODO |
| Related Documents | [API Architecture](./API-Architecture.md), [Request and Response Standards](./Request-and-Response-Standards.md), [Data Access and Persistence](../Data/Data-Access-and-Persistence.md), [Reporting and Analytics Architecture](../Data/Reporting-and-Analytics-Architecture.md), [API Security](../Security/API-Security.md), [Tenant Isolation](../Multi-Tenancy/Tenant-Isolation.md), [Data Classification and Sensitivity](../Data/Data-Classification-and-Sensitivity.md) |
| Dependencies | [API Architecture](./API-Architecture.md), [Request and Response Standards](./Request-and-Response-Standards.md), [Data Access and Persistence](../Data/Data-Access-and-Persistence.md) |

---

## Applicable Version

This document defines V1 collection query standards. All modules exposing collection endpoints must conform to these patterns. The standards ensure consistent, secure, and performant collection access across the platform.

---

## Purpose

This document defines how API consumers query collections — pagination, filtering, sorting, and search. It establishes the rules that prevent unbounded queries, ensure tenant isolation, protect performance, and provide consistent behavior across all modules.

Collections are the most performance-sensitive and security-sensitive API surface. An unbounded collection query can exhaust database resources. A poorly designed filter can bypass tenant isolation. An uncontrolled expansion can create N+1 explosions. This document prevents all of these.

---

## Scope

This document covers:

- Pagination strategies (cursor, offset) and when to use each.
- Filtering conventions, operators, and safety rules.
- Sorting requirements and constraints.
- Search versus transactional filtering distinction.
- Query complexity limits and performance protection.
- Field selection and resource expansion direction.
- Security and multi-tenancy constraints on queries.

This document does not cover:

- Exact query-string syntax or parameter names (development standards).
- Database index design (module implementation).
- Search engine configuration (infrastructure documentation).
- Reporting or analytics query patterns (see [Reporting and Analytics Architecture](../Data/Reporting-and-Analytics-Architecture.md)).
- Full-text search engine selection (infrastructure decision).

---

## Mandatory Principles

| # | Principle |
|---|-----------|
| 1 | All collections must be bounded |
| 2 | Default and maximum page sizes are required conceptually |
| 3 | Stable ordering is required for reliable pagination |
| 4 | Tenant constraints cannot be removed through query parameters |
| 5 | Filtering cannot bypass authorization |
| 6 | Unsupported filters fail predictably |
| 7 | Search and transactional filtering are different capabilities |
| 8 | Query complexity must be limited |
| 9 | Expansions must not create uncontrolled N+1 behavior |
| 10 | Sensitive fields cannot become filterable or sortable by accident |
| 11 | Cursor pagination is preferred where datasets are large or frequently changing |
| 12 | Offset pagination may be acceptable for simple bounded administrative lists |

---

## 1. Collection Query Principles

| Principle | Detail |
|-----------|--------|
| Bounded | Every collection query returns a bounded number of results. No endpoint returns all records. |
| Paginated | Every collection uses pagination. No exceptions. |
| Tenant-scoped | Every query operates within the authenticated tenant boundary. Always. |
| Authorized | Query results include only resources the caller is authorized to see. |
| Predictable | Same query with same data returns same results (stable ordering). |
| Safe | Query parameters cannot cause unbounded resource consumption, bypass security, or expose sensitive data. |
| Consistent | Same filtering, sorting, and pagination conventions across all modules. |
| Documented | Available filters, sorts, and expansions are documented per collection endpoint. |

---

## 2. Pagination Requirements

**All collections must be bounded.**

| Requirement | Detail |
|-------------|--------|
| Always paginated | No collection endpoint returns unbounded results |
| Default page size | Every collection has a platform-defined default page size |
| Maximum page size | Every collection has a maximum page size that cannot be exceeded |
| Page size in response | Response includes the effective page size used |
| More-results indicator | Response indicates whether additional pages exist |
| Navigation | Response includes the mechanism to retrieve the next page |
| First page implicit | The first request (no pagination parameter) returns the first page |

---

## 3. Cursor Pagination

**Cursor pagination is preferred where datasets are large or frequently changing.**

| Aspect | Detail |
|--------|--------|
| Mechanism | Opaque cursor token encodes position. Consumer passes cursor to get next page. |
| Stability | Insensitive to insertions and deletions between pages (no skipped/duplicated items) |
| Statefulness | Stateless on server. Cursor encodes everything needed to resume. |
| Direction | Forward-only in V1. Backward pagination is future. |
| Cursor format | Opaque string. Consumers must not parse or construct cursors. |
| Expiration | Cursors may expire after a reasonable period (hours, not minutes). |
| Response structure | `{ cursor: "...", hasMore: true }` in pagination metadata |

| Advantage | Detail |
|-----------|--------|
| Consistent under writes | New records between pages do not cause duplicates or skips |
| Scalable | No COUNT(*) needed. No OFFSET overhead. |
| Efficient | Database can use index to seek directly to cursor position |

| Limitation | Detail |
|------------|--------|
| No random page access | Cannot jump to "page 5" directly |
| No total count (cheap) | Total count requires separate query (expensive on large sets) |
| Forward-only (V1) | Cannot go to previous page without re-querying from start |

---

## 4. Offset Pagination

**Offset pagination may be acceptable for simple bounded administrative lists.**

| Aspect | Detail |
|--------|--------|
| Mechanism | `offset` + `limit` (or `page` + `pageSize`) parameters |
| When appropriate | Small, bounded, infrequently changing datasets (admin lists, configuration items) |
| When inappropriate | Large datasets, frequently changing data, customer-facing collections |
| Risk | Inserts/deletes between pages cause item duplication or skipping |
| Performance | OFFSET grows expensive as page number increases (database must scan past offset rows) |
| Total count | Can provide total count feasibly for small datasets |

---

## 5. Stable Ordering

**Stable ordering is required for reliable pagination.**

| Rule | Detail |
|------|--------|
| Deterministic | Same query always returns same order (given same data) |
| Tie-breaking | Sort fields must include a unique tiebreaker (typically resource ID or creation timestamp) |
| Default order | Every collection has a defined default sort order (not arbitrary/random) |
| Cursor dependency | Cursor pagination requires stable ordering to function correctly |
| Documentation | Default sort order is documented per collection |

---

## 6. Page-Size Limits

**Default and maximum page sizes are required conceptually.**

| Parameter | Direction |
|-----------|-----------|
| Default page size | Platform-wide default (e.g., 20-50 items). Specific endpoints may override with justification. |
| Maximum page size | Platform-wide maximum (e.g., 100-200 items). Cannot be exceeded regardless of consumer request. |
| Minimum page size | At least 1. Consumer cannot request 0 items (use HEAD or count endpoint for existence checks). |
| Consumer override | Consumer may request a specific page size up to the maximum |
| Response indicates actual | Response pagination metadata includes the effective page size used |

---

## 7. Filtering

| Rule | Detail |
|------|--------|
| Declarative | Consumers declare filter criteria. Server applies them. |
| Server-enforced | Filtering is server-side. Consumers never receive unfiltered data to filter locally. |
| Allowlist | Only documented, explicitly supported filter fields are accepted |
| Unsupported rejected | **Unsupported filters fail predictably.** Requesting a filter on an unsupported field returns a 400 error, not silently ignored results. |
| Additive | Multiple filters are combined with AND logic (intersection) |
| Tenant-implicit | **Tenant constraints cannot be removed through query parameters.** Tenant filtering is always applied regardless of other filters. |
| Authorization-implicit | **Filtering cannot bypass authorization.** The filter is applied AFTER tenant and authorization scoping. |
| Default filters | Active resources by default. Include archived/deleted only through explicit filter. |

---

## 8. Filter Operators

| Operator Category | Examples | Notes |
|-------------------|----------|-------|
| Equality | `status=active` | Exact match |
| Inequality | `status!=cancelled` | Exclude specific values |
| List inclusion | `status=active,pending` | Match any of multiple values (OR within field) |
| Greater/less than | `createdAfter=2026-01-01`, `amountMin=100` | Range boundaries |
| Range | `createdAfter=...&createdBefore=...` | Combined boundaries |
| Boolean | `isActive=true` | True/false filtering |
| Null/non-null | `assignedTo=null`, `assignedTo!=null` | Presence/absence of value |
| Text contains | Limited — see Search section | Not a general SQL LIKE |

| Rule | Detail |
|------|--------|
| Documented per endpoint | Each collection documents which operators are available for which fields |
| Type-safe | Filter values are validated against the field's type |
| Case sensitivity | Documented per field. Typically case-insensitive for text, exact for IDs. |

---

## 9. Sorting

| Rule | Detail |
|------|--------|
| Explicit parameter | Consumer specifies sort field and direction |
| Allowlist | Only documented, explicitly supported sort fields are accepted |
| Unsupported rejected | Requesting sort on an unsupported field returns 400 |
| Default sort | Every collection has a documented default sort when none is specified |
| Direction | Ascending (asc) and descending (desc) per field |
| Stable tiebreaker | Sort always includes a final unique-field tiebreaker for determinism |

---

## 10. Multi-Field Sorting

| Rule | Detail |
|------|--------|
| Supported | Consumers may sort by multiple fields (primary, secondary, etc.) |
| Limit | Maximum number of sort fields is limited (e.g., 3) to prevent query complexity |
| Order matters | First field is primary sort, second is secondary, etc. |
| Each has direction | Each field can independently be asc or desc |
| Tiebreaker appended | Platform appends ID as final tiebreaker even if consumer does not specify it |

---

## 11. Search

**Search and transactional filtering are different capabilities.**

| Aspect | Transactional Filtering | Search |
|--------|------------------------|--------|
| Data source | Transactional database (authoritative) | Search index (derived, eventually consistent) |
| Freshness | Real-time | Near-real-time (seconds to minutes lag) |
| Capability | Exact match, range, boolean on indexed fields | Full-text, fuzzy, relevance-scored |
| Use case | "Show me active orders for this customer" | "Find products matching 'blue widget'" |
| Consistency | ACID-consistent with writes | Eventually consistent with writes |
| Performance | Optimized for specific indexed queries | Optimized for text search and relevance |

| Rule | Detail |
|------|--------|
| Different endpoints (direction) | Search is a distinct capability from collection filtering. May be exposed on a separate search endpoint. |
| Lag acknowledged | Search results may lag behind transactional state. Freshness metadata communicated. |
| Security same | Search results are tenant-scoped and authorization-filtered, same as transactional queries. |
| Not a bypass | Search cannot return resources the consumer is not authorized to see. |

---

## 12. Exact Matching

| Rule | Detail |
|------|--------|
| Primary filter mechanism | Most filtering uses exact matching (status = active, customerId = X) |
| Case rules | IDs and enum values: case-sensitive. Human text fields: typically case-insensitive (documented per field). |
| Type matching | Filter value must match field type. String for strings, boolean for booleans. |
| Null matching | Explicit null matching supported for nullable fields |

---

## 13. Partial Matching

| Rule | Detail |
|------|--------|
| Limited availability | Partial matching (starts-with, contains) is available only on explicitly designated fields |
| Not general-purpose | Not every string field supports partial matching (performance implications) |
| Index-backed | Partial matching requires supporting database index. Only offered where performance allows. |
| Prefix preferred | Starts-with is efficient (index-friendly). Contains requires full-scan or search index. |
| Search for full-text | "Contains" across multiple fields is a search operation, not a filter |

---

## 14. Date Ranges

| Rule | Detail |
|------|--------|
| Boundary parameters | `after` / `before` or `from` / `to` semantics per field |
| Inclusive/exclusive | Documented per endpoint. Typically: `after` is exclusive (>), `from` is inclusive (>=). |
| ISO 8601 | Date/time values in ISO 8601 format |
| UTC | Date-time ranges use UTC |
| Date-only | Date-only ranges (without time) represent the full day in UTC |
| Validation | End must be after start. Invalid ranges return 400. |
| Open ranges | Either boundary may be omitted (unbounded on one side) |

---

## 15. Numeric Ranges

| Rule | Detail |
|------|--------|
| Boundary parameters | `min` / `max` or `greaterThan` / `lessThan` per field |
| Inclusive | Min/max are typically inclusive (>=, <=). Documented per endpoint. |
| Type validation | Numeric filter values validated against field type (integer, decimal) |
| Open ranges | Either boundary may be omitted |
| Not for money | Monetary range filtering uses the amount field with explicit currency context |

---

## 16. Status Filtering

| Rule | Detail |
|------|--------|
| Common filter | Status is the most common filter across all resource types |
| Multi-value | Consumers may filter by multiple statuses: `status=active,pending` |
| Default | Default filter includes only active resources unless explicitly expanded |
| Archived | Archived resources require explicit filter to include |
| Deleted | Deleted resources are never returned through standard collection queries |
| Valid values | Filter values are validated against the resource's known status values |

---

## 17. Tenant Constraints

**Tenant constraints cannot be removed through query parameters.**
**Filtering cannot bypass authorization.**

| Rule | Detail |
|------|--------|
| Implicit always | Tenant filtering is applied before any consumer-specified filter |
| Not a parameter | Tenant is not a query parameter. It is resolved from authentication context. |
| Cannot be expanded | No filter parameter can expand results beyond the authenticated tenant |
| Cannot be removed | No query construct removes the tenant filter |
| Cross-tenant | Only platform-admin APIs with explicit cross-tenant authorization can query across tenants |
| Sub-scoping allowed | Consumer may further narrow within their tenant (specific store, specific customer) |

---

## 18. Field Selection

| Aspect | V1 Direction |
|--------|-------------|
| Sparse fieldsets | Not supported in V1. All resource fields are returned. |
| Rationale | Simplicity. Consistent response shapes. Cacheable. Field selection adds complexity. |
| Future | V2+ may support `fields` parameter for bandwidth optimization |
| Sensitive fields | Sensitive fields are excluded by default regardless (classification-driven, not consumer-selected) |

---

## 19. Resource Expansion

**Expansions must not create uncontrolled N+1 behavior.**

| Rule | Detail |
|------|--------|
| Opt-in | Related resources are referenced by ID by default. Expansion is explicit. |
| Documented | Available expansions are documented per endpoint |
| Allowlist | Only supported expansions are accepted. Unsupported returns 400. |
| Depth limit | Maximum one level of expansion. No nested expansion (expand the expanded resource). |
| Collection limit | Expanding a to-many relationship within a collection has item limits |
| Performance | Expansions are batch-loaded (not per-item N+1 queries) |
| Not for filtering | Cannot filter the parent collection based on expanded child properties (that requires a different query) |

---

## 20. Query Complexity

**Query complexity must be limited.**

| Limit | Purpose |
|-------|---------|
| Maximum page size | Prevents single-request resource exhaustion |
| Maximum filter count | Prevents overly complex WHERE clauses |
| Maximum sort fields | Prevents queries that cannot use indexes efficiently |
| Maximum expansion count | Prevents response explosion |
| Query timeout | Prevents long-running queries from blocking resources |
| Rate limiting | Per-consumer limits prevent query flooding |
| Total result scan limit | Internal: queries that would scan excessive rows are terminated |

| Rule | Detail |
|------|--------|
| Complexity measured | Filters + sorts + expansions contribute to query complexity |
| Over-limit fails fast | Exceeding complexity limits returns 400 (not slow execution) |
| Documented limits | Complexity limits are published in documentation |

---

## 21. Query Timeouts

| Rule | Detail |
|------|--------|
| Server-enforced | Queries exceeding a configured time limit are cancelled |
| Consumer receives error | Timeout returns 504 or appropriate error with guidance |
| Not retry-infinite | Consumers should not retry identical complex queries indefinitely |
| Guidance | Error response suggests simplifying the query (fewer filters, smaller page, less expansion) |
| Internal monitoring | Queries approaching timeout are logged for optimization review |

---

## 22. Index Awareness

| Rule | Detail |
|------|--------|
| Filterable = indexed | Fields exposed as filterable must have supporting database indexes |
| Sortable = indexed | Fields exposed as sortable must have supporting database indexes |
| No accidental exposure | **Sensitive fields cannot become filterable or sortable by accident.** Adding a filter requires explicit design decision including index creation. |
| Performance review | New filter/sort fields require performance review before exposure |
| Module responsibility | Module teams ensure indexes exist for all exposed filter/sort fields |

---

## 23. Empty Results

| Rule | Detail |
|------|--------|
| Not an error | An empty result set is a valid, successful response (HTTP 200) |
| Consistent structure | Empty results use the same collection response structure with `items: []` |
| Pagination metadata | Even empty results include pagination metadata (`hasMore: false`, `cursor: null`) |
| Distinguished from not-found | A collection with no matching items (200 + empty) is distinct from a parent resource not found (404) |

---

## 24. Result Metadata

| Metadata | Always Present | Detail |
|----------|:-:|--------|
| `cursor` | Yes | Next-page cursor (null if no more pages) |
| `hasMore` | Yes | Boolean indicating whether more results exist |
| `pageSize` | Yes | Effective page size used |
| `totalCount` | No | Optional. Expensive on large datasets. Provided only where feasible and requested. |
| Applied filters | No | Future: echo back applied filters for debugging |
| Data freshness | Conditional | For search-backed results: last index update timestamp |

---

## 25. Consistency During Pagination

| Rule | Detail |
|------|--------|
| Cursor isolates from inserts | New records added after the first page do not cause duplicates on subsequent pages |
| Cursor isolates from deletes | Deleted records do not cause items to be skipped |
| Not snapshot | Pagination does not guarantee a point-in-time snapshot. Records may change state between pages. |
| Acceptable inconsistency | A record that transitions from "active" to "archived" between pages may disappear. This is acceptable. |
| Strong consistency not promised | Collection queries are consistent with the transactional database but are not isolated snapshots |

---

## 26. Search-Index Lag

| Rule | Detail |
|------|--------|
| Lag is expected | Search indexes are eventually consistent. Freshly created or modified records may not appear immediately in search. |
| Freshness communicated | Search responses indicate when the index was last updated |
| Not authoritative | Search results are derived. The transactional database is authoritative. |
| Reconciliation | If a consumer finds a result via search but gets 404 on direct access, the item was deleted after indexing. This is expected. |
| Not hidden | **Analytical lag must not be hidden** (from [Reporting and Analytics Architecture](../Data/Reporting-and-Analytics-Architecture.md)). Same applies to search. |

---

## 27. Future Advanced Query Language

| Aspect | Detail |
|--------|--------|
| V1 | Simple query parameters per field. Documented operators. Sufficient for V1 needs. |
| Future trigger | If query complexity demands exceed simple parameters (complex boolean logic, nested conditions) |
| Direction | Structured query body (POST for complex queries) or filter expression language |
| Not V1 | V1 does not need a query language. Simple parameterized filters are sufficient. |
| GraphQL alternative | GraphQL may serve advanced query needs if adopted (evaluated separately) |

---

## Pagination Decision Matrix

| Factor | Cursor Pagination | Offset Pagination |
|--------|:-:|:-:|
| Large dataset (1000+) | **Recommended** | Not recommended |
| Frequently changing data | **Recommended** | Not recommended |
| Customer-facing collections | **Recommended** | Acceptable |
| Small bounded admin lists (<100 items typically) | Acceptable | **Acceptable** |
| Need random page access | Not supported | Supported |
| Need total count cheaply | Expensive (separate query) | Feasible |
| Performance at scale | Excellent (index seek) | Degrades with offset size |
| Consistency under writes | Excellent (no skip/duplicate) | Poor (items shift) |
| V1 default | **Yes** | Selective use |

### V1 Pagination Direction

| Collection Type | Recommended Pagination |
|----------------|----------------------|
| Products (storefront, admin) | Cursor |
| Orders (customer, admin) | Cursor |
| Customers (admin) | Cursor |
| Inventory items | Cursor |
| Audit events | Cursor |
| Search results | Cursor |
| Configuration items (small lists) | Offset acceptable |
| Organization members (small list) | Offset acceptable |
| Store list (bounded per org) | Offset acceptable |
| Webhook registrations (small list) | Offset acceptable |

---

## Query Capability Matrix

| Capability | Transactional Queries | Search Queries |
|-----------|:---:|:---:|
| Exact match filtering | Yes | Yes |
| Range filtering (date, numeric) | Yes | Yes |
| Multi-value filtering (IN) | Yes | Yes |
| Full-text search | No | **Yes** |
| Fuzzy matching | No | **Yes** |
| Relevance scoring | No | **Yes** |
| Highlighting | No | **Yes** |
| Faceted results | No | **Yes** (future) |
| Real-time freshness | **Yes** | Near-real-time |
| ACID consistency | **Yes** | Eventually consistent |
| Tenant-scoped | **Yes** | **Yes** |
| Authorization-filtered | **Yes** | **Yes** |

---

## Security and Performance Considerations

### Security

| Concern | Protection |
|---------|-----------|
| Tenant bypass via filter | Tenant filter is applied first and cannot be overridden by query parameters |
| Authorization bypass via filter | Results are authorization-filtered. Cannot request resources beyond access. |
| Sensitive field filtering | Sensitive fields (PII, financial) are not filterable/sortable unless explicitly designed with security review |
| Enumeration via filters | Filtering on existence (e.g., `email=test@example.com`) on unauthorized resources returns empty, not 403 |
| Query injection | Filter values are parameterized. Never interpolated into queries. |
| Information via timing | Complex queries do not reveal existence through timing differences (constant-time not-found vs. found) |

### Performance

| Concern | Protection |
|---------|-----------|
| Unbounded queries | Mandatory pagination with maximum page size |
| Expensive sorts | Only indexed fields are sortable |
| Expensive filters | Only indexed fields are filterable |
| N+1 in expansion | Expansions use batch loading, never per-item queries |
| Count queries | Total count is optional (expensive on large tables) |
| Query timeout | Server-enforced timeout on all queries |
| Rate limiting | Per-consumer rate limits prevent query flooding |
| Scan limits | Internal: queries scanning excessive rows are terminated |

---

## V1 versus Future Search Capability

| Capability | V1 | V2+ |
|-----------|:---:|:---:|
| Parameterized filtering | Yes | Yes |
| Cursor pagination | Yes | Yes |
| Offset pagination (bounded) | Yes (selective) | Yes (selective) |
| Sorting with tiebreaker | Yes | Yes |
| Multi-value filter (IN) | Yes | Yes |
| Date range filtering | Yes | Yes |
| Full-text search (basic) | Yes (search endpoint) | Enhanced |
| Fuzzy search | No | Yes |
| Relevance scoring | No | Yes |
| Faceted search | No | Yes |
| Field selection | No | Yes |
| Nested expansion (multi-level) | No | Evaluated |
| Advanced query language | No | Evaluated |
| Saved searches/views | No | Yes |
| Search suggestions/autocomplete | No | Yes |

---

## Version Gate

| Version | Collection Query Gate |
|---------|---------------------|
| V1 | All collections paginated (cursor default). Maximum page sizes enforced. Stable ordering with tiebreaker. Documented filter fields per endpoint (indexed). Status filtering with active default. Tenant constraint always applied. Unsupported filters return 400. Basic text search endpoint (search index). Expansion with depth-1 limit and batch loading. Query timeout enforced. |
| V2 | Field selection support. Enhanced search (fuzzy, relevance, highlighting). Faceted search. Backward cursor pagination. Saved searches. Search suggestions. Advanced sort capabilities. |
| V3 | Advanced query language evaluated. GraphQL query surface (if adopted). Aggregate queries on collections. Real-time search with minimal lag. |

---

## Decision Summary

| Decision | Rationale |
|----------|-----------|
| Cursor pagination as default | Consistent under concurrent writes. Scales efficiently. No performance degradation at depth. Industry standard for large APIs (Stripe, Shopify). |
| Offset allowed for bounded admin lists | Small, rarely-changing lists (stores, config) benefit from total count and page jumping. Risk is minimal at small scale. |
| Unsupported filters return 400 | Silent ignoring of filters creates confusion (consumer thinks they are filtering but are not). Explicit failure is clearer. |
| Sensitive fields not filterable by default | Filtering on email, phone, or financial data enables enumeration attacks. Explicit security review required before exposing. |
| Expansion depth limited to 1 | Nested expansion creates exponential response growth and N+1 risk. One level is practical without complexity explosion. |
| Search separate from transactional filtering | Different consistency guarantees, different backends, different capabilities. Mixing them confuses consumers about freshness. |
| No advanced query language in V1 | Simple parameterized filters cover V1 needs. Query language adds parsing complexity, injection risk, and documentation burden. |
| Tenant filter is architectural, not optional | Tenant isolation is not a "nice filter" that consumers can turn off. It is a security boundary enforced at the platform level. |

---

## Alternatives Considered

| Alternative | Rejected Because |
|------------|-----------------|
| Offset pagination as default | Poor performance at depth. Inconsistent under writes (skip/duplicate). Not suitable for large or active datasets. |
| Silent ignore of unsupported filters | Consumer believes they are filtering but are not. Leads to incorrect assumptions and subtle bugs. |
| Allow filtering on all fields | Performance disaster (no index). Security risk (sensitive field enumeration). Uncontrolled query complexity. |
| Unlimited page size | Single request can exhaust database/memory. Denial-of-service vector. Must be bounded. |
| No expansion (always separate requests) | Forces N+1 at the client level. Excessive round trips for common patterns. Controlled expansion is better. |
| GraphQL for all queries in V1 | Over-complex for V1. Authorization is harder. Caching is harder. Rate limiting is harder. REST with filters is sufficient. |
| Allow consumers to remove tenant filter | Security boundary violation. Tenant isolation is not optional regardless of convenience. |
| Arbitrary sort on any field | Only indexed fields can be sorted efficiently. Arbitrary sort causes full-table scans. |

---

## Architecture Impact

| Concern | Impact |
|---------|--------|
| Module design | Modules must define supported filters, sorts, and expansions per collection. Must create indexes for all filterable/sortable fields. |
| Database | Every filterable/sortable field requires a supporting index. Cursor pagination requires ordered index access. |
| Search infrastructure | Platform requires a search index for full-text search (separate from transactional DB). |
| Performance | Query timeout, scan limits, and rate limiting are platform-enforced infrastructure. |
| Security | Tenant filter is platform-enforced (not module-implemented per query). Authorization filtering is consistent. |
| Documentation | Every collection endpoint documents its available filters, sorts, expansions, and their limitations. |

---

## Implementation Impact

| Area | Impact |
|------|--------|
| Modules | Must declare filterable/sortable fields. Must create database indexes. Must implement expansion with batch loading. Must support cursor generation/parsing. Must document available query parameters. |
| Platform | Must provide cursor pagination infrastructure. Must enforce page size limits. Must apply tenant filter before module code. Must enforce query timeout. Must provide search index integration. |
| Frontend | Must implement cursor-based navigation (next page, not page numbers for large lists). Must handle empty results gracefully. Must respect rate limits. |
| SDKs | Must provide pagination helpers (auto-paginate iterators). Must expose filter/sort parameters cleanly. |
| Testing | Must test: tenant isolation not bypassed by filters, unsupported filters rejected, pagination stability under writes, expansion batch loading (no N+1). |
| Operations | Must monitor query performance. Must alert on timeout-approaching queries. Must manage search index health and lag. |

---

## Security Responsibilities

| Role | Query Security Responsibilities |
|------|-------------------------------|
| API Query Architect | Defines query standards. Reviews new filter/sort field proposals for security implications. |
| Module Teams | Declare filterable fields. Create indexes. Implement expansion safely. Ensure no sensitive-field exposure without review. |
| Platform Team | Enforces tenant filter. Enforces page-size limits. Enforces query timeout. Applies rate limiting. Manages search index. |
| Security Team | Reviews sensitive-field filterability proposals. Validates tenant isolation cannot be bypassed. Reviews enumeration risk. |
| Operations | Monitors query performance. Manages search index lag. Responds to query timeout alerts. |

---

## Multi-Tenancy Responsibilities

| Responsibility | Detail |
|---------------|--------|
| Tenant filter is infrastructure | Applied by platform before any consumer-specified filter. Not a module concern. |
| Cross-tenant impossible through queries | No query parameter, filter, or sort can return data from other tenants |
| Sub-tenant scoping | Within a tenant, consumers may further scope (specific store, specific customer) |
| Search is tenant-scoped | Search index queries are tenant-scoped. No cross-tenant search results. |
| Expansion is tenant-scoped | Expanded resources must belong to the same tenant as the parent |

---

## Out of Scope

This document does not define:

- Exact query-string parameter names or syntax (development standards).
- Database index specifications per module (module implementation).
- Search engine product selection (infrastructure decision).
- Reporting/analytics query patterns (see [Reporting and Analytics Architecture](../Data/Reporting-and-Analytics-Architecture.md)).
- Full-text search ranking algorithm (search infrastructure).
- GraphQL query patterns (future — evaluated separately).

---

## Future Considerations

- **GraphQL** — Flexible query surface for complex frontend needs (evaluated in V2+).
- **Faceted search** — Search results with aggregate facets (category counts, price ranges).
- **Saved searches** — Consumers save and reuse query configurations.
- **Search suggestions** — Autocomplete and "did you mean" for text search.
- **Aggregate queries** — COUNT, SUM, AVG on collections without fetching items.
- **Real-time search** — Minimal lag between write and search availability.
- **Advanced filter expressions** — Boolean logic, nested conditions, computed filters.
- **Backward pagination** — Navigate to previous pages with cursor.

---

## Future Refactoring Triggers

This document should be revisited when:

- Query complexity demands exceed simple parameterized filters (trigger for query language evaluation).
- Search lag becomes unacceptable for use cases (trigger for real-time search investment).
- Frontend needs require flexible field selection (trigger for sparse fieldsets implementation).
- Collection sizes exceed cursor pagination efficiency (trigger for advanced partitioning).
- Consumers need aggregate counts without fetching items (trigger for count/aggregate endpoints).
- GraphQL adoption decision is made (trigger for GraphQL query standards).

---

## Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 0.1 | 2026-07-21 | API Query and Collection Architecture Lead | Initial draft — pagination, filtering, sorting, and search standards |
