# Naming Conventions

## Metadata

| Field | Value |
|-------|-------|
| Title | Kairo Platform Naming Conventions |
| Document ID | KAI-PROD-008 |
| Status | Draft |
| Version | 0.1 |
| Target Release | N/A |
| Owner | Chief Product Architect |
| Created | 2026-07-15 |
| Last Updated | 2026-07-15 |
| Reviewers | TODO |
| Related Documents | [Kairo Platform](./Kairo-Platform.md), [Product Ecosystem](./Product-Ecosystem.md), [Documentation Standards](../00-Governance/Documentation-Standards.md) |
| Dependencies | None |

---

## Purpose

Consistent naming across the platform eliminates ambiguity, reduces cognitive load, and ensures that developers, contributors, and automated systems can navigate the ecosystem predictably. These conventions apply to every product, module, service, and artifact in the Kairo ecosystem.

Names are permanent decisions. Renaming after adoption is expensive and disruptive. Apply these conventions from the start.

---

## General Rules

- Names are descriptive and specific. They communicate purpose without requiring context.
- Names use domain terminology, not implementation terminology.
- Abbreviations are avoided unless universally understood (API, SDK, URL, ID).
- Names do not include version numbers, status indicators, or temporal references.
- Names are in English.
- Casing conventions are defined per category and must be followed without exception.

---

## Products

**Convention:** `Kairo {ProductName}`

**Casing:** Title Case

Products are named with the `Kairo` prefix followed by a single descriptive word that represents the business domain.

| Example | Correct |
|---------|---------|
| Kairo Commerce | Yes |
| Kairo ERP | Yes |
| Kairo Payments | Yes |
| Kairo Identity | Yes |
| Kairo POS | Yes |
| Kairo AI | Yes |
| Kairo OMS | Yes |

| Example | Incorrect | Reason |
|---------|-----------|--------|
| Commerce Engine | Missing Kairo prefix |
| Kairo Commerce Platform | Redundant; Kairo is already the platform |
| Kairo eCommerce | Inconsistent casing |
| KairoCommerce | Missing space |

---

## Modules

**Convention:** `{domain}.{subdomain}` or `{domain}`

**Casing:** PascalCase

Modules represent bounded contexts within a product. Module names reflect the business domain they own.

| Example | Product |
|---------|---------|
| Catalog | Commerce |
| Pricing | Commerce |
| Inventory | Commerce |
| Orders | Commerce |
| Fulfillment | Commerce |
| Customers | Commerce |
| Promotions | Commerce |
| Tax | Commerce |
| Accounting | ERP |
| Procurement | ERP |

| Example | Incorrect | Reason |
|---------|-----------|--------|
| ProductService | Implementation-oriented, not domain-oriented |
| OrderMgmt | Abbreviation |
| catalog_module | Wrong casing |

---

## Services

**Convention:** `{product}.{module}` or `{module}Service`

**Casing:** PascalCase

Service names identify the product and module they belong to. Internal service names reflect the domain, not the technology.

| Example | Context |
|---------|---------|
| Commerce.Catalog | Catalog service within Commerce |
| Commerce.Orders | Orders service within Commerce |
| Identity.Authentication | Authentication service within Identity |
| Payments.Transactions | Transactions service within Payments |

| Example | Incorrect | Reason |
|---------|-----------|--------|
| CatalogAPI | Conflates service with transport |
| OrderProcessor | Implementation-oriented |
| svc-catalog | Abbreviated, wrong casing |

---

## Repositories

**Convention:** `kairo-{product}` or `kairo-{product}-{component}`

**Casing:** lowercase with hyphens (kebab-case)

Repository names use the `kairo-` prefix for platform-wide identification.

| Example | Purpose |
|---------|---------|
| kairo-commerce | Commerce product source code |
| kairo-identity | Identity product source code |
| kairo-payments | Payments product source code |
| kairo-platform | Shared platform infrastructure |
| kairo-docs | Architecture and documentation repository |
| kairo-commerce-sdk-dotnet | Commerce .NET SDK |
| kairo-commerce-sdk-node | Commerce Node.js SDK |

| Example | Incorrect | Reason |
|---------|-----------|--------|
| commerce | Missing kairo prefix |
| Kairo-Commerce | Wrong casing |
| kairo_commerce | Underscores instead of hyphens |
| kairo-commerce-v2 | Version in name |

---

## Namespaces

**Convention:** `Kairo.{Product}.{Module}.{SubModule}`

**Casing:** PascalCase with dot separation

Namespaces mirror the product and module hierarchy.

| Example | Context |
|---------|---------|
| Kairo.Commerce | Commerce product root |
| Kairo.Commerce.Catalog | Catalog module |
| Kairo.Commerce.Orders | Orders module |
| Kairo.Commerce.Pricing | Pricing module |
| Kairo.Identity | Identity product root |
| Kairo.Platform.Events | Platform event infrastructure |

| Example | Incorrect | Reason |
|---------|-----------|--------|
| Commerce.Catalog | Missing Kairo root |
| Kairo.Commerce.Catalog.Services.Internal | Exposes implementation structure |
| kairo.commerce | Wrong casing |

---

## SDKs

**Convention:** `kairo-{product}-sdk-{language}`

**Casing:** lowercase with hyphens (kebab-case)

SDK names identify the product and target language or platform.

| Example | Target |
|---------|--------|
| kairo-commerce-sdk-dotnet | .NET |
| kairo-commerce-sdk-node | Node.js |
| kairo-commerce-sdk-python | Python |
| kairo-commerce-sdk-go | Go |
| kairo-payments-sdk-dotnet | .NET |

| Example | Incorrect | Reason |
|---------|-----------|--------|
| kairo-sdk | Missing product identifier |
| kairo-commerce-csharp | Inconsistent language naming |
| KairoCommerceSDK | Wrong casing for package/repository name |

---

## Packages

**Convention varies by ecosystem:**

| Ecosystem | Convention | Example |
|-----------|-----------|---------|
| NuGet (.NET) | `Kairo.{Product}.{Module}` | Kairo.Commerce.Catalog |
| npm (Node.js) | `@kairo/{product}-{module}` | @kairo/commerce-catalog |
| PyPI (Python) | `kairo-{product}-{module}` | kairo-commerce-catalog |
| Go | `github.com/kairo/{product}` | github.com/kairo/commerce |

| Example | Incorrect | Reason |
|---------|-----------|--------|
| Kairo.Commerce.Catalog.Helpers | Exposes internal structure |
| @kairo/sdk | Too vague |
| kairo_commerce | Wrong separator for npm |

---

## Events

**Convention:** `{product}.{module}.{entity}.{action}`

**Casing:** PascalCase with dot separation

Event names describe what happened in domain terms. They use past tense for the action.

| Example | Meaning |
|---------|---------|
| Commerce.Orders.Order.Created | An order was created |
| Commerce.Orders.Order.Completed | An order was completed |
| Commerce.Orders.Order.Cancelled | An order was cancelled |
| Commerce.Inventory.Stock.Reserved | Stock was reserved |
| Commerce.Inventory.Stock.Adjusted | Stock was adjusted |
| Commerce.Catalog.Product.Published | A product was published |
| Commerce.Pricing.PriceList.Updated | A price list was updated |
| Payments.Transactions.Payment.Captured | A payment was captured |
| Identity.Authentication.User.LoggedIn | A user logged in |

| Example | Incorrect | Reason |
|---------|-----------|--------|
| OrderCreated | Missing product and module context |
| Commerce.CreateOrder | Imperative, not past tense |
| order.created | Wrong casing |
| Commerce.Orders.Order.Create | Not past tense; sounds like a command |

---

## APIs

### Endpoint Paths

**Convention:** `/api/{version}/{product}/{module}/{resource}`

**Casing:** lowercase with hyphens for multi-word resources

| Example | Purpose |
|---------|---------|
| /api/v1/commerce/catalog/products | List products |
| /api/v1/commerce/catalog/products/{id} | Get a specific product |
| /api/v1/commerce/orders/orders | List orders |
| /api/v1/commerce/inventory/stock-levels | List stock levels |
| /api/v1/commerce/pricing/price-lists | List price lists |
| /api/v1/identity/users | List users |

| Example | Incorrect | Reason |
|---------|-----------|--------|
| /api/v1/getProducts | Verb in path; not RESTful |
| /api/v1/commerce/catalog/Products | Wrong casing |
| /api/v1/commerce/catalog/product | Singular for collection endpoint |
| /api/commerce/catalog/products | Missing version |

### API Header Prefix

**Convention:** `X-Kairo-{Name}`

| Example | Purpose |
|---------|---------|
| X-Kairo-Tenant-Id | Tenant identifier |
| X-Kairo-Channel-Id | Channel context |
| X-Kairo-Request-Id | Request correlation |

---

## Databases

**Convention:** `kairo_{product}_{module}`

**Casing:** lowercase with underscores (snake_case)

Database names identify the product and module they serve. Each module owns its data and has its own logical database or schema.

| Example | Owner |
|---------|-------|
| kairo_commerce_catalog | Commerce Catalog module |
| kairo_commerce_orders | Commerce Orders module |
| kairo_commerce_inventory | Commerce Inventory module |
| kairo_commerce_pricing | Commerce Pricing module |
| kairo_identity_auth | Identity Authentication module |

| Example | Incorrect | Reason |
|---------|-----------|--------|
| commerce | Too vague; missing kairo prefix |
| kairo-commerce-catalog | Hyphens instead of underscores |
| KairoCatalog | Wrong casing; missing product |
| kairo_commerce_db | Redundant suffix |

---

## Queues

**Convention:** `kairo.{product}.{module}.{purpose}`

**Casing:** lowercase with dot separation

Queue names identify the product, module, and purpose of the queue.

| Example | Purpose |
|---------|---------|
| kairo.commerce.orders.processing | Order processing queue |
| kairo.commerce.inventory.adjustments | Inventory adjustment queue |
| kairo.payments.transactions.settlement | Payment settlement queue |
| kairo.commerce.fulfillment.shipments | Shipment processing queue |

| Example | Incorrect | Reason |
|---------|-----------|--------|
| order-queue | Missing product and module context |
| kairo.commerce.queue1 | Non-descriptive |
| KAIRO.COMMERCE.ORDERS | Wrong casing |

---

## Topics

**Convention:** `kairo.{product}.{module}.events`

**Casing:** lowercase with dot separation

Topics aggregate events by product and module. Individual event types are distinguished by the event name within the message, not by separate topics.

| Example | Content |
|---------|---------|
| kairo.commerce.orders.events | All order-related events |
| kairo.commerce.catalog.events | All catalog-related events |
| kairo.commerce.inventory.events | All inventory-related events |
| kairo.payments.transactions.events | All payment transaction events |
| kairo.identity.authentication.events | All authentication events |

| Example | Incorrect | Reason |
|---------|-----------|--------|
| kairo.commerce.orders.order-created | Too granular; one topic per event type |
| orders | Missing hierarchy |
| kairo.commerce.events | Too broad; should scope to module |

---

## Future Products

When a new product is added to the ecosystem, all naming conventions apply from the first commit:

1. Product name follows the `Kairo {ProductName}` convention.
2. Repository is created as `kairo-{product}`.
3. Namespaces begin with `Kairo.{Product}`.
4. Events use `{Product}.{Module}.{Entity}.{Action}`.
5. APIs use `/api/{version}/{product}/{module}/{resource}`.
6. Databases use `kairo_{product}_{module}`.
7. Queues and topics follow their respective conventions.

No exceptions are granted for new products. Consistency at creation is dramatically cheaper than renaming after adoption.

---

## Enforcement

- Naming conventions are reviewed during code review and architecture review.
- New modules, services, and events must reference this document in their design proposals.
- Violations discovered after deployment are treated as technical debt and scheduled for correction.
- Exceptions require an ADR documenting the deviation and its justification.
