---
name: hexagonal-architecture
description: Guarantee that application architecture follows the hexagonal-architecture pattern
metadata:
  version: 1.0.0
  auto-invoke:
    - Designing architecture
    - Creating ports and adapters
    - Implementing use cases
    - Refactoring architecture
---

# Critical rules

- ALWAYS keep domain and use-case logic framework-agnostic
- ALWAYS define ports (interfaces/contracts) in the application or domain layer
- ALWAYS implement adapters outside the domain and use-case layers
- NEVER allow infrastructure details to leak into the domain or use-case layers
- ALWAYS direct dependencies inwards (domain <= application <= adapters)
- ALWAYS validate architectural boundaries when modifying code

---

# Core Concepts

## Layers and Responsibilities

- **Domain**: Entities, Value Objects, domain services, domain rules
- **Application (Use Cases)**: Orchestrates domain logic, defines ports
- **Adapters**: Implement ports for external systems (DB, HTTP, messaging, files)
- **Infrastructure**: Technical details used by adapters (frameworks, libraries, drivers)

## Ports and Adapters

- **Ports** are interfaces that describe what the application needs (outbound) or exposes (inbound)
- **Adapters** implement ports for concrete technologies
- **Inbound adapters** call use cases (HTTP, CLI, jobs)
- **Outbound adapters** fulfil ports (repositories, gateways, external services)

---

# Workflow

1. **Identify use cases** and name them by business intent
2. **Define domain model** with rules enforced by entities and Value Objects
3. **Create ports** for external interactions required by use cases
4. **Implement use cases** that depend only on ports and domain model
5. **Implement adapters** to satisfy ports for specific technologies
6. **Wire dependencies** in the composition root only
7. **Verify boundaries** with dependency checks and reviews

---

# Decision Trees

## Where does this class belong?

```
Is it a domain rule or invariant?
├─ YES → Domain
└─ NO → Is it a use-case orchestration?
          ├─ YES → Application
          └─ NO → Does it connect to an external system?
                    ├─ YES → Adapter
                    └─ NO → Infrastructure helper
```

## When to create a port?

```
Does a use case need to reach outside the application boundary?
├─ YES → Define a port in the application or domain layer
└─ NO → Keep logic inside the use case or domain model
```

---

# Examples

## Good: Use Case depends on a port

```
UseCase depends on UserRepositoryPort (interface)
Adapter implements UserRepositoryPort using a database client
Domain entity is used only in Domain and Use Case
```

## Bad: Use Case depends on infrastructure

```
UseCase imports a database client or framework types directly
Domain entity annotated with framework annotations
```

---

# Common Pitfalls

- **Framework leakage** — Domain or use case imports infrastructure types
- **Missing ports** — Use cases call adapters directly
- **Wrong dependency direction** — Adapters depend on domain, not the other way around
- **Anemic domain** — Domain holds data only, rules live in use cases or adapters
- **Overly generic ports** — Ports that mirror framework APIs rather than domain intent

---

# Checklist

Before submitting code, verify:

- [ ] Domain model contains business rules and invariants
- [ ] Use cases depend only on ports and domain model
- [ ] Ports are interfaces/contracts defined inside the application or domain layer
- [ ] Adapters implement ports and live outside the core layers
- [ ] Infrastructure details are confined to adapters and composition root
- [ ] Dependency direction is strictly inwards
- [ ] No framework annotations or types appear in domain or use-case code

---

# Related Skills

- `code-quality` — For Clean Code, SOLID and Object Calisthenics
- `java-language` — For Java-specific language guidance

---
