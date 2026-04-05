# xUnit Integration Testing

## Rules
- One test class per controller or feature area
- Tests must be able to fail — assert real outcomes, not just status codes or non-null values
- Each test asserts only what it's specifically testing — don't duplicate assertions from other tests
- Write descriptive test names using the pattern `MethodUnderTest_Scenario_ExpectedResult`
- Always use `async Task` for test methods — never sync
- Use raw xUnit `Assert` — do not introduce FluentAssertions unless already present in the project

> **Note on FluentAssertions:** A popular third-party library that makes assertions read like English (`result.Id.Should().Be(42)`). Consider it for new greenfield projects. Never introduce it into a project that doesn't already use it.

---

## Test Structure

### Test Organization

Each test class uses `IClassFixture<T>` for shared test data and `IAsyncLifetime` for per-test setup and cleanup. Centralize `HttpClient` creation (e.g. via a factory helper) so shared configuration like `AllowAutoRedirect = false` is applied consistently.

```csharp
public class MyControllerTests : IClassFixture<SharedFixture>, IAsyncLifetime
{
    private readonly SharedFixture _shared;
    private readonly HttpClient _client;
    private readonly TestDataTracker _tracker = new();

    public MyControllerTests(SharedFixture shared)
    {
        _shared = shared;
        _client = TestClientFactory.Create(shared.Factory);
    }

    public async Task InitializeAsync()
    {
        await AuthHelper.LoginAsync(_client);
    }

    public Task DisposeAsync()
    {
        _client.Dispose();
        return Task.CompletedTask;
    }
}
```

### Shared vs Per-Test Data

**Read-only tests** use the shared fixture's pre-seeded data — no per-test seeding needed.

**Mutating tests** (create, update, delete, etc.) create their own data via helpers so they don't corrupt shared state.

```csharp
// Read-only — use shared data
var response = await _client.GetAsync($"/api/Resource/{_shared.Item1.Id}");

// Mutating — create your own
var item = await ResourceHelper.CreateAsync(_client, _tracker);
await _client.DeleteAsync($"/api/Resource/{item.Id}");
```

### Use Real Models from the Application

Deserialize API responses into the actual server models rather than creating private test-only records. If the test project references the server project, these models stay in sync automatically.

```csharp
// Good — uses the real model
var item = await response.Content.ReadFromJsonAsync<MyModel>();

// Bad — duplicate record that can drift out of sync
private record MyModelResponse(int Id, string Name);
```

---

## Test Data Management

### Verify Seeding Requirements Before Writing Tests

Before writing tests for a controller, check what data the endpoints actually return and ensure the necessary seed data exists. Tests should assert on **real data values** — a test that passes because a list is empty is not verifying meaningful behavior.

### Seed via API Helpers

Use helpers to seed data through the real API. This validates the create path and produces realistic test data.

```csharp
// Good — creates via API, auto-tracked for cleanup
var item = await ResourceHelper.CreateAsync(_client, _tracker);

// Bad — hardcoded IDs or data that may not exist
var itemId = 42;
```

### Test Data Cleanup

Centralize cleanup in the shared fixture's `DisposeAsync`. Individual test classes should not perform their own cleanup.

---

## Authentication

Centralize authentication in a helper. Support multiple test user roles where relevant.

```csharp
await AuthHelper.LoginAsync(_client);                        // default
await AuthHelper.LoginAsync(_client, TestUser.LimitedUser);  // specific role
```

---

## Waiting and Timeouts

Never use `Thread.Sleep`. Integration tests hit an in-memory test server — responses are synchronous from the test's perspective. Poll for expected state with a timeout if needed.

---

## File Organization

```
MyProject.Tests.Integration/
├── Assets/           # Static fixtures (CSV files, seed data, etc.)
├── Helpers/          # Auth, data creation, cleanup utilities
├── Controllers/      # One test class per controller or feature area
└── README.md
```

New helpers follow the pattern: create via API, register with tracker for cleanup.

---

## Additional Resources

- [ASP.NET Core Integration Testing](https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests)
- [xUnit Documentation](https://xunit.net/docs/getting-started/netcore/cmdline)
