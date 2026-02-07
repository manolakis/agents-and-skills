---
name: java-language
description: >
  Write, refactor and review production-grade Java code whilst strictly adhering to the language
  level configured to the project (e.g., as defined in asdf `.tool-versions` file or `pom.xml`).
  Use this skill for Java-only tasks: API design, core language features, refactoring, readability,
  correctness, performance and concurrency. Do not provide build-tool instructions or modify build
  files.
compatibility: >
  Works with any Java language level. The agent must detect and follow the project's configured
  source/target/release version when available (e.g. from configuration files). If unclear, ask the
  user to confirm the intended language level.
metadata:
  version: 1.0.0
  auto-invoke:
    - Writing Java code
    - Reviewing Java code
    - Refactoring Java code
---

# Critical rules

- ALWAYS adhere to the project's configured Java language level
- NEVER use features from a newer Java version than configured
- NEVER use raw types or suppress warnings without justification

For general code-quality principles (Clean Code, SOLID and Object Calisthenics), follow the
`code-quality` skill.

---

# Language Level Detection

1. Check `.tool-versions` for Java version
2. Check `pom.xml` for `maven.compiler.source`/`target`/`release`
3. Check `build.gradle` for `sourceCompatibility`/`targetCompatibility`
4. If unclear, ask the user to confirm the Java version

---

# Code Quality Standards

## Naming Conventions
- **Classes/Interfaces:** `PascalCase` (e.g., `UserRepository`)
- **Methods/Variables:** `camelCase` (e.g., `calculateTotal`)
- **Constants:** `UPPER_SNAKE_CASE` (e.g., `MAX_RETRY_ATTEMPTS`)
- **Packages:** `lowercase` (e.g., `com.example.domain`)

## Java-Specific Guidance
- Use generics to maintain type safety; avoid raw types
- Prefer `Optional` for absent return values; avoid `Optional` for fields and parameters
- Use `try-with-resources` for `AutoCloseable` resources
- Use `Objects.requireNonNull` (or equivalent) for mandatory arguments
- Use `var`, `record`, `sealed` and pattern matching only when allowed by the configured Java version
- Prefer immutable data for concurrency; use `java.util.concurrent` over manual locking when appropriate
- Avoid shared mutable state; favour thread confinement and clear ownership boundaries
- Use the module system (`module-info.java`) only if the project already adopts modules

---

# Workflow

When writing or refactoring Java code:

1. **Detect language level** using detection workflow above
2. **Select language features** that are valid for the configured Java version
3. **Apply Java naming conventions** and package structure rules
4. **Ensure type safety** with generics and avoid raw types
5. **Handle resources safely** with `try-with-resources`
6. **Review** against Critical Rules and Java-Specific Guidance

---

# Examples

## Good: Type-safe Generics

```java
var names = new ArrayList<String>();
```

## Bad: Raw Types

```java
var names = new ArrayList();
```

## Good: Try-with-resources

```java
try (var reader = Files.newBufferedReader(path)) {
    return reader.readLine();
}
```

## Bad: Manual Resource Handling

```java
var reader = Files.newBufferedReader(path);
var line = reader.readLine();

reader.close();
```

---

# Common Pitfalls

- Using features from newer Java versions than configured
- Using raw types instead of generics
- Suppressing warnings without justification
- Misusing `Optional` (e.g., using it for fields or parameters)
- Failing to close resources or avoiding `try-with-resources`

---

# Testing in Java

This section defines Java-specific testing patterns, frameworks, and practices. For general testing principles (Given-When-Then, test independence, coverage), see the `testing` skill.

## Testing Frameworks

- **JUnit 5 (Jupiter)**: Primary testing framework with built-in assertions
- **Mockito**: For creating test doubles (mocks, spies, stubs)

## Test Structure Pattern

Follow **Given-When-Then** with `@DisplayName` annotation (for regular tests) or `name` attribute (for parameterized tests):

### Regular Tests with @DisplayName

```java
@Test
@DisplayName("""
    Given [context]
    When [action]
    Then [outcome]
    """)
void descriptiveMethodName() {
    var input = createTestInput();
    var dependency = mock(Dependency.class);
    when(dependency.doSomething()).thenReturn(expected);

    var result = sut.performOperation(input);

    assertEquals(expected, result);
    verify(dependency).doSomething();
}
```

### Parameterized Tests with name Attribute

```java
@ParameterizedTest(name = """
    Given blank input {0}
    When validating
    Then validation fails
    """)
@ValueSource(strings = {"", "  ", "\t", "\n"})
void testBlankInputsRejected(String input) {
    assertThrows(ValidationException.class, () ->
        validator.validate(input)
    );
}

@ParameterizedTest(name = """
    Given the {0} email address
    When validating the email
    Then validation result matches expected {1} result
    """)
@CsvSource({
    "john@example.com, true",
    "invalid-email, false",
    "test@, false"
})
void testEmailValidation(String email, boolean expected) {
    var result = validator.isValidEmail(email);

    assertEquals(expected, result);
}

@ParameterizedTest(name = """
    Given test case with input {0}
    When processing
    Then result matches expected {1}
    """)
@MethodSource("provideTestCases")
void testWithMethodSource(String input, String expected) {
    var result = service.process(input);

    assertEquals(expected, result);
}

private static Stream<Arguments> provideTestCases() {
    return Stream.of(
        Arguments.of("input1", "output1"),
        Arguments.of("input2", "output2")
    );
}
```

## Critical Java Testing Rules

1. **Self-contained tests**: Create ALL test data locally within each test method
2. **No shared state**: Avoid class-level variables that are mutated across tests
3. **Use `var` for local variables**: Improves readability
4. **Use `@DisplayName` with triple-quote strings** for Given-When-Then descriptions (regular tests with `@Test`)
5. **Use `name` attribute in `@ParameterizedTest`** for Given-When-Then descriptions with parameter placeholders (`{0}`, `{1}`, etc.)
6. **Static imports**: For assertions and Mockito methods

## Self-Contained Test Pattern (Critical)

```java
class UserServiceTest {
    // ✅ Good: No mutable class-level fields

    @Test
    @DisplayName("""
        Given a valid user request
        When creating a user
        Then the user is persisted correctly
        """)
    void testUserCreation() {
        var email = "user@example.com";
        var name = "John Doe";
        var repository = mock(UserRepository.class);
        var service = new UserService(repository);

        var user = service.createUser(email, name);

        verify(repository).save(any());
    }

    @Test
    @DisplayName("""
        Given a duplicate email
        When creating a user
        Then an exception is thrown
        """)
    void testDuplicateEmail() {
        var email = "user@example.com";
        var repository = mock(UserRepository.class);
        when(repository.existsByEmail(email)).thenReturn(true);
        var service = new UserService(repository);

        assertThrows(DuplicateUserException.class, () ->
            service.createUser(email, "John")
        );
    }
}
```

## Anti-Pattern: Shared Mutable State

```java
// ❌ BAD: Do NOT do this
class UserServiceTest {
    // ❌ Shared mutable fields
    private UserRepository repository;
    private UserService service;
    private User testUser;

    @BeforeEach
    void setUp() {
        repository = mock(UserRepository.class);  // ❌ Shared across tests
        service = new UserService(repository);     // ❌ Shared across tests
    }

    @Test
    void testOne() {
        testUser = service.createUser("test@example.com");  // ❌ Mutates shared field
    }

    @Test
    void testTwo() {
        // ❌ This test might depend on testOne's state
        assertNotNull(testUser);
    }
}

// ✅ GOOD: Self-contained tests
class UserServiceTest {
    @Test
    @DisplayName("""
        Given a new user
        When creating the user
        Then the user is created successfully
        """)
    void testOne() {
        // ✅ All setup local to this test
        var repository = mock(UserRepository.class);
        var service = new UserService(repository);

        var user = service.createUser("test@example.com", "Test");

        assertNotNull(user);
    }
}
```

## Mockito Patterns

### Creating Mocks Locally

```java
@Test
@DisplayName("""
    Given a mocked repository
    When executing a command
    Then the repository is called correctly
    """)
void testWithMock() {
    var repository = mock(CommandRepository.class);
    var handler = new CommandHandler(repository);
    var command = new TestCommand();
    var context = mock(Context.class);

    handler.handle(command, context);

    verify(repository).save(any());
}
```

### Using Spies

```java
@Test
@DisplayName("""
    Given a spy on the handler
    When dispatching a command
    Then the handler is invoked
    """)
void testWithSpy() {
    var commandHandler = spy(TestCommandHandler.class);
    var commandBus = CommandBus.newInstance()
        .registerCommandHandler(TestCommand.class, commandHandler);
    var command = new TestCommand();
    var context = mock(Context.class);

    commandBus.dispatch(command, context);

    verify(commandHandler).handle(command, context);
}
```

### Argument Matchers

```java
@Test
void testArgumentMatchers() {
    var repository = mock(UserRepository.class);
    var service = new UserService(repository);

    service.createUser("user@example.com", "John");

    verify(repository).save(argThat(user ->
        user.getEmail().equals("user@example.com") &&
        user.getName().equals("John") &&
        user.getStatus() == UserStatus.ACTIVE
    ));
}
```

### InOrder Verification

```java
@Test
@DisplayName("""
    Given multiple middlewares
    When dispatching a command
    Then middlewares are invoked in order
    """)
void testExecutionOrder() {
    var middleware1 = spy(TestMiddleware1.class);
    var middleware2 = spy(TestMiddleware2.class);
    var commandBus = CommandBus.newInstance()
        .registerMiddleware(middleware1)
        .registerMiddleware(middleware2);

    commandBus.dispatch(new TestCommand(), mock(Context.class));

    var inOrder = inOrder(middleware1, middleware2);
    inOrder.verify(middleware1).invoke(any(), any(), any());
    inOrder.verify(middleware2).invoke(any(), any(), any());
}
```

## Assertion Patterns

### JUnit 5 Assertions

```java
@Test
void testWithJUnitAssertions() {
    var result = service.execute(command);

    assertNotNull(result);
    assertEquals("expected", result.getValue());
    assertTrue(result.isSuccess());
    assertFalse(result.hasErrors());
}
```

### Exception Assertions

```java
@Test
@DisplayName("""
    Given an invalid command
    When executing the handler
    Then an IllegalArgumentException is thrown
    """)
void testExceptionThrown() {
    var handler = new CommandHandler();

    var exception = assertThrows(
        IllegalArgumentException.class,
        () -> handler.handle(null, mock(Context.class))
    );

    assertTrue(exception.getMessage().startsWith("Command cannot be null"));
}


```

## Parameterized Tests

```java
@ParameterizedTest(name = """
    Given blank input
    When validating
    Then validation fails
    """)
@ValueSource(strings = {"", "  ", "\t", "\n"})
void testBlankInputsRejected(String input) {
    assertThrows(ValidationException.class, () ->
        validator.validate(input)
    );
}

@ParameterizedTest(name = """
    Given an email address
    When validating the email
    Then validation result matches expected
    """)
@CsvSource({
    "john@example.com, true",
    "invalid-email, false",
    "test@, false"
})
void testEmailValidation(String email, boolean expected) {
    var result = validator.isValidEmail(email);

    assertEquals(expected, result);
}

@ParameterizedTest(name = """
    Given a test case
    When processing the input
    Then the result matches expected
    """)
@MethodSource("provideTestCases")
void testWithMethodSource(TestCase testCase) {
    var result = service.process(testCase.input());
    assertEquals(testCase.expected(), result);
}

private static Stream<TestCase> provideTestCases() {
    return Stream.of(
        new TestCase("input1", "output1"),
        new TestCase("input2", "output2")
    );
}
```

## Testing with Test Containers (Integration Tests)

```java
@SpringBootTest
@Testcontainers
class UserRepositoryIntegrationTest {
    @Container
    private static PostgreSQLContainer<?> postgres =
        new PostgreSQLContainer<>("postgres:15-alpine");

    @Autowired
    private UserRepository repository;

    @Test
    @DisplayName("""
        Given a valid user
        When saving to database
        Then the user is persisted correctly
        """)
    void testPersistence() {
        var email = "test@example.com";
        var user = User.create(email, "Test User");

        repository.save(user);

        var found = repository.findByEmail(email);
        assertTrue(found.isPresent());
        assertEquals("Test User", found.get().getName());
    }
}
```

## Test Utilities and Helpers

### Test Data Builders

```java
// TestCommandBuilder.java
public class TestCommandBuilder {
    private String userId = "default-user-id";
    private String action = "default-action";

    private TestCommandBuilder() {}

    public static TestCommandBuilder aCommand() {
        return new TestCommandBuilder();
    }

    public TestCommandBuilder withUserId(String userId) {
        this.userId = userId;
        return this;
    }

    public TestCommandBuilder withAction(String action) {
        this.action = action;
        return this;
    }

    public TestCommand build() {
        return new TestCommand(userId, action);
    }
}

// Usage in tests
@Test
void testWithBuilder() {
    var command = TestCommandBuilder.aCommand()
        .withUserId("user-123")
        .withAction("create")
        .build();
}
```

### Object Mother Pattern

```java
// TestCommands.java
public class TestCommands {
    private TestCommands() {}

    public static TestCommand validCommand() {
        return new TestCommand("user-123", "create");
    }

    public static TestCommand invalidCommand() {
        return new TestCommand(null, null);
    }

    public static TestCommand unauthorizedCommand() {
        return new TestCommand("guest-user", "admin-action");
    }
}

// Usage in tests
@Test
void testWithObjectMother() {
    var command = TestCommands.validCommand();
}
```

## Static Imports (Recommended)

```java
import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;
import static org.mockito.ArgumentMatchers.*;

class MyTest {
    @Test
    void test() {
        // Clean, readable test code
        var mock = mock(Repository.class);
        when(mock.find(any())).thenReturn(Optional.of(data));

        var result = service.execute(command);

        assertNotNull(result);
        verify(mock).find(any());
    }
}
```

## Test Organization

### Package Structure

```
src/test/java/
└── com/
    └── example/
        ├── domain/
        │   ├── UserTest.java
        │   └── EmailTest.java
        ├── application/
        │   ├── CreateUserHandlerTest.java
        │   └── GetUserQueryHandlerTest.java
        ├── infrastructure/
        │   └── UserRepositoryAdapterTest.java
        └── mocks/
            ├── TestCommand.java
            ├── TestCommandHandler.java
            └── TestCommands.java  // Object Mother
```

### Test Class Naming

- Domain tests: `ClassNameTest` (e.g., `UserTest`, `EmailTest`)
- Handler tests: `HandlerNameTest` (e.g., `CreateUserHandlerTest`)
- Integration tests: `ClassNameIntegrationTest`
- Adapter tests: `AdapterNameTest`

---

# Related Skills

- `hexagonal-architecture`: For layering and port/adapter patterns
- `testing`: For language-agnostic testing principles, coverage goals, and patterns
- `code-quality`: For general code quality guidelines

---
