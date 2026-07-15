# Product Boundaries

## Metadata

| Field | Value |
|-------|-------|
| Title | Kairo Product Boundaries |
| Document ID | KAI-PROD-004 |
| Status | Draft |
| Version | 0.1 |
| Target Release | N/A |
| Owner | Chief Product Architect |
| Created | 2026-07-15 |
| Last Updated | 2026-07-15 |
| Reviewers | TODO |
| Related Documents | [Kairo Platform](./Kairo-Platform.md), [Kairo Commerce](./Kairo-Commerce.md), [Product Ecosystem](./Product-Ecosystem.md) |
| Dependencies | None |

---

## Purpose

This document defines the boundaries of every product in the Kairo ecosystem. Clear boundaries prevent overlap, eliminate ambiguity about ownership, and ensure that capabilities are built in the right product.

When a new capability is proposed, this document determines where it belongs. If a capability does not clearly fit within an existing product, it either belongs to the shared platform or becomes a candidate for a new product.

Boundary violations create long-term structural problems. A capability placed in the wrong product becomes a dependency that distorts both products over time. This document exists to prevent that.

---

## Boundary Principles

- **Each product owns one business domain.** Products do not reach into adjacent domains.
- **Ownership is exclusive.** A capability belongs to exactly one product. There is no shared ownership.
- **Integration is not ownership.** A product may consume data from another product, but consuming data does not grant ownership of that data.
- **When in doubt, it is a platform concern.** If a capability is needed by multiple products and does not belong to any single business domain, it belongs to the shared platform.
- **Boundaries are defined by business domain, not by technical convenience.** The question is "which business domain owns this concept?" not "which codebase is easiest to modify?"

---

## Kairo Commerce

### Owns

| Domain | Examples |
|--------|----------|
| Catalog | Products, variants, categories, attributes, product types |
| Pricing | Price lists, price rules, customer-specific pricing, currency-aware pricing |
| Inventory | Stock levels, reservations, stock movements, multi-location inventory |
| Orders | Order creation, order lifecycle, line items, order calculations |
| Cart | Cart management, cart pricing, cart-level discounts, cart persistence |
| Promotions | Discount rules, promotional campaigns, coupon codes, stacking rules |
| Fulfillment orchestration | Shipping methods, fulfillment workflows, shipment tracking references |
| Tax calculation | Tax rules, tax zones, tax categories, tax application to orders and carts |
| Commerce customers | Customer profiles, addresses, customer groups within commerce context |
| Returns | Return requests, return authorization, refund initiation |
| Channels | Sales channels, channel-specific catalog and pricing |

### Does Not Own

| Capability | Belongs To | Reason |
|-----------|-----------|--------|
| User authentication | Shared Platform | Cross-product concern |
| Payment processing | Kairo Payments | Separate financial domain |
| Accounting entries | Kairo ERP | Financial accounting is an ERP concern |
| In-store transactions | Kairo POS | Physical retail is a separate domain |
| Demand forecasting | Kairo AI | Predictive analytics is an AI concern |
| Customer identity and login | Kairo Identity | Identity management is a separate domain |
| Content management | External systems | Not a Kairo concern |
| Email and notifications | Shared Platform | Cross-product concern |

### Boundary Examples

- A product's price is Commerce. The accounting journal entry when that product is sold is ERP.
- A cart's discount calculation is Commerce. The financial reconciliation of that discount is ERP.
- A shipment tracking reference stored on an order is Commerce. The logistics optimization of that shipment is outside Kairo.
- A customer's shipping address is Commerce. That customer's login credentials are Identity.

---

## Kairo ERP

### Owns

| Domain | Examples |
|--------|----------|
| Accounting | Chart of accounts, journal entries, ledgers, fiscal periods |
| Procurement | Purchase orders, supplier management, receiving |
| Financial reporting | Profit and loss, balance sheets, financial summaries |
| Expense management | Expense tracking, approval workflows, cost allocation |
| Asset management | Fixed asset tracking, depreciation |
| Budgeting | Budget definitions, budget tracking, variance analysis |
| Supplier management | Supplier profiles, supplier terms, supplier performance |

### Does Not Own

| Capability | Belongs To | Reason |
|-----------|-----------|--------|
| Product catalog | Kairo Commerce | Catalog is a commerce concept |
| Order management | Kairo Commerce | Orders originate in commerce |
| Payment processing | Kairo Payments | Financial transactions are a payments concern |
| In-store hardware | Kairo POS | Physical retail operations |
| User authentication | Shared Platform | Cross-product concern |
| Inventory levels | Kairo Commerce | Inventory is a commerce operation |

### Boundary Examples

- When an order is placed in Commerce, ERP receives an event and creates the corresponding accounting entries. Commerce owns the order. ERP owns the accounting.
- A supplier's payment terms are ERP. The products that supplier provides are referenced in Commerce's catalog.
- Purchase cost of goods is ERP. The selling price of those goods is Commerce.

---

## Kairo POS

### Owns

| Domain | Examples |
|--------|----------|
| In-store transactions | Register sessions, in-store sales, receipt generation |
| Store management | Store locations, register configurations, staff assignments |
| Physical retail workflows | Cash management, shift operations, drawer reconciliation |
| In-store customer interaction | Walk-in customer handling, loyalty lookups at register |
| Offline operation | Transaction queuing, offline mode, sync on reconnect |
| Hardware integration | Receipt printers, barcode scanners, payment terminals |

### Does Not Own

| Capability | Belongs To | Reason |
|-----------|-----------|--------|
| Product catalog | Kairo Commerce | Catalog is shared; POS consumes it |
| Inventory management | Kairo Commerce | POS triggers inventory changes; Commerce owns the data |
| Pricing and discounts | Kairo Commerce | POS applies prices defined in Commerce |
| Payment processing | Kairo Payments | POS initiates payments; Payments processes them |
| Accounting | Kairo ERP | POS generates transactions; ERP accounts for them |
| Online orders | Kairo Commerce | E-commerce is Commerce's domain |

### Boundary Examples

- A product's price displayed at the register comes from Commerce. The register session and cash drawer belong to POS.
- When a sale is completed at POS, an inventory adjustment event is sent to Commerce. POS does not modify inventory directly.
- An in-store refund is initiated by POS. The financial reversal is handled by Payments. The accounting entry is created by ERP.

---

## Kairo Identity

### Owns

| Domain | Examples |
|--------|----------|
| User authentication | Login, logout, session management, credential storage |
| Authorization | Roles, permissions, access policies, permission evaluation |
| Customer identity | Customer registration, profile management, identity verification |
| Federation | SSO, OAuth providers, external identity provider integration |
| API keys and tokens | API key lifecycle, token issuance, token revocation |
| Tenant identity | Tenant provisioning, tenant-scoped identity boundaries |
| Security policies | Password policies, MFA, account lockout, session expiration |

### Does Not Own

| Capability | Belongs To | Reason |
|-----------|-----------|--------|
| Customer commerce data | Kairo Commerce | Commerce-specific customer attributes |
| Customer accounting data | Kairo ERP | Financial relationship data |
| User activity in products | Each product | Products own their domain-specific activity records |
| Payment credentials | Kairo Payments | Card data and payment tokens are a payments concern |
| Audit logging | Shared Platform | Audit spans all products |

### Boundary Examples

- A customer's email and password are Identity. That customer's order history is Commerce.
- A staff member's role and permissions are Identity. Which register that staff member operates is POS.
- An API key's creation and revocation are Identity. What that API key accesses in Commerce is governed by Commerce's permission model using Identity's authorization framework.

---

## Kairo Payments

### Owns

| Domain | Examples |
|--------|----------|
| Payment processing | Charge initiation, capture, authorization, settlement |
| Refund processing | Refund execution, partial refunds, refund status tracking |
| Payment methods | Payment method types, saved payment methods, method selection |
| Payment provider integration | Provider configuration, provider routing, failover |
| Transaction records | Payment transaction log, reconciliation data, settlement records |
| Payout management | Merchant payouts, payout schedules, payout reconciliation |
| Financial compliance | PCI compliance boundaries, tokenization, sensitive data handling |

### Does Not Own

| Capability | Belongs To | Reason |
|-----------|-----------|--------|
| Order totals | Kairo Commerce | Commerce calculates what is owed |
| Accounting entries | Kairo ERP | ERP records the financial impact |
| Register cash handling | Kairo POS | Physical cash is a POS concern |
| Pricing and discounts | Kairo Commerce | Payment processes the amount; Commerce determines it |
| Customer identity | Kairo Identity | Who is paying is an identity concern |

### Boundary Examples

- Commerce calculates the order total. Payments charges that amount. ERP records the revenue.
- A refund is requested in Commerce. Payments executes the financial reversal. ERP adjusts the accounting.
- A saved credit card token is Payments. The customer who owns that card is Identity. The order that card was used for is Commerce.

---

## Kairo AI

### Owns

| Domain | Examples |
|--------|----------|
| Predictive analytics | Demand forecasting, sales prediction, trend detection |
| Optimization | Pricing optimization, inventory reorder suggestions, search ranking |
| Intelligence | Anomaly detection, fraud signals, operational insights |
| Data processing | Cross-product data aggregation for analysis, model training data preparation |
| Recommendations | Product recommendations, related items, personalization signals |

### Does Not Own

| Capability | Belongs To | Reason |
|-----------|-----------|--------|
| Product catalog | Kairo Commerce | AI reads catalog data; Commerce owns it |
| Pricing rules | Kairo Commerce | AI suggests prices; Commerce enforces them |
| Inventory levels | Kairo Commerce | AI predicts demand; Commerce manages stock |
| Financial data | Kairo ERP | AI analyzes patterns; ERP owns the records |
| Transaction processing | Kairo Payments | AI detects anomalies; Payments processes transactions |
| User identity | Kairo Identity | AI personalizes; Identity authenticates |

### Boundary Examples

- AI predicts that a product will sell out next week. Commerce decides whether to reorder. ERP processes the purchase order.
- AI suggests an optimal price. Commerce applies it through its pricing engine. AI does not modify prices directly.
- AI detects a suspicious transaction pattern. Payments decides whether to flag or block the transaction. AI provides the signal, not the enforcement.

---

## Shared Platform

### Owns

| Domain | Examples |
|--------|----------|
| API gateway | Routing, rate limiting, API versioning, request validation |
| Event infrastructure | Event bus, event schemas, delivery guarantees, subscription management |
| Multi-tenancy | Tenant isolation, tenant configuration, tenant-scoped data boundaries |
| Audit logging | Cross-product audit trail, compliance logging, retention policies |
| Notification infrastructure | Email delivery, webhook dispatch, notification templates |
| Observability | Logging standards, metrics collection, distributed tracing |
| Developer tooling | SDKs, CLI, sandbox environments, API documentation hosting |
| Configuration management | Feature flags, environment configuration, platform settings |

### Does Not Own

| Capability | Belongs To | Reason |
|-----------|-----------|--------|
| Business logic of any domain | Respective product | Each product owns its domain logic |
| Domain-specific data models | Respective product | Products define their own entities |
| Product-specific APIs | Respective product | Products expose their own endpoints |
| Product-specific permissions | Respective product | Products define what actions exist; platform enforces access |

### Boundary Examples

- The platform provides the event bus. Commerce defines what events it publishes. ERP decides which Commerce events it subscribes to.
- The platform enforces tenant isolation. Commerce defines what data exists within a tenant's scope.
- The platform provides the API gateway. Each product registers its own routes and handles its own requests.

---

## Resolving Boundary Disputes

When a capability does not clearly belong to a single product:

1. **Identify the business domain.** Which domain concept does this capability serve? The answer determines ownership.
2. **Apply the single-owner rule.** Assign ownership to exactly one product. Do not split a capability across products.
3. **Prefer the platform for cross-cutting concerns.** If two or more products need the same generic capability, it belongs to the platform.
4. **Record the decision.** Create an ADR documenting the boundary decision, the alternatives considered, and the rationale.
5. **Update this document.** Add the capability to the appropriate product's boundary definition.

Boundary decisions are structural. They are difficult to reverse once implemented. Invest the time to get them right.
