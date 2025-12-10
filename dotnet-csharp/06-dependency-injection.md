# Dependency Injection (DI)

**Level:** 🚦 Intermediate  
**Tags:** `DI`, `IoC`, `architecture`, `testing`, `interview`

---

## Why this matters

Dependency Injection isn't just a pattern—it's **the architectural foundation of modern .NET applications**. ASP.NET Core, Entity Framework, and most production .NET systems are built entirely around DI. Not understanding it means you can't effectively work in professional .NET environments.

**Testability revolution:** Before DI became mainstream, testing was painful:
- Classes directly instantiated their dependencies (tight coupling)
- Mocking dependencies required complex workarounds or test frameworks
- Integration tests were slow and fragile
- Test coverage was difficult to achieve

With DI, testing becomes straightforward—you simply inject mock implementations. This difference is transformative: companies can achieve 80%+ test coverage with DI, vs struggling to reach 30% without it. Better tests mean fewer production bugs, faster deployment cycles, and higher confidence in changes.

**Loose coupling and maintainability:** In large codebases (100K+ lines), tight coupling becomes a nightmare:
- Changing one class breaks dozens of others
- Refactoring is risky and time-consuming
- Adding features requires touching many files
- Understanding code dependencies is difficult

DI inverts these dependencies, making code modular and maintainable. When you need to swap logging implementations, change databases, or replace external APIs, DI lets you make the change in one place (the registration) rather than hunting through the codebase.

**Service lifetime mastery:** Understanding Transient, Scoped, and Singleton lifetimes is critical because getting it wrong causes:
- **Memory leaks:** Singleton capturing Scoped service keeps it alive forever (captive dependency)
- **Concurrency bugs:** Sharing Transient state across requests causes race conditions
- **Database errors:** Incorrect DbContext lifetime causes "already tracking entity" errors
- **Performance issues:** Creating expensive services too frequently (should be Singleton)

These bugs are subtle and often only appear in production under load, making them particularly dangerous.

**ASP.NET Core architecture:** The entire ASP.NET Core framework is designed around DI:
- Controllers get dependencies injected
- Middleware uses DI for services
- Configuration is injected via IOptions<T>
- Logging, authentication, authorization all use DI

Not understanding DI means you can't follow the framework's conventions or leverage its full power. You'll write code that fights the framework instead of working with it.

**Cross-cutting concerns:** DI enables elegant solutions to cross-cutting concerns:
- Logging: Inject ILogger<T> everywhere it's needed
- Caching: Inject IMemoryCache or IDistributedCache
- Authentication: Inject IHttpContextAccessor or custom auth services
- Configuration: Inject IOptions<T> for strongly-typed settings
- Feature flags: Inject feature flag services

Without DI, these concerns get scattered throughout the codebase in inconsistent ways.

**Production patterns:** Professional .NET development patterns require DI:
- Repository pattern for data access
- Unit of Work pattern for transactions
- Strategy pattern for algorithms
- Decorator pattern for cross-cutting concerns
- Factory pattern for dynamic object creation

All of these patterns are vastly simpler with DI than without it.

**Interview expectations:** When interviewers ask about DI, they're evaluating:
- Can you design maintainable, testable architectures?
- Do you understand service lifetimes and their implications?
- Can you debug common DI problems (circular dependencies, captive dependencies)?
- Can you work effectively in ASP.NET Core?
- Do you know when to use interfaces vs concrete types?
- Can you explain IoC principles and their benefits?
- Do you understand the trade-offs of different DI containers?

DI knowledge is often a litmus test: developers who understand it can work on enterprise codebases; those who don't struggle with modern .NET development. It's not just about knowing the syntax—it's about understanding how to architect maintainable, testable, professional software systems.

---

## Core Ideas

### 1. **Dependency Injection inverts control of object creation**

**Without DI (tightly coupled):**
```csharp
public class OrderService
{
    private readonly EmailService _emailService;
    
    public OrderService()
    {
        _emailService = new EmailService(); // ❌ Tight coupling
    }
}
```

**With DI (loosely coupled):**
```csharp
public class OrderService
{
    private readonly IEmailService _emailService;
    
    public OrderService(IEmailService emailService) // ✅ Dependency injected
    {
        _emailService = emailService;
    }
}
```

**Mental model:**  
"Don't call us, we'll call you. Let the container create and inject dependencies."

---

### 2. **Service lifetimes control object lifecycle**

| Lifetime | Behavior | When to use |
|----------|----------|-------------|
| **Transient** | New instance every time | Lightweight, stateless services |
| **Scoped** | One instance per request/scope | DbContext, per-request data |
| **Singleton** | Single instance for app lifetime | Configuration, caches, shared state |

```csharp
// Registration
services.AddTransient<IEmailService, EmailService>();
services.AddScoped<IOrderService, OrderService>();
services.AddSingleton<IConfigService, ConfigService>();
```

**Key rules:**
- ✅ Singleton can depend on Singleton
- ✅ Scoped can depend on Scoped or Singleton
- ✅ Transient can depend on anything
- ❌ Singleton should NOT depend on Scoped or Transient (captive dependency)

---

### 3. **Constructor injection is preferred**

```csharp
// ✅ Constructor injection (preferred)
public class OrderService
{
    private readonly ILogger<OrderService> _logger;
    private readonly IEmailService _emailService;
    
    public OrderService(ILogger<OrderService> logger, IEmailService emailService)
    {
        _logger = logger ?? throw new ArgumentNullException(nameof(logger));
        _emailService = emailService ?? throw new ArgumentNullException(nameof(emailService));
    }
}

// ⚠️ Property injection (rare, for optional dependencies)
public class OrderService
{
    public ILogger<OrderService> Logger { get; set; }
}

// ❌ Method injection (avoid)
public void ProcessOrder(Order order, IEmailService emailService)
{
    // Mixing concerns
}
```

**Why constructor injection:**
- Dependencies are explicit and required
- Object is always in valid state
- Easier to test
- Compiler catches missing dependencies

---

### 4. **Program.cs registration in .NET 6+**

```csharp
var builder = WebApplication.CreateBuilder(args);

// Register services
builder.Services.AddTransient<IEmailService, EmailService>();
builder.Services.AddScoped<IOrderService, OrderService>();
builder.Services.AddSingleton<IConfigService, ConfigService>();

// Built-in services
builder.Services.AddControllers();
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("Default")));

var app = builder.Build();

// Middleware pipeline
app.MapControllers();
app.Run();
```

---

### 5. **DI enables testability**

```csharp
// Production code
public class OrderService
{
    private readonly IEmailService _emailService;
    
    public OrderService(IEmailService emailService)
    {
        _emailService = emailService;
    }
    
    public void ProcessOrder(Order order)
    {
        // Process order...
        _emailService.SendConfirmation(order);
    }
}

// Test code with mock
public class OrderServiceTests
{
    [Fact]
    public void ProcessOrder_SendsConfirmationEmail()
    {
        // Arrange
        var mockEmail = new Mock<IEmailService>();
        var service = new OrderService(mockEmail.Object);
        var order = new Order();
        
        // Act
        service.ProcessOrder(order);
        
        // Assert
        mockEmail.Verify(e => e.SendConfirmation(order), Times.Once);
    }
}
```

---

## Examples

### Example 1 — Basic registration and injection

```csharp
// Define interface and implementation
public interface IWeatherService
{
    Task<WeatherForecast> GetForecastAsync(string city);
}

public class WeatherService : IWeatherService
{
    private readonly HttpClient _httpClient;
    private readonly ILogger<WeatherService> _logger;
    
    public WeatherService(HttpClient httpClient, ILogger<WeatherService> logger)
    {
        _httpClient = httpClient;
        _logger = logger;
    }
    
    public async Task<WeatherForecast> GetForecastAsync(string city)
    {
        _logger.LogInformation("Fetching weather for {City}", city);
        // Implementation...
        return new WeatherForecast();
    }
}

// Register in Program.cs
builder.Services.AddHttpClient<IWeatherService, WeatherService>();

// Use in controller
[ApiController]
[Route("[controller]")]
public class WeatherController : ControllerBase
{
    private readonly IWeatherService _weatherService;
    
    public WeatherController(IWeatherService weatherService)
    {
        _weatherService = weatherService;
    }
    
    [HttpGet("{city}")]
    public async Task<WeatherForecast> Get(string city)
    {
        return await _weatherService.GetForecastAsync(city);
    }
}
```

---

### Example 2 — Service lifetimes in action

```csharp
public interface IOperationService
{
    Guid OperationId { get; }
}

public class OperationService : IOperationService
{
    public Guid OperationId { get; } = Guid.NewGuid();
}

// Register with different lifetimes
builder.Services.AddTransient<IOperationService>(sp => 
    new OperationService());  // New GUID each time

builder.Services.AddScoped<IOperationService>(sp => 
    new OperationService());  // Same GUID per request

builder.Services.AddSingleton<IOperationService>(sp => 
    new OperationService());  // Same GUID for entire app

// Test in controller
[HttpGet]
public IActionResult Get(
    [FromServices] IOperationService op1,
    [FromServices] IOperationService op2)
{
    // Transient: op1.OperationId != op2.OperationId
    // Scoped: op1.OperationId == op2.OperationId (same request)
    // Singleton: Always same GUID across all requests
    
    return Ok(new { op1.OperationId, OpId2 = op2.OperationId });
}
```

---

### Example 3 — Factory pattern with DI

```csharp
// Service interface
public interface INotificationService
{
    Task SendAsync(string message);
}

public class EmailNotificationService : INotificationService
{
    public Task SendAsync(string message) => Task.CompletedTask;
}

public class SmsNotificationService : INotificationService
{
    public Task SendAsync(string message) => Task.CompletedTask;
}

// Factory
public interface INotificationServiceFactory
{
    INotificationService Create(NotificationType type);
}

public class NotificationServiceFactory : INotificationServiceFactory
{
    private readonly IServiceProvider _serviceProvider;
    
    public NotificationServiceFactory(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }
    
    public INotificationService Create(NotificationType type)
    {
        return type switch
        {
            NotificationType.Email => _serviceProvider.GetRequiredService<EmailNotificationService>(),
            NotificationType.Sms => _serviceProvider.GetRequiredService<SmsNotificationService>(),
            _ => throw new ArgumentException("Invalid notification type")
        };
    }
}

// Registration
builder.Services.AddTransient<EmailNotificationService>();
builder.Services.AddTransient<SmsNotificationService>();
builder.Services.AddSingleton<INotificationServiceFactory, NotificationServiceFactory>();
```

---

### Example 4 — Options pattern

```csharp
// Configuration class
public class SmtpSettings
{
    public string Host { get; set; }
    public int Port { get; set; }
    public string Username { get; set; }
    public string Password { get; set; }
}

// appsettings.json
{
  "SmtpSettings": {
    "Host": "smtp.example.com",
    "Port": 587,
    "Username": "user@example.com",
    "Password": "password"
  }
}

// Register options
builder.Services.Configure<SmtpSettings>(
    builder.Configuration.GetSection("SmtpSettings"));

// Inject options
public class EmailService
{
    private readonly SmtpSettings _settings;
    
    public EmailService(IOptions<SmtpSettings> options)
    {
        _settings = options.Value;
    }
    
    public void SendEmail()
    {
        // Use _settings.Host, _settings.Port, etc.
    }
}
```

---

### Example 5 — Resolving services manually

```csharp
// In Program.cs after app.Build()
using (var scope = app.Services.CreateScope())
{
    var services = scope.ServiceProvider;
    
    try
    {
        // Resolve and use service
        var dbContext = services.GetRequiredService<AppDbContext>();
        await SeedData.InitializeAsync(dbContext);
    }
    catch (Exception ex)
    {
        var logger = services.GetRequiredService<ILogger<Program>>();
        logger.LogError(ex, "An error occurred seeding the DB.");
    }
}

// Or in a service
public class SomeService
{
    private readonly IServiceProvider _serviceProvider;
    
    public SomeService(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }
    
    public void DoWork()
    {
        using var scope = _serviceProvider.CreateScope();
        var scopedService = scope.ServiceProvider
            .GetRequiredService<IScopedService>();
        
        scopedService.Execute();
    }
}
```

---

## Common Pitfalls

### ❌ Pitfall 1: Captive dependency (Singleton captures Scoped)

```csharp
// ❌ BAD - Singleton captures Scoped DbContext
public class CachingService // Registered as Singleton
{
    private readonly AppDbContext _dbContext; // Scoped!
    
    public CachingService(AppDbContext dbContext)
    {
        _dbContext = dbContext; // DbContext lives forever!
    }
}

// ✅ Solution: Inject IServiceProvider and create scopes
public class CachingService
{
    private readonly IServiceProvider _serviceProvider;
    
    public CachingService(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }
    
    public async Task<Data> GetDataAsync()
    {
        using var scope = _serviceProvider.CreateScope();
        var dbContext = scope.ServiceProvider.GetRequiredService<AppDbContext>();
        return await dbContext.Data.ToListAsync();
    }
}
```

---

### ❌ Pitfall 2: Registering concrete types

```csharp
// ❌ Tight coupling
builder.Services.AddScoped<EmailService>();

// ✅ Use interface
builder.Services.AddScoped<IEmailService, EmailService>();
```

---

### ❌ Pitfall 3: Service locator anti-pattern

```csharp
// ❌ Service locator - hard to test, hides dependencies
public class OrderService
{
    public void ProcessOrder(Order order)
    {
        var emailService = ServiceLocator.Get<IEmailService>(); // ❌
        emailService.Send(order);
    }
}

// ✅ Constructor injection
public class OrderService
{
    private readonly IEmailService _emailService;
    
    public OrderService(IEmailService emailService)
    {
        _emailService = emailService;
    }
}
```

---

### ❌ Pitfall 4: Circular dependencies

```csharp
// ❌ Circular dependency
public class ServiceA
{
    public ServiceA(ServiceB serviceB) { }
}

public class ServiceB
{
    public ServiceB(ServiceA serviceA) { } // ❌ Circular!
}

// ✅ Solution: Extract interface or rethink design
public interface IServiceA { }

public class ServiceA : IServiceA
{
    public ServiceA(IServiceB serviceB) { }
}

public interface IServiceB { }

public class ServiceB : IServiceB
{
    public ServiceB(IServiceA serviceA) { } // Now works
}
```

---

### ❌ Pitfall 5: Disposing scoped services too early

```csharp
// ❌ Don't manually dispose services injected by DI
public class MyService
{
    public MyService(AppDbContext dbContext)
    {
        dbContext.Dispose(); // ❌ Container manages lifecycle!
    }
}

// ✅ Let DI container handle disposal
public class MyService
{
    private readonly AppDbContext _dbContext;
    
    public MyService(AppDbContext dbContext)
    {
        _dbContext = dbContext; // Container disposes at end of scope
    }
}
```

---

## Quick Self-Check

1. What's the difference between Transient, Scoped, and Singleton?
2. Why is constructor injection preferred?
3. What is a captive dependency?
4. When should you inject `IServiceProvider`?
5. How does DI improve testability?

<details>
<summary>Answers</summary>

1. Transient = new instance always; Scoped = one per request/scope; Singleton = one for entire app
2. Makes dependencies explicit, ensures valid state, easier to test, compile-time safety
3. When a longer-lived service (Singleton) captures a shorter-lived dependency (Scoped), keeping it alive inappropriately
4. When you need to resolve services at runtime or create scopes manually (use sparingly)
5. Allows injecting mock/fake implementations for testing without modifying production code

</details>

---

## Further Practice

1. **Refactor** a tightly-coupled class to use DI
2. **Create** a custom health check service with Scoped lifetime
3. **Reproduce** the captive dependency problem and fix it
4. **Implement** a decorator pattern using DI
5. **Write tests** for a service using mocked dependencies

---

## Key Takeaways

✅ DI inverts control—container creates and injects dependencies  
✅ Use Transient for stateless, Scoped for per-request, Singleton for shared  
✅ Constructor injection makes dependencies explicit and testable  
✅ Avoid captive dependencies (Singleton → Scoped)  
✅ Use interfaces to enable loose coupling and testability  
✅ Service locator is an anti-pattern—inject instead  
✅ Options pattern for configuration injection
