# .NET Web Development with Obsidian & Claude Code #

> Created entirely based on my convo with Gemini and asking it at the end to generate a markdown
>
> * __Where this should be utilized and taken to the next step__ is at my GH Template project [__Dotnet Minimal API with AI Rules__](https://github.com/hasilo-org/gh-template-for-dotnet-minimal-api-with-ai-rules)

## Comprehensive Engineering Strategy: .NET Web Development with Obsidian & Claude Code ##

This strategy guide codifies the development workflow, directory separation, and technical constraints established for building modern, scalable .NET web applications using __Claude Code__ as an autonomous execution engine and __Obsidian__ as a human-controlled knowledge vault.

## 1. Architectural Philosophy & Strategy Overview

### The Paradox of LLMs in Codebase Scaling 

LLMs and automated AI agents are stateless by default. Left to their own devices, they frequently revert to generic boilerplate styles, generate code that leaks layers, or suffer from session drift once a terminal window closes or context windows reset.

### The Core Solution: Dual-Vault Context Engineering

To enforce continuous governance over your architecture, the workspace is split into two distinct sub-directories located inside the solution root:

1. __`.docs/` (Human Knowledge Vault):__ A __strict read-only area for the AI__. This is your personal workspace where you document strategic requirements, paste internet research, write design dictates, and reference implementation templates. The AI reads this to understand constraints but is forbidden from modifying its content.
2. __`.wiki/` (AI Execution Space):__ A __read-write sandbox managed entirely by Claude Code__. The AI uses this directory to build, maintain, and update its own execution logs, feature checklists, and system plans. This bridges state continuity across short-lived terminal sessions without polluting human documentation.

## 2. Project Boundary & Configuration Setup

### Root Level: `CLAUDE.md`

Claude Code automatically looks for a local orchestration file named `CLAUDE.md` at the directory root to dynamically override its internal system prompt. It forces the agentic model to obey formatting rules, testing standards, and build tasks.

Create this file exactly at the root of your .NET solution directory:

```markdown
# Claude Code .NET Assistant Configuration

You are an expert .NET 10 Web API developer. Always check `.docs/` and `.wiki/` before modifying code.

## System & Build Commands
- **Build Solution:** `dotnet build`
- **Run All Tests:** `dotnet test`
- **EF Migration:** `dotnet ef migrations add [Name] --project src/YourProject.Infrastructure/ --startup-project src/YourProject.Web/`
- **Update Database:** `dotnet ef database update --project src/YourProject.Infrastructure/ --startup-project src/YourProject.Web/`

## Strict Behavioral Rules
1. **CRITICAL:** Do NOT modify, delete, or add any files inside the `.docs/` directory. This is human-only space.
2. **SOLID PRINCIPLES:** All generated code must strictly adhere to SOLID design principles:
   - *Single Responsibility:* Feature services must do one thing (e.g., handle one slice's business logic).
   - *Open/Closed:* Use abstraction or behavior configuration instead of hardcoded conditional flags for extensions.
   - *Interface Segregation:* Client code must not depend on fat interfaces; split repository or service interfaces by feature needs if applicable.
   - *Dependency Inversion:* High-level modules must depend on abstractions. Inject dependencies via constructors—never instantiate them inline.
3. **MEDIANTR BAN:** Never use MediatR or MediatR.Contracts. It has transitioned to a commercial license. Instead, use native C# Dependency Injection with direct service injection into Minimal API endpoint groups.
4. **CONFIGURATION STIPULATION:** Never inject or pass around `IConfiguration` inside feature domains or controllers. You must strictly use the strongly-typed **Options Pattern** via `IOptions<T>` or `IOptionsSnapshot<T>`.
5. **TESTING:** Always write tests using **xUnit**, **Moq**, and **FluentAssertions** inside the `tests/` directory.

## Contextual Routing (Read these notes first)
- **Architecture & DI Registration:** Read `.docs/Architecture/Minimal-API-Pattern.md`
- **Configuration & Options:** Read `.docs/Architecture/Options-Pattern.md`
- **Testing & Mocking:** Read `.docs/Testing/Test-Strategy.md`

## AI Workspace (.wiki/) Execution Plan
- You are required to maintain your own internal state, task checklists, and code mapping inside the `.wiki/` directory.
- Update `.wiki/Current-Execution-Plan.md` before and after every feature implementation to track what files you modified.
```

## 3. Human Knowledge Vault (`.docs/`) — Read-Only Snippets

Place these structural guidelines, best practices, and templates inside your `.docs/` folder. They prevent the AI from generating obsolete patterns and enforce modern C# paradigms.

### File: `.docs/Architecture/Minimal-API-Pattern.md`

````markdown
# Architectural Pattern: Vertical Slices, EF Core, & FluentValidation

## EF Core Registration Syntax
Register the `DbContext` in `Program.cs` utilizing the connection string extracted via the Options pattern.

```csharp
// Program.cs DB Context Registration Template
builder.Services.AddDbContext<ApplicationDbContext>((serviceProvider, options) =>
{
    var databaseOptions = serviceProvider.GetRequiredService<IOptions<DatabaseOptions>>().Value;
    
    options.UseSqlServer(databaseOptions.ConnectionString, sqlOptions =>
    {
        sqlOptions.MigrationsAssembly(typeof(ApplicationDbContext).Assembly.FullName);
        sqlOptions.EnableRetryOnFailure(maxRetryCount: 5);
    });
});
```
````

## Feature Slice with FluentValidation
Every feature slice requiring request validation must utilize `FluentValidation`. Validate requests immediately at the beginning of service execution to enforce the Single Responsibility Principle.

```csharp
namespace MyProject.Features.Orders;

public record CreateOrderRequest(Guid ProductId, int Quantity);
public record OrderResponse(Guid Id, string Status);

// Single Responsibility: Validation Rules
public class CreateOrderValidator : AbstractValidator<CreateOrderRequest>
{
    public CreateOrderValidator()
    {
        RuleFor(x => x.ProductId).NotEmpty().WithMessage("Product ID is required.");
        RuleFor(x => x.Quantity).GreaterThan(0).WithMessage("Quantity must be greater than zero.");
    }
}

// Single Responsibility: Domain Execution Strategy
public class CreateOrderService
{
    private readonly ApplicationDbContext _db;
    private readonly IValidator<CreateOrderRequest> _validator;

    public CreateOrderService(ApplicationDbContext db, IValidator<CreateOrderRequest> validator)
    {
        _db = db;
        _validator = validator;
    }

    public async Task<IResult> HandleAsync(CreateOrderRequest request)
    {
        var validationResult = await _validator.ValidateAsync(request);
        if (!validationResult.IsValid)
        {
            return Results.ValidationProblem(validationResult.ToDictionary());
        }

        // Business execution logic goes here...
        return Results.Ok(new OrderResponse(Guid.NewGuid(), "Created"));
    }
}

// Minimal API Route Mapping
public static class CreateOrderEndpoint
{
    public static void MapCreateOrder(this IEndpointRouteBuilder app)
    {
        app.MapPost("/api/orders", async (CreateOrderRequest request, CreateOrderService service) =>
        {
            return await service.HandleAsync(request);
        })
        .WithName("CreateOrder")
        .WithOpenApi();
    }
}
```

### File: `.docs/Architecture/Options-Pattern.md`

````markdown

# Configuration Standard: Options Pattern

We strictly prohibit injecting `IConfiguration` into custom services or endpoints. This violates the Dependency Inversion and Interface Segregation principles by leaking global configuration layout into granular domains.

## Step 1: Define the Options Record

```c#
namespace MyProject.Infrastructure.Options;

public record DatabaseOptions
{
    public const string SectionName = "Database";
    
    [Required]
    public string ConnectionString { get; init; } = string.Empty;
}
```

## Step 2: Bind Options in Program.cs

```c#
builder.Services.AddOptions<DatabaseOptions>()
    .Bind(builder.Configuration.GetSection(DatabaseOptions.SectionName))
    .ValidateDataAnnotations()
    .ValidateOnStart();
```

## Step 3: Consume Options via Constructor Injection

```c#
public class PaymentProcessingService
{
    private readonly DatabaseOptions _options;

    public PaymentProcessingService(IOptions<DatabaseOptions> options)
    {
        _options = options.Value;
    }
}
```

````
### File: `.docs/Testing/Test-Strategy.md`

````markdown
# Testing Strategy: xUnit, Moq, & FluentAssertions

All tests must be written inside the `tests/` directory using **xUnit** and **Moq**. Avoid using native xUnit `Assert.*` statements—strictly use FluentAssertions `.Should()` expressions for clean, domain-readable tests.

## Unit Testing & Mocking Standards
- Initialize mocks in the constructor to keep tests dry and isolated.
- Use `It.IsAny<T>()` and `.ReturnsAsync()` for setting up behavior.
- Always use `.Verify()` to check behavior side effects on commands.

### Reference Unit Testing Template
```csharp
using Xunit;
using FluentAssertions;
using Moq;

namespace MyProject.UnitTests.Features.Orders;

public class CreateOrderServiceTests
{
    private readonly CreateOrderService _sut; // System Under Test
    private readonly Mock<IOrderRepository> _orderRepoMock;

    public CreateOrderServiceTests()
    {
        _orderRepoMock = new Mock<IOrderRepository>();
        _sut = new CreateOrderService(_orderRepoMock.Object, new CreateOrderValidator());
    }

    [Fact]
    public async Task HandleAsync_GivenValidRequest_ShouldReturnCreatedStatus()
    {
        // Arrange
        var request = new CreateOrderRequest(Guid.NewGuid(), 3);
        
        _orderRepoMock
            .Setup(repo => repo.SaveAsync(It.IsAny<Order>()))
            .ReturnsAsync(true);

        // Act
        var result = await _sut.HandleAsync(request);

        // Assert
        result.Should().NotBeNull();
        
        // Verify mock behavior
        _orderRepoMock.Verify(repo => repo.SaveAsync(It.IsAny<Order>()), Times.Once);
    }
}
```
````

## Integration Testing Strategy [PLACEHOLDER]
*Human Note: The exact database tool for integration testing is currently under evaluation (e.g., Testcontainers vs. Respawn vs. SQLite In-Memory).*

### Rules for Claude:
- Until this placeholder is updated by the human engineer, do NOT write physical infrastructure integration tests that depend on a live external database.
- Focus strictly on unit testing feature slices using the Moq framework defined above.
```

---

## 4. AI Workspace Vault (`.wiki/`) — Read-Write Sandbox

When execution commands run, Claude Code evaluates your constraints and writes its own runtime logs to track dependencies, codebase changes, and task lists without interfering with human files.

### File Sample: `.wiki/Current-Execution-Plan.md`
```markdown
# AI Current Execution Plan

## Target Objective
Scaffold the `UpdateOrder` feature slice.

## Architectural Conformance Verification
- **SOLID Audit:** Created separate validation models to insulate business execution logic (SRP Compliance).
- **MediatR Check:** Confirmed zero references to MediatR namespace. Registered feature dependencies via direct container mapping.
- **Options Compliance:** Evaluated database dependency via `IOptions<DatabaseOptions>`. Global `IConfiguration` boundary was preserved.
- **Integration Test Status:** **PAUSED.** Test strategy file specifies integration infrastructure is frozen via a placeholder. Skipping external database integration verification.

## Target Execution Checklist
- [ ] Implement `UpdateOrder` Minimal API Endpoint and Feature Service.
- [ ] Build xUnit + Moq unit tests matching the AAA layout standard.
- [ ] Run execution test loop using `dotnet test` until terminal exit code is 0.
```

---

## 5. Execution Prompts to Launch the Cycle

When opening a new session in your terminal environment with Claude Code, issue this bootstrap prompt to activate the system boundaries and lock in the workflow:

```bash
claude /run "Review all design rules in root CLAUDE.md and technical snippets inside .docs/. Making sure your hands stay completely off the .docs/ directory, initialize a planning log inside .wiki/Current-Execution-Plan.md detailing how you intend to scaffold our first feature while honoring the MediatR ban, Options requirements, and integration test pause."
```

***

<FollowUp>
Now that the blueprint is complete, let me know:
* Do you want to add a snippet for **logging standards (ILogger)** or **Structured Logging mapping** to the `CLAUDE.md` requirements?
* Would you like me to outline how you can configure **Obsidian properties/frontmatter** to easily filter your human research documents within your vault interface?
</FollowUp>
```



