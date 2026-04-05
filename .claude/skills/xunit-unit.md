# xUnit Unit Testing

## Rules
- One test class per source class
- Tests must be able to fail — assert real outcomes, not just that values are non-null
- Each test asserts only what it's specifically testing — don't duplicate assertions from other tests
- Write descriptive test names using the pattern `MethodUnderTest_Scenario_ExpectedResult`
- Always use `async Task` for test methods — never sync
- Use raw xUnit `Assert` — do not introduce FluentAssertions unless already present in the project

> **Note on FluentAssertions:** A popular third-party library that makes assertions read like English (`result.Id.Should().Be(42)`). Consider it for new greenfield projects. Never introduce it into a project that doesn't already use it.

---

## Test Structure

Each test class targets a single source class. The constructor sets up mocks and the system under test (SUT). No base class or fixture needed — unit tests are self-contained.

```csharp
public class MyServiceTests
{
    private readonly Mock<IMyRepository> _mockRepo = new();
    private readonly MyService _service;

    public MyServiceTests()
    {
        _service = new MyService(_mockRepo.Object);
    }

    [Fact]
    public async Task GetByIdAsync_WithValidId_ReturnsItem()
    {
        var item = TestDataFactory.CreateItem();

        _mockRepo
            .Setup(r => r.GetByIdAsync(item.Id))
            .ReturnsAsync(item);

        var result = await _service.GetByIdAsync(item.Id);

        Assert.Equal(item, result);
        _mockRepo.Verify(r => r.GetByIdAsync(item.Id), Times.Once);
    }
}
```

---

## Moq Patterns

### Setup — tell the mock what to return

```csharp
_mockRepo.Setup(r => r.GetByIdAsync(42)).ReturnsAsync(expected);
_mockRepo.Setup(r => r.GetByIdAsync(It.IsAny<int>())).ReturnsAsync(expected);
_mockRepo.Setup(r => r.GetByIdAsync(999)).ThrowsAsync(new NotFoundException());
```

### Verify — assert the mock was called correctly

```csharp
_mockRepo.Verify(r => r.GetByIdAsync(42), Times.Once);
_mockRepo.Verify(r => r.DeleteAsync(It.IsAny<int>()), Times.Never);
```

---

## Helpers

### TestDataFactory

Use a `TestDataFactory` instead of manually constructing model objects. Override only the fields relevant to your test. If the project doesn't have one yet, create it following this pattern.

```csharp
var item = TestDataFactory.CreateItem();
var inactive = TestDataFactory.CreateItem(isActive: false);
```

When adding new model types, add corresponding factory methods with sensible defaults and named optional parameters.

### TestAuthHelper

Use a `TestAuthHelper` to build `ClaimsPrincipal` objects without the ceremony of claims, identities, and schemes. If the project doesn't have one yet, create it following this pattern.

```csharp
var principal = TestAuthHelper.CreateAuthenticatedUser("user-id");
var principal = TestAuthHelper.CreateUnauthenticatedUser();
```

---

## What to Test vs What to Skip

### Good candidates
- Exception handlers and middleware — verify correct status codes and response bodies
- Authorization handlers — verify succeed/fail decisions based on user claims
- Services with business logic — validation, transformation, branching logic
- Extension methods — pure functions with clear inputs and outputs

### Skip or defer
- Pure pass-through services (service just calls repository and returns) — low value
- Controllers — better covered by integration tests
- Repository implementations — these hit the database; test via integration tests

---

## File Organization

```
MyProject.Tests.Unit/
├── Helpers/          # TestDataFactory, TestAuthHelper, shared utilities
├── Infrastructure/   # Exception handlers, auth handlers, middleware
├── Services/         # Service-layer tests
└── README.md
```

- Mirror the source project's folder structure
- One test class per source class, named `{ClassName}Tests.cs`

---

## Additional Resources

- [xUnit Documentation](https://xunit.net/docs/getting-started/netcore/cmdline)
- [Moq Quickstart](https://github.com/devlooped/moq/wiki/Quickstart)
- [Unit Testing Best Practices (.NET)](https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices)
