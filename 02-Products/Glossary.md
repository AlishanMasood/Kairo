# Glossary

## Metadata

| Field | Value |
|-------|-------|
| Title | Kairo Business Glossary |
| Document ID | KAI-PROD-009 |
| Status | Draft |
| Version | 0.1 |
| Target Release | N/A |
| Owner | Chief Product Architect |
| Created | 2026-07-15 |
| Last Updated | 2026-07-15 |
| Reviewers | TODO |
| Related Documents | [Domain Model](./Domain-Model.md), [Commerce Domain](./Commerce-Domain.md), [Naming Conventions](./Naming-Conventions.md) |
| Dependencies | None |

---

## Purpose

This is the official vocabulary for the Kairo platform. Every document, API, event, and conversation within the ecosystem must use these terms with the definitions provided here.

Ambiguous terminology leads to ambiguous design. When two people use the same word to mean different things, the system they build will reflect that confusion. This glossary eliminates that risk.

If a term is not listed here and is needed, it should be added through the standard document change process before being used in approved documents.

---

## Platform Terms

### API

Application Programming Interface. The programmatic interface through which developers interact with Kairo products. All Kairo capabilities are delivered API-first.

### Audit

A tamper-evident record of significant actions performed within the platform. Audit entries capture who did what, when, and within which context. Audit is a platform concern that spans all products.

### Channel

A distinct sales context within Commerce. Each channel has its own catalog visibility, pricing, inventory, and promotions. Examples: online store, mobile app, wholesale portal.

### Event

A record of something that happened within the platform. Events are published by products when significant state changes occur and consumed by other products or external systems. Events are named in past tense and follow the naming conventions.

### Extension

A mechanism for developers to add custom behavior to the platform without modifying its core. Extensions operate through defined extension points such as webhooks, custom fields, and workflow hooks.

### Integration

A connection between the Kairo platform and an external system. Integrations are configured per organization and enable data flow between Kairo and third-party services such as payment providers, shipping carriers, and tax services.

### Multi-Tenancy

The platform's ability to serve multiple independent organizations from a single deployment while maintaining strict data isolation between them.

### Namespace

A hierarchical identifier that organizes code, packages, and services within the platform. Namespaces follow the `Kairo.{Product}.{Module}` convention.

### Platform

The shared infrastructure and services that all Kairo products are built upon. The platform provides identity, multi-tenancy, event infrastructure, API gateway, audit, and extensibility.

### Product

A self-contained, independently adoptable application within the Kairo ecosystem that owns a specific business domain. Examples: Kairo Commerce, Kairo Payments, Kairo Identity.

### Module

A bounded context within a product that owns a specific subdomain. Modules have explicit boundaries, own their data, and communicate with other modules through defined interfaces. Examples: Catalog, Pricing, Orders.

### SDK

Software Development Kit. A client library provided by Kairo for a specific programming language that simplifies interaction with Kairo APIs.

### Tenant

See Organization. In platform context, a tenant is the unit of data isolation.

### Webhook

An HTTP callback configured by a developer to receive notifications when specific events occur within the platform. Webhooks are the primary mechanism for external systems to react to Kairo events.

---

## Identity Terms

### Authentication

The process of verifying that a user or system is who they claim to be. Handled by the platform's identity layer.

### Authorization

The process of determining whether an authenticated user or system is permitted to perform a specific action. Governed by roles and permissions.

### Permission

A specific action that a user or role is allowed to perform. Permissions are scoped per product and per organization.

### Role

A named collection of permissions assigned to users. Roles simplify access management by grouping related permissions.

### Session

A time-bound authenticated state for a user. Sessions are created upon successful authentication and expire according to security policies.

### Token

A credential issued after authentication that represents a user's identity and permissions. Tokens are used to authorize API requests.

### User

An authenticated individual who interacts with the Kairo platform. Users may be staff members, administrators, or developers. Users are distinct from customers.

---

## Organization Terms

### Organization

The top-level business entity operating on the Kairo platform. All data, users, and configurations are scoped to an organization. An organization represents a single business or tenant.

### Store

A commercial operation within an organization. A store represents a business that sells goods or services. An organization may operate multiple stores.

### Branch

A physical location associated with a store. Branches represent brick-and-mortar sites where in-store operations occur. A store may have multiple branches.

### Warehouse

A physical location where inventory is stored. Warehouses are used for fulfillment and inventory management. A warehouse may serve one or more stores.

---

## Catalog Terms

### Attribute

A named property that describes a characteristic of a product or variant. Attributes may be used for display, filtering, or variant differentiation. Examples: color, size, weight, material.

### Category

A hierarchical grouping used to organize products in the catalog. Categories can be nested to form a tree structure.

### Product

A sellable item defined in the catalog. A product represents a conceptual item that may exist in multiple variants. A product without variants is itself the purchasable unit.

### Product Type

A template that defines which attributes a product of that type possesses. Product types ensure structural consistency across similar products.

### Variant

A specific, purchasable version of a product defined by a unique combination of attributes. A t-shirt in size Large and color Blue is a variant. Each variant has its own SKU, price, and inventory.

### SKU

Stock Keeping Unit. A unique identifier for a specific variant at the inventory level. SKUs are used for inventory tracking, order line items, and fulfillment.

---

## Pricing Terms

### Currency

A monetary unit in which prices are expressed and transactions are conducted. Kairo supports multi-currency operations.

### Discount

A reduction in price applied through a promotion or coupon. Discounts may be percentage-based, fixed-amount, or conditional.

### Price List

A named collection of prices for products and variants. Price lists are scoped by currency and may be associated with specific channels, customer groups, or contexts.

### Promotion

A rule that modifies pricing under specific conditions to incentivize purchases. Promotions may apply automatically or require a coupon code.

### Coupon

A code that a customer enters to activate a specific promotion. Coupons may have usage limits, expiration dates, and eligibility conditions.

---

## Inventory Terms

### Inventory

The total quantity of a specific variant available across all locations. Inventory is tracked per variant per location.

### Reservation

A temporary hold placed on inventory while a purchase is in progress. Reservations prevent overselling during the checkout process. They are released if the purchase is not completed.

### Stock Level

The quantity of a specific variant available at a specific location at a given point in time.

### Stock Movement

A recorded change in inventory quantity at a specific location. Stock movements capture the reason for change (sale, return, adjustment, transfer, receipt).

---

## Order Terms

### Cart

A temporary, server-side collection of items representing a customer's intent to purchase. Carts include real-time price calculations, promotional discounts, and tax. A cart becomes an order upon checkout.

### Checkout

The process of converting a cart into a confirmed order. Checkout finalizes pricing, applies tax, validates inventory, and initiates payment.

### Line Item

A single entry in a cart or order referencing a specific variant, its quantity, and the resolved price. An order typically contains one or more line items.

### Order

A confirmed purchase transaction. An order records what was bought, by whom, at what price, with what tax, and through which channel. Orders have a defined lifecycle from creation through fulfillment.

### Return

A process initiated when a customer sends back purchased goods. Returns may result in refunds, exchanges, or store credit. Returns affect inventory and financial records.

---

## Customer Terms

### Customer

A person or business that purchases goods or services through the Kairo platform. Customers have commerce-specific profiles (addresses, order history, group membership) managed by Commerce. Their identity credentials are managed by Identity.

### Customer Group

A classification assigned to customers that determines eligibility for specific pricing, promotions, or policies. Examples: wholesale, VIP, employee.

### Guest

A purchaser who completes a transaction without creating a full customer profile. Guest checkout allows purchases without registration.

---

## Payment Terms

### Capture

The process of collecting funds that were previously authorized. Capture transfers the authorized amount from the customer's payment method.

### Authorization

A temporary hold placed on funds by a payment provider, confirming that the customer's payment method can cover the transaction amount. Authorization does not transfer funds.

### Payment

A financial transaction in which funds are transferred from a customer to the merchant in exchange for goods or services.

### Refund

The return of funds to a customer, typically as a result of a return, cancellation, or dispute. Refunds reverse all or part of a previous payment.

### Settlement

The process by which authorized and captured funds are transferred to the merchant's account by the payment provider.

---

## Fulfillment Terms

### Fulfillment

The process of delivering ordered goods to the customer. Fulfillment includes picking, packing, shipping, and delivery confirmation.

### Shipment

A physical package created to fulfill part or all of an order. An order may be fulfilled through multiple shipments.

### Shipping Method

A delivery option available to customers during checkout. Shipping methods define carrier, speed, and cost.

### Tracking

A reference provided by a shipping carrier that allows the customer and the platform to monitor delivery progress.

---

## Subscription Terms

### Subscription

A recurring purchase agreement where a customer receives goods or services on a scheduled basis. Subscriptions generate recurring orders and payments.

### Billing Cycle

The recurring interval at which a subscription is charged. Examples: monthly, quarterly, annually.

### Renewal

The automatic continuation of a subscription at the end of a billing cycle, generating a new charge and order.

---

## ERP Terms

### Journal Entry

A record of a financial transaction in the accounting ledger. Journal entries debit and credit specific accounts.

### Ledger

A collection of accounts that records all financial transactions for an organization.

### Purchase Order

A formal request to a supplier to provide goods or services at agreed terms. Purchase orders are managed by Procurement.

### Supplier

A business that provides goods or services to the organization. Suppliers are managed within the Procurement domain.

---

## Usage Rules

- Use these terms exactly as defined. Do not create synonyms or alternative names for the same concept.
- If a document uses a term differently from this glossary, the document is incorrect and must be updated.
- When introducing a new concept that requires a term, propose an addition to this glossary before using the term in other documents.
- API field names, event names, and documentation must align with this glossary.
- If a term has different meanings in different contexts, the glossary must define each meaning with its context clearly stated.
