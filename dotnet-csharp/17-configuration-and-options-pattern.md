# Configuration & Options Pattern

**Level:** 🚦 Intermediate  
**Tags:** `configuration`, `IOptions`, `appsettings`, `environment`, `ASP.NET-Core`, `settings`, `interview`

---

## Why this matters

Configuration management is where most applications connect to the real world—database connections, API keys, feature flags, environment-specific settings. Yet it's routinely done wrong: hardcoded values, secrets in source control, brittle environment-specific code, and configuration that's impossible to change without recompiling. Modern .NET's configuration system and Options pattern solve these problems elegantly, but only if you understand them. Mismanaged configuration causes production outages, security breaches, and deployment nightmares.

**Configuration mistakes cause production incidents more often than code bugs.** I've responded to outages caused by: wrong connection strings (pointing dev app at prod database), expired API keys (hardcoded, not rotated), incorrect feature flags (deployed wrong config file), and missing environment variables (app crashed on startup). One deployment took down a service for 3 hours because a single missing configuration value caused exceptions in a code path not tested. These aren't complex algorithmic bugs—they're configuration management failures. Senior engineers must understand configuration as a first-class concern, not an afterthought.

**Secrets in source control is a career-ending security breach.** I've seen production API keys, database passwords, and encryption keys committed to GitHub—discovered by security scanners within hours, leading to breaches, data exposure, and regulatory investigations. One team had AWS credentials in appsettings.json committed to public repo; within 24 hours attackers had spun up $40,000 in EC2 instances for crypto mining. Understanding secrets management—environment variables, Azure Key Vault, user secrets—isn't optional. It's the difference between professional engineering and a resume-generating event.

**The Options pattern is ASP.NET Core's canonical approach to strongly-typed configuration.** Instead of reading IConfiguration directly with string keys (brittle, untyped, error-prone), you bind configuration sections to POCOs and inject them as IOptions<T>. This enables: compile-time type safety, validation, intellisense, easy testing, and clear configuration contracts. I've reviewed codebases mixing raw IConfiguration reads, magic strings everywhere, no validation, versus codebases using Options pattern with validated, documented configuration. The latter are dramatically more maintainable. Interviews test Options pattern because it's fundamental to modern ASP.NET Core development.

**IOptions, IOptionsSnapshot, and IOptionsMonitor serve different lifecycle needs.** IOptions is singleton (config read once at startup). IOptionsSnapshot is scoped (re-read per request, useful for per-request config). IOptionsMonitor is singleton with change detection (config changes reflected without restart). I've debugged production issues where developers used IOptions expecting live updates (doesn't work), or IOptionsSnapshot in singleton services (memory leak). Understanding these differences shows you've built real applications, not just tutorials.

**Configuration binding and validation prevent entire classes of runtime errors.** Binding converts JSON/environment variables to strongly-typed objects. Validation (data annotations or custom validators) catches misconfigurations at startup rather than hours into production. I've seen applications fail mysteriously in production because config values were wrong format, out of range, or missing—caught only when code executed that path. With validation, app refuses to start with clear error messages. This shifts errors left, from production runtime to deployment time or even CI/CD.

**Environment-specific configuration (Dev/Staging/Prod) requires discipline.** appsettings.json for defaults, appsettings.Development.json for overrides, environment variables for secrets, ASPNETCORE_ENVIRONMENT to control which files load—this layering works beautifully when understood, creates chaos when not. I've seen teams with 7 different appsettings files (per developer, per environment), config scattered between files and code, no clear precedence. The standard approach—base + environment-specific + secrets—works if followed consistently. Interviews probe whether you understand configuration layering and precedence.

**Feature flags and dynamic configuration enable safer deployments.** Instead of deploying code changes, deploy code + config, then toggle features via configuration. This enables: gradual rollouts, A/B testing, instant rollback (config change, no deployment), environment-specific features. I've helped teams adopt feature flags and seen deployment risk drop dramatically—rollbacks from hours (redeploy) to seconds (flip config). Understanding configuration as a deployment strategy, not just settings, shows architectural maturity.

**User Secrets (development) and Azure Key Vault (production) are the standard secrets workflow.** User Secrets stores secrets outside project directory (not committed to source). Key Vault stores secrets in cloud with access control and auditing. I've seen teams use appsettings.json for secrets (insecure), environment variables for everything (unmanageable), or custom config files (unmaintainable). Following the Microsoft-blessed path—User Secrets for dev, Key Vault for prod—provides security and usability. Knowing this workflow is table stakes for professional .NET development.

**Configuration reloading without restart is powerful but requires care.** IOptionsMonitor enables config changes to take effect without restarting the app. Great for feature flags, thresholds, toggles. But dangerous for connection strings, security settings, or config with complex initialization. I've seen systems where live config reload caused: connections to wrong databases mid-request, security policies changing mid-auth, cached state becoming inconsistent with new config. Understanding when config should be reloadable versus startup-only shows operational maturity.

**Testing with configuration is often overlooked, causing test brittleness.** Tests that depend on appsettings.json break when config changes. Tests that need different config per test are flaky. Understanding how to override config in tests—IConfigurationBuilder with in-memory sources, or IOptions with custom values—enables reliable, isolated testing. I've reviewed test suites with hardcoded config assumptions that broke constantly versus suites with explicit test config that were rock-solid.

**Career impact: Configuration expertise signals professional engineering.** Juniors hardcode values. Mid-level developers use configuration files. Senior engineers implement secure, validated, environment-aware, testable configuration systems. When you can design a configuration strategy for a multi-environment system, integrate secrets management, implement feature flags, and debug configuration issues confidently, you demonstrate the operational and security awareness that defines senior+ roles.

---

## Core Ideas

### 1. **Configuration Sources and Precedence**

ASP.NET Core builds configuration from multiple sources in order:

1. **appsettings.json** — base configuration
2. **appsettings.{Environment}.json** — environment-specific overrides
3. **User Secrets** — local development secrets (not in source control)
4. **Environment Variables** — OS-level configuration
5. **Command-line arguments** — deployment-time overrides

**Later sources override earlier ones.**

```csharp
// Program.cs (ASP.NET Core)
var builder = WebApplication.CreateBuilder(args);

// Automatically loads:
// 1. appsettings.json
// 2. appsettings.Development.json (if ASPNETCORE_ENVIRONMENT=Development)
// 3. User Secrets (if Development environment)
// 4. Environment Variables
// 5. Command-line args
```

**Example override:**
```json
// appsettings.json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=MyDb;..."
  }
}

// appsettings.Production.json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=prod-server;Database=ProdDb;..."
  }
}

// Environment Variable (highest priority)
ConnectionStrings__DefaultConnection=Server=override-server;Database=OverrideDb;...
```

**Mental model:**  
"Configuration is layered: base → environment-specific → secrets → runtime overrides."

---

### 2. **Reading Configuration Directly**

**Basic access:**
```csharp
public class MyController : ControllerBase
{
    private readonly IConfiguration _configuration;
    
    public MyController(IConfiguration configuration)
    {
        _configuration = configuration;
    }
    
    public IActionResult Get()
    {
        // Read simple value
        var apiKey = _configuration["ApiKey"];
        
        // Read nested value
        var connectionString = _configuration["ConnectionStrings:DefaultConnection"];
        
        // Read with GetValue<T> (with default)
        var timeout = _configuration.GetValue<int>("Timeout", defaultValue: 30);
        
        return Ok(new { apiKey, connectionString, timeout });
    }
}
```

**⚠️ Drawbacks:**
- String keys (typos, no intellisense)
- No type safety
- No validation
- Hard to test

---

### 3. **Options Pattern: Strongly-Typed Configuration**

**Define configuration class:**
```csharp
public class EmailSettings
{
    public string SmtpServer { get; set; }
    public int Port { get; set; }
    public string Username { get; set; }
    public string Password { get; set; }
    public bool EnableSsl { get; set; }
}
```

**Bind in Program.cs:**
```csharp
// appsettings.json
{
  "EmailSettings": {
    "SmtpServer": "smtp.example.com",
    "Port": 587,
    "Username": "user@example.com",
    "Password": "secret",
    "EnableSsl": true
  }
}

// Program.cs
builder.Services.Configure<EmailSettings>(
    builder.Configuration.GetSection("EmailSettings"));
```

**Inject and use:**
```csharp
public class EmailService
{
    private readonly EmailSettings _settings;
    
    public EmailService(IOptions<EmailSettings> options)
    {
        _settings = options.Value;
    }
    
    public async Task SendEmail(string to, string subject, string body)
    {
        // Use strongly-typed settings
        var client = new SmtpClient(_settings.SmtpServer, _settings.Port)
        {
            EnableSsl = _settings.EnableSsl,
            Credentials = new NetworkCredential(_settings.Username, _settings.Password)
        };
        
        await client.SendMailAsync(new MailMessage("from@example.com", to, subject, body));
    }
}
```

**Benefits:**
- ✅ Type safety
- ✅ Intellisense
- ✅ Validation
- ✅ Testable

---

### 4. **IOptions vs IOptionsSnapshot vs IOptionsMonitor**

| Interface | Lifetime | Updates | Use Case |
|-----------|----------|---------|----------|
| **IOptions<T>** | Singleton | No | Static config read once at startup |
| **IOptionsSnapshot<T>** | Scoped | Per request | Config that varies per request |
| **IOptionsMonitor<T>** | Singleton | Real-time | Config that changes at runtime |

**IOptions (Most Common):**
```csharp
// Singleton: config read once
public class MyService
{
    private readonly AppSettings _settings;
    
    public MyService(IOptions<AppSettings> options)
    {
        _settings = options.Value;  // Read once, cached
    }
}
```

**IOptionsSnapshot:**
```csharp
// Scoped: re-read per request (e.g., tenant-specific config)
public class MyController : ControllerBase
{
    private readonly TenantSettings _settings;
    
    public MyController(IOptionsSnapshot<TenantSettings> options)
    {
        _settings = options.Value;  // Fresh per request
    }
}
```

**IOptionsMonitor:**
```csharp
// Singleton with change detection
public class CacheService
{
    private readonly IOptionsMonitor<CacheSettings> _optionsMonitor;
    
    public CacheService(IOptionsMonitor<CacheSettings> optionsMonitor)
    {
        _optionsMonitor = optionsMonitor;
        
        // React to config changes
        _optionsMonitor.OnChange(settings =>
        {
            Console.WriteLine($"Cache settings changed: {settings.MaxSize}");
            ReconfigureCache(settings);
        });
    }
    
    public void UseSettings()
    {
        var current = _optionsMonitor.CurrentValue;  // Always latest
    }
}
```

---

### 5. **Configuration Validation**

**Data Annotations:**
```csharp
using System.ComponentModel.DataAnnotations;

public class DatabaseSettings
{
    [Required]
    [MinLength(10)]
    public string ConnectionString { get; set; }
    
    [Range(1, 100)]
    public int MaxRetries { get; set; }
    
    [Range(0, 60)]
    public int TimeoutSeconds { get; set; }
}

// Enable validation
builder.Services.AddOptions<DatabaseSettings>()
    .Bind(builder.Configuration.GetSection("Database"))
    .ValidateDataAnnotations()
    .ValidateOnStart();  // Fail fast at startup if invalid
```

**Custom validation:**
```csharp
public class ApiSettings
{
    public string BaseUrl { get; set; }
    public string ApiKey { get; set; }
}

// Custom validator
public class ApiSettingsValidator : IValidateOptions<ApiSettings>
{
    public ValidateOptionsResult Validate(string name, ApiSettings options)
    {
        if (string.IsNullOrEmpty(options.BaseUrl))
            return ValidateOptionsResult.Fail("BaseUrl is required");
        
        if (!Uri.TryCreate(options.BaseUrl, UriKind.Absolute, out _))
            return ValidateOptionsResult.Fail("BaseUrl must be a valid URL");
        
        if (string.IsNullOrEmpty(options.ApiKey))
            return ValidateOptionsResult.Fail("ApiKey is required");
        
        return ValidateOptionsResult.Success;
    }
}

// Register validator
builder.Services.AddSingleton<IValidateOptions<ApiSettings>, ApiSettingsValidator>();
builder.Services.AddOptions<ApiSettings>()
    .Bind(builder.Configuration.GetSection("Api"))
    .ValidateOnStart();
```

---

### 6. **User Secrets (Development)**

**User Secrets** store sensitive data outside project directory (not committed to source control).

**Initialize User Secrets:**
```bash
# Initialize user secrets for project
dotnet user-secrets init

# Set a secret
dotnet user-secrets set "ApiKeys:OpenAI" "sk-xxxxxxxxxxxxx"

# Set connection string
dotnet user-secrets set "ConnectionStrings:DefaultConnection" "Server=localhost;..."

# List secrets
dotnet user-secrets list

# Remove secret
dotnet user-secrets remove "ApiKeys:OpenAI"

# Clear all secrets
dotnet user-secrets clear
```

**Stored location:**
- **Windows:** `%APPDATA%\Microsoft\UserSecrets\<user_secrets_id>\secrets.json`
- **macOS/Linux:** `~/.microsoft/usersecrets/<user_secrets_id>/secrets.json`

**Automatically loaded in Development environment.**

---

### 7. **Azure Key Vault (Production)**

**Azure Key Vault** securely stores secrets in cloud with access control and audit logging.

**Setup:**
```bash
# Install package
dotnet add package Azure.Extensions.AspNetCore.Configuration.Secrets
```

**Configure Key Vault:**
```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

if (builder.Environment.IsProduction())
{
    var keyVaultUrl = builder.Configuration["KeyVault:Url"];
    
    builder.Configuration.AddAzureKeyVault(
        new Uri(keyVaultUrl),
        new DefaultAzureCredential());  // Uses managed identity
}
```

**Access secrets:**
```csharp
// Secrets in Key Vault are automatically available in IConfiguration
var apiKey = builder.Configuration["ApiKeys--OpenAI"];  // Note: -- instead of :

// Or via Options pattern
builder.Services.Configure<ApiSettings>(builder.Configuration.GetSection("ApiKeys"));
```

**Key Vault naming:**
- Vault secret: `ApiKeys--OpenAI` (double dash becomes colon in config)
- Configuration key: `ApiKeys:OpenAI`

---

### 8. **Environment Variables**

**Setting environment variables:**

**Windows (PowerShell):**
```powershell
$env:ConnectionStrings__DefaultConnection = "Server=prod;..."
$env:ASPNETCORE_ENVIRONMENT = "Production"
```

**Linux/macOS:**
```bash
export ConnectionStrings__DefaultConnection="Server=prod;..."
export ASPNETCORE_ENVIRONMENT="Production"
```

**Docker:**
```yaml
# docker-compose.yml
services:
  myapp:
    image: myapp:latest
    environment:
      - ASPNETCORE_ENVIRONMENT=Production
      - ConnectionStrings__DefaultConnection=Server=prod;...
```

**Hierarchical keys:**
- `ConnectionStrings:DefaultConnection` → `ConnectionStrings__DefaultConnection`
- `Logging:LogLevel:Default` → `Logging__LogLevel__Default`

---

## Examples

### Example 1 — Complete Options pattern implementation

```csharp
// appsettings.json
{
  "EmailSettings": {
    "SmtpServer": "smtp.gmail.com",
    "Port": 587,
    "FromAddress": "noreply@example.com",
    "EnableSsl": true
  }
}

// Settings class
public class EmailSettings
{
    [Required]
    public string SmtpServer { get; set; }
    
    [Range(1, 65535)]
    public int Port { get; set; }
    
    [Required]
    [EmailAddress]
    public string FromAddress { get; set; }
    
    public bool EnableSsl { get; set; } = true;
}

// Program.cs
builder.Services.AddOptions<EmailSettings>()
    .Bind(builder.Configuration.GetSection("EmailSettings"))
    .ValidateDataAnnotations()
    .ValidateOnStart();

builder.Services.AddScoped<IEmailService, EmailService>();

// Service
public class EmailService : IEmailService
{
    private readonly EmailSettings _settings;
    
    public EmailService(IOptions<EmailSettings> options)
    {
        _settings = options.Value;
    }
    
    public async Task SendAsync(string to, string subject, string body)
    {
        using var client = new SmtpClient(_settings.SmtpServer, _settings.Port)
        {
            EnableSsl = _settings.EnableSsl
        };
        
        await client.SendMailAsync(_settings.FromAddress, to, subject, body);
    }
}
```

---

### Example 2 — Configuration with multiple environments

```json
// appsettings.json (base)
{
  "Logging": {
    "LogLevel": {
      "Default": "Information"
    }
  },
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=DevDb;"
  }
}

// appsettings.Development.json
{
  "Logging": {
    "LogLevel": {
      "Default": "Debug"
    }
  }
}

// appsettings.Production.json
{
  "Logging": {
    "LogLevel": {
      "Default": "Warning"
    }
  },
  "ConnectionStrings": {
    "DefaultConnection": "Server=prod-server;Database=ProdDb;"
  }
}
```

**Run with environment:**
```bash
# Development
dotnet run --environment Development

# Production
dotnet run --environment Production
```

---

### Example 3 — Testing with custom configuration

```csharp
public class MyServiceTests
{
    [Fact]
    public void TestWithCustomConfig()
    {
        // Arrange: Build in-memory configuration
        var config = new ConfigurationBuilder()
            .AddInMemoryCollection(new Dictionary<string, string>
            {
                ["ApiSettings:BaseUrl"] = "https://test-api.com",
                ["ApiSettings:ApiKey"] = "test-key"
            })
            .Build();
        
        var options = Options.Create(new ApiSettings
        {
            BaseUrl = "https://test-api.com",
            ApiKey = "test-key"
        });
        
        var service = new MyService(options);
        
        // Act & Assert
        // ...
    }
}
```

---

## Common Pitfalls

### ❌ 1. Committing secrets to source control

```json
// ❌ BAD: Secrets in appsettings.json (committed to Git)
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=prod;User=admin;Password=P@ssw0rd;"
  }
}

// ✅ GOOD: Use User Secrets (dev) or Key Vault (prod)
dotnet user-secrets set "ConnectionStrings:DefaultConnection" "..."
```

---

### ❌ 2. Using IOptions in singleton services expecting updates

```csharp
// ❌ BAD: IOptions doesn't reload
public class SingletonService
{
    public SingletonService(IOptions<Settings> options)
    {
        _settings = options.Value;  // Read once, never updated
    }
}

// ✅ GOOD: Use IOptionsMonitor for reloadable config
public class SingletonService
{
    public SingletonService(IOptionsMonitor<Settings> monitor)
    {
        _monitor = monitor;
    }
    
    public void DoWork()
    {
        var current = _monitor.CurrentValue;  // Always fresh
    }
}
```

---

### ❌ 3. Not validating configuration

```csharp
// ❌ BAD: No validation, fails at runtime
builder.Services.Configure<Settings>(
    builder.Configuration.GetSection("Settings"));

// ✅ GOOD: Validate at startup
builder.Services.AddOptions<Settings>()
    .Bind(builder.Configuration.GetSection("Settings"))
    .ValidateDataAnnotations()
    .ValidateOnStart();
```

---

### ❌ 4. Hardcoding environment checks

```csharp
// ❌ BAD: Hardcoded logic
if (env == "Production")
{
    useProductionDb = true;
}

// ✅ GOOD: Use configuration
var dbSettings = builder.Configuration.GetSection("Database").Get<DatabaseSettings>();
```

---

## Quick Self-Check

You should be able to answer these out loud:

1. What's the configuration precedence order in ASP.NET Core?
2. What's the difference between IOptions, IOptionsSnapshot, and IOptionsMonitor?
3. How do you securely store secrets in development vs production?
4. What's the Options pattern and why use it over IConfiguration directly?
5. How do you validate configuration at startup?
6. How do environment variables map to hierarchical config keys?
7. What's User Secrets and where are they stored?
8. How do you override configuration for testing?

If you struggle with any of these, revisit the relevant section.

---

## Further Practice

1. Create an ASP.NET Core app with appsettings.json and environment-specific overrides.
2. Implement Options pattern with validation for database settings.
3. Set up User Secrets for local development secrets.
4. Configure Azure Key Vault integration for production secrets.
5. Write unit tests that inject custom configuration via IOptions.

---

## Mini Interview Cheat Sheet

**Configuration precedence:**
appsettings.json → appsettings.{Environment}.json → User Secrets → Environment Variables → Command-line

**Options pattern:**
"Strongly-typed configuration via POCOs. Bind config section to class, inject as IOptions<T>. Provides type safety, validation, and testability."

**IOptions variants:**
- **IOptions** — Singleton, read once
- **IOptionsSnapshot** — Scoped, per request
- **IOptionsMonitor** — Singleton, live updates

**Secrets management:**
- **Dev:** User Secrets (`dotnet user-secrets set`)
- **Prod:** Azure Key Vault or environment variables
- **Never:** appsettings.json in source control

**Validation:**
```csharp
builder.Services.AddOptions<Settings>()
    .Bind(config.GetSection("Settings"))
    .ValidateDataAnnotations()
    .ValidateOnStart();
```

---
