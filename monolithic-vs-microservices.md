# Monolithic vs Microservices Architecture

## Monolithic Architecture

A single, unified codebase where all components (UI, business logic, data access) are tightly coupled and deployed as one unit.

**Characteristics:**
- One codebase, one build, one deployment
- Shared database, shared memory space
- Components communicate via in-process function calls

**Pros:**
- Simple to develop, test, and deploy early on
- Easier debugging (single process, single stack trace)
- No network latency between components
- Simpler transactions (ACID within one DB)

**Cons:**
- Scaling means scaling the entire app, even if only one part is under load
- Large codebase becomes hard to reason about as team/product grows
- One bug can crash the whole system
- Tech stack is locked in — hard to adopt new languages/frameworks per component
- Slower CI/CD as the codebase grows (long build/test times)

## Microservices Architecture

The application is split into small, independently deployable services, each owning a specific business capability and often its own database.

**Characteristics:**
- Services communicate over the network (REST, gRPC, message queues)
- Each service can be built, deployed, and scaled independently
- Decentralized data management (database-per-service pattern)

**Pros:**
- Independent scaling — scale only the hot service
- Teams can own services end-to-end, deploy independently
- Fault isolation — one service failing doesn't necessarily crash others
- Freedom to pick different tech stacks per service

**Cons:**
- Distributed systems complexity: network failures, latency, partial failures
- Data consistency is hard — no single ACID transaction across services (need sagas, eventual consistency)
- Operational overhead: service discovery, load balancing, monitoring, distributed tracing, orchestration (Kubernetes etc.)
- Testing is harder (integration/contract testing across services)
- Debugging requires distributed tracing (e.g., Jaeger, X-Ray) since a single request spans multiple services

## Key Trade-off Axes

| Axis | Monolith | Microservices |
|---|---|---|
| Deployment | Single unit | Independent per service |
| Scaling | Whole app | Per service |
| Data | Shared DB | DB per service |
| Team structure | Works for small teams | Suits multiple autonomous teams |
| Failure mode | Whole app down | Isolated (if designed well) |
| Ops complexity | Low | High |

## When to Choose Which

- **Monolith**: early-stage products, small teams, unclear domain boundaries — you want speed and simplicity, and can always split out services later (this is often the recommended path — "monolith first").
- **Microservices**: mature product with well-understood domain boundaries, multiple teams needing independent deployment cycles, and components with very different scaling needs.

## Notes

Amazon Live and most large-scale Amazon systems operate in a microservices-heavy environment — service-to-service calls via internal RPC frameworks, per-service ownership, and operational overhead (on-call, distributed debugging) are common day-to-day realities.

Topics worth exploring further:
- Service communication patterns (sync vs async)
- Saga pattern for distributed transactions
- Service discovery and load balancing
- Distributed tracing tools (Jaeger, X-Ray)
- Connection to networking fundamentals (DNS, HTTP, load balancers)
