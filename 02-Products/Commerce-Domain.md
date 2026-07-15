# Commerce Domain

## Metadata

| Field | Value |
|-------|-------|
| Title | Kairo Commerce Business Domain |
| Document ID | KAI-PROD-006 |
| Status | Draft |
| Version | 0.1 |
| Target Release | TODO |
| Owner | Chief Product Architect |
| Created | 2026-07-15 |
| Last Updated | 2026-07-15 |
| Reviewers | TODO |
| Related Documents | [Kairo Commerce](./Kairo-Commerce.md), [Product Boundaries](./Product-Boundaries.md), [Domain Model](./Domain-Model.md) |
| Dependencies | [Kairo Platform](./Kairo-Platform.md) |

---

## Purpose

This document defines the Commerce domain in detail. It establishes the business responsibilities, capabilities, boundaries, dependencies, and terminology that all future Commerce modules, architecture documents, and development work must align with.

This is the authoritative reference for what Commerce is, what it owns, and where it ends.

---

## Commerce Responsibilities

Kairo Commerce is responsible for the complete lifecycle of selling goods and services through digital channels. Specifically:

- **Defining what can be sold** — Managing the catalog of products, their variants, attributes, and relationships.
- **Determining what things cost** — Resolving prices based on price lists, customer context, channel, currency, and promotional rules.
- **Tracking what is available** — Maintaining inventory levels across locations, handling reservations, and recording stock movements.
- **Managing the purchase process** — Supporting cart creation, checkout calculation, and order placement.
- **Processing orders** — Managing the order lifecycle from creation through fulfillment, including cancellations and returns.
- **Orchestrating fulfillment** — Coordinating shipment creation, tracking, and delivery status.
- **Applying promotional incentives** — Evaluating discount rules, coupon codes, and campaign conditions.
- **Calculating tax obligations** — Determining applicable taxes based on jurisdiction, product type, and customer status.
- **Managing commerce customers** — Maintaining customer profiles, addresses, and group memberships within the commerce context.
- **Supporting multi-channel operations** — Enabling distinct catalogs, pricing, inventory, and promotions per sales channel.

---

## Commerce Business Capabilities

### Catalog Management

The ability to define, organize, and maintain the full range of sellable items.

- Product creation and lifecycle management
- Variant management with attribute combinations
- Category hierarchies and product categorization
- Product attributes and custom attribute definitions
- Product type templates that define attribute structure
- Product relationships (related, cross-sell, up-sell, bundled)
- Media references for product images and assets
- Channel-scoped catalog visibility

### Pricing

The ability to determine the correct price for any item in any context.

- Price list management with currency support
- Multiple price lists per channel, customer group, or context
- Tiered and volume-based pricing
- Customer-specific and group-specific pricing
- Scheduled price changes with effective date ranges
- Price overrides for exceptional cases
- Currency conversion and multi-currency support

### Inventory Management

The ability to know what is available and where.

- Stock level tracking per product variant per location
- Stock reservations during the purchase process
- Stock movements and adjustment recording
- Multi-location inventory with location-scoped availability
- Reorder threshold awareness
- Inventory reconciliation support
- Channel-scoped inventory visibility

### Cart

The ability to manage a pre-order shopping state with accurate calculations.

- Server-side cart creation and persistence
- Item addition, removal, and quantity management
- Real-time price resolution from active price lists
- Promotional rule evaluation and discount application
- Tax calculation based on shipping destination
- Shipping method selection and rate calculation
- Cart expiration and cleanup
- Cart-to-order conversion at checkout

### Order Management

The ability to record, track, and manage purchase transactions through their lifecycle.

- Order creation from completed carts or direct API calls
- Order status lifecycle with defined transitions
- Line item management with resolved prices, taxes, and discounts
- Order total calculation and verification
- Payment initiation and payment status tracking
- Partial and full cancellation
- Return request creation and return authorization
- Refund initiation
- Order history and audit trail
- Order notes and internal annotations

### Promotions and Discounts

The ability to create incentives that influence purchasing behavior.

- Rule-based discount engine
- Percentage and fixed-amount discounts
- Coupon code management with usage limits
- Automatic promotions based on cart conditions
- Buy-X-get-Y and bundle discount rules
- Stacking rules that control how discounts combine
- Eligibility conditions based on customer, channel, product, or cart value
- Campaign scheduling with start and end dates

### Tax Calculation

The ability to determine and apply correct tax to transactions.

- Tax zone definitions based on geographic regions
- Tax rate management per zone and product category
- Tax category assignment to products
- Tax-inclusive and tax-exclusive pricing support
- Tax exemption handling
- Integration points for external tax calculation services
- Tax summary on carts and orders

### Fulfillment Orchestration

The ability to coordinate the delivery of ordered goods.

- Shipping method definitions and configuration
- Shipping rate calculation or external rate lookup
- Shipment creation against order line items
- Partial fulfillment support
- Tracking reference storage and status updates
- Fulfillment location selection based on inventory
- Integration points for external shipping carriers

### Customer Management

The ability to represent and manage the people and businesses that purchase.

- Customer profile creation and maintenance
- Multiple shipping and billing addresses per customer
- Customer group assignment for pricing and promotion eligibility
- Customer notes and internal metadata
- Guest checkout support without requiring a full customer profile
- Customer search and lookup

### Channel Management

The ability to operate distinct sales contexts from a single platform.

- Channel creation and configuration
- Channel-scoped catalog, pricing, inventory, and promotions
- Channel-specific settings and defaults
- Cross-channel order visibility
- Independent channel operations without mutual interference

---

## Commerce Boundaries

### Inside Commerce

| Concept | Reason |
|---------|--------|
| Product catalog and variants | Core to defining what is sold |
| Price resolution and price lists | Core to determining cost |
| Stock levels and reservations | Core to knowing availability |
| Carts and checkout calculation | Core to the purchase process |
| Orders and order lifecycle | Core to recording transactions |
| Promotions and coupon codes | Core to commerce incentives |
| Tax calculation | Inseparable from pricing and checkout |
| Shipping methods and fulfillment orchestration | Core to completing the sale |
| Commerce customer profiles | Commerce-specific customer data |
| Sales channels | Commerce-specific context scoping |

### Outside Commerce

| Concept | Owner | Reason |
|---------|-------|--------|
| User login and authentication | Identity | Not commerce-specific |
| User roles and permissions | Identity | Cross-product concern |
| Payment authorization and capture | Payments | Separate financial domain |
| Accounting and journal entries | ERP | Financial record-keeping |
| Purchase orders and procurement | ERP | Acquisition, not selling |
| In-store register operations | POS | Physical retail domain |
| Content pages and editorial | External CMS | Not a commerce concern |
| Email templates and delivery | Platform Notifications | Cross-product infrastructure |
| Demand forecasting | AI | Analytical, not operational |
| Search ranking algorithms | AI | Intelligence layer, not commerce logic |
| Audit trail storage | Platform Audit | Cross-product infrastructure |
| Webhook delivery mechanics | Platform Integrations | Cross-product infrastructure |

---

## Commerce Dependencies

### Depends On

| Dependency | What Commerce Needs |
|-----------|-------------------|
| Identity | Authentication of API consumers, authorization of operations, customer credential management |
| Organizations | Tenant context for all commerce data |
| Platform Event Infrastructure | Publishing commerce events for consumption by other products and external systems |
| Platform API Gateway | Routing and rate limiting for commerce API endpoints |
| Platform Integrations | Framework for connecting to external tax services, shipping carriers, and other third-party systems |

### Depended On By

| Consumer | What They Need From Commerce |
|---------|----------------------------|
| Payments | Order totals and payment initiation requests |
| ERP | Order completion events for accounting entries |
| POS | Catalog, pricing, and inventory data for in-store sales |
| AI | Historical order, pricing, and inventory data for analysis |
| Reporting | Commerce data for business metrics and dashboards |

---

## Commerce Terminology

| Term | Definition |
|------|-----------|
| Product | A sellable item defined in the catalog. May have variants. |
| Variant | A specific version of a product defined by a combination of attributes (e.g., size, color). |
| SKU | Stock Keeping Unit. A unique identifier for a specific variant at the inventory level. |
| Attribute | A named property of a product or variant (e.g., color, weight, material). |
| Category | A hierarchical grouping used to organize products in the catalog. |
| Product Type | A template that defines which attributes a product of that type has. |
| Price List | A named collection of prices, typically scoped by currency, channel, or customer group. |
| Cart | A temporary, server-side collection of items representing a customer's intent to purchase. |
| Order | A confirmed purchase transaction with resolved prices, taxes, and customer information. |
| Line Item | A single entry in a cart or order, referencing a product variant, quantity, and resolved price. |
| Promotion | A rule that modifies pricing under specific conditions. |
| Coupon | A code that activates a specific promotion when applied. |
| Tax Zone | A geographic region with a defined set of tax rates. |
| Tax Category | A classification applied to products that determines which tax rates apply. |
| Fulfillment | The process of delivering ordered goods to the customer. |
| Shipment | A physical package created to fulfill part or all of an order. |
| Channel | A distinct sales context with its own catalog, pricing, and inventory configuration. |
| Reservation | A temporary hold on inventory while a purchase is in progress. |
| Stock Movement | A recorded change in inventory quantity at a specific location. |
| Customer Group | A classification that determines pricing and promotion eligibility. |

---

## Future Growth

Commerce will deepen within its defined boundaries over time:

- **Subscription commerce** — Recurring orders, billing cycles, and subscription lifecycle management. This extends the order domain into recurring patterns.
- **B2B commerce** — Quote management, approval workflows, negotiated pricing, and company account hierarchies. This extends the customer and pricing domains.
- **Marketplace capabilities** — Multi-vendor catalog, vendor-scoped orders, and split fulfillment. This extends catalog and order management.
- **Advanced promotions** — Loyalty programs, points-based rewards, and tiered membership benefits. This extends the promotions domain.
- **Returns and exchanges** — Formalized return merchandise authorization, exchange workflows, and restocking. This deepens order lifecycle management.

Growth follows these rules:

- New capabilities must fit within Commerce's defined boundaries.
- Capabilities that belong to other domains become candidates for other products, not Commerce extensions.
- Existing APIs and behaviors are not broken when new capabilities are added.
- Each new capability is documented and approved before development begins.

---

## Out of Scope

Commerce explicitly does not handle and will not grow into:

- **Payment processing** — Commerce initiates payments. It does not process them, store payment credentials, or communicate with payment networks.
- **Accounting** — Commerce records what was sold. It does not maintain ledgers, generate financial statements, or manage fiscal periods.
- **Content management** — Commerce stores product data. Editorial content, landing pages, and media management belong to external systems.
- **User identity** — Commerce references customers. It does not manage authentication, passwords, or login flows.
- **Physical retail operations** — Commerce handles digital sales channels. Register sessions, cash drawers, and in-store hardware belong to POS.
- **Logistics optimization** — Commerce orchestrates fulfillment. Route planning, warehouse management, and carrier negotiations are outside scope.
- **Marketing automation** — Commerce provides purchase data. Campaign management, audience segmentation, and email marketing are outside scope.
- **Analytics and intelligence** — Commerce emits data. Deriving insights, building models, and generating predictions belong to AI or external analytics systems.
