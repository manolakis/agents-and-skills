---
name: adr-compliance
description: Ensure work aligns with and updates Architecture Decision Records (ADRs)
metadata:
  version: 1.0.0
  auto-invoke:
    - Making architectural changes
    - Introducing new dependencies
    - Changing system boundaries
    - Modifying cross-cutting concerns
---

# Critical rules

- ALWAYS check existing ADRs before proposing architectural changes
- NEVER contradict an ADR without either updating it or writing a superseding ADR
- ALWAYS document architectural decisions that impact structure, boundaries or dependencies
- ALWAYS keep ADRs concise, factual and written in British English
- NEVER edit historical ADR content; add a new ADR to supersede it

---

# Scope

ADRs are required when:
- Introducing a new architectural pattern or framework
- Changing boundaries between domain, application, and adapters
- Adding or replacing infrastructure dependencies
- Making decisions that affect runtime characteristics (performance, security, resilience)
- Establishing cross-cutting concerns (logging, observability, error handling)

ADRs are not required for:
- Pure refactoring with no architectural impact
- Test-only changes
- Cosmetic or formatting changes

---

# Workflow

1. **Locate ADRs** by checking `.adr-dir` file if it exists, otherwise use `docs/adr` by default
2. **Review relevant decisions** from the ADR directory
3. **Assess impact** of the proposed change against existing ADRs
4. **Choose action**:
   - Follow existing ADR (no new ADR)
   - Supersede with a new ADR if the decision changes
   - Suggest the creation of a new ADR if no decision exists
4. **Draft ADR** using the standard format
5. **Reference the ADR** in the change description or PR summary

---

# ADR Format

Use a simple, consistent structure:

```
# Title

## Status
Accepted | Proposed | Superseded | Deprecated

## Context
What problem are we solving and why now?

## Decision
What is the decision?

## Consequences
What are the trade-offs and impacts?
```

---

# Decision Tree

```
Does the change affect architecture or system boundaries?
├─ YES → Is there an ADR covering this?
│         ├─ YES → Follow it
│         └─ NO → Create a new ADR
└─ NO → No ADR needed
```

---

# Common Pitfalls

- **Silent divergence** — Changing architecture without updating ADRs
- **Overwriting history** — Editing old ADRs instead of superseding them
- **Vague consequences** — Not stating trade-offs or impacts clearly
- **Mixed scope** — Combining multiple unrelated decisions in one ADR

---

# Checklist

Before submitting changes, verify:

- [ ] Relevant ADRs were reviewed
- [ ] New architectural decisions are captured in a new ADR
- [ ] Superseded decisions have a new ADR referencing the prior decision
- [ ] ADR is concise and written in British English
- [ ] ADR status is appropriate and consistent

---

# Related Skills

- `hexagonal-architecture` — For architectural boundaries and ports/adapters
- `code-quality` — For Clean Code, SOLID and Object Calisthenics

---
