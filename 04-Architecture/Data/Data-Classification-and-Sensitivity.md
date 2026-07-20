# Data Classification and Sensitivity

## Metadata

| Field | Value |
|-------|-------|
| Title | Kairo Data Classification and Sensitivity Model |
| Document ID | KAI-DATA-003 |
| Status | Draft |
| Version | 0.1 |
| Target Release | V1 |
| Owner | Data Security and Privacy Architecture Lead |
| Created | 2026-07-20 |
| Last Updated | 2026-07-20 |
| Reviewers | TODO |
| Related Documents | [Data Protection](../Security/Data-Protection.md), [Data Architecture](./Data-Architecture.md), [Data Ownership](./Data-Ownership.md), [Compliance Readiness](../Security/Compliance-Readiness.md), [Tenant Isolation](../Multi-Tenancy/Tenant-Isolation.md), [Secrets and Key Management](../Security/Secrets-and-Key-Management.md), [Audit and Security Monitoring](../Security/Audit-and-Security-Monitoring.md) |
| Dependencies | [Data Protection](../Security/Data-Protection.md), [Data Architecture](./Data-Architecture.md) |

---

## Purpose

This document defines the data classification and sensitivity model for the Kairo platform. It extends [Data Protection](../Security/Data-Protection.md) with operational classification rules that apply across all data surfaces — backend, frontend, logs, events, search, cache, exports, and support tooling.

Classification determines how data is handled at every point in its lifecycle. When a developer writes code, they must know whether the field they are handling is Public (log freely), Confidential (access-control required), or Restricted (never log, never expose). This document provides that clarity.

This document does not duplicate [Data Protection](../Security/Data-Protection.md). It consumes the classification framework defined there and extends it with per-surface handling rules, responsibilities, and prohibitions.

---

## Scope

This document covers:

- Classification definitions with detailed handling rules per surface.
- Sensitivity model for all data categories across the platform.
- Per-surface (backend, frontend, logs, events, etc.) classification responsibilities.
- Explicit prohibitions for sensitive data.
- Handling matrices for operational implementation.

This document does not cover:

- Classification framework definition (defined in [Data Protection](../Security/Data-Protection.md)).
- Encryption algorithms or key management (defined in [Secrets and Key Management](../Security/Secrets-and-Key-Management.md)).
- Compliance procedures or legal conclusions.
- Database schema or storage implementation.

---

## Classification Definitions

### Public

| Aspect | Detail |
|--------|--------|
| Description | Data intentionally published and accessible to any authenticated request within the tenant |
| Examples | Published product names, public prices, category names, published media URLs, store name |
| Ownership | Respective business module |
| Allowed access | Any request with valid tenant context (publishable key or higher) |
| Storage | Standard storage. Volume encryption (infrastructure default). |
| Encryption | Volume-level (infrastructure default). No additional application-level encryption required. |
| Logging | May appear in logs freely |
| Export | No additional restrictions beyond tenant scoping |
| Retention | While the entity is active |
| Non-production | May be used in non-production environments |
| Masking | Not required |
| Backup | Standard backup treatment |
| Deletion | Removed when entity is deleted or tenant is decommissioned |
| Incident severity if exposed cross-tenant | High (isolation failure, even for public data) |
| V1 requirements | Identify and document public fields per module |
| Future requirements | None beyond V1 |

---

### Internal

| Aspect | Detail |
|--------|--------|
| Description | Operational data supporting platform function, not intended for external or customer-facing exposure |
| Examples | Event processing state, queue metadata, internal identifiers, system health metrics, feature flag state, background job state |
| Ownership | Platform services or respective modules |
| Allowed access | Authorized workforce and platform services |
| Storage | Standard storage. Volume encryption. |
| Encryption | Volume-level (infrastructure default) |
| Logging | May appear in operational logs |
| Export | Organization admin export only |
| Retention | Per operational need |
| Non-production | May be used in non-production environments |
| Masking | Not required for authorized access |
| Backup | Standard backup treatment |
| Deletion | Removed per operational lifecycle |
| Incident severity if exposed externally | Medium |
| V1 requirements | Identify internal-only fields. Exclude from public API responses. |
| Future requirements | None beyond V1 |

---

### Confidential

| Aspect | Detail |
|--------|--------|
| Description | Business and customer data requiring controlled access and explicit authorization |
| Examples | Customer email addresses, phone numbers, shipping addresses, order details, invoice data, integration configuration, business reports, revenue figures |
| Ownership | Respective business module |
| Allowed access | Workforce users with explicit permission. Customers accessing their own data. |
| Storage | Volume encryption (infrastructure). Considered for additional application-level protection in future. |
| Encryption | Volume-level required. Application-level recommended for PII fields. |
| Logging | Access events may be logged (who accessed what). **Field values must not appear in logs.** |
| Export | Controlled. Audit logged. Organization admin authorization for bulk export. |
| Retention | While business relationship exists + compliance period |
| Non-production | Anonymized or synthetic only. Production confidential data must not be used in non-production. |
| Masking | Partial masking in UI where appropriate (e.g., email: `j***@example.com`). Full value only for authorized access. |
| Backup | Encrypted backup. Per-tenant extraction supported. |
| Deletion | Deleted upon tenant deletion or customer erasure request |
| Incident severity if exposed | High |
| V1 requirements | Identify confidential fields. Enforce access control. Exclude from logs. Mask in responses where appropriate. |
| Future requirements | Field-level encryption for PII. Advanced masking policies. |

---

### Restricted

| Aspect | Detail |
|--------|--------|
| Description | The most sensitive platform data. Access is structurally limited to the minimum services that require it. |
| Examples | Password hashes, API key material (hashed), encryption keys, MFA secrets, signing keys, recovery codes |
| Ownership | Platform Identity / Secret Store |
| Allowed access | Platform identity and cryptographic services only. Never returned through any API. Never visible to administrators. |
| Storage | Dedicated secret store or application-level encrypted storage |
| Encryption | Mandatory application-level encryption above volume encryption |
| Logging | **Never logged under any circumstance.** Access to the storage is audited (who accessed, not what value). |
| Export | **Never exported.** Credential rotation replaces old values. |
| Retention | While credential is active. Immediately deleted upon revocation/rotation. |
| Non-production | Non-production environments use separate restricted data. Production restricted data never enters non-production. |
| Masking | Not applicable — never exposed. No partial display. |
| Backup | Encrypted. Secret store backup is separate from business data backup. |
| Deletion | Immediately on revocation or rotation |
| Incident severity if exposed | Critical |
| V1 requirements | All restricted data identified. Stored in secret store or with application-level encryption. Never logged. Never in API responses. |
| Future requirements | HSM-backed storage. Hardware key protection. |

---

### Personal Data

| Aspect | Detail |
|--------|--------|
| Description | Data that identifies or relates to a natural person. Subject to data protection regulations. |
| Examples | Customer names, email addresses, phone numbers, shipping addresses, IP addresses, purchase history, account creation date |
| Ownership | Respective business module (Customers, Orders) |
| Allowed access | Purpose-limited. Accessible only for the purpose it was collected. |
| Storage | Volume encryption required. Application-level encryption for defined PII fields. |
| Encryption | Volume-level required. Application-level for sensitive personal fields (email, phone, address). |
| Logging | Access events logged (who accessed). **Field values never logged.** |
| Export | Subject to data portability. Customer can request export. Controlled bulk export (audited). |
| Retention | While purpose exists. Deleted on customer request or after retention period. |
| Non-production | **Never used in non-production.** Synthetic or anonymized data only. |
| Masking | Masked in API responses unless the caller has explicit PII access permission. Masked in support tooling. |
| Backup | Encrypted. Subject to the same deletion requirements (removed from backups as they rotate). |
| Deletion | On customer request (right of erasure) or retention expiry. Deletion is verified and audited. |
| Incident severity if exposed | High to Critical (depending on volume and nature) |
| V1 requirements | Identify all personal data fields. Apply access control. Exclude from logs. Support deletion. Support export. No production PII in non-production. |
| Future requirements | Consent tracking. Per-field encryption with tenant keys. Automated erasure. Regional data residency. |

---

### Authentication Data

| Aspect | Detail |
|--------|--------|
| Description | Data used to verify identity or grant access |
| Examples | Access tokens, refresh tokens, session identifiers, OAuth state, temporary auth codes |
| Ownership | Platform Identity service |
| Allowed access | Platform authentication services. Client applications (their own tokens only, in transit). |
| Storage | Short-lived in-memory or session store. Not in persistent business databases. |
| Encryption | Encrypted in transit (TLS). Encrypted at rest if persisted (session store). |
| Logging | **Never logged.** Session identifiers may be logged (hashed or truncated) for correlation. Full token values never. |
| Export | Never exported |
| Retention | While session is active. Deleted on expiration or revocation. |
| Non-production | Non-production environments use separate authentication data |
| Masking | Never displayed. Not applicable. |
| Backup | Not backed up (ephemeral). Session store replication for availability only. |
| Deletion | On session expiration or revocation |
| Incident severity if exposed | Critical (enables account takeover) |
| V1 requirements | Tokens never logged. Never in persistent storage beyond session store. Never in API response bodies (except at issuance). |
| Future requirements | Token binding. Hardware-backed session keys. |

---

### Financial Data

| Aspect | Detail |
|--------|--------|
| Description | Data relating to financial transactions, revenue, and commercial terms |
| Examples | Order totals, transaction amounts, refund amounts, price overrides, cost prices, margin data, revenue reports |
| Ownership | Orders, Pricing, Payments (future) |
| Allowed access | Authorized workforce with financial permission. Customers see their own transaction amounts. |
| Storage | Volume encryption. Standard business data storage. |
| Encryption | Volume-level required. Application-level for highly sensitive financial fields (cost prices, margins). |
| Logging | Transaction IDs and amounts may be logged for reconciliation. Cost/margin data not logged. |
| Export | Controlled. Audit logged. Financial data export requires elevated permission. |
| Retention | Per financial regulations and business requirements |
| Non-production | Synthetic financial data only. No production financial data in non-production. |
| Masking | Cost prices and margins masked from non-finance roles. Transaction amounts visible to authorized users. |
| Backup | Encrypted backup. Financial data retention may exceed general business data retention. |
| Deletion | Per regulatory retention requirements. May be retained longer than other business data. |
| Incident severity if exposed | High |
| V1 requirements | Identify financial fields. Apply role-based access. Audit financial operations. Exclude cost/margin from general access. |
| Future requirements | Financial audit trail. Reconciliation automation. |

---

### Payment-Related Data

| Aspect | Detail |
|--------|--------|
| Description | Data associated with payment processing. Subject to PCI scope. |
| Examples | Payment provider tokens, transaction references, payment method types, last-four digits of card (masked), payment status |
| Ownership | Payments service (future). Orders module (payment references in V1). |
| Allowed access | Payment service and authorized finance roles. Never general workforce access. |
| Storage | Application-level encryption for payment tokens. Provider tokenization (no raw card data). |
| Encryption | Mandatory application-level encryption above volume encryption |
| Logging | Transaction IDs logged for reconciliation. **Card numbers, CVVs, full account numbers never logged.** |
| Export | Subject to PCI scope restrictions. Limited export. Audit logged. |
| Retention | Per payment provider and regulatory requirements |
| Non-production | Test-mode data only. No production payment data in non-production. Provider test credentials only. |
| Masking | Card numbers always masked (last 4 only). Payment tokens never displayed. |
| Backup | Encrypted. PCI-scoped data may have separate backup treatment. |
| Deletion | Per provider retention requirements. Tokens deleted when payment method is removed. |
| Incident severity if exposed | Critical |
| V1 requirements | No raw card data stored (provider tokenization). Payment tokens encrypted. Card data never logged. PCI scope minimized. |
| Future requirements | Dedicated PCI boundary. PCI-specific backup and access controls. Formal PCI assessment. |

---

### Commercially Sensitive Data

| Aspect | Detail |
|--------|--------|
| Description | Business data that provides competitive advantage or whose exposure would harm the business |
| Examples | Supplier cost prices, profit margins, volume discounts, negotiated terms, unreleased pricing strategies, internal KPIs |
| Ownership | Respective business module (Pricing, Procurement future) |
| Allowed access | Restricted to defined business roles (finance, purchasing). Not general workforce. |
| Storage | Volume encryption. Standard business data storage. |
| Encryption | Volume-level required |
| Logging | Not logged |
| Export | Controlled. Requires elevated authorization. Audit logged. |
| Retention | While commercially relevant |
| Non-production | Synthetic data only |
| Masking | Masked from roles without explicit commercial-data permission |
| Backup | Standard encrypted backup |
| Deletion | When no longer commercially relevant |
| Incident severity if exposed externally | High |
| V1 requirements | Identify commercially sensitive fields. Apply role-based access. Exclude from general responses. |
| Future requirements | Advanced field-level access policies. |

---

### Operational Security Data

| Aspect | Detail |
|--------|--------|
| Description | Data about the platform's security posture, configuration, and operations |
| Examples | Security policies, firewall rules, rate limit configurations, infrastructure topology, vulnerability scan results, penetration test findings |
| Ownership | Platform operations / Security team |
| Allowed access | Security and operations teams only. Never exposed to tenants. |
| Storage | Platform-internal storage. Not in tenant-accessible databases. |
| Encryption | Volume-level required |
| Logging | Operational logs only (not tenant-visible) |
| Export | Never exported to tenants. Internal use only. |
| Retention | While operationally relevant |
| Non-production | Security testing data uses separate controls |
| Masking | Not applicable (never tenant-visible) |
| Backup | Platform-internal backup |
| Deletion | When superseded or no longer relevant |
| Incident severity if exposed | High (reveals attack surface) |
| V1 requirements | Operational security data is never in tenant-accessible APIs, responses, or logs. |
| Future requirements | Formal classification of operational security assets. |

---

### Audit Evidence

| Aspect | Detail |
|--------|--------|
| Description | Immutable records of significant platform actions for accountability and compliance |
| Examples | Authentication events, authorization decisions, data changes, administrative actions, lifecycle events |
| Ownership | Audit service |
| Allowed access | Organization admins (their tenant's events). Platform compliance team (all events). |
| Storage | Append-only storage. Immutable. Separate from business data. |
| Encryption | Volume-level encryption |
| Logging | Audit IS the log. Not separately logged. |
| Export | Tenant audit export (scoped to organization). Compliance export (authorized). |
| Retention | Compliance-driven (months to years). Outlives business data. |
| Non-production | Test audit data generated normally in non-production. No production audit data moved to non-production. |
| Masking | Audit entries do not contain sensitive field values (they reference by ID). No masking needed. |
| Backup | Immutable backup. Longer retention than business data backup. |
| Deletion | Only on retention expiry. Never manually deletable. |
| Incident severity if audit integrity is compromised | Critical (destroys accountability) |
| V1 requirements | Immutable storage. Append-only. Tenant-scoped access. Retention enforced. |
| Future requirements | Cryptographic tamper-evidence. Compliance framework mapping. |

---

### Derived Analytical Data

| Aspect | Detail |
|--------|--------|
| Description | Data produced by aggregating or transforming authoritative data for analysis |
| Examples | Sales aggregates, conversion rates, trend data, usage metrics, recommendation models |
| Ownership | Analytics service (future). Platform metrics (operational). |
| Allowed access | Business roles (tenant analytics). Platform team (aggregate platform analytics, de-identified). |
| Storage | Analytical storage (future). Platform metrics infrastructure. |
| Encryption | Volume-level encryption |
| Logging | Aggregate values may appear in logs. Individual-level data follows source classification. |
| Export | Tenant analytics exportable by org admin. Platform analytics internal only. |
| Retention | While analytically useful. Rebuildable from authoritative sources. |
| Non-production | Synthetic or anonymized |
| Masking | Aggregated data does not require masking. Individual-level data follows source classification. |
| Backup | Rebuildable from sources. Backup optional. |
| Deletion | When no longer needed. Can be rebuilt. |
| Incident severity if exposed | Depends on content (aggregated = Medium, individual-level = follows source classification) |
| V1 requirements | Platform operational metrics only. Individual-tenant analytics is future. |
| Future requirements | Formal analytics pipeline. Anonymization. Purpose limitation. Consent awareness. |

---

## Explicit Prohibitions

| Prohibition | Rationale | Applies To |
|------------|-----------|-----------|
| **Secrets in logs** | Logs are widely accessible and retained. Secret exposure in logs is a high-probability breach vector. | All systems |
| **Authentication tokens in analytics** | Analytics data has broader access. Tokens in analytics pipelines create credential exposure. | Analytics, events, exports |
| **Full payment credentials in application storage** | Raw card data creates PCI scope. Provider tokenization eliminates this need. | All storage |
| **Unmasked restricted data in lower environments** | Non-production environments have weaker security. Restricted data exposure in dev/staging is unacceptable. | All non-production |
| **Sensitive personal data in test fixtures without explicit protection** | Test fixtures are often broadly accessible. Unprotected PII in test data creates exposure risk. | Testing infrastructure |
| **Sensitive exports without authorization and auditing** | Uncontrolled export of sensitive data is a data exfiltration vector. | Export mechanisms |
| **Treating encrypted data as non-sensitive** | Encryption is a control, not a reclassification. Encrypted Restricted data is still Restricted. | All systems |
| **Storing unnecessary personal data** | Data minimization is a principle. Collecting data "just in case" creates liability without value. | All collection points |

---

## Classification Responsibilities by Surface

### Backend

| Responsibility | Detail |
|---------------|--------|
| Field classification | Every entity field is classified in its module specification |
| Access control | Classification determines who can read/write each field |
| Validation | Personal and restricted data collection validated against necessity |
| Event payloads | Events must not include restricted fields. Personal fields are included only when the consuming service requires them. |
| Error responses | Error messages never include classified data values |

### Frontend

| Responsibility | Detail |
|---------------|--------|
| No secrets | Frontend never handles restricted data (API keys, credentials, tokens beyond its own session) |
| Masking | Frontend displays masked values for confidential/personal fields where appropriate |
| No persistent storage of sensitive data | Frontend does not persist classified data beyond the session |
| Token handling | Authentication tokens stored using secure client-side mechanisms |

### SDKs

| Responsibility | Detail |
|---------------|--------|
| No embedded secrets | SDKs never contain secret API keys or credentials |
| Secure token storage | SDK guides specify secure storage for authentication tokens |
| No logging of sensitive data | SDK logging must not capture request/response bodies containing classified data |
| Classification guidance | SDK documentation identifies which fields are sensitive |

### Logs

| Responsibility | Detail |
|---------------|--------|
| Never log: Restricted | Passwords, tokens, keys, secrets, card data — never, under any circumstance |
| Never log: Personal field values | Email addresses, phone numbers, physical addresses — values excluded |
| May log: Identifiers | Request IDs, entity IDs, organization IDs, user IDs — acceptable |
| May log: Public data | Product names, category names — acceptable |
| May log: Financial amounts | Transaction amounts for reconciliation — acceptable |
| Automatic redaction | Platform logging framework redacts known sensitive patterns |

### Events

| Responsibility | Detail |
|---------------|--------|
| No secrets in event payloads | Events never contain passwords, tokens, or API keys |
| Personal data minimized | Events include only the personal data that consuming services require |
| Tenant context in envelope | Organization ID in event envelope (not in payload body) |
| Webhook payloads follow same rules | Outbound webhooks must not include restricted or unnecessary personal data |

### Search

| Responsibility | Detail |
|---------------|--------|
| Searchable fields respect classification | Restricted fields are never indexed |
| Personal data indexing | Personal fields may be indexed for search (with tenant scoping) but not stored in cleartext in the index where avoidable |
| Search results respect access control | Results filtered by caller's authorization |
| Tenant-scoped | Search never returns cross-tenant data |

### Cache

| Responsibility | Detail |
|---------------|--------|
| Tenant-scoped keys | All cached classified data includes tenant context in the key |
| No persistent cache of restricted data | Restricted data is cached in-memory only (never persistent distributed cache) |
| TTL on classified data | Cached personal and confidential data has defined expiration |
| Cache does not reduce classification | Cached data retains its original classification |

### Reports

| Responsibility | Detail |
|---------------|--------|
| Aggregation preferred | Reports prefer aggregated data over individual records |
| Access controlled | Reports showing confidential or personal data require appropriate permission |
| Masking in reports | Personal data in reports follows the same masking rules as API responses |
| Tenant-scoped | Reports never span tenant boundaries (for tenant-visible reports) |

### Exports

| Responsibility | Detail |
|---------------|--------|
| Authorization required | All exports of confidential, personal, or financial data require explicit authorization |
| Audit logged | Every export operation is recorded |
| Scoped to tenant | Exports contain only the authenticated tenant's data |
| Secrets excluded | Exports never include restricted data |
| Classification preserved | Exported data documentation indicates the classification of contained fields |

### Backups

| Responsibility | Detail |
|---------------|--------|
| All backups encrypted | Volume-level encryption minimum |
| Classification preserved | Backup data retains classification of the source |
| Restoration scoped | Restoration must not cross-contaminate tenants |
| Retention aligned | Backup retention follows the source data's retention requirements |
| Restricted data backup | Secret store backed up separately with additional controls |

### Support Tooling

| Responsibility | Detail |
|---------------|--------|
| Masked by default | Support tools mask personal and confidential fields by default |
| Reveal requires authorization | Unmasking requires elevated permission and is audited |
| Read-only by default | Support access to classified data is read-only unless explicitly authorized |
| Time-limited | Support access sessions expire automatically |
| Audited | Every support access to classified data is recorded |

---

## Classification Handling Matrix

| Classification | Log Values | Cache | Search Index | Events | Exports | Non-Production | Masking | Encryption (App) | Deletion on Request |
|---------------|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| Public | Yes | Yes | Yes | Yes | Yes | Yes | No | No | On entity deletion |
| Internal | Yes | Yes | Yes | Yes | Controlled | Yes | No | No | On lifecycle |
| Confidential | No (values) | Yes (TTL) | Yes | Minimal | Authorized + audit | No (anonymized) | Yes | Recommended | On erasure request |
| Restricted | Never | In-memory only | Never | Never | Never | Never (separate) | N/A | Required | Immediately on revocation |
| Personal | No (values) | Yes (TTL) | Limited | Minimal | Customer export | Never (synthetic) | Yes | Required (sensitive fields) | On erasure request |
| Authentication | Never | Session store | Never | Never | Never | Separate | N/A | Required | On expiration/revocation |
| Financial | Limited | Yes (TTL) | Limited | Transaction IDs | Authorized + audit | No (synthetic) | Partial | Recommended | Per regulation |
| Payment | Never (card) | Limited | Never | Transaction IDs | PCI-restricted | Test mode only | Always | Required | Per provider rules |
| Commercially Sensitive | No | Yes (TTL) | Limited | No | Authorized + audit | No (synthetic) | Yes | Recommended | When irrelevant |
| Operational Security | Internal only | No | No | No | Never (tenant) | Separate | N/A | Volume | When superseded |
| Audit Evidence | N/A | No | Limited | N/A | Authorized | No | No | Volume | Retention expiry only |
| Derived Analytical | Source rules | Optional | Optional | Aggregate only | Authorized | Synthetic | Per source | Volume | Rebuildable |

---

## Version Gate

| Version | Data Classification Gate |
|---------|------------------------|
| V1 | Every module identifies classified fields in its specification. Restricted data is in secret store or application-encrypted. Personal data is never in logs or non-production. Payment data uses provider tokenization. Masking is applied for confidential/personal fields. Export and deletion capabilities are operational. Classification handling matrix is followed. |
| V2 | Per-field encryption for sensitive personal data. Advanced masking policies (per-role). Analytics pipeline respects classification. Non-production data generation is automated (synthetic). |
| V3 | Field-level encryption with tenant keys. Data residency classification. Automated classification validation in CI/CD. Compliance mapping per classification. |

---

## Decision Summary

| Decision | Rationale |
|----------|-----------|
| 12 classifications cover all platform data | Fewer would be too coarse (no distinction between personal and financial). More would be unmanageable. 12 provides actionable granularity. |
| Encryption does not reduce classification | An encrypted secret is still a secret. Encryption is a control mechanism, not a reclassification. Classification determines handling rules regardless of encryption state. |
| Personal data never in non-production | Non-production environments have weaker security. Real PII in dev/staging creates disproportionate risk for testing convenience. |
| Logs are the primary prohibition target | Logs are widely accessible, long-retained, and often exported. They are the highest-probability vector for accidental sensitive data exposure. |
| Per-surface responsibility matrices | Different surfaces handle data differently. A blanket rule is insufficient. Each surface has specific handling requirements. |
| Prohibitions are architectural, not advisory | "Should not log secrets" is advice. "Secrets are architecturally prevented from entering logs" is a control. This document defines controls. |

---

## Alternatives Considered

| Alternative | Rejected Because |
|------------|-----------------|
| Fewer classifications (3-4 levels) | Insufficient granularity. Cannot distinguish personal data handling (privacy rules) from payment data handling (PCI rules) from credential handling (immediate revocation). |
| Per-field classification only (no categories) | Categories enable batch rules. Without categories, every field requires individual rule definition. Categories provide patterns that fields inherit. |
| Non-production uses production data (masked) | Even masked production data carries risk. Synthetic data generation is safer and does not create data handling obligations for non-production. |
| Encryption removes classification requirement | Creates false confidence. A breached encryption key exposes everything. Classification ensures defence-in-depth regardless of encryption state. |
| Logging framework handles all redaction automatically | Automatic redaction is required but insufficient alone. Classification awareness prevents sensitive data from being constructed into log messages in the first place. |

---

## Trade-offs

| Trade-off | Accepted Because |
|-----------|-----------------|
| 12 classifications add complexity for developers | Developers learn the categories once. The alternative (ambiguous handling) leads to security incidents. Module specifications document per-field classification. |
| No production personal data in non-production limits test realism | Synthetic data generators produce realistic test data. The security risk of real PII in dev environments exceeds the testing benefit. |
| Masking reduces functionality for support | Support can unmask with authorization (audited). Default masking prevents casual exposure. |
| Event payloads must minimize personal data | Events with less data are less useful. But events are distributed and stored. Minimization limits exposure surface. |
| Per-surface responsibility adds documentation | Each surface has different constraints. A single rule set would either over-restrict some surfaces or under-protect others. |

---

## Architecture Impact

| Concern | Impact |
|---------|--------|
| Module design | Every module classifies every field in its specification. Classification determines access control, logging, caching, and export behavior. |
| Data access layer | Must support field-level access filtering based on classification and caller permission. |
| Logging framework | Must redact classified data patterns automatically. Must prevent construction of log messages containing restricted/personal values. |
| Event system | Event payloads must be reviewed against classification. Restricted data never enters events. Personal data is minimized. |
| Caching | Cache interface must enforce classification-aware TTL and storage restrictions (no persistent cache for restricted data). |
| Search | Index configuration must exclude restricted fields. Personal data indexing follows defined rules. |
| API design | Response models must support field-level masking based on caller permission and classification. |
| Testing | Test data generation must produce synthetic data for classified fields. No production PII in test fixtures. |
| Export | Export mechanisms must check authorization per classification level and audit all exports. |

---

## Implementation Impact

| Area | Impact |
|------|--------|
| Modules | Must classify every field. Must use platform logging (with redaction). Must not include restricted data in events or responses. Must support masking for confidential/personal fields. |
| Platform logging | Must implement pattern-based redaction. Must prevent classified data in log messages. Must be used by all modules (no custom loggers). |
| Platform cache | Must enforce tenant-scoped keys. Must prevent persistent caching of restricted data. Must support TTL for classified data. |
| Events | Must review payload content against classification. Must exclude restricted data. Must minimize personal data. |
| Exports | Must check authorization per classification. Must audit all exports. Must exclude restricted data. |
| Testing | Must use synthetic data for classified fields. Must not import production PII. Must validate that classification rules are followed. |
| Support tooling | Must mask by default. Must require authorization for unmasking. Must audit access to classified data. |

---

## Security Responsibilities

| Role | Classification Responsibilities |
|------|-------------------------------|
| Data Security Lead | Defines classification model. Reviews classification decisions. Maintains handling rules. |
| Module Teams | Classify every field. Follow handling rules. Exclude sensitive data from logs and events. |
| Platform Team | Implement redaction framework, masking infrastructure, classification-aware caching, and export controls. |
| Security Team | Validate classification compliance. Review event payloads. Audit export operations. |
| Operations | Manage backup classification alignment. Ensure non-production data hygiene. |

---

## Multi-Tenancy Responsibilities

| Responsibility | Detail |
|---------------|--------|
| Cross-tenant exposure of any classification is Critical | Even public data exposed cross-tenant is an isolation failure |
| Classification does not change by tenant | All tenants follow the same classification rules |
| Per-tenant configuration of masking (future) | Future: tenants may configure additional masking beyond platform defaults |
| Export scoping | All exports are tenant-scoped regardless of classification |

---

## Out of Scope

This document does not:

- Define specific field classifications per module — defined in module specifications.
- Define encryption algorithms or key management — defined in [Secrets and Key Management](../Security/Secrets-and-Key-Management.md).
- Make legal conclusions about privacy regulations — [Compliance Readiness](../Security/Compliance-Readiness.md) addresses regulatory readiness without claims.
- Define database schemas or storage implementation.
- Define the logging framework implementation — defined in development standards.
- Duplicate [Data Protection](../Security/Data-Protection.md) — this document extends it with operational classification rules.

---

## Future Considerations

- **Automated classification scanning** — CI/CD tooling that detects unclassified fields or classification violations.
- **Classification metadata in API schemas** — Machine-readable classification on API response fields for SDK generation.
- **Dynamic masking policies** — Per-role, per-context masking that adapts to the caller's access level.
- **Data loss prevention (DLP)** — Runtime detection of classified data appearing in unauthorized locations.
- **Classification-aware data lineage** — Tracking how classified data flows and transforms through the system.
- **Consent-aware classification** — Classification rules that adapt based on customer consent state.

---

## Future Refactoring Triggers

This document should be revisited when:

- A new data category does not fit existing classifications.
- Privacy regulations impose specific classification requirements.
- PCI assessment identifies classification gaps for payment data.
- A data exposure incident reveals classification or handling failures.
- The analytics pipeline introduces new classification challenges for derived data.
- Per-tenant encryption keys change the encryption landscape for classified data.
- Multi-region deployment introduces data residency as a classification dimension.

---

## Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 0.1 | 2026-07-20 | Data Security and Privacy Architecture Lead | Initial draft |
