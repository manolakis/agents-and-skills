---
name: testing
description: >
  Language-agnostic testing principles, patterns, and practices. Defines universal testing concepts,
  test design strategies, and quality standards applicable to any programming language or framework.
  Language-specific testing patterns (frameworks, assertions, mocking libraries) should be defined
  in the respective language skills.
compatibility: >
  Works with any programming language, testing framework, or project structure. Language-specific
  implementations should reference and extend these principles in their respective language skills.
metadata:
  version: 1.0.0
---

# Critical Rules

These rules are **non-negotiable** and must be followed for all tests:

## 1. Test Coverage Requirements

- **Every public interface** in domain and application layers must have tests
- **Every use case handler** (command/query/operation) must have tests
- **Every adapter implementation** must have tests
- **Critical business logic** must have 100% branch coverage
- **Edge cases and error paths** must be explicitly tested

## 2. Test Naming

Tests must clearly communicate their intent using **Given-When-Then** format:

```
Given [initial context/state]
When [action/event occurs]
Then [expected outcome]
```

The test method name should be descriptive and reflect the scenario.

## 3. Test Structure (Given-When-Then)

Every test must follow the **Given-When-Then** pattern:

```
test_scenario_description() {
    // Given: Set up the initial context and test data
    [setup code]

    // When: Execute the action under test
    [action code]

    // Then: Verify the expected outcome
    [assertion code]
}
```

## 4. Test Independence (Critical)

- Tests must **NOT depend on execution order**
- Tests must **NOT share mutable state** between test cases
- Each test must **create its own test data locally**
- Each test must be **fully self-contained and runnable in isolation**
- Avoid class-level variables that are mutated across tests

**Why:** Shared state creates coupling between tests, making them fragile and hard to debug.

## 5. Test Isolation

- Tests should be **repeatable** (same result every time)
- Tests should be **deterministic** (no randomness unless explicitly tested)
- Tests should **not depend on external state** (databases, files, network)
- Use test doubles (mocks, stubs, fakes) for external dependencies

## 6. One Logical Assertion per Test

Each test should verify **one logical concept**, though it may have multiple assertion statements:

```
// Good: All assertions verify the same concept
test_user_creation_sets_all_required_fields() {
    // Given-When-Then...

    // Then: All fields are set correctly
    assert user.id is not null
    assert user.email equals "user@example.com"
    assert user.status equals ACTIVE
}

// Bad: Testing multiple unrelated concepts
test_user_operations() {
    // Tests creation, update, deletion all in one
}
```

---

## Workflow

Follow this process when writing tests:

### Step 1: Identify What to Test

1. **Read the code** to understand its behavior and contract
2. **Identify public interfaces** (these define the contract to test)
3. **Identify critical paths** (happy path + failure paths)
4. **Identify edge cases** (null, empty, boundary values, invalid inputs)

### Step 2: Plan Test Scenarios

For each operation, identify scenarios using Given-When-Then:

- **Happy path**: Given valid input, When operation executes, Then expected result
- **Edge cases**: Given boundary values, When operation executes, Then handled correctly
- **Error cases**: Given invalid input, When operation executes, Then appropriate error
- **State transitions**: Given current state, When event occurs, Then new state reached

### Step 3: Write Self-Contained Tests

Each test must:
- Create its own test data locally (no shared class variables)
- Set up required context and dependencies
- Execute the operation under test
- Verify the expected outcome
- Be independent of other tests

### Step 4: Verify Coverage

- Run coverage analysis
- Ensure critical paths are covered
- Add tests for uncovered branches
- Review for missed edge cases

### Step 5: Review Test Quality

Ask yourself:
- Are tests readable and express clear intent?
- Are tests fully self-contained?
- Do they test behavior, not implementation details?
- Are they fast enough?
- Will they help future developers understand the code?

---

## Decision Trees

### What Type of Test Should I Write?

```
Is it domain logic with no external dependencies?
├─ YES → Unit test (no test doubles)
│         Use real domain objects
└─ NO → Does it interact with external systems?
    ├─ YES → Integration test or Unit test with test doubles
    │         Mock external dependencies
    └─ NO → Does it coordinate between components?
        ├─ YES → Integration test
        │         Test real collaboration
        └─ NO → Unit test (minimal or no test doubles)
```

### Should I Mock This Dependency?

```
Is it an external system (database, API, filesystem, network)?
├─ YES → Use test double (mock/stub/fake)
└─ NO → Is it infrastructure layer?
    ├─ YES → Use test double
    └─ NO → Is it domain logic?
        ├─ YES → Use real object
        └─ NO → Is it slow or non-deterministic?
            ├─ YES → Use test double
            └─ NO → Use real object
```

### How Many Test Cases Do I Need?

```
For each public operation:
1. Happy path (at least 1 test)
2. For each input parameter:
   - Null case (if nullable)
   - Empty case (if applicable: collections, strings)
   - Invalid case (if validation exists)
   - Boundary values (min, max, zero, negative)
3. For each error condition:
   - Test the condition that triggers it
   - Verify expected error behavior
4. For each branch in logic:
   - Test true path
   - Test false path
```

---

## Test Types

### Unit Tests

**Purpose**: Test a single component in isolation

**Characteristics:**
- Fast (< 100ms per test)
- No external dependencies
- Test doubles for collaborators
- Test single responsibility

**When to use:**
- Testing domain entities and value objects
- Testing application use cases with mocked ports
- Testing algorithms and business rules

### Integration Tests

**Purpose**: Test interaction between components

**Characteristics:**
- Slower (< 1 second per test)
- May use real infrastructure (in-memory databases, test containers)
- Test port/adapter integration
- Verify component collaboration

**When to use:**
- Testing adapters against real implementations
- Testing database queries
- Testing API endpoints
- Testing message handling

### Contract Tests

**Purpose**: Verify interface contracts are respected

**Characteristics:**
- Test port interfaces are correctly implemented by adapters
- Can use consumer-driven contract testing
- Ensure compatibility between components

**When to use:**
- Testing adapter implementations match port contracts
- Testing external API integrations
- Preventing breaking changes

---

## Patterns and Best Practices

### Self-Contained Tests (Critical)

**Always create test data locally within each test:**

```
// Good: Self-contained test
test_user_creation() {
    // Given: Create test data locally
    var email = "user@example.com"
    var name = "John Doe"

    // When
    var user = createUser(email, name)

    // Then
    assert user.email equals email
}

// Bad: Shared state between tests
class UserTests {
    var sharedEmail = "user@example.com"  // ❌ Don't do this
    var sharedUser = null                  // ❌ Don't do this

    test_creation() {
        sharedUser = createUser(sharedEmail)  // ❌ Mutates shared state
    }

    test_retrieval() {
        assert sharedUser is not null  // ❌ Depends on previous test
    }
}
```

### Test Data Creation

**Option 1: Inline Creation**
```
test_scenario() {
    // Given: Create directly in test
    var product = Product(id: "123", name: "Widget", price: 9.99)

    // When-Then...
}
```

**Option 2: Factory Functions/Builders**
```
test_scenario() {
    // Given: Use builder/factory
    var product = buildProduct(id: "123", name: "Widget")

    // When-Then...
}
```

**Critical**: Each test must create its own data, even if using builders.

### Testing Exceptions/Errors

Tests should verify both that errors occur and their details:

```
test_invalid_input_throws_error() {
    // Given: Invalid data
    var invalidEmail = "not-an-email"

    // When-Then: Verify error is thrown
    assertThrows(ValidationError) {
        createUser(invalidEmail)
    }
}

test_error_contains_helpful_message() {
    // Given-When-Then
    try {
        createUser("invalid")
        fail("Expected exception")
    } catch (ValidationError e) {
        assert e.message contains "email"
        assert e.message contains "invalid format"
    }
}
```

### Parameterized Tests

When testing multiple similar cases, use parameterized tests:

```
test_validation_rejects_blank_inputs(input: String) {
    // Test with multiple blank values: "", "  ", "\t", "\n"

    // When-Then
    assertThrows(ValidationError) {
        validate(input)
    }
}
```

---

## Anti-Patterns to Avoid

### ❌ Shared Mutable State

```
// Bad: State shared across tests
class ServiceTests {
    var service = new Service()  // ❌ Shared
    var testData = []            // ❌ Shared

    test_one() {
        testData.add("item")  // ❌ Mutates shared state
    }
}

// Good: Each test creates its own state
class ServiceTests {
    test_one() {
        var service = new Service()  // ✅ Local
        var testData = ["item"]       // ✅ Local
    }
}
```

### ❌ Testing Implementation Details

```
// Bad: Tests internal structure
test_uses_correct_algorithm() {
    service.calculate(input)
    verify_internal_method_was_called()  // ❌ Tests "how"
}

// Good: Tests behavior
test_calculation_returns_correct_result() {
    // Given-When
    var result = service.calculate(input)

    // Then
    assert result equals expected  // ✅ Tests "what"
}
```

### ❌ Test Interdependence

```
// Bad: Tests depend on execution order
test_step1_creation() {
    sharedUser = create()  // ❌ Sets shared state
}

test_step2_modification() {
    modify(sharedUser)  // ❌ Depends on step1
}

// Good: Independent tests
test_creation() {
    var user = create()  // ✅ Local setup
    assert user.isValid()
}

test_modification() {
    var user = create()  // ✅ Local setup
    modify(user)
    assert user.wasModified()
}
```

### ❌ Slow Tests

```
// Bad: Uses sleep/delays
test_async_completion() {
    startAsync()
    sleep(5000)  // ❌ Slow and brittle
    assert isComplete()
}

// Good: Uses proper synchronization
test_async_completion() {
    var latch = new CountDownLatch(1)
    startAsync(onComplete: -> latch.countDown())
    assert latch.await(1, SECONDS)  // ✅ Fast and reliable
}
```

---

## Coverage Guidelines

### Minimum Coverage Targets

| Layer | Line Coverage | Branch Coverage |
|-------|--------------|-----------------|
| Domain | 100% | 100% |
| Application (Handlers) | 95% | 90% |
| Infrastructure (Adapters) | 80% | 75% |
| API/Controllers | 75% | 70% |

### What to Measure

- **Line coverage**: Every line of code executed
- **Branch coverage**: Every branch (if/else, switch) taken
- **Path coverage**: Different execution paths tested

### When 100% Coverage is Not the Goal

Some code doesn't require 100% coverage:
- Trivial getters/setters (if no logic)
- Framework-generated code
- Configuration classes
- Pure logging/debugging code

**However:** Complex getters with logic, validation, or transformations **do** need tests.

---

## Testing Hexagonal Architecture

### Testing Domain Layer

- **No test doubles**: Domain should have no external dependencies
- **Test business logic**: Focus on rules, invariants, and domain behavior
- **Test value objects**: Equality, validation, immutability
- **Test entities**: State transitions, business rules
- **Use real objects**: All domain objects are real instances

### Testing Application Layer (Use Cases)

- **Mock ports**: External dependencies through port interfaces
- **Test orchestration**: Verify correct collaboration between components
- **Test transaction boundaries**: If applicable
- **Self-contained setup**: Each test creates its own mocks locally

### Testing Adapters (Infrastructure)

- **Integration tests**: Use real or in-memory implementations when possible
- **Test contracts**: Verify adapter matches port interface
- **Test error handling**: Network failures, timeouts, retries
- **Test data mapping**: Correct transformation between external and domain models

---

## Notes

- **Tests are documentation**: They show how to use your code
- **Tests are specifications**: They define expected behavior
- **Tests enable refactoring**: Change implementation with confidence
- **Self-contained tests are maintainable**: No hidden dependencies
- **Fast feedback is crucial**: Keep tests fast for rapid iteration
- **Test behavior, not implementation**: Focus on "what", not "how"

---
