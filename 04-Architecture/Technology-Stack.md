# Technology Stack

## Metadata

| Field | Value |
|-------|-------|
| Title | Kairo Approved Technology Stack |
| Document ID | KAI-ARCH-004 |
| Status | Draft |
| Version | 0.1 |
| Target Release | V1 |
| Owner | Chief Software Architect |
| Created | 2026-07-15 |
| Last Updated | 2026-07-15 |
| Reviewers | TODO |
| Related Documents | [Architecture Overview](./Architecture-Overview.md), [Architecture Principles](./Architecture-Principles.md), [Technical Philosophy](../01-Foundation/Technical-Philosophy.md) |
| Dependencies | None |

---

## Purpose

This document defines the approved technologies for the Kairo platform. Every technology listed here has been evaluated against the platform's architectural principles and selected for specific, documented reasons.

Technologies not listed here are not approved for use. Introducing a new technology requires an ADR documenting the evaluation, the alternatives considered, and the architectural impact.

---

## Selection Criteria

All technology choices are evaluated against these criteria:

- **Alignment with architectural principles** — Does it support modularity, statelessness, observability, and cloud-native operation?
- **Long-term viability** — Is the technology actively maintained, broadly adopted, and likely to be supported for years?
- **Developer experience** — Does it improve or degrade the daily experience of engineers working on the platform?
- **Operational simplicity** — Can it be deployed, monitored, and maintained without specialized expertise?
- **Community and ecosystem** — Is there a healthy community, good documentation, and a mature ecosystem of tooling?
- **Vendor independence** — Does it avoid deep lock-in to a specific vendor or cloud provider?

---

## Backend

### .NET / C#

| Attribute | Detail |
|-----------|--------|
| Purpose | Primary language and runtime for all platform services |
| Selection Reason | Strong type system reduces runtime errors. Excellent performance characteristics for API workloads. Mature ecosystem for building enterprise-grade services. Cross-platform runtime. First-class support for dependency injection, middleware pipelines, and async patterns that align with the platform's architectural needs. |
| Alternatives Considered | **Node.js/TypeScript** — Strong ecosystem but weaker runtime performance for compute-heavy operations and less mature tooling for complex domain modeling. **Go** — Excellent performance but less expressive type system for complex business domain modeling. **Java/Kotlin** — Viable but heavier runtime footprint and slower startup times for cloud-native workloads. |
| Future Evolution | Track .NET release cadence. Adopt new language features and runtime improvements as they stabilize. Evaluate minimal APIs and AOT compilation for specific workloads where startup time is critical. |

### ASP.NET Core

| Attribute | Detail |
|-----------|--------|
| Purpose | Web framework for API endpoints, middleware, and request pipeline |
| Selection Reason | Native integration with the .NET ecosystem. High-performance request pipeline. Built-in support for dependency injection, configuration, logging, and health checks. Middleware architecture aligns with the platform's layered design. |
| Alternatives Considered | **FastEndpoints** — Lighter weight but smaller community and less mature ecosystem. **Carter** — Minimal API wrapper with less architectural structure. |
| Future Evolution | Adopt minimal API patterns where appropriate for lightweight endpoints. Evaluate gRPC integration for internal service communication in later versions. |

### Entity Framework Core

| Attribute | Detail |
|-----------|--------|
| Purpose | Object-relational mapping and data access |
| Selection Reason | Strongly-typed query construction reduces data access errors. Migration tooling supports schema evolution. Supports multiple database providers, reducing vendor lock-in. Integrates natively with ASP.NET Core's dependency injection. |
| Alternatives Considered | **Dapper** — Higher performance for read-heavy workloads but requires manual mapping and lacks migration tooling. May be used alongside EF Core for specific performance-critical queries. **Raw ADO.NET** — Maximum control but significant development overhead for standard operations. |
| Future Evolution | Evaluate compiled queries for hot paths. Consider Dapper for specific read-optimized query paths where EF Core overhead is measurable. |

### MediatR

| Attribute | Detail |
|-----------|--------|
| Purpose | In-process mediator for command/query separation and pipeline behaviors |
| Selection Reason | Enables clean separation of request handling from controller logic. Pipeline behaviors support cross-cutting concerns (validation, logging, authorization) without coupling them to handlers. Aligns with the modular architecture by keeping module logic independent of transport. |
| Alternatives Considered | **Direct service injection** — Simpler but couples controllers to business logic and lacks pipeline behavior support. **Custom mediator** — Avoids external dependency but replicates well-tested functionality. |
| Future Evolution | Evaluate whether pipeline complexity justifies the abstraction as the codebase matures. |

### FluentValidation

| Attribute | Detail |
|-----------|--------|
| Purpose | Request validation with strongly-typed rules |
| Selection Reason | Separates validation logic from business logic. Strongly-typed rules catch errors at compile time. Integrates with MediatR pipeline for automatic validation. Readable rule definitions that serve as documentation. |
| Alternatives Considered | **Data Annotations** — Simpler but limited expressiveness for complex validation rules. **Custom validation** — No external dependency but higher maintenance cost. |
| Future Evolution | Stable. No significant evolution expected. |

---

## Database

### PostgreSQL

| Attribute | Detail |
|-----------|--------|
| Purpose | Primary relational data store for all modules |
| Selection Reason | Robust, feature-rich relational database with strong ACID compliance. Excellent JSON support for semi-structured data. Proven at scale. Open source with no licensing cost. Available as a managed service on all major cloud providers, supporting vendor independence. Strong community and extensive tooling ecosystem. |
| Alternatives Considered | **SQL Server** — Capable but introduces Microsoft licensing dependency. **MySQL** — Less feature-rich for complex queries and JSON operations. **CockroachDB** — Distributed SQL but adds operational complexity premature for V1. |
| Future Evolution | Evaluate read replicas for scaling read-heavy workloads. Consider logical replication for cross-module data needs. Evaluate time-series extensions for audit and event data. |

---

## Caching

### Redis

| Attribute | Detail |
|-----------|--------|
| Purpose | Distributed caching, session storage, and rate limiting |
| Selection Reason | Industry-standard distributed cache with sub-millisecond latency. Supports multiple data structures (strings, hashes, sets, sorted sets) that align with different caching patterns. Built-in expiration and eviction policies. Available as a managed service on all major cloud providers. |
| Alternatives Considered | **Memcached** — Simpler but lacks data structure support and persistence options. **In-memory caching only** — Violates stateless service principle and does not work with horizontal scaling. |
| Future Evolution | Evaluate Redis Streams for lightweight event processing. Consider Redis clustering for high-availability caching as load increases. |

---

## Messaging

### RabbitMQ

| Attribute | Detail |
|-----------|--------|
| Purpose | Message broker for asynchronous event delivery between modules |
| Selection Reason | Mature, reliable message broker with strong delivery guarantees. Supports multiple exchange types (direct, topic, fanout) that map to different event routing patterns. Built-in dead-letter handling for failed message processing. Well-understood operationally with extensive monitoring tooling. Available as managed service on major cloud providers. |
| Alternatives Considered | **Apache Kafka** — Superior for high-throughput event streaming but adds operational complexity disproportionate to V1 requirements. May be evaluated for later versions. **Azure Service Bus / AWS SQS** — Capable but creates cloud vendor lock-in. **MassTransit abstraction** — Consider as a transport abstraction layer to ease future broker migration. |
| Future Evolution | Evaluate Kafka for event sourcing and high-throughput scenarios in later versions. Consider MassTransit as a transport abstraction to decouple from the specific broker. |

---

## Storage

### Object Storage (S3-compatible)

| Attribute | Detail |
|-----------|--------|
| Purpose | Binary asset storage for media, documents, and file uploads |
| Selection Reason | Scalable, durable object storage with industry-standard S3-compatible API. Available from all major cloud providers and self-hosted solutions (MinIO). Cost-effective for large volumes of binary data. CDN integration for global delivery. |
| Alternatives Considered | **Local filesystem** — Violates cloud-native and stateless principles. **Database BLOB storage** — Adds unnecessary load to the primary database. |
| Future Evolution | Integrate CDN for public asset delivery. Evaluate lifecycle policies for cost optimization. |

---

## CI/CD

### GitHub Actions

| Attribute | Detail |
|-----------|--------|
| Purpose | Continuous integration, testing, and deployment automation |
| Selection Reason | Native integration with GitHub repository hosting. YAML-based workflow definitions that are version-controlled alongside code. Extensive marketplace of pre-built actions. Supports matrix builds for multi-target testing. Sufficient for current team size and workflow complexity. |
| Alternatives Considered | **Azure DevOps** — More enterprise features but adds platform complexity. **GitLab CI** — Requires GitLab hosting migration. **Jenkins** — Self-hosted operational burden without proportional benefit. |
| Future Evolution | Evaluate self-hosted runners for performance-sensitive builds. Add deployment workflows as infrastructure matures. |

---

## Infrastructure

### Docker

| Attribute | Detail |
|-----------|--------|
| Purpose | Application containerization for consistent build and deployment |
| Selection Reason | Industry-standard containerization. Ensures consistency between development, testing, and production environments. Supports the cloud-native and stateless service principles. Foundation for orchestrated deployment. |
| Alternatives Considered | **Direct host deployment** — Inconsistent environments and manual dependency management. **Podman** — Compatible but smaller ecosystem. |
| Future Evolution | Optimize container images for size and startup time. Evaluate multi-stage builds for production artifacts. |

### Kubernetes

| Attribute | Detail |
|-----------|--------|
| Purpose | Container orchestration for deployment, scaling, and operations |
| Selection Reason | Industry-standard orchestration platform. Supports horizontal scaling, health-based routing, rolling deployments, and self-healing. Available as a managed service on all major cloud providers. Aligns with cloud-native principles. |
| Alternatives Considered | **Docker Compose** — Suitable for development but insufficient for production orchestration. **Cloud-specific container services (ECS, Cloud Run)** — Creates vendor lock-in. **Bare metal/VM deployment** — Manual scaling and operations overhead. |
| Future Evolution | Evaluate Helm charts for standardized deployment. Consider service mesh (Istio/Linkerd) when inter-service communication complexity justifies it. |

---

## Monitoring

### Prometheus + Grafana

| Attribute | Detail |
|-----------|--------|
| Purpose | Metrics collection, storage, alerting, and visualization |
| Selection Reason | Prometheus is the industry standard for cloud-native metrics. Pull-based collection model works well with Kubernetes. PromQL provides powerful query capabilities. Grafana provides flexible dashboarding with broad data source support. Both are open source with large communities. |
| Alternatives Considered | **Datadog** — Excellent but significant cost at scale and vendor lock-in. **CloudWatch/Azure Monitor** — Cloud-specific lock-in. **InfluxDB** — Capable time-series database but less ecosystem integration than Prometheus. |
| Future Evolution | Evaluate Thanos or Cortex for long-term metric storage. Add business-level dashboards as capabilities mature. |

---

## Logging

### Structured Logging (Serilog) + Centralized Aggregation

| Attribute | Detail |
|-----------|--------|
| Purpose | Structured log emission from application services and centralized log aggregation |
| Selection Reason | Serilog provides structured logging with rich context (request ID, tenant, module) that is essential for debugging in a multi-tenant, modular system. Structured logs can be parsed, filtered, and aggregated programmatically. Supports multiple output sinks for flexibility in aggregation tooling. |
| Alternatives Considered | **Microsoft.Extensions.Logging alone** — Less expressive structured logging. **NLog** — Capable but less community momentum for structured logging patterns. |
| Future Evolution | Evaluate OpenTelemetry for unified logging, metrics, and tracing. Select centralized aggregation platform (ELK, Loki, or managed service) based on operational requirements. |

### Distributed Tracing (OpenTelemetry)

| Attribute | Detail |
|-----------|--------|
| Purpose | End-to-end request tracing across modules and services |
| Selection Reason | Vendor-neutral standard for distributed tracing. Supports .NET natively. Enables request correlation from gateway through modules to event processing. Compatible with multiple tracing backends (Jaeger, Zipkin, managed services). |
| Alternatives Considered | **Application Insights** — Azure-specific lock-in. **Jaeger client directly** — Vendor-specific instrumentation. |
| Future Evolution | Adopt OpenTelemetry for metrics and logging as the standard matures, creating a unified telemetry pipeline. |

---

## Testing

### xUnit

| Attribute | Detail |
|-----------|--------|
| Purpose | Unit and integration test framework |
| Selection Reason | Modern .NET test framework with strong community adoption. Supports parallel test execution, dependency injection in tests, and clean fixture patterns. Integrates with CI/CD pipelines natively. |
| Alternatives Considered | **NUnit** — Capable but less idiomatic for modern .NET patterns. **MSTest** — Less community tooling and extension support. |
| Future Evolution | Stable. No significant evolution expected. |

### Testcontainers

| Attribute | Detail |
|-----------|--------|
| Purpose | Integration testing with real infrastructure dependencies |
| Selection Reason | Enables integration tests against real PostgreSQL, Redis, and RabbitMQ instances in disposable containers. Eliminates the gap between test and production behavior. Supports the principle of testing against real dependencies rather than mocks for integration tests. |
| Alternatives Considered | **In-memory test databases** — Behavior differences between in-memory and production databases create false confidence. **Shared test infrastructure** — State pollution between test runs. |
| Future Evolution | Stable. Expand container definitions as infrastructure dependencies grow. |

---

## Architecture Impact

| Layer | Impact |
|-------|--------|
| Backend | .NET/C# with ASP.NET Core provides the runtime, framework, and patterns for all modules. MediatR enforces command/query separation. FluentValidation ensures consistent input validation. |
| Data | PostgreSQL with EF Core provides relational data management. Redis provides caching and session storage. Both support the multi-tenant, modular architecture. |
| Communication | RabbitMQ enables event-driven communication between modules. Decouples producers from consumers. Supports at-least-once delivery. |
| Operations | Docker and Kubernetes provide the deployment and scaling foundation. Prometheus, Grafana, Serilog, and OpenTelemetry provide the observability stack. |
| Quality | xUnit and Testcontainers enable reliable testing against real dependencies. GitHub Actions automates the build, test, and deployment pipeline. |

---

## Version Gate

| Version | Technology Expectations |
|---------|------------------------|
| V1 | All listed technologies are in use. Stack is proven for single-product deployment. Observability covers core operations. |
| V2 | Stack supports increased load. Caching and messaging are optimized. Observability covers business metrics. CI/CD includes automated deployment. |
| V3 | Stack supports multi-product deployment. Evaluate Kafka for high-throughput events. Evaluate service mesh for inter-product communication. OpenTelemetry is unified across logs, metrics, and traces. |
| Future | Stack supports geographic distribution. Long-term storage solutions for metrics and logs are in place. Technology refresh cycle is established for evaluating replacements. |

---

## Out of Scope

This document does not cover:

- Installation or setup instructions — documented in development onboarding guides
- Configuration details — documented in deployment and operations guides
- Cloud provider selection — documented separately as an infrastructure decision
- Frontend technology choices — frontend is the developer's domain; the platform is headless
- Third-party SaaS integrations (payment providers, shipping carriers) — documented per integration

---

## Technology Governance

- All technologies listed here are approved for use. No approval process is needed to use them.
- Technologies not listed here require an ADR before adoption. The ADR must evaluate the technology against the selection criteria and document alternatives considered.
- Technology versions are tracked and upgraded on a regular cadence. Security patches are applied promptly.
- Technology removal requires an ADR documenting the migration plan and timeline.
- Experimentation with unapproved technologies is permitted in research branches. Production use requires formal approval.

---

## Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 0.1 | 2026-07-15 | Chief Software Architect | Initial draft |
