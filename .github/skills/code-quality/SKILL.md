---
name: code-quality
description: Ensure production-grade code through systematic quality review across any language
metadata:
  version: 1.0.0
  auto-invoke:
    - Writing code
    - Reviewing code
    - Refactoring code
    - Implementing new features
---

# Critical rules

- ALWAYS use British English in code, comments and documentation
- ALWAYS follow Clean Code, SOLID and Object Calisthenics principles
- NEVER modify build files (e.g., `pom.xml`, `build.gradle`, `package.json`, `Cargo.toml`) unless explicitly requested
- ALWAYS write self-documenting code; add comments only when necessary
- ALWAYS validate domain constraints in Value Objects
- NEVER expose mutable state through public methods
- ALWAYS prefer composition over inheritance

**Note:** These principles are language-agnostic and apply to all programming languages.

---

# Workflow

1. **Review naming** — Ensure all identifiers reveal intent without abbreviations
2. **Check method/function size** — Methods should be < 20 lines, ideally < 10
3. **Verify single responsibility** — Each class/module/function does one thing
4. **Assess nesting** — Maximum one level of indentation per method/function (extract logic if needed)
5. **Eliminate conditional branches** — Use early returns, guard clauses, and polymorphism instead of `else`
6. **Wrap primitives** — Domain concepts must use Value Objects
7. **Review dependencies** — Ensure dependency inversion (depend on abstractions)
8. **Check immutability** — Prefer immutable objects and limit mutability
9. **Validate tests** — Ensure behaviour is covered with clear test names

---

# Code Quality Standards

## Clean Code Principles
- **Single purpose:** Methods should do one thing and do it well
- **Short methods:** Keep methods under 20 lines (ideally < 10)
- **Descriptive names:** Use names that reveal intent without comments
- **No side effects:** Functions should not modify state unexpectedly
- **Immutability:** Prefer immutable objects; use immutability features of your language
- **Low coupling:** Minimise dependencies between modules
- **High cohesion:** Keep related functionality together

## Code Readability
- **Use whitespace strategically:** Separate logical blocks with blank lines to improve readability
- **Avoid dense code blocks:** Large sections of code without separation are hard to follow
- **One blank line between logical sections:** Between variable declaration and first operation, between setup and action, between action and assertion
- **Group related statements:** Keep related operations together, separate different concerns

### Examples

✅ **Good: Clear separation of logical blocks**
```
var user = new User("john@example.com");
var email = user.getEmail();

var result = validateEmail(email);  // Clear separation from setup

assert(result);  // Separated from action
assertEquals("john@example.com", user.getEmail());
```

❌ **Bad: Dense code without breathing room**
```
var user = new User("john@example.com");
var email = user.getEmail();
var result = validateEmail(email);
assert(result);
assertEquals("john@example.com", user.getEmail());
```

## SOLID Principles
- **Single Responsibility:** One class, one reason to change
- **Open/Closed:** Open for extension, closed for modification
- **Liskov Substitution:** Subtypes must be substitutable for base types
- **Interface Segregation:** Small, focused interfaces (prefer roles over wide contracts)
- **Dependency Inversion:** Depend on abstractions, not concretions

## Object Calisthenics
1. **One level of indentation per method** — Extract nested logic into separate functions/methods
2. **Avoid conditional chains** — Prefer guard clauses and polymorphism over `else`
3. **Wrap all primitives** — Domain concepts deserve Value Objects (e.g., `EmailAddress`, `Money`)
4. **First-class collections** — If a class has a collection, it should have no other instance variables
5. **One dot per line** — Avoid method/property chains (Law of Demeter)
6. **Don't abbreviate names** — `user` not `usr`, `calculate` not `calc`
7. **Keep entities small** — Typical classes <= 50 lines, typical methods <= 10 lines
8. **Limit instance variables** — Prefer fewer instance variables to promote cohesion
9. **Encapsulation** — Tell objects what to do instead of asking for state and manipulating it

---

# Decision Trees

## When to Create a Value Object

```
Does the primitive have domain meaning?
├─ YES → Does it have validation rules?
│         ├─ YES → Create Value Object with validation
│         └─ NO → Consider if it will in future; if likely, create Value Object
└─ NO → Keep as primitive (e.g., loop counters, temporary variables)
```

## When to Extract a Method/Function

```
Is the method/function > 10 lines?
├─ YES → Extract cohesive blocks into separate methods
└─ NO → Does it have nested conditionals?
          ├─ YES → Extract conditional logic into named methods
          └─ NO → Does it mix abstraction levels?
                    ├─ YES → Separate high-level orchestration from low-level details
                    └─ NO → Method is acceptable
```

## When to Use Inheritance vs Composition

```
Is there a true "is-a" relationship?
├─ YES → Can you guarantee Liskov Substitution?
│         ├─ YES → Inheritance may be appropriate
│         └─ NO → Use composition with interfaces
└─ NO → ALWAYS use composition
```

---

# Examples

## Good: Value Object with Validation

A Value Object encapsulates a domain concept:
- Validates the value during creation
- Immutable after creation
- Examples: `EmailAddress`, `Money`, `UserId`, `OrderNumber`
- The object ensures its own invariants are maintained

## Good: Small, Single-Purpose Method

A method or function that:
- Has one clear purpose
- Is under 10 lines long
- Has no side effects
- Has clear input/output contracts

## Good: Tell, Don't Ask

Instead of extracting state and manipulating it:
```
✓ Tell the object what to do: user.activate()
✗ Don't extract state and act on it externally
✗ Don't expose getters/setters for internal mutation
```

## Good: Early Return / Guard Clauses

Structure decision logic with guards at the start:
```
Guard clause: if (condition) return early;
Guard clause: if (condition) return early;
Guard clause: if (condition) return early;

// Happy path at normal indentation level
```

## Good: Dependency Inversion (Hexagonal Architecture)

```
✓ Define an interface/protocol for external dependencies
✓ Inject the abstraction into business logic
✓ Implement the abstraction in adapters for specific technologies
✓ Business logic does not depend on technology details
```

## Bad: Primitive Obsession

```
✗ Passing domain concepts as primitives (String, Integer, etc.)
✗ Scattering validation logic across multiple functions
✓ Wrap domain concepts in Value Objects
✓ Validation happens once during object creation
```

## Bad: Multiple Responsibilities

```
✗ One class/module handling: user creation, email, logging, reporting
✓ Split into separate classes/modules:
  - UserRegistrationService (handle user creation)
  - EmailService (send emails)
  - AuditLogger (log activity)
  - ReportGenerator (generate reports)
```

## Bad: Deep Nesting

Deep conditional nesting obscures intent:
```
✗ Multiple nested levels: if (x) { if (y) { if (z) { ... } } }
✓ Use guard clauses to exit early
✓ Keep happy path at original indentation
```

## Bad: Excessive Chaining (Train Wreck)

```
✗ Violates Law of Demeter: object.property().method().thing()
✗ Tight coupling to internal structure
✓ Ask the object directly for what you need
✓ Encapsulate navigation in methods
```

---

# Common Pitfalls

- **Ignoring immutability** — Objects should not change after creation unless essential
- **Over-using inheritance** — Inheritance creates tight coupling; prefer composition
- **Exposing internal state** — Using getters/setters breaks encapsulation
- **Writing God classes/modules** — Classes that know or do too much
- **Deep nesting** — More than one level of indentation obscures intent
- **Abbreviations** — `usr`, `mgr`, `calc` are unclear; spell out full words
- **Framework coupling** — Business logic depending on infrastructure/framework details
- **Mutable Value Objects** — Value Objects must be immutable
- **Ignoring null/option safety** — Use explicit Optional/Maybe types to represent nullable values
- **Comments that explain bad code** — Refactor the code instead of explaining it

---

# Checklist

Before submitting code, verify:

- [ ] All primitives with domain meaning are wrapped in Value Objects
- [ ] No method/function exceeds reasonable length (ideally < 10 lines)
- [ ] No class/module exceeds reasonable size (ideally <= 50 lines)
- [ ] Maximum one level of indentation per method/function
- [ ] No `else` keywords or equivalents (use early returns or polymorphism)
- [ ] All objects/variables are immutable unless mutability is essential
- [ ] No getters/setters exposing internal mutable state
- [ ] Domain logic is independent from infrastructure/framework details
- [ ] Tests cover all business rules with expressive names
- [ ] British English spelling throughout

---

# Related Skills

- `conventional-commits` — For commit message standards

---
