# Middleware and HTTP Pipeline

🚦 **Intermediate** | ⏱ 50-60 min

---

## 📝 Why This Matters

The ASP.NET Core middleware pipeline is the foundation of every web request in your application. Understanding how requests flow through middleware, how to create custom middleware, and how the pipeline processes authentication, routing, and error handling is essential for building robust web applications and APIs.

In real-world applications, middleware handles cross-cutting concerns: logging every request, authenticating users, compressing responses, handling errors, enforcing CORS policies, rate limiting, caching, and more. These concerns affect all endpoints, so middleware provides a centralized place to implement them once rather than repeating code in every controller action.

The middleware pipeline's order matters critically. Authentication must run before authorization. Routing must run before endpoints. Exception handling must run early to catch errors from later middleware. Understanding this execution order prevents bugs like "authorization ran before authentication" or "exception handler didn't catch the error because it ran too late."

From a practical perspective, custom middleware lets you implement powerful features: request/response logging with timing, correlation IDs for tracing requests across services, tenant identification in multi-tenant systems, API key validation, circuit breakers for external services, and response compression. These patterns separate infrastructure concerns from business logic, keeping controllers clean and focused.

Performance also matters. Middleware runs on every request, so inefficient middleware impacts every user. Understanding short-circuiting (terminating the pipeline early) lets you handle errors or cached responses without executing expensive downstream middleware. Async middleware prevents blocking threads during I/O operations.

In interviews, middleware questions test architectural understanding: "How does ASP.NET Core handle requests?", "What's the order of middleware execution?", "How do you implement cross-cutting concerns?", "When would you create custom middleware?" Being able to explain the pipeline, draw it on a whiteboard, and discuss real-world middleware scenarios shows you understand how web frameworks work under the hood.

Middleware also connects to other concepts. The authentication middleware integrates with identity providers (OAuth, JWT). The routing middleware connects to endpoint routing and MVC controllers. The static files middleware serves files from disk. Understanding these integrations shows holistic knowledge of the framework.

Modern ASP.NET Core has evolved significantly. The minimal API pattern in .NET 6+ simplifies middleware configuration. Endpoint routing (introduced in 2.2, refined in 3.0+) separates routing from endpoint execution. Understanding these improvements and when to use minimal APIs vs traditional MVC shows you're current with the framework.

From a debugging perspective, knowing the middleware pipeline helps troubleshoot issues: "Why is my auth not working?" (check middleware order), "Why aren't my routes matching?" (check routing middleware placement), "Why is my exception handler not catching errors?" (check if it's registered early enough). The pipeline model makes request flow explicit and debuggable.

---

## 🎯 Core Ideas

### 1. **The Middleware Pipeline Concept**

**Understanding the pipeline:**

```csharp
// Middleware forms a pipeline where each component can:
// 1. Execute code before calling next middleware
// 2. Call the next middleware
// 3. Execute code after next middleware returns

// Request flows forward through middleware:
// Request → Middleware 1 → Middleware 2 → Middleware 3 → Endpoint

// Response flows backward:
// Endpoint → Middleware 3 → Middleware 2 → Middleware 1 → Response

// Visual representation:
//
//    Request →
//    ┌─────────────────────────────────────────┐
//    │ Middleware 1 (Logging)                  │
//    │   ↓ Before next                         │
//    │   ┌─────────────────────────────────┐   │
//    │   │ Middleware 2 (Authentication)   │   │
//    │   │   ↓ Before next                 │   │
//    │   │   ┌─────────────────────────┐   │   │
//    │   │   │ Middleware 3 (Routing)  │   │   │
//    │   │   │   ↓                     │   │   │
//    │   │   │   Endpoint (Controller) │   │   │
//    │   │   │   ↓                     │   │   │
//    │   │   └─────────────────────────┘   │   │
//    │   │   ↑ After next                  │   │
//    │   └─────────────────────────────────┘   │
//    │   ↑ After next                          │
//    └─────────────────────────────────────────┘
//    ← Response
```

**Basic middleware configuration in Program.cs:**

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// Middleware 1: Exception handling (should be FIRST)
app.UseExceptionHandler("/error");

// Middleware 2: HTTPS redirection
app.UseHttpsRedirection();

// Middleware 3: Static files (short-circuits if file found)
app.UseStaticFiles();

// Middleware 4: Routing (matches routes)
app.UseRouting();

// Middleware 5: Authentication (identifies user)
app.UseAuthentication();

// Middleware 6: Authorization (checks permissions)
app.UseAuthorization();

// Middleware 7: Endpoints (executes matched endpoint)
app.MapControllers();

app.Run();
```

### 2. **Inline Middleware with Use, Run, and Map**

**app.Use: Middleware that calls next:**

```csharp
app.Use(async (context, next) =>
{
    // Code before calling next middleware
    Console.WriteLine($"Request: {context.Request.Method} {context.Request.Path}");
    var startTime = DateTime.UtcNow;
    
    // Call next middleware
    await next(context);
    
    // Code after next middleware completes
    var duration = (DateTime.UtcNow - startTime).TotalMilliseconds;
    Console.WriteLine($"Response: {context.Response.StatusCode} ({duration}ms)");
});

// Order matters!
app.Use(async (context, next) =>
{
    Console.WriteLine("First middleware: before");
    await next(context);
    Console.WriteLine("First middleware: after");
});

app.Use(async (context, next) =>
{
    Console.WriteLine("Second middleware: before");
    await next(context);
    Console.WriteLine("Second middleware: after");
});

// Output for a request:
// First middleware: before
// Second middleware: before
// Second middleware: after
// First middleware: after
```

**app.Run: Terminal middleware (doesn't call next):**

```csharp
app.Run(async context =>
{
    // This is terminal - pipeline ends here
    await context.Response.WriteAsync("Hello, World!");
    // No next middleware will execute
});

// ⚠️ Any middleware after app.Run will never execute!
app.Use(async (context, next) =>
{
    Console.WriteLine("Never executes!");
    await next(context);
});
```

**app.Map: Branch the pipeline based on path:**

```csharp
// Branch for /api path
app.Map("/api", apiApp =>
{
    apiApp.Use(async (context, next) =>
    {
        Console.WriteLine("API middleware");
        await next(context);
    });
    
    apiApp.Run(async context =>
    {
        await context.Response.WriteAsync("API Response");
    });
});

// Branch for /admin path
app.Map("/admin", adminApp =>
{
    adminApp.Use(async (context, next) =>
    {
        // Check admin authentication
        if (!context.User.IsInRole("Admin"))
        {
            context.Response.StatusCode = 403;
            return; // Short-circuit
        }
        await next(context);
    });
    
    adminApp.Run(async context =>
    {
        await context.Response.WriteAsync("Admin Panel");
    });
});

// Default route (not matched by above)
app.Run(async context =>
{
    await context.Response.WriteAsync("Main App");
});
```

**app.MapWhen: Conditional branching:**

```csharp
// Branch for requests with specific query string
app.MapWhen(
    context => context.Request.Query.ContainsKey("debug"),
    debugApp =>
    {
        debugApp.Run(async context =>
        {
            await context.Response.WriteAsync("Debug mode enabled");
        });
    });

// Branch for mobile user agents
app.MapWhen(
    context => context.Request.Headers["User-Agent"].ToString().Contains("Mobile"),
    mobileApp =>
    {
        mobileApp.Run(async context =>
        {
            await context.Response.WriteAsync("Mobile version");
        });
    });
```

### 3. **Custom Middleware Class**

**Creating reusable middleware:**

```csharp
// Middleware class
public class RequestTimingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<RequestTimingMiddleware> _logger;
    
    public RequestTimingMiddleware(RequestDelegate next, ILogger<RequestTimingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        var startTime = DateTime.UtcNow;
        var stopwatch = Stopwatch.StartNew();
        
        try
        {
            // Call next middleware
            await _next(context);
        }
        finally
        {
            stopwatch.Stop();
            _logger.LogInformation(
                "Request {Method} {Path} completed with {StatusCode} in {Duration}ms",
                context.Request.Method,
                context.Request.Path,
                context.Response.StatusCode,
                stopwatch.ElapsedMilliseconds);
        }
    }
}

// Extension method for registration
public static class RequestTimingMiddlewareExtensions
{
    public static IApplicationBuilder UseRequestTiming(this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<RequestTimingMiddleware>();
    }
}

// Usage in Program.cs
app.UseRequestTiming();
```

**Middleware with dependencies:**

```csharp
public class ApiKeyMiddleware
{
    private readonly RequestDelegate _next;
    private readonly IConfiguration _configuration;
    
    public ApiKeyMiddleware(RequestDelegate next, IConfiguration configuration)
    {
        _next = next;
        _configuration = configuration;
    }
    
    public async Task InvokeAsync(HttpContext context, IApiKeyValidator validator)
    {
        // Can inject scoped/transient services in InvokeAsync
        if (!context.Request.Headers.TryGetValue("X-API-Key", out var apiKey))
        {
            context.Response.StatusCode = 401;
            await context.Response.WriteAsync("API Key missing");
            return; // Short-circuit pipeline
        }
        
        if (!await validator.ValidateAsync(apiKey!))
        {
            context.Response.StatusCode = 403;
            await context.Response.WriteAsync("Invalid API Key");
            return;
        }
        
        // API key valid, continue pipeline
        await _next(context);
    }
}

// Registration
services.AddScoped<IApiKeyValidator, ApiKeyValidator>();
app.UseMiddleware<ApiKeyMiddleware>();
```

**Middleware that modifies request/response:**

```csharp
public class RequestHeaderMiddleware
{
    private readonly RequestDelegate _next;
    
    public RequestHeaderMiddleware(RequestDelegate next)
    {
        _next = next;
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        // Add custom header to request
        context.Request.Headers["X-Correlation-Id"] = Guid.NewGuid().ToString();
        
        // Modify response headers (before response starts)
        context.Response.OnStarting(() =>
        {
            context.Response.Headers["X-Content-Type-Options"] = "nosniff";
            context.Response.Headers["X-Frame-Options"] = "DENY";
            context.Response.Headers["X-XSS-Protection"] = "1; mode=block";
            return Task.CompletedTask;
        });
        
        await _next(context);
    }
}
```

### 4. **Common Built-in Middleware**

**Exception handling middleware:**

```csharp
// Development: detailed error page
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
else
{
    // Production: custom error handler
    app.UseExceptionHandler("/error");
    
    // Or inline:
    app.UseExceptionHandler(errorApp =>
    {
        errorApp.Run(async context =>
        {
            context.Response.StatusCode = 500;
            context.Response.ContentType = "application/json";
            
            var error = context.Features.Get<IExceptionHandlerFeature>();
            if (error != null)
            {
                var logger = context.RequestServices.GetRequiredService<ILogger<Program>>();
                logger.LogError(error.Error, "Unhandled exception");
                
                await context.Response.WriteAsJsonAsync(new
                {
                    error = "An error occurred",
                    requestId = context.TraceIdentifier
                });
            }
        });
    });
}

// Status code pages for 404, 403, etc.
app.UseStatusCodePages(async context =>
{
    context.HttpContext.Response.ContentType = "application/json";
    await context.HttpContext.Response.WriteAsJsonAsync(new
    {
        statusCode = context.HttpContext.Response.StatusCode,
        message = $"Status code: {context.HttpContext.Response.StatusCode}"
    });
});
```

**Authentication and authorization:**

```csharp
// Configure services
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = builder.Configuration["Jwt:Issuer"],
            ValidAudience = builder.Configuration["Jwt:Audience"],
            IssuerSigningKey = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Key"]!))
        };
    });

builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("AdminOnly", policy => policy.RequireRole("Admin"));
    options.AddPolicy("Over18", policy => policy.RequireClaim("Age", "18"));
});

// Middleware pipeline (ORDER MATTERS!)
app.UseRouting();
app.UseAuthentication(); // First: identify user
app.UseAuthorization();  // Second: check permissions
app.MapControllers();

// Usage in controller
[ApiController]
[Route("api/[controller]")]
[Authorize] // Requires authentication
public class AccountController : ControllerBase
{
    [HttpGet("profile")]
    public IActionResult GetProfile()
    {
        var userId = User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
        return Ok(new { userId });
    }
    
    [HttpGet("admin")]
    [Authorize(Policy = "AdminOnly")] // Requires admin role
    public IActionResult AdminOnly()
    {
        return Ok("Admin area");
    }
}
```

**CORS middleware:**

```csharp
// Configure CORS
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowSpecificOrigin", policy =>
    {
        policy.WithOrigins("https://example.com")
              .AllowAnyHeader()
              .AllowAnyMethod()
              .AllowCredentials();
    });
    
    options.AddPolicy("AllowAll", policy =>
    {
        policy.AllowAnyOrigin()
              .AllowAnyHeader()
              .AllowAnyMethod();
    });
});

// Use CORS (BEFORE routing, auth, etc.)
app.UseCors("AllowSpecificOrigin");

app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();

// Per-endpoint CORS
app.MapGet("/api/public", () => "Public data")
    .RequireCors("AllowAll");
```

**Response compression:**

```csharp
// Configure compression
builder.Services.AddResponseCompression(options =>
{
    options.EnableForHttps = true;
    options.Providers.Add<GzipCompressionProvider>();
    options.Providers.Add<BrotliCompressionProvider>();
});

builder.Services.Configure<GzipCompressionProviderOptions>(options =>
{
    options.Level = CompressionLevel.Fastest;
});

// Use compression (EARLY in pipeline, before static files)
app.UseResponseCompression();
app.UseStaticFiles();
```

**Static files middleware:**

```csharp
// Serve files from wwwroot
app.UseStaticFiles();

// Serve from custom directory
app.UseStaticFiles(new StaticFileOptions
{
    FileProvider = new PhysicalFileProvider(
        Path.Combine(builder.Environment.ContentRootPath, "MyStaticFiles")),
    RequestPath = "/files"
});
// Access: https://example.com/files/image.jpg → MyStaticFiles/image.jpg

// Configure caching
app.UseStaticFiles(new StaticFileOptions
{
    OnPrepareResponse = ctx =>
    {
        // Cache for 1 year
        ctx.Context.Response.Headers[HeaderNames.CacheControl] = 
            "public,max-age=31536000";
    }
});
```

### 5. **Middleware Order and Short-Circuiting**

**Critical middleware order:**

```csharp
var app = builder.Build();

// 1. Exception handling (FIRST - catches all errors)
app.UseExceptionHandler("/error");

// 2. HTTPS redirection
app.UseHttpsRedirection();

// 3. Response compression (before static files)
app.UseResponseCompression();

// 4. Static files (can short-circuit for static content)
app.UseStaticFiles();

// 5. Routing (matches routes but doesn't execute endpoints yet)
app.UseRouting();

// 6. CORS (after routing, before auth)
app.UseCors();

// 7. Authentication (identifies user)
app.UseAuthentication();

// 8. Authorization (checks permissions based on identity)
app.UseAuthorization();

// 9. Custom middleware (your business logic)
app.UseRequestTiming();

// 10. Endpoints (executes matched endpoint)
app.MapControllers();

app.Run();
```

**Short-circuiting the pipeline:**

```csharp
// Middleware can stop the pipeline by not calling next()
app.Use(async (context, next) =>
{
    // Check cache
    var cachedResponse = GetFromCache(context.Request.Path);
    if (cachedResponse != null)
    {
        // Short-circuit: return cached response without calling next
        context.Response.ContentType = "application/json";
        await context.Response.WriteAsync(cachedResponse);
        return; // Pipeline stops here
    }
    
    // Not cached, continue pipeline
    await next(context);
    
    // After next returns, cache the response
    CacheResponse(context.Request.Path, context.Response);
});

// Static files middleware short-circuits if file found
app.UseStaticFiles(); // If file exists, returns it without calling next

// All requests beyond here didn't match a static file
app.UseRouting();
app.MapControllers();
```

### 6. **Real-World Middleware Patterns**

**Request/Response logging with correlation ID:**

```csharp
public class RequestResponseLoggingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<RequestResponseLoggingMiddleware> _logger;
    
    public RequestResponseLoggingMiddleware(
        RequestDelegate next, 
        ILogger<RequestResponseLoggingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        // Generate correlation ID
        var correlationId = Guid.NewGuid().ToString();
        context.Items["CorrelationId"] = correlationId;
        
        // Log request
        _logger.LogInformation(
            "[{CorrelationId}] Request: {Method} {Path} {QueryString}",
            correlationId,
            context.Request.Method,
            context.Request.Path,
            context.Request.QueryString);
        
        // Capture response body (requires buffering)
        var originalBody = context.Response.Body;
        using var responseBody = new MemoryStream();
        context.Response.Body = responseBody;
        
        var stopwatch = Stopwatch.StartNew();
        
        try
        {
            await _next(context);
        }
        finally
        {
            stopwatch.Stop();
            
            // Log response
            _logger.LogInformation(
                "[{CorrelationId}] Response: {StatusCode} in {Duration}ms",
                correlationId,
                context.Response.StatusCode,
                stopwatch.ElapsedMilliseconds);
            
            // Copy response body back to original stream
            responseBody.Seek(0, SeekOrigin.Begin);
            await responseBody.CopyToAsync(originalBody);
        }
    }
}
```

**Rate limiting middleware:**

```csharp
public class RateLimitingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly IMemoryCache _cache;
    private readonly int _requestLimit = 100;
    private readonly TimeSpan _timeWindow = TimeSpan.FromMinutes(1);
    
    public RateLimitingMiddleware(RequestDelegate next, IMemoryCache cache)
    {
        _next = next;
        _cache = cache;
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        var clientId = GetClientId(context);
        var cacheKey = $"rate_limit_{clientId}";
        
        if (!_cache.TryGetValue(cacheKey, out int requestCount))
        {
            requestCount = 0;
        }
        
        if (requestCount >= _requestLimit)
        {
            context.Response.StatusCode = 429; // Too Many Requests
            await context.Response.WriteAsync("Rate limit exceeded");
            return; // Short-circuit
        }
        
        requestCount++;
        _cache.Set(cacheKey, requestCount, _timeWindow);
        
        context.Response.Headers["X-RateLimit-Limit"] = _requestLimit.ToString();
        context.Response.Headers["X-RateLimit-Remaining"] = 
            (_requestLimit - requestCount).ToString();
        
        await _next(context);
    }
    
    private string GetClientId(HttpContext context)
    {
        // Use API key, user ID, or IP address
        if (context.Request.Headers.TryGetValue("X-API-Key", out var apiKey))
            return apiKey!;
        
        return context.Connection.RemoteIpAddress?.ToString() ?? "unknown";
    }
}
```

**Tenant identification middleware (multi-tenancy):**

```csharp
public class TenantMiddleware
{
    private readonly RequestDelegate _next;
    
    public TenantMiddleware(RequestDelegate next)
    {
        _next = next;
    }
    
    public async Task InvokeAsync(HttpContext context, ITenantProvider tenantProvider)
    {
        // Extract tenant from subdomain
        var host = context.Request.Host.Host;
        var subdomain = host.Split('.')[0];
        
        // Or from header
        if (context.Request.Headers.TryGetValue("X-Tenant-Id", out var tenantId))
        {
            subdomain = tenantId!;
        }
        
        // Or from route
        if (context.Request.RouteValues.TryGetValue("tenant", out var routeTenant))
        {
            subdomain = routeTenant?.ToString()!;
        }
        
        // Set tenant in context
        var tenant = await tenantProvider.GetTenantAsync(subdomain);
        if (tenant == null)
        {
            context.Response.StatusCode = 404;
            await context.Response.WriteAsync("Tenant not found");
            return;
        }
        
        context.Items["Tenant"] = tenant;
        
        await _next(context);
    }
}
```

---

## ⚠️ Common Pitfalls

### 1. **Wrong Middleware Order**

```csharp
// ❌ BAD: Authorization before authentication
app.UseAuthorization();  // Checks permissions first
app.UseAuthentication(); // Then identifies user (too late!)

// ✅ GOOD: Authentication before authorization
app.UseAuthentication(); // First: identify user
app.UseAuthorization();  // Then: check permissions
```

### 2. **Modifying Response After It's Started**

```csharp
// ❌ Can't modify headers after response starts
app.Use(async (context, next) =>
{
    await next(context);
    
    // Response already started by next middleware!
    context.Response.Headers["X-Custom"] = "Value"; // Exception!
});

// ✅ Use OnStarting callback
app.Use(async (context, next) =>
{
    context.Response.OnStarting(() =>
    {
        context.Response.Headers["X-Custom"] = "Value";
        return Task.CompletedTask;
    });
    
    await next(context);
});
```

### 3. **Forgetting to Call next() or Return**

```csharp
// ❌ Neither calls next nor returns - hangs!
app.Use(async (context, next) =>
{
    if (context.Request.Path == "/test")
    {
        await context.Response.WriteAsync("Test");
        // Forgot to return! Continues to next() call below
    }
    await next(context); // Always executes
});

// ✅ Return after writing response
app.Use(async (context, next) =>
{
    if (context.Request.Path == "/test")
    {
        await context.Response.WriteAsync("Test");
        return; // Stop here
    }
    await next(context);
});
```

### 4. **Blocking Middleware**

```csharp
// ❌ Synchronous code blocks thread
app.Use((context, next) =>
{
    Thread.Sleep(1000); // Blocks thread!
    return next(context);
});

// ✅ Use async
app.Use(async (context, next) =>
{
    await Task.Delay(1000); // Releases thread
    await next(context);
});
```

### 5. **Expensive Middleware on Every Request**

```csharp
// ❌ Expensive operation on every request
app.Use(async (context, next) =>
{
    var data = await ExpensiveDatabase.Query(); // Runs for ALL requests!
    await next(context);
});

// ✅ Cache or optimize
private static readonly AsyncLazy<Data> _cachedData = 
    new AsyncLazy<Data>(() => ExpensiveDatabase.Query());

app.Use(async (context, next) =>
{
    var data = await _cachedData.Value; // Computed once
    await next(context);
});
```

### 6. **Not Disposing Resources**

```csharp
// ❌ Resource leak
app.Use(async (context, next) =>
{
    var stream = new MemoryStream();
    // Do work...
    await next(context);
    // Stream never disposed!
});

// ✅ Use using or try/finally
app.Use(async (context, next) =>
{
    using var stream = new MemoryStream();
    // Do work...
    await next(context);
}); // Stream disposed here
```

---

## ✅ Self-Check Questions

1. **What's the correct order: authentication or authorization first?**
   - Authentication first (identify user)
   - Then authorization (check permissions)
   - Auth* must come after routing, before endpoints

2. **When does middleware short-circuit the pipeline?**
   - When it doesn't call `next()`
   - Static files middleware short-circuits if file found
   - Useful for caching, rate limiting, early returns

3. **How do you inject scoped services into middleware?**
   - Constructor: only singleton services
   - `InvokeAsync` parameters: can inject scoped/transient services
   - DI injects per-request services in InvokeAsync

4. **What happens if you modify response headers after response starts?**
   - Throws exception - headers already sent
   - Use `Response.OnStarting()` callback to modify headers
   - Must modify before calling `WriteAsync()`

5. **Why use async middleware?**
   - Releases thread during I/O operations
   - Improves scalability under load
   - Essential for web applications handling many requests

---

## 🏋️ Practice

1. **Build a request logging middleware** that logs method, path, status code, duration, and correlation ID for every request.

2. **Create an API key authentication middleware** that validates API keys from headers, short-circuits on invalid keys, and adds user claims to the context.

3. **Implement a caching middleware** that caches GET responses in memory, serves cached responses on subsequent requests, and invalidates cache on non-GET methods.

4. **Build a rate limiting middleware** that limits requests per IP address, returns 429 status when exceeded, and adds rate limit headers to responses.

5. **Create a tenant resolution middleware** for a multi-tenant app that identifies tenant from subdomain or header, loads tenant config, and stores it in HttpContext.

---

## 🎤 Interview Cheat Sheet

**Quick answers for common questions:**

**"Explain the ASP.NET Core middleware pipeline."**
> "The middleware pipeline processes HTTP requests through a series of components. Each middleware can execute code before and after calling the next middleware in the pipeline. Requests flow forward through middleware, and responses flow backward. Order matters critically—for example, authentication must run before authorization, and exception handling should be first to catch all errors. Middleware can short-circuit by not calling next, which is useful for caching or early returns."

**"What's the difference between app.Use, app.Run, and app.Map?"**
> "`app.Use` adds middleware that calls the next component. `app.Run` is terminal—it doesn't call next, ending the pipeline. `app.Map` branches the pipeline based on request path, creating sub-pipelines for different routes. Use `Use` for typical middleware, `Run` for endpoints, and `Map` for path-based routing of middleware."

**"How do you create custom middleware?"**
> "I create a class with a constructor taking `RequestDelegate next` and any singleton dependencies, plus an `InvokeAsync` method that can inject scoped services. The method executes code before calling `await next(context)`, then code after it returns. I create an extension method like `UseMyMiddleware()` for clean registration. For simple cases, I use inline middleware with `app.Use(async (context, next) => {...})`."

**"Why does middleware order matter?"**
> "Middleware executes in registration order, and each layer can affect what comes next. Authentication must identify the user before authorization checks permissions. Exception handling must be first to catch errors from all middleware. Routing must run before endpoints. Static files should be early to short-circuit quickly. Wrong order causes bugs like authorization failing because the user wasn't authenticated yet."

**"How do you handle errors in middleware?"**
> "I use `UseExceptionHandler` middleware at the top of the pipeline to catch all exceptions. In development, I use `UseDeveloperExceptionPage` for detailed errors. In production, I configure a custom error handler that logs the exception, returns a sanitized error response, and includes a correlation ID. I can also use try-catch within middleware to handle specific errors locally."

**"When would you short-circuit the pipeline?"**
> "When I can fulfill the request without downstream middleware—for example, returning a cached response, rejecting an unauthenticated request, serving a static file, or hitting a rate limit. Short-circuiting improves performance by avoiding unnecessary middleware execution. I short-circuit by not calling `await next(context)` and instead writing the response directly."

**Key phrases that impress:**
- "Middleware pipeline processes requests in registration order"
- "Authentication before authorization, exception handling first"
- "Short-circuiting for caching and early returns"
- "Inject scoped services via InvokeAsync parameters"
- "OnStarting callback for modifying response headers"
- "Async middleware for scalability"

---

**TL;DR for interviews:**

**Pipeline basics:**
- Request flows forward, response flows backward
- Each middleware: before → next() → after
- Order matters critically!

**Critical order:**
1. Exception handling (first)
2. HTTPS redirection
3. Static files
4. Routing
5. CORS
6. Authentication
7. Authorization
8. Custom middleware
9. Endpoints (last)

**Three types:**
- `app.Use`: calls next
- `app.Run`: terminal (no next)
- `app.Map`: branch by path

**Custom middleware:**
- Constructor: `RequestDelegate next`, singleton deps
- `InvokeAsync`: can inject scoped services
- Call `await next(context)` to continue pipeline
- Return without calling next to short-circuit

**Common patterns:**
- Request logging with correlation ID
- API key validation
- Rate limiting
- Response caching
- Tenant identification

**Gotchas:**
- Auth* order: authentication BEFORE authorization
- Can't modify headers after response starts (use OnStarting)
- Must call next() OR return, not neither
- Use async to avoid blocking threads

Remember: Middleware handles cross-cutting concerns. Order matters. Short-circuit when possible.
