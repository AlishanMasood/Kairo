# Data Protection

## Metadata

| Field | Value |
|-------|-------|
| Title | Kairo Data Protection Architecture |
| Document ID | KAI-SEC-006 |
| Status | Draft |
| Version | 0.1 |
| Target Release | V1 |
| Owner | Data Security Architect |
| Created | 2026-07-20 |
| Last Updated | 2026-07-20 |
| Reviewers | TODO |
| Related Documents | [Security Architecture](./Security-Architecture.md), [Threat Model](./Threat-Model.md), [Platform Services](../../05-Platform-Core/Platform-Services.md), [Organization Model](../../05-Platform-Core/Organization-Model.md), [Identity and Authentication](./Identity-and-Authentication.md), [API Security](./API-Security.md), [Glossary](../../02-Products/Glossary.md) |
| Dependencies | [Security Architecture](./Security-Architecture.md) |

---

## Purpose

This document defines how the Kairo platform protects data throughout its lifecycle — from creation through storage, processing, transmission, export, and eventual deletion. It establishes data classification, encryption requirements, access controls, and handling rules for every category of data the platform manages.

Data protection is a structural property of the platform. It is enforced by architecture, not by individual developer decisions. The platform makes it difficult to mishandle data and easy to handle it correctly.

---

## Scope

This document covers:

- Data classification and handling rules for all data categories.
- Encryption requirements (transit and rest).
- Data minimization, retention, and deletion.
- Log redaction and non-production data rules.
- Data ownership and portability.
- V1 baseline and future regulatory capabilities.

This document does not cover:

- Database schemas or specific storage implementations.
- Specific encryption algorithms or libraries.
- Compliance audit procedures (documented separately when compliance programs are established).
- Incident response for data breaches (documented in operational guides).

---

## Data Protection Objectives

| Objective | Description |
|-----------|-------------|
| **Protect confidentiality** | Data is accessible only to authorized principals within the correct tenant boundary. |
| **Maintain integrity** | Data is accurate, complete, and unmodified by unauthorized actors. |
| **Ensure availability** | Data is accessible to authorized principals when needed. |
| **Enforce isolation** | Tenant data is never visible to other tenants, regardless of any failure in application logic. |
| **Support compliance** | Data handling enables compliance with applicable regulations without requiring platform re-architecture. |
| **Enable portability** | Tenants can export their data in a usable format. The platform does not hold data hostage. |
| **Minimize exposure** | The platform collects and retains only the data necessary for its function, for only as long as necessary. |

---

## Data Classification

Every piece of data in the platform is classified into one of the following categories. Classification determines handling rules, access controls, logging policy, and retention.

### Classification Matrix

| Classification | Examples | Allowed Access | Logging Policy | Retention Direction | Export Restrictions |
|---------------|----------|---------------|---------------|--------------------|--------------------|
| **Public** | Published product names, public catalog data, published prices | Any authenticated request within the tenant | Full logging permitted | Retained while the entity exists | No restrictions within tenant |
| **Internal** | Order processing state, internal event metadata, system configuration | Authorized workforce and service identities | Logged without sensitive context | Retained per operational need | Organization admin only |
| **Confidential** | Customer email addresses, phone numbers, shipping addresses, order details | Authorized workforce with explicit permission; customer self-access | Log access events, not field values | Retained while business relationship exists + compliance period | Controlled export with audit |
| **Restricted** | Password hashes, API key hashes, encryption keys, MFA secrets | Platform identity service only; never exposed through APIs | Never logged. Access audited. | Retained while credential is active | Never exported |
| **Payment** | Payment provider tokens, transaction references, refund details | Payment service and authorized finance roles | Transaction IDs logged; amounts logged; card data never logged | Per payment provider and regulatory requirements | Subject to PCI scope restrictions |
| **Personal** | Customer names, addresses, emails, purchase history, IP addresses | Authorized access within purpose limitation | Access events logged; field values not logged | Deleted upon request or after retention period | Subject to data portability rights |

---

## Public Data

Data that is intentionally published and accessible to any authenticated request within the tenant.

| Aspect | Rule |
|--------|------|
| Examples | Published product titles, descriptions, public prices, category names, published media |
| Access | Any request with valid tenant context (publishable key or higher) |
| Modification | Requires explicit write permission |
| Logging | May appear in logs freely |
| Caching | May be cached (tenant-scoped) |
| Export | No additional restrictions |

---

## Internal Data

Operational data that supports platform function but is not intended for external exposure.

| Aspect | Rule |
|--------|------|
| Examples | Event processing state, queue metadata, internal identifiers, system health metrics, feature flag state |
| Access | Authorized workforce and platform services |
| Modification | Platform services or authorized administrators |
| Logging | May appear in operational logs |
| Caching | Permitted for operational efficiency |
| Export | Available to organization administrators for operational purposes |

---

## Confidential Data

Business and customer data that requires controlled access and protection.

| Aspect | Rule |
|--------|------|
| Examples | Customer email, phone, address; order details; invoice data; integration configuration; business reports |
| Access | Workforce users with explicit permission; customers accessing their own data |
| Modification | Authorized users within their permission scope |
| Logging | Access events logged (who accessed what, when). Field values excluded from logs. |
| Caching | Permitted with tenant scoping and appropriate TTL |
| Export | Controlled. Audit logged. Organization admin authorization required for bulk export. |

---

## Restricted Data

The most sensitive data in the platform. Access is structurally limited to the minimum services that require it.

| Aspect | Rule |
|--------|------|
| Examples | Password hashes, API key material, encryption keys, MFA secrets, signing keys |
| Access | Platform identity and cryptographic services only. Never returned through any API. |
| Modification | Platform services only through defined credential management flows |
| Logging | **Never logged under any circumstance.** Access to the secret store is audited. |
| Caching | Permitted only in memory for operational necessity. Never in shared or persistent caches. |
| Export | **Never exported.** Credential rotation replaces old values. |

---

## Personal Data

Data that identifies or relates to a natural person. Subject to data protection regulations.

| Aspect | Rule |
|--------|------|
| Examples | Customer names, email addresses, phone numbers, shipping addresses, IP addresses, purchase history, browsing signals |
| Access | Purpose-limited. Accessible only for the purpose it was collected. |
| Modification | Customer self-service or authorized workforce with purpose justification |
| Logging | Access events logged. Field values never logged. |
| Retention | Retained only while the purpose exists. Deleted upon customer request or after retention period. |
| Export | Subject to data portability. Customer can request export of their personal data. |
| Deletion | Subject to right of deletion. Customer can request deletion with defined exceptions (legal retention obligations). |

---

## Payment-Related Data

Data associated with financial transactions and payment processing.

| Aspect | Rule |
|--------|------|
| Examples | Payment provider tokens, transaction IDs, payment amounts, refund references, payment method types |
| Access | Payment service and authorized finance roles. Never accessible to general workforce without explicit permission. |
| Modification | Payment service only through defined transaction flows |
| Logging | Transaction IDs and amounts may be logged for reconciliation. Card numbers, CVVs, and full account numbers are **never logged**. |
| Storage | **The platform must not store payment data that can be replaced by provider tokenization.** Raw card numbers are never stored. Provider tokens are stored instead. |
| PCI boundary | Payment data is handled within the PCI scope boundary defined in the [Security Architecture](./Security-Architecture.md). |

---

## Authentication and Credential Data

Data used to verify identity or grant access.

| Aspect | Rule |
|--------|------|
| Examples | Password hashes, OAuth tokens, API key hashes, session tokens, MFA seeds, recovery codes |
| Access | Platform identity service only |
| Storage | Hashed with modern algorithms (passwords). Encrypted at rest (tokens, seeds). |
| Logging | **Passwords, tokens, secrets, or card security codes must never appear in logs.** |
| API exposure | Never returned through APIs. Password fields are write-only. Token values are shown only at creation time. |
| Rotation | Supported without downtime. Old credentials invalidated after rotation. |

---

## Encryption in Transit

All data in transit is encrypted.

### Requirements

| Channel | Encryption | Standard |
|---------|-----------|----------|
| Client → API Gateway | TLS | Minimum TLS 1.2. TLS 1.3 preferred. |
| API Gateway → Application | TLS or internal transport security | Encrypted within the deployment boundary |
| Application → Database | TLS | Authenticated, encrypted connections |
| Application → Cache | TLS | Encrypted connections to Redis |
| Application → Message Broker | TLS | Encrypted connections to RabbitMQ |
| Platform → External Services | TLS | Minimum TLS 1.2 |
| Outbound Webhooks | TLS | HTTPS required for webhook endpoints |

### Rules

- Plaintext HTTP is never accepted from external clients.
- Certificate validation is mandatory for outbound connections.
- Internal service communication uses encrypted transport even within the deployment boundary.
- Deprecated TLS versions (1.0, 1.1) are rejected.

---

## Encryption at Rest

Sensitive data is encrypted when stored.

### Requirements

| Data Category | Encryption at Rest | Key Management |
|--------------|-------------------|----------------|
| Restricted (credentials, keys) | Mandatory — application-level encryption | Platform-managed keys with rotation |
| Payment (provider tokens) | Mandatory — application-level encryption | Dedicated payment encryption keys |
| Personal (customer PII) | Mandatory — field-level or volume-level encryption | Platform-managed keys |
| Confidential (business data) | Volume-level encryption (database/storage encryption) | Infrastructure-managed keys |
| Internal | Volume-level encryption | Infrastructure-managed keys |
| Public | Volume-level encryption (included with infrastructure encryption) | Infrastructure-managed keys |

### Rules

- All storage volumes are encrypted at the infrastructure level (database, cache, object storage, message broker).
- Sensitive fields (credentials, payment tokens, PII) receive additional application-level encryption above volume encryption.
- Encryption keys are managed through the platform's secret store. They are never stored alongside the data they protect.
- Key rotation is supported without data re-encryption downtime (envelope encryption pattern).

---

## Field-Level Protection Direction

For future versions, the platform will support field-level encryption and access controls.

### V1 Approach

- Restricted and payment data fields are encrypted at the application level before storage.
- PII fields are protected by volume-level encryption and access controls.
- Field-level access filtering is implemented for defined sensitive fields (masking in API responses based on permission).

### Future Direction

- Per-field encryption with per-tenant keys, enabling cryptographic tenant isolation.
- Field-level access policies that control which roles can decrypt which fields.
- Searchable encryption for specific use cases (searching encrypted customer data without decryption).

---

## Data Minimization

The platform collects and stores only the data necessary for its defined function.

### Principles

- Do not collect data "in case it is useful later." Collect what is needed for a defined purpose.
- Do not store derived data that can be recalculated on demand.
- Do not retain data beyond its defined retention period.
- Do not copy data between systems unless the receiving system has a defined need.
- Default form fields to optional unless the field is required for the operation.

### Application

| Context | Minimization Rule |
|---------|-----------------|
| Customer registration | Collect only what is required for account creation. Additional data is collected when needed (e.g., address at checkout, not registration). |
| Order data | Retain order details for business and compliance purposes. Remove or anonymize personal data after retention period. |
| Logs | Log operational context (request ID, endpoint, status, duration). Exclude request/response bodies containing personal data. |
| Analytics | Aggregate rather than store individual-level data where possible. |

---

## Purpose Limitation

Data collected for one purpose is not used for a different purpose without explicit justification.

### Rules

- Customer data collected for order fulfillment is not used for marketing without separate consent.
- Authentication data (login times, IP addresses) is used for security, not for business analytics.
- Integration credentials are used for the configured integration, not shared with other services.
- The platform does not use tenant data for platform improvement, model training, or any purpose beyond serving that tenant.

---

## Retention and Deletion

### Retention Rules

| Data Category | Retention Direction |
|--------------|-------------------|
| Active business data (products, active orders) | Retained while the entity is active |
| Completed orders | Retained for a defined business and compliance period |
| Customer personal data | Retained while business relationship exists + compliance period. Deleted on request (with legal exceptions). |
| Authentication credentials | Retained while active. Deleted immediately upon revocation/rotation. |
| Payment data | Retained per payment provider requirements and financial regulations |
| Audit records | Retained for compliance period (longer than business data) |
| Logs | Retained for operational period (days to weeks). Not indefinite. |

### Deletion Rules

- **Indefinite data retention without purpose is prohibited.** Every data category has a defined retention period or a triggering condition for deletion.
- Customer deletion requests result in deletion or anonymization of personal data within a defined timeframe.
- Deletion of personal data does not delete audit records (which retain the action record but may anonymize the actor).
- Deletion is verified. Deleted data does not persist in caches, backups beyond retention, or secondary stores.

---

## Backup Protection

### Requirements

- Backups are encrypted at rest with the same or stronger protection as the primary data.
- Backup access is restricted to operations personnel with explicit authorization.
- Backup restoration is audited.
- Backups follow the same retention schedule as the data they contain. Old backups are deleted when they exceed the retention period.
- Backups do not circumvent deletion. Data deleted from the primary store is not recoverable from backups beyond the backup retention window.

---

## Export Protection

### Rules

- **Uncontrolled data exports are prohibited.** All bulk data exports require authorization and are audit logged.
- Export operations are subject to rate limiting and quota controls.
- Exported data respects classification. Restricted data is never included in exports.
- Export formats do not include internal system identifiers or metadata not relevant to the tenant.
- Organization administrators can export their organization's data (data portability).
- Customer personal data export follows data portability requirements (machine-readable format).

---

## Log Redaction

### Mandatory Redactions

The following data must **never** appear in logs, regardless of log level:

- Passwords (plaintext or hashed)
- Access tokens and refresh tokens
- API key values (secret keys)
- MFA secrets and recovery codes
- Card numbers, CVVs, or card security codes
- Encryption keys or signing keys
- Integration credentials (provider API keys, secrets)
- Full customer email addresses in operational logs (use truncated or hashed identifiers)

### Logging Principles

- Log the operation, not the data. "Customer 12345 updated their address" is appropriate. The actual address value is not.
- Log identifiers, not values. Request ID, entity ID, and field name are appropriate. Field values for personal or confidential data are not.
- Error logs must not include request bodies or response bodies that contain sensitive data.
- If a log entry would contain sensitive data, the logging framework must redact it automatically.

---

## Sensitive Data Masking

### API Response Masking

- Sensitive fields are masked in API responses when the caller does not have permission to view the full value.
- Masking shows a partial value (e.g., email: `j***@example.com`, card: `****1234`) to confirm existence without exposing the full value.
- Masking is applied server-side. The full value never leaves the platform unless the caller is explicitly authorized.

### Administrative Display

- Administrative interfaces that display sensitive data must require elevated permissions.
- Full values are revealed only when operationally necessary, with audit logging.

---

## Data Residency Considerations

### V1 Approach

- V1 operates in a single deployment region. Data residency is determined by the deployment location.
- The platform does not offer per-tenant regional data placement in V1.
- The deployment region is documented and communicated to tenants.

### Future Direction

- Per-tenant or per-region data residency for organizations with regulatory requirements.
- Data sovereignty controls that prevent data from leaving a defined geographic boundary.
- Multi-region deployments with region-pinned tenant data.

Data residency is a future capability. It is not required for V1 but the architecture must not preclude it.

---

## Non-Production Data

### Rules

- **Production personal data must not be copied casually into development environments.** Non-production environments use synthetic, anonymized, or redacted data.
- If production data is needed for debugging, it must be anonymized before transfer and accessed only by authorized personnel with audit logging.
- Non-production environments use separate credentials, separate encryption keys, and separate secret stores from production.
- Test data generators provide realistic but fictional data for development and testing.
- Staging environments that require realistic data volumes use anonymized production snapshots, not raw copies.

---

## Data Access Auditing

### What Is Audited

| Access Type | Audit Requirement |
|-------------|------------------|
| Read access to personal data | Logged with accessor identity, resource, and timestamp |
| Write access to any data | Logged with actor, action, resource, old/new values (excluding sensitive field values) |
| Bulk export | Logged with requestor, scope, volume, and timestamp |
| Direct database access (operations) | Logged with accessor, query scope, and justification |
| Backup restoration | Logged with requestor, scope, and justification |
| Data deletion | Logged with requestor, scope, and retention justification |

### Audit Rules

- Data access auditing is separate from operational logging. Audit records have longer retention and immutability guarantees.
- Audit entries for data access never contain the accessed data values (which would create a secondary exposure).
- Audit records are themselves protected as Internal/Confidential data.

---

## Data Ownership and Portability

### Ownership

- **Tenant data belongs to the tenant.** The platform stores and processes it on behalf of the tenant. The platform does not claim ownership of tenant business data.
- The platform owns platform operational data (infrastructure metrics, system logs, aggregate anonymized usage data).
- Audit records are jointly owned — the tenant owns the business context; the platform owns the compliance record.

### Portability

- Tenants can export their business data in a standard, machine-readable format.
- Customer personal data is exportable by the customer (data portability right) or by the organization administrator.
- Export formats are documented and stable. Format changes follow the platform's versioning and deprecation policies.
- Data portability does not include platform internals (internal IDs, processing metadata, system configuration).
- **Returning internal fields through public API models is prohibited.** Export models are explicitly designed for external consumption.

---

## V1 Baseline

| Capability | V1 Status |
|-----------|-----------|
| Data classification applied to all data categories | Required |
| TLS for all external communication | Required |
| TLS for all internal communication (database, cache, broker) | Required |
| Volume-level encryption at rest (all storage) | Required |
| Application-level encryption for restricted and payment data | Required |
| Log redaction for all prohibited data categories | Required |
| No production personal data in non-production environments | Required |
| Audit logging for data access to personal data | Required |
| Audit logging for bulk exports | Required |
| Data retention policies defined per category | Required |
| Customer data deletion capability | Required |
| Tenant data export capability | Required |
| API response masking for sensitive fields | Required |
| No internal fields in public API models | Required |
| Payment data tokenization (no raw card storage) | Required |
| Backup encryption | Required |

## Future Regulatory and Regional Capabilities

| Capability | Target Version | Description |
|-----------|---------------|-------------|
| Per-tenant data residency | V2+ | Tenant data pinned to a specific geographic region |
| GDPR consent management | V2+ | Consent tracking and purpose-linked data processing |
| Right to erasure automation | V2+ | Automated personal data deletion across all stores and services |
| Data processing agreements | V2+ | Platform support for DPA requirements |
| Cross-border data transfer controls | V3+ | Controls for international data movement |
| Per-field encryption with tenant keys | V3+ | Cryptographic tenant isolation at the field level |
| Data Loss Prevention (DLP) | V3+ | Automated detection of sensitive data in inappropriate locations |
| Regional compliance profiles | Future | Pre-configured compliance settings per jurisdiction |
| Data sovereignty certification | Future | Formal verification of data residency compliance |
| Searchable encryption | Future | Query encrypted fields without decryption |

---

## Version Gate

| Version | Data Protection Gate |
|---------|---------------------|
| V1 | All V1 baseline capabilities are operational. Classification is applied. Encryption covers all layers. Log redaction is verified. Non-production data rules are enforced. Deletion and export work for tenant and customer data. |
| V2 | Per-tenant data residency is available for regulated tenants. GDPR-aligned consent and erasure are operational. Data retention automation deletes expired data. DPA support is documented. |
| V3 | Per-field encryption with tenant keys is available. Cross-border transfer controls are operational. DLP detects sensitive data in logs and non-production environments automatically. |

---

## Decision Summary

| Decision | Rationale |
|----------|-----------|
| Classification-driven handling | Classification provides clear, enforceable rules. Without classification, handling decisions are ad-hoc and inconsistent. |
| Never log credentials or secrets | Log exposure is a common breach vector. Architectural prohibition removes the risk entirely. |
| Provider tokenization for payment data | Raw card storage creates PCI scope. Tokenization eliminates the need to store card data at all. |
| No production PII in non-production | Development environments have weaker security. Real personal data in development creates unnecessary exposure. |
| Tenant owns their data | Data ownership builds trust. Platform lock-in through data holding destroys it. |
| Explicit API response models | Serializing internal objects risks leaking internal fields. Explicit models make exposure impossible by default. |
| Retention requires purpose | Indefinite retention creates unbounded liability. Purpose-linked retention defines a clear endpoint. |

---

## Architecture Impact

| Concern | Impact |
|---------|--------|
| Data model | Every field is classified. Classification metadata drives encryption, logging, and access decisions. |
| API design | Response models are explicit contracts. Internal fields are never included. Sensitive fields support masking. |
| Logging infrastructure | The logging framework automatically redacts prohibited data patterns. Redaction is not optional. |
| Module design | Modules handle data according to classification rules. Modules use platform encryption services, not custom implementations. |
| Testing | Tests verify log redaction, response filtering, and classification enforcement. Production data is never used in test environments. |
| Operations | Backup, export, and deletion operations follow classification-driven rules. Access to production data is audited. |

---

## Implementation Impact

| Area | Impact |
|------|--------|
| Modules | Must classify all fields in their specifications. Must use platform encryption for restricted and payment data. Must not log sensitive field values. Must define retention for all data categories. |
| APIs | Must use explicit response models. Must implement field masking for sensitive data. Must not expose internal identifiers. |
| Logging | Must use the platform logging framework with built-in redaction. Must never construct log messages containing sensitive values. |
| Database | Must use encrypted connections. Must apply volume encryption. Must support field-level encryption where required. |
| CI/CD | Must not contain production data. Must use synthetic test data. Must verify that log redaction rules are enforced. |
| Operations | Must encrypt backups. Must audit data access. Must enforce retention schedules. Must anonymize data before non-production use. |

---

## Security Responsibilities

| Role | Data Protection Responsibilities |
|------|-------------------------------|
| Data Security Architect | Defines data protection architecture. Maintains classification standards. Reviews data-handling changes. |
| Platform Team | Implements encryption services, log redaction framework, export capabilities, and deletion mechanisms. |
| Product Teams | Classify fields in module specifications. Use platform encryption and logging services. Define retention per data category. |
| Operations | Manage encryption keys. Enforce backup policies. Audit data access. Execute retention schedules. Anonymize data for non-production. |
| Organization Administrators | Manage data export. Handle customer deletion requests. Configure retention where configurable. |

---

## Out of Scope

This document does not define:

- Specific encryption algorithms or key lengths — defined in implementation specifications.
- Database schemas for data storage — defined in module specifications.
- Compliance procedures for specific regulations (GDPR, CCPA, PCI DSS) — defined in future compliance documentation (dependency identified).
- Data architecture (schema design, data flow, data warehouse) — defined in future Data Architecture documentation (dependency identified).
- Incident response for data breaches — defined in operational documentation.

---

## Future Considerations

- **Data lineage** — Tracking where data originated, how it was transformed, and where it flows.
- **Automated classification** — Tooling that detects unclassified fields and suggests classification.
- **Consent-aware processing** — Processing pipelines that check consent before operating on personal data.
- **Privacy-preserving analytics** — Differential privacy or aggregation techniques for analytics without individual-level exposure.
- **Data clean rooms** — Controlled environments for data analysis without exposing raw data.
- **Compliance dashboard** — Real-time visibility into data protection posture per regulation.

---

## Future Refactoring Triggers

This document should be revisited when:

- A specific regulatory compliance program is initiated (GDPR, PCI DSS, SOC 2).
- The Data Architecture phase is formally defined (data flow patterns, storage strategy).
- Multi-region deployment is introduced (data residency becomes an active requirement).
- The Payments product enters development (PCI scope requires detailed data handling rules).
- A data breach or near-miss occurs (validate classification and controls).
- New data categories emerge that do not fit existing classification levels.

---

## Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 0.1 | 2026-07-20 | Data Security Architect | Initial draft |
