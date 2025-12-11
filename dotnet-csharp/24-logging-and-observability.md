# Logging and Observability

🚦 **Intermediate** | ⏱ 50-60 min

---

## 📝 Why This Matters

Logging and observability are critical for operating production systems. When something goes wrong at 3 AM, logs are your primary tool for understanding what happened. When performance degrades, metrics show you where the bottleneck is. When a request fails across microservices, distributed tracing reveals which service broke. Without proper logging and observability, you're flying blind.

In real-world applications, logging serves multiple purposes: debugging during development, troubleshooting production issues, auditing user actions, monitoring system health, and analyzing usage patterns. The difference between good and poor logging is the difference between quickly identifying "Payment service failed because the database connection timed out after the database server ran out of memory at 2:47 AM" versus "Something went wrong."

Modern observability goes beyond simple logging. It encompasses three pillars: logs (discrete events), metrics (aggregated measurements), and traces (request flows). Together, these give you complete visibility into system behavior. Metrics show CPU spiked to 95%, logs show which requests failed, and traces show the slow database query causing the spike.

From a practical perspective, structured logging (logging objects instead of strings) enables powerful querying in log aggregation systems like Application Insights, Splunk, or ELK. Instead of parsing log strings with regex, you query: `level=Error AND component=PaymentService AND duration>5000`. This transforms logs from text files to queryable data.

Performance matters too. Excessive logging impacts throughput and response times. Synchronous logging blocks threads. Logging sensitive data creates security and compliance risks. Understanding logging best practices—async logging, appropriate log levels, filtering, and sampling—prevents these issues.

In interviews, logging questions test production-readiness: "How do you debug production issues?", "What logging levels do you use?", "How do you implement correlation IDs?", "What's structured logging?" Being able to discuss logging strategies, observability tools, and real-world debugging experiences shows you've operated production systems.

ASP.NET Core's `ILogger` abstraction is powerful because it separates logging from log providers. Your code logs to `ILogger`, and you configure whether logs go to console, file, Application Insights, Seq, or multiple destinations. This flexibility lets you adapt logging without changing application code.

Correlation IDs are essential in distributed systems. A single user request might touch 5 microservices—how do you find all logs for that request? Correlation IDs thread through all services, letting you trace the entire request flow. Without them, distributed debugging is nearly impossible.

From a cost perspective, logs are expensive to store and analyze at scale. A high-traffic system generates terabytes of logs monthly. Understanding log sampling, retention policies, and choosing what to log versus what to capture in metrics directly impacts operational costs.

Modern Application Performance Monitoring (APM) tools like Application Insights, New Relic, or Datadog provide real-time monitoring, alerting, and diagnostics. Understanding these tools and how to instrument code for them is increasingly expected for production applications.

---

## 🎯 Core Ideas

### 1. **ILogger Basics**

**Using ILogger in ASP.NET Core:**

```csharp
public class OrderController : ControllerBase
{
    private readonly ILogger<OrderController> _logger;
    
    public OrderController(ILogger<OrderController> logger)
    {
        _logger = logger;
    }
    
    [HttpPost]
    public async Task<IActionResult> CreateOrder(Order order)
    {
        _logger.LogInformation("Creating order for customer {CustomerId}", order.CustomerId);
        
        try
        {
            var result = await _orderService.CreateAsync(order);
            _logger.LogInformation(
                "Order {OrderId} created successfully", 
                result.OrderId);
            return Ok(result);
        }
        catch (Exception ex)
        {
            _logger.LogError(
                ex, 
                "Failed to create order for customer {CustomerId}", 
                order.CustomerId);
            return StatusCode(500);
        }
    }
}
```

**Log levels (least to most severe):**

```csharp
// Trace: Very detailed, typically only in development
_logger.LogTrace("Entering method GetUser with id {UserId}", userId);

// Debug: Diagnostic information useful for debugging
_logger.LogDebug("Cache hit for key {CacheKey}", key);

// Information: General flow of the application
_logger.LogInformation("User {UserId} logged in successfully", userId);

// Warning: Unexpected but recoverable issues
_logger.LogWarning("API rate limit approaching: {CurrentCount}/{MaxCount}", 95, 100);

// Error: Application errors that don't stop execution
_logger.LogError(ex, "Failed to process payment for order {OrderId}", orderId);

// Critical: Severe errors requiring immediate attention
_logger.LogCritical(ex, "Database connection lost, application cannot function");
```

**Structured logging with properties:**

```csharp
// ❌ BAD: String interpolation - can't query efficiently
_logger.LogInformation($"User {userId} ordered product {productId}");

// ✅ GOOD: Structured properties - queryable
_logger.LogInformation(
    "User {UserId} ordered product {ProductId}", 
    userId, 
    productId);

// In Application Insights, you can query:
// traces | where customDimensions.UserId == "123"
// traces | where customDimensions.ProductId == "ABC"

// Multiple properties
_logger.LogInformation(
    "Order placed: OrderId={OrderId}, CustomerId={CustomerId}, Total={Total:C}, Items={ItemCount}",
    order.Id,
    order.CustomerId,
    order.Total,
    order.Items.Count);
```

### 2. **Configuring Logging**

**appsettings.json configuration:**

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information",
      "Microsoft.EntityFrameworkCore": "Warning",
      "MyApp": "Debug"
    },
    "Console": {
      "LogLevel": {
        "Default": "Debug"
      },
      "FormatterName": "json",
      "FormatterOptions": {
        "SingleLine": true,
        "IncludeScopes": true,
        "TimestampFormat": "yyyy-MM-dd HH:mm:ss ",
        "UseUtcTimestamp": true
      }
    }
  }
}
```

**Programmatic configuration:**

```csharp
var builder = WebApplication.CreateBuilder(args);

// Clear default providers
builder.Logging.ClearProviders();

// Add specific providers
builder.Logging.AddConsole();
builder.Logging.AddDebug();
builder.Logging.AddEventSourceLogger();

// Add Application Insights
builder.Services.AddApplicationInsightsTelemetry();

// Set minimum levels
builder.Logging.SetMinimumLevel(LogLevel.Warning);

// Filter specific categories
builder.Logging.AddFilter("Microsoft.EntityFrameworkCore", LogLevel.Warning);
builder.Logging.AddFilter("MyApp.Services", LogLevel.Debug);

var app = builder.Build();
```

**Environment-specific configuration:**

```csharp
// appsettings.Development.json
{
  "Logging": {
    "LogLevel": {
      "Default": "Debug"  // Verbose in development
    }
  }
}

// appsettings.Production.json
{
  "Logging": {
    "LogLevel": {
      "Default": "Warning"  // Only warnings+ in production
    }
  }
}
```

### 3. **Log Scopes for Context**

**Adding context with scopes:**

```csharp
public class OrderService
{
    private readonly ILogger<OrderService> _logger;
    
    public OrderService(ILogger<OrderService> logger)
    {
        _logger = logger;
    }
    
    public async Task ProcessOrderAsync(Order order)
    {
        // Create scope with context that applies to all logs within
        using (_logger.BeginScope("OrderId={OrderId} CustomerId={CustomerId}", 
            order.Id, order.CustomerId))
        {
            _logger.LogInformation("Processing order");
            
            await ValidateOrder(order);
            // Logs: Processing order [OrderId=123, CustomerId=456]
            
            await ChargePayment(order);
            // Logs include OrderId and CustomerId automatically
            
            await ShipOrder(order);
            // All logs within scope include the context
            
            _logger.LogInformation("Order processed successfully");
        }
    }
    
    private async Task ValidateOrder(Order order)
    {
        _logger.LogDebug("Validating order");
        // Inherits scope: OrderId and CustomerId included
    }
}
```

**Nested scopes:**

```csharp
public async Task ProcessBatchAsync(List<Order> orders)
{
    using (_logger.BeginScope("BatchId={BatchId}", Guid.NewGuid()))
    {
        _logger.LogInformation("Processing batch of {Count} orders", orders.Count);
        
        foreach (var order in orders)
        {
            using (_logger.BeginScope("OrderId={OrderId}", order.Id))
            {
                _logger.LogInformation("Processing order");
                // Logs include both BatchId and OrderId
                
                await ProcessOrderAsync(order);
            }
        }
        
        _logger.LogInformation("Batch processing complete");
    }
}
```

### 4. **Correlation IDs for Distributed Tracing**

**Middleware to add correlation ID:**

```csharp
public class CorrelationIdMiddleware
{
    private readonly RequestDelegate _next;
    private const string CorrelationIdHeader = "X-Correlation-Id";
    
    public CorrelationIdMiddleware(RequestDelegate next)
    {
        _next = next;
    }
    
    public async Task InvokeAsync(HttpContext context, ILogger<CorrelationIdMiddleware> logger)
    {
        // Get or generate correlation ID
        var correlationId = context.Request.Headers[CorrelationIdHeader].FirstOrDefault()
                          ?? Guid.NewGuid().ToString();
        
        // Add to response headers
        context.Response.Headers[CorrelationIdHeader] = correlationId;
        
        // Add to HttpContext for access throughout request
        context.Items["CorrelationId"] = correlationId;
        
        // Log with correlation ID using scope
        using (logger.BeginScope("CorrelationId={CorrelationId}", correlationId))
        {
            logger.LogInformation(
                "Request started: {Method} {Path}",
                context.Request.Method,
                context.Request.Path);
            
            try
            {
                await _next(context);
                
                logger.LogInformation(
                    "Request completed: {StatusCode}",
                    context.Response.StatusCode);
            }
            catch (Exception ex)
            {
                logger.LogError(ex, "Request failed");
                throw;
            }
        }
    }
}

// Registration
app.UseMiddleware<CorrelationIdMiddleware>();
```

**Propagating correlation ID to downstream services:**

```csharp
public class OrderService
{
    private readonly HttpClient _httpClient;
    private readonly IHttpContextAccessor _httpContextAccessor;
    
    public OrderService(HttpClient httpClient, IHttpContextAccessor httpContextAccessor)
    {
        _httpClient = httpClient;
        _httpContextAccessor = httpContextAccessor;
    }
    
    public async Task<Inventory> CheckInventoryAsync(string productId)
    {
        // Get correlation ID from current request
        var correlationId = _httpContextAccessor.HttpContext?.Items["CorrelationId"]?.ToString();
        
        // Create request with correlation ID
        var request = new HttpRequestMessage(HttpMethod.Get, $"/api/inventory/{productId}");
        if (!string.IsNullOrEmpty(correlationId))
        {
            request.Headers.Add("X-Correlation-Id", correlationId);
        }
        
        var response = await _httpClient.SendAsync(request);
        response.EnsureSuccessStatusCode();
        
        return await response.Content.ReadFromJsonAsync<Inventory>();
    }
}
```

### 5. **Application Insights Integration**

**Setting up Application Insights:**

```csharp
// Install: Microsoft.ApplicationInsights.AspNetCore

var builder = WebApplication.CreateBuilder(args);

// Add Application Insights
builder.Services.AddApplicationInsightsTelemetry(options =>
{
    options.ConnectionString = builder.Configuration["ApplicationInsights:ConnectionString"];
    options.EnableAdaptiveSampling = true;  // Sample logs intelligently
    options.EnableDependencyTrackingTelemetryModule = true;  // Track HTTP calls, SQL, etc.
});

// Configure telemetry
builder.Services.Configure<TelemetryConfiguration>(config =>
{
    config.TelemetryInitializers.Add(new CustomTelemetryInitializer());
});

var app = builder.Build();
```

**Custom telemetry:**

```csharp
public class OrderService
{
    private readonly TelemetryClient _telemetryClient;
    private readonly ILogger<OrderService> _logger;
    
    public OrderService(TelemetryClient telemetryClient, ILogger<OrderService> logger)
    {
        _telemetryClient = telemetryClient;
        _logger = logger;
    }
    
    public async Task<Order> CreateOrderAsync(Order order)
    {
        var stopwatch = Stopwatch.StartNew();
        
        try
        {
            var result = await ProcessOrderAsync(order);
            
            // Track custom event
            _telemetryClient.TrackEvent("OrderCreated", new Dictionary<string, string>
            {
                ["OrderId"] = result.Id.ToString(),
                ["CustomerId"] = order.CustomerId.ToString(),
                ["TotalAmount"] = order.Total.ToString("C")
            });
            
            // Track custom metric
            _telemetryClient.TrackMetric("OrderValue", order.Total);
            
            return result;
        }
        catch (Exception ex)
        {
            // Track exception
            _telemetryClient.TrackException(ex, new Dictionary<string, string>
            {
                ["CustomerId"] = order.CustomerId.ToString(),
                ["Operation"] = "CreateOrder"
            });
            
            throw;
        }
        finally
        {
            stopwatch.Stop();
            
            // Track request duration
            _telemetryClient.TrackDependency(
                "OrderProcessing",
                "CreateOrder",
                startTime: DateTimeOffset.UtcNow - stopwatch.Elapsed,
                duration: stopwatch.Elapsed,
                success: true);
        }
    }
}
```

**Querying Application Insights (KQL):**

```kusto
// All errors in last 24 hours
traces
| where timestamp > ago(24h)
| where severityLevel >= 3  // Error or Critical
| project timestamp, message, severityLevel, customDimensions
| order by timestamp desc

// Requests by customer
traces
| where timestamp > ago(1h)
| where message contains "OrderCreated"
| extend CustomerId = tostring(customDimensions.CustomerId)
| summarize Count=count() by CustomerId
| order by Count desc

// Slow requests
requests
| where timestamp > ago(1h)
| where duration > 5000  // > 5 seconds
| project timestamp, name, duration, resultCode
| order by duration desc

// Error rate
requests
| where timestamp > ago(24h)
| summarize Total=count(), Errors=countif(success == false) by bin(timestamp, 1h)
| extend ErrorRate = round(100.0 * Errors / Total, 2)
| project timestamp, Total, Errors, ErrorRate
```

### 6. **Structured Logging with Serilog**

**Setting up Serilog:**

```csharp
// Install: Serilog.AspNetCore, Serilog.Sinks.Console, Serilog.Sinks.File

using Serilog;

var builder = WebApplication.CreateBuilder(args);

// Configure Serilog
Log.Logger = new LoggerConfiguration()
    .ReadFrom.Configuration(builder.Configuration)
    .Enrich.FromLogContext()
    .Enrich.WithMachineName()
    .Enrich.WithThreadId()
    .WriteTo.Console()
    .WriteTo.File(
        path: "logs/app-.log",
        rollingInterval: RollingInterval.Day,
        retainedFileCountLimit: 7,
        outputTemplate: "{Timestamp:yyyy-MM-dd HH:mm:ss} [{Level:u3}] {Message:lj}{NewLine}{Exception}")
    .WriteTo.Seq("http://localhost:5341")  // Structured log server
    .CreateLogger();

// Use Serilog for ASP.NET Core
builder.Host.UseSerilog();

try
{
    var app = builder.Build();
    
    // Request logging middleware
    app.UseSerilogRequestLogging(options =>
    {
        options.EnrichDiagnosticContext = (diagnosticContext, httpContext) =>
        {
            diagnosticContext.Set("UserAgent", httpContext.Request.Headers["User-Agent"]);
            diagnosticContext.Set("ClientIp", httpContext.Connection.RemoteIpAddress);
        };
    });
    
    app.Run();
}
catch (Exception ex)
{
    Log.Fatal(ex, "Application start-up failed");
}
finally
{
    Log.CloseAndFlush();
}
```

**appsettings.json for Serilog:**

```json
{
  "Serilog": {
    "Using": ["Serilog.Sinks.Console", "Serilog.Sinks.File"],
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "Microsoft": "Warning",
        "System": "Warning"
      }
    },
    "WriteTo": [
      {
        "Name": "Console",
        "Args": {
          "outputTemplate": "[{Timestamp:HH:mm:ss} {Level:u3}] {Message:lj}{NewLine}{Exception}"
        }
      },
      {
        "Name": "File",
        "Args": {
          "path": "logs/app-.log",
          "rollingInterval": "Day",
          "retainedFileCountLimit": 7,
          "outputTemplate": "{Timestamp:yyyy-MM-dd HH:mm:ss.fff} [{Level:u3}] {Message:lj}{NewLine}{Exception}"
        }
      }
    ],
    "Enrich": ["FromLogContext", "WithMachineName", "WithThreadId"]
  }
}
```

### 7. **Performance and Best Practices**

**Avoiding expensive logging:**

```csharp
// ❌ BAD: Always executes expensive operation
_logger.LogDebug($"User details: {GetExpensiveUserDetails(userId)}");
// GetExpensiveUserDetails() runs even if Debug logging is disabled!

// ✅ GOOD: Check if log level is enabled first
if (_logger.IsEnabled(LogLevel.Debug))
{
    _logger.LogDebug("User details: {UserDetails}", GetExpensiveUserDetails(userId));
}

// ✅ Even better: Use message templates (deferred evaluation)
_logger.LogDebug("User details: {UserDetails}", () => GetExpensiveUserDetails(userId));
```

**Async logging for performance:**

```csharp
// Most logging providers are async internally
// Just use ILogger normally, it won't block

_logger.LogInformation("Order processed"); // Non-blocking

// For custom file logging, use async I/O
public class AsyncFileLogger
{
    private readonly SemaphoreSlim _lock = new(1, 1);
    
    public async Task LogAsync(string message)
    {
        await _lock.WaitAsync();
        try
        {
            await File.AppendAllTextAsync("log.txt", $"{DateTime.UtcNow}: {message}\n");
        }
        finally
        {
            _lock.Release();
        }
    }
}
```

**Sampling to reduce log volume:**

```csharp
// Application Insights adaptive sampling
builder.Services.AddApplicationInsightsTelemetry(options =>
{
    options.EnableAdaptiveSampling = true;  // Samples automatically
});

// Or configure sampling explicitly
builder.Services.Configure<TelemetryConfiguration>(config =>
{
    var builder = config.DefaultTelemetrySink.TelemetryProcessorChainBuilder;
    builder.UseSampling(samplingPercentage: 10.0); // 10% of logs
    builder.Build();
});

// Custom sampling logic
public class SamplingLogger
{
    private readonly ILogger _logger;
    private int _counter = 0;
    
    public void LogSampled(string message)
    {
        // Log every 100th call
        if (Interlocked.Increment(ref _counter) % 100 == 0)
        {
            _logger.LogInformation(message);
        }
    }
}
```

**Filtering sensitive data:**

```csharp
// ❌ NEVER log sensitive data
_logger.LogInformation("User {Email} logged in with password {Password}", 
    email, password);  // ❌ Password in logs!

// ✅ GOOD: Filter sensitive fields
public class Order
{
    public string CustomerId { get; set; }
    
    [SensitiveData]  // Custom attribute
    public string CreditCardNumber { get; set; }
}

public static class LoggingExtensions
{
    public static void LogOrderPlaced(this ILogger logger, Order order)
    {
        // Mask sensitive fields before logging
        logger.LogInformation(
            "Order placed: CustomerId={CustomerId}, CardNumber={CardNumber}",
            order.CustomerId,
            MaskCardNumber(order.CreditCardNumber));
    }
    
    private static string MaskCardNumber(string cardNumber)
    {
        if (string.IsNullOrEmpty(cardNumber) || cardNumber.Length < 4)
            return "****";
        
        return "****-****-****-" + cardNumber.Substring(cardNumber.Length - 4);
    }
}
```

---

## ⚠️ Common Pitfalls

### 1. **String Interpolation Instead of Structured Logging**

```csharp
// ❌ BAD: Can't query properties
_logger.LogInformation($"User {userId} placed order {orderId}");

// ✅ GOOD: Structured properties
_logger.LogInformation("User {UserId} placed order {OrderId}", userId, orderId);
// Queryable: customDimensions.UserId == "123"
```

### 2. **Logging Sensitive Data**

```csharp
// ❌ Security risk
_logger.LogInformation("Login attempt: {Email} / {Password}", email, password);

// ✅ Don't log passwords, tokens, credit cards
_logger.LogInformation("Login attempt: {Email}", email);
```

### 3. **Expensive Operations in Log Statements**

```csharp
// ❌ Executes even when logging disabled
_logger.LogDebug($"Data: {JsonSerializer.Serialize(largeObject)}");

// ✅ Check if enabled first
if (_logger.IsEnabled(LogLevel.Debug))
{
    _logger.LogDebug("Data: {Data}", JsonSerializer.Serialize(largeObject));
}
```

### 4. **Not Using Log Scopes**

```csharp
// ❌ Repetitive, error-prone
_logger.LogInformation("OrderId={OrderId}: Processing started", orderId);
_logger.LogInformation("OrderId={OrderId}: Validating", orderId);
_logger.LogInformation("OrderId={OrderId}: Complete", orderId);

// ✅ Use scope for context
using (_logger.BeginScope("OrderId={OrderId}", orderId))
{
    _logger.LogInformation("Processing started");
    _logger.LogInformation("Validating");
    _logger.LogInformation("Complete");
}
```

### 5. **Wrong Log Levels**

```csharp
// ❌ Everything as Information
_logger.LogInformation("Cache hit");  // Should be Debug
_logger.LogInformation("User not found");  // Should be Warning
_logger.LogInformation("Database connection failed");  // Should be Error!

// ✅ Use appropriate levels
_logger.LogDebug("Cache hit");
_logger.LogWarning("User {UserId} not found", userId);
_logger.LogError(ex, "Database connection failed");
```

### 6. **Not Propagating Correlation IDs**

```csharp
// ❌ Loses trace context across services
var response = await _httpClient.GetAsync("/api/orders");

// ✅ Propagate correlation ID
var correlationId = _httpContext.Items["CorrelationId"]?.ToString();
var request = new HttpRequestMessage(HttpMethod.Get, "/api/orders");
request.Headers.Add("X-Correlation-Id", correlationId);
var response = await _httpClient.SendAsync(request);
```

---

## ✅ Self-Check Questions

1. **What's the difference between structured logging and string interpolation?**
   - Structured: properties are queryable, efficient filtering
   - String interpolation: everything is a string, hard to query
   - Structured logging enables powerful analysis in log systems

2. **When should you use each log level?**
   - Trace/Debug: detailed diagnostics, development only
   - Information: normal flow, important events
   - Warning: unexpected but recoverable issues
   - Error: failures that don't stop the app
   - Critical: severe failures requiring immediate action

3. **What is a correlation ID and why is it important?**
   - Unique identifier for a request across all services
   - Enables tracing a single user request through distributed systems
   - Essential for debugging microservices

4. **How do log scopes help?**
   - Add context to all logs within a scope
   - Avoid repeating the same properties
   - Nested scopes accumulate context

5. **What sensitive data should never be logged?**
   - Passwords, tokens, API keys
   - Credit card numbers, SSN, PII
   - Anything that violates security/compliance

---

## 🏋️ Practice

1. **Implement a logging middleware** that logs all requests with correlation IDs, duration, status codes, and captures errors.

2. **Create a custom TelemetryInitializer** for Application Insights that adds user ID, tenant ID, and environment to all telemetry.

3. **Build a performance monitoring system** that tracks custom metrics (order count, revenue, processing time) and sends them to Application Insights.

4. **Implement structured logging** for a service, ensuring all logs are queryable, correlation IDs propagate to downstream services, and sensitive data is masked.

5. **Create a log analysis tool** that parses JSON logs, extracts errors, calculates response time percentiles, and identifies slow requests.

---

## 🎤 Interview Cheat Sheet

**Quick answers for common questions:**

**"How do you implement logging in ASP.NET Core?"**
> "I inject `ILogger<T>` into classes via dependency injection. ASP.NET Core provides logging out of the box—I just configure providers in Program.cs. I use structured logging with named properties like `_logger.LogInformation('User {UserId} created order {OrderId}', userId, orderId)` so logs are queryable in systems like Application Insights. I set appropriate log levels and filter noisy namespaces like EF Core to Warning."

**"What's the difference between log levels?"**
> "Trace and Debug are for detailed diagnostics, typically disabled in production. Information is for normal application flow like 'User logged in'. Warning is for unexpected but recoverable issues like 'API rate limit approaching'. Error is for failures that don't stop the app like 'Failed to send email'. Critical is for severe failures like 'Database unreachable'. In production, I typically set minimum level to Warning to reduce noise and costs."

**"How do you handle correlation IDs?"**
> "I use middleware to extract or generate a correlation ID from the X-Correlation-Id header, store it in HttpContext.Items, and include it in all logs using log scopes. When calling downstream services, I propagate the correlation ID in request headers. This lets me trace a single user request across all microservices by querying logs for that correlation ID. It's essential for debugging distributed systems."

**"What is structured logging?"**
> "Structured logging treats logs as data, not just text. Instead of string interpolation like `$'User {userId} logged in'`, I use message templates: `_logger.LogInformation('User {UserId} logged in', userId)`. This preserves userId as a queryable property in log systems like Application Insights. I can then query `customDimensions.UserId == '123'` instead of parsing strings with regex. It enables powerful filtering and analysis."

**"How do you prevent logging sensitive data?"**
> "I never log passwords, tokens, credit cards, or PII. For objects that might contain sensitive data, I create safe logging methods that mask or exclude those fields. For example, when logging credit cards, I show only the last 4 digits: '****-****-****-1234'. I also review logs regularly and educate the team on what's safe to log. Some companies use automated tools to scan logs for sensitive patterns."

**"How do you troubleshoot production issues?"**
> "I start with centralized logs in Application Insights or similar. I search for errors in the timeframe users reported issues, filter by correlation ID or user ID, and examine the full request flow. I look at custom metrics to see if something spiked—requests, errors, durations. Distributed tracing shows which service failed in a microservice chain. If logs are insufficient, I add more detailed logging and deploy. Good observability is crucial—I can't fix what I can't see."

**Key phrases that impress:**
- "Structured logging with queryable properties"
- "Correlation IDs for distributed tracing"
- "Log scopes to add context"
- "Appropriate log levels to reduce noise"
- "Never log sensitive data—mask or exclude PII"
- "Application Insights for centralized logging and metrics"

---

**TL;DR for interviews:**

**ILogger basics:**
- Inject `ILogger<T>` via DI
- Use structured logging: `LogInformation("User {UserId}", userId)`
- Not string interpolation: `$"User {userId}"`

**Log levels:**
- Trace/Debug: detailed diagnostics
- Information: normal flow
- Warning: unexpected but OK
- Error: failures
- Critical: severe failures

**Structured logging:**
- Named properties: `{UserId}`, `{OrderId}`
- Queryable in log systems
- Better than string parsing

**Correlation IDs:**
- Unique ID per request
- Propagate across services
- Essential for distributed tracing

**Best practices:**
- Use log scopes for context
- Check `IsEnabled()` before expensive operations
- Never log passwords, tokens, PII
- Set appropriate minimum log levels
- Use Application Insights or similar for production

**Common mistakes:**
- String interpolation instead of structured logging
- Logging sensitive data
- Everything at Information level
- Not propagating correlation IDs
- Expensive operations in log statements

Remember: Logs are your eyes in production. Structure them well, protect sensitive data, and always include context.
