# Design Patterns in C#

🚦 **Intermediate** | ⏱ 50-60 min

---

## 📝 Why This Matters

Design patterns are proven solutions to common software design problems. While it's tempting to think of them as just academic concepts, they're actually the vocabulary of experienced developers—the shared language that lets teams communicate complex ideas quickly and effectively.

In real-world C# development, design patterns matter because they directly impact your codebase's maintainability, testability, and scalability. When you see a `DbContext` in Entity Framework, you're using the Unit of Work pattern. When you inject services through constructors, you're using Dependency Injection (a form of Inversion of Control). When you register services with different lifetimes in ASP.NET Core, you're dealing with variations of the Singleton and Factory patterns.

Understanding patterns helps you read and understand existing codebases faster. When you join a new team and see a class named `OrderFactory` or `NotificationService`, the pattern names immediately tell you what to expect about how these classes behave. This shared vocabulary reduces cognitive load and speeds up onboarding.

Patterns also help you avoid reinventing the wheel—or worse, inventing a square wheel. Many developers have struggled with the same problems you face: "How do I create objects when their type isn't known until runtime?" (Factory), "How do I add behavior without modifying existing classes?" (Decorator), "How do I let objects communicate without tight coupling?" (Observer). Patterns capture the lessons learned from solving these problems thousands of times across thousands of projects.

However, patterns can be misused. The most common mistake is pattern obsession—applying patterns everywhere "just in case" or to make code look more "professional." This leads to over-engineered codebases with unnecessary abstractions that make simple tasks complicated. Every pattern adds complexity; only use them when they solve a real problem you actually have.

In C#, patterns integrate deeply with language features. LINQ is built on the Iterator pattern. Events use the Observer pattern. The `IDisposable` interface implements the Dispose pattern. Modern C# features like records, pattern matching, and init-only properties have changed how we implement certain patterns, making some simpler and others less necessary.

From an interview perspective, you need to know both the classic patterns and how they manifest in real C# frameworks. Interviewers often ask about Repository pattern (especially with Entity Framework), Singleton pattern (and its problems with DI containers), Factory pattern (especially Abstract Factory), and Strategy pattern (particularly for polymorphic behavior). Being able to explain when NOT to use a pattern is often as important as knowing when to use it.

Understanding design patterns also helps with system design interviews. When designing a notification system, you might use Observer. For a payment processing system, Strategy pattern helps handle different payment methods. For a caching layer, Proxy or Decorator might be appropriate. Patterns give you building blocks for architectural discussions.

The key is balance: know the patterns, understand where they naturally fit, but don't force them. A simple `if` statement is often better than an elaborate Strategy pattern if you only have two strategies that rarely change. A static method might be better than a full Factory if your object creation is truly simple. Patterns are tools, not goals.

---

## 🎯 Core Ideas

### 1. **Creational Patterns: Object Creation**

**Singleton Pattern**

Ensures a class has only one instance and provides a global access point.

```csharp
// ❌ Thread-unsafe, classic singleton (DON'T USE)
public class LegacySingleton
{
    private static LegacySingleton _instance;
    
    private LegacySingleton() { }
    
    public static LegacySingleton Instance
    {
        get
        {
            if (_instance == null)
                _instance = new LegacySingleton();
            return _instance;
        }
    }
}

// ✅ Thread-safe, lazy initialization
public sealed class ThreadSafeSingleton
{
    private static readonly Lazy<ThreadSafeSingleton> _instance = 
        new Lazy<ThreadSafeSingleton>(() => new ThreadSafeSingleton());
    
    private ThreadSafeSingleton() 
    {
        Console.WriteLine("Singleton initialized");
    }
    
    public static ThreadSafeSingleton Instance => _instance.Value;
    
    public void DoWork() => Console.WriteLine("Working...");
}

// Usage
var singleton = ThreadSafeSingleton.Instance; // Initializes here
singleton.DoWork();

// ⚠️ In modern C# with DI, prefer this:
public interface IConfigurationService
{
    string GetSetting(string key);
}

public class ConfigurationService : IConfigurationService
{
    // No singleton code needed!
    public string GetSetting(string key) => $"Value for {key}";
}

// In Program.cs or Startup.cs
services.AddSingleton<IConfigurationService, ConfigurationService>();
```

**When to use Singleton:**
- Configuration management (though DI is usually better)
- Logging (though DI is usually better)
- Caching services
- Connection pools

**When NOT to use:**
- With DI containers (use `AddSingleton` instead)
- For state that varies per user/request (use scoped lifetime)
- When testability matters (hard to mock/test)
- When you need different instances in tests vs production

**Factory Pattern**

Creates objects without exposing instantiation logic.

```csharp
// Simple Factory
public interface INotification
{
    void Send(string message);
}

public class EmailNotification : INotification
{
    public void Send(string message) => 
        Console.WriteLine($"📧 Email: {message}");
}

public class SmsNotification : INotification
{
    public void Send(string message) => 
        Console.WriteLine($"📱 SMS: {message}");
}

public class PushNotification : INotification
{
    public void Send(string message) => 
        Console.WriteLine($"🔔 Push: {message}");
}

public static class NotificationFactory
{
    public static INotification Create(string type)
    {
        return type.ToLower() switch
        {
            "email" => new EmailNotification(),
            "sms" => new SmsNotification(),
            "push" => new PushNotification(),
            _ => throw new ArgumentException($"Unknown type: {type}")
        };
    }
}

// Usage
var notification = NotificationFactory.Create("email");
notification.Send("Hello!");

// ✅ Better: Factory with DI
public interface INotificationFactory
{
    INotification Create(string type);
}

public class NotificationFactory : INotificationFactory
{
    private readonly IServiceProvider _serviceProvider;
    
    public NotificationFactory(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }
    
    public INotification Create(string type)
    {
        return type.ToLower() switch
        {
            "email" => _serviceProvider.GetRequiredService<EmailNotification>(),
            "sms" => _serviceProvider.GetRequiredService<SmsNotification>(),
            "push" => _serviceProvider.GetRequiredService<PushNotification>(),
            _ => throw new ArgumentException($"Unknown type: {type}")
        };
    }
}

// Registration
services.AddTransient<EmailNotification>();
services.AddTransient<SmsNotification>();
services.AddTransient<PushNotification>();
services.AddSingleton<INotificationFactory, NotificationFactory>();
```

**Abstract Factory Pattern**

Creates families of related objects.

```csharp
// Abstract products
public interface IButton
{
    void Render();
}

public interface ITextBox
{
    void Render();
}

// Concrete products for Windows
public class WindowsButton : IButton
{
    public void Render() => Console.WriteLine("🪟 Windows Button");
}

public class WindowsTextBox : ITextBox
{
    public void Render() => Console.WriteLine("🪟 Windows TextBox");
}

// Concrete products for macOS
public class MacButton : IButton
{
    public void Render() => Console.WriteLine("🍎 Mac Button");
}

public class MacTextBox : ITextBox
{
    public void Render() => Console.WriteLine("🍎 Mac TextBox");
}

// Abstract factory
public interface IUIFactory
{
    IButton CreateButton();
    ITextBox CreateTextBox();
}

// Concrete factories
public class WindowsUIFactory : IUIFactory
{
    public IButton CreateButton() => new WindowsButton();
    public ITextBox CreateTextBox() => new WindowsTextBox();
}

public class MacUIFactory : IUIFactory
{
    public IButton CreateButton() => new MacButton();
    public ITextBox CreateTextBox() => new MacTextBox();
}

// Usage
IUIFactory factory = Environment.OSVersion.Platform == PlatformID.Win32NT
    ? new WindowsUIFactory()
    : new MacUIFactory();

var button = factory.CreateButton();
var textBox = factory.CreateTextBox();
button.Render();
textBox.Render();
```

**Builder Pattern**

Constructs complex objects step by step.

```csharp
public class HttpRequest
{
    public string Url { get; set; }
    public string Method { get; set; }
    public Dictionary<string, string> Headers { get; set; }
    public string Body { get; set; }
    public int TimeoutSeconds { get; set; }
}

public class HttpRequestBuilder
{
    private readonly HttpRequest _request = new();
    
    public HttpRequestBuilder WithUrl(string url)
    {
        _request.Url = url;
        return this;
    }
    
    public HttpRequestBuilder WithMethod(string method)
    {
        _request.Method = method;
        return this;
    }
    
    public HttpRequestBuilder WithHeader(string key, string value)
    {
        _request.Headers ??= new Dictionary<string, string>();
        _request.Headers[key] = value;
        return this;
    }
    
    public HttpRequestBuilder WithBody(string body)
    {
        _request.Body = body;
        return this;
    }
    
    public HttpRequestBuilder WithTimeout(int seconds)
    {
        _request.TimeoutSeconds = seconds;
        return this;
    }
    
    public HttpRequest Build()
    {
        if (string.IsNullOrEmpty(_request.Url))
            throw new InvalidOperationException("URL is required");
        if (string.IsNullOrEmpty(_request.Method))
            _request.Method = "GET";
        return _request;
    }
}

// Usage - fluent interface
var request = new HttpRequestBuilder()
    .WithUrl("https://api.example.com/users")
    .WithMethod("POST")
    .WithHeader("Content-Type", "application/json")
    .WithHeader("Authorization", "Bearer token123")
    .WithBody("{\"name\": \"John\"}")
    .WithTimeout(30)
    .Build();

// ✅ Modern C# alternative: init-only properties + object initializer
public class HttpRequest
{
    public required string Url { get; init; }
    public string Method { get; init; } = "GET";
    public Dictionary<string, string> Headers { get; init; } = new();
    public string? Body { get; init; }
    public int TimeoutSeconds { get; init; } = 30;
}

var request = new HttpRequest
{
    Url = "https://api.example.com/users",
    Method = "POST",
    Headers = new() 
    { 
        ["Content-Type"] = "application/json",
        ["Authorization"] = "Bearer token123"
    },
    Body = "{\"name\": \"John\"}"
};
```

### 2. **Structural Patterns: Object Composition**

**Repository Pattern**

Abstracts data access logic.

```csharp
public interface IRepository<T> where T : class
{
    Task<T?> GetByIdAsync(int id);
    Task<IEnumerable<T>> GetAllAsync();
    Task AddAsync(T entity);
    Task UpdateAsync(T entity);
    Task DeleteAsync(int id);
}

public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
}

// Generic repository implementation
public class Repository<T> : IRepository<T> where T : class
{
    private readonly DbContext _context;
    
    public Repository(DbContext context)
    {
        _context = context;
    }
    
    public async Task<T?> GetByIdAsync(int id)
    {
        return await _context.Set<T>().FindAsync(id);
    }
    
    public async Task<IEnumerable<T>> GetAllAsync()
    {
        return await _context.Set<T>().ToListAsync();
    }
    
    public async Task AddAsync(T entity)
    {
        await _context.Set<T>().AddAsync(entity);
        await _context.SaveChangesAsync();
    }
    
    public async Task UpdateAsync(T entity)
    {
        _context.Set<T>().Update(entity);
        await _context.SaveChangesAsync();
    }
    
    public async Task DeleteAsync(int id)
    {
        var entity = await GetByIdAsync(id);
        if (entity != null)
        {
            _context.Set<T>().Remove(entity);
            await _context.SaveChangesAsync();
        }
    }
}

// Specialized repository for complex queries
public interface IProductRepository : IRepository<Product>
{
    Task<IEnumerable<Product>> GetExpensiveProductsAsync(decimal minPrice);
    Task<IEnumerable<Product>> SearchByNameAsync(string searchTerm);
}

public class ProductRepository : Repository<Product>, IProductRepository
{
    private readonly ApplicationDbContext _context;
    
    public ProductRepository(ApplicationDbContext context) : base(context)
    {
        _context = context;
    }
    
    public async Task<IEnumerable<Product>> GetExpensiveProductsAsync(decimal minPrice)
    {
        return await _context.Products
            .Where(p => p.Price >= minPrice)
            .OrderByDescending(p => p.Price)
            .ToListAsync();
    }
    
    public async Task<IEnumerable<Product>> SearchByNameAsync(string searchTerm)
    {
        return await _context.Products
            .Where(p => p.Name.Contains(searchTerm))
            .ToListAsync();
    }
}

// Usage in service
public class ProductService
{
    private readonly IProductRepository _productRepository;
    
    public ProductService(IProductRepository productRepository)
    {
        _productRepository = productRepository;
    }
    
    public async Task<Product?> GetProductAsync(int id)
    {
        return await _productRepository.GetByIdAsync(id);
    }
    
    public async Task<IEnumerable<Product>> GetPremiumProductsAsync()
    {
        return await _productRepository.GetExpensiveProductsAsync(100m);
    }
}
```

**Decorator Pattern**

Adds behavior to objects dynamically.

```csharp
public interface IDataService
{
    Task<string> GetDataAsync(string key);
}

// Base implementation
public class DataService : IDataService
{
    public async Task<string> GetDataAsync(string key)
    {
        await Task.Delay(100); // Simulate database call
        return $"Data for {key}";
    }
}

// Logging decorator
public class LoggingDataService : IDataService
{
    private readonly IDataService _inner;
    private readonly ILogger<LoggingDataService> _logger;
    
    public LoggingDataService(IDataService inner, ILogger<LoggingDataService> logger)
    {
        _inner = inner;
        _logger = logger;
    }
    
    public async Task<string> GetDataAsync(string key)
    {
        _logger.LogInformation("Getting data for key: {Key}", key);
        var result = await _inner.GetDataAsync(key);
        _logger.LogInformation("Retrieved data: {Result}", result);
        return result;
    }
}

// Caching decorator
public class CachingDataService : IDataService
{
    private readonly IDataService _inner;
    private readonly IMemoryCache _cache;
    
    public CachingDataService(IDataService inner, IMemoryCache cache)
    {
        _inner = inner;
        _cache = cache;
    }
    
    public async Task<string> GetDataAsync(string key)
    {
        if (_cache.TryGetValue(key, out string cachedValue))
        {
            return cachedValue;
        }
        
        var result = await _inner.GetDataAsync(key);
        _cache.Set(key, result, TimeSpan.FromMinutes(5));
        return result;
    }
}

// Retry decorator
public class RetryDataService : IDataService
{
    private readonly IDataService _inner;
    private readonly int _maxRetries;
    
    public RetryDataService(IDataService inner, int maxRetries = 3)
    {
        _inner = inner;
        _maxRetries = maxRetries;
    }
    
    public async Task<string> GetDataAsync(string key)
    {
        for (int i = 0; i < _maxRetries; i++)
        {
            try
            {
                return await _inner.GetDataAsync(key);
            }
            catch (Exception) when (i < _maxRetries - 1)
            {
                await Task.Delay(TimeSpan.FromSeconds(Math.Pow(2, i))); // Exponential backoff
            }
        }
        throw new Exception($"Failed after {_maxRetries} attempts");
    }
}

// Composing decorators
IDataService service = new DataService();
service = new RetryDataService(service);
service = new CachingDataService(service, memoryCache);
service = new LoggingDataService(service, logger);

var data = await service.GetDataAsync("user:123");
// Order matters: Logging → Caching → Retry → Base
```

**Adapter Pattern**

Converts one interface to another.

```csharp
// Third-party library we can't modify
public class LegacyPrinter
{
    public void PrintInUpperCase(string text)
    {
        Console.WriteLine(text.ToUpper());
    }
}

// Our application interface
public interface IPrinter
{
    void Print(string text, bool uppercase = false);
}

// Adapter
public class PrinterAdapter : IPrinter
{
    private readonly LegacyPrinter _legacyPrinter;
    
    public PrinterAdapter(LegacyPrinter legacyPrinter)
    {
        _legacyPrinter = legacyPrinter;
    }
    
    public void Print(string text, bool uppercase = false)
    {
        if (uppercase)
        {
            _legacyPrinter.PrintInUpperCase(text);
        }
        else
        {
            Console.WriteLine(text);
        }
    }
}

// Usage
IPrinter printer = new PrinterAdapter(new LegacyPrinter());
printer.Print("Hello World", uppercase: true);
printer.Print("Normal text");
```

### 3. **Behavioral Patterns: Object Interaction**

**Strategy Pattern**

Defines a family of algorithms and makes them interchangeable.

```csharp
// Strategy interface
public interface IPaymentStrategy
{
    Task<PaymentResult> ProcessPaymentAsync(decimal amount);
}

public class PaymentResult
{
    public bool Success { get; set; }
    public string TransactionId { get; set; }
    public string Message { get; set; }
}

// Concrete strategies
public class CreditCardPayment : IPaymentStrategy
{
    public async Task<PaymentResult> ProcessPaymentAsync(decimal amount)
    {
        await Task.Delay(500); // Simulate API call
        return new PaymentResult
        {
            Success = true,
            TransactionId = Guid.NewGuid().ToString(),
            Message = $"Credit card payment of ${amount} processed"
        };
    }
}

public class PayPalPayment : IPaymentStrategy
{
    public async Task<PaymentResult> ProcessPaymentAsync(decimal amount)
    {
        await Task.Delay(300);
        return new PaymentResult
        {
            Success = true,
            TransactionId = Guid.NewGuid().ToString(),
            Message = $"PayPal payment of ${amount} processed"
        };
    }
}

public class CryptoPayment : IPaymentStrategy
{
    public async Task<PaymentResult> ProcessPaymentAsync(decimal amount)
    {
        await Task.Delay(1000);
        return new PaymentResult
        {
            Success = true,
            TransactionId = Guid.NewGuid().ToString(),
            Message = $"Cryptocurrency payment of ${amount} processed"
        };
    }
}

// Context
public class PaymentProcessor
{
    private IPaymentStrategy _strategy;
    
    public void SetStrategy(IPaymentStrategy strategy)
    {
        _strategy = strategy;
    }
    
    public async Task<PaymentResult> ProcessAsync(decimal amount)
    {
        if (_strategy == null)
            throw new InvalidOperationException("Payment strategy not set");
        
        return await _strategy.ProcessPaymentAsync(amount);
    }
}

// Usage
var processor = new PaymentProcessor();

processor.SetStrategy(new CreditCardPayment());
var result1 = await processor.ProcessAsync(99.99m);

processor.SetStrategy(new PayPalPayment());
var result2 = await processor.ProcessAsync(49.99m);

// ✅ Better: Strategy with DI
public class PaymentService
{
    private readonly IEnumerable<IPaymentStrategy> _strategies;
    
    public PaymentService(IEnumerable<IPaymentStrategy> strategies)
    {
        _strategies = strategies;
    }
    
    public async Task<PaymentResult> ProcessAsync(string method, decimal amount)
    {
        var strategy = method.ToLower() switch
        {
            "creditcard" => _strategies.OfType<CreditCardPayment>().First(),
            "paypal" => _strategies.OfType<PayPalPayment>().First(),
            "crypto" => _strategies.OfType<CryptoPayment>().First(),
            _ => throw new ArgumentException($"Unknown payment method: {method}")
        };
        
        return await strategy.ProcessPaymentAsync(amount);
    }
}

// Registration
services.AddTransient<IPaymentStrategy, CreditCardPayment>();
services.AddTransient<IPaymentStrategy, PayPalPayment>();
services.AddTransient<IPaymentStrategy, CryptoPayment>();
```

**Observer Pattern**

Defines one-to-many dependency between objects.

```csharp
// Using C# events (built-in Observer pattern)
public class StockTicker
{
    // Event declaration
    public event EventHandler<StockPriceChangedEventArgs>? PriceChanged;
    
    private decimal _price;
    
    public decimal Price
    {
        get => _price;
        set
        {
            if (_price != value)
            {
                var oldPrice = _price;
                _price = value;
                OnPriceChanged(new StockPriceChangedEventArgs
                {
                    OldPrice = oldPrice,
                    NewPrice = value,
                    Timestamp = DateTime.UtcNow
                });
            }
        }
    }
    
    protected virtual void OnPriceChanged(StockPriceChangedEventArgs e)
    {
        PriceChanged?.Invoke(this, e);
    }
}

public class StockPriceChangedEventArgs : EventArgs
{
    public decimal OldPrice { get; set; }
    public decimal NewPrice { get; set; }
    public DateTime Timestamp { get; set; }
}

// Observers
public class PriceDisplay
{
    public void Subscribe(StockTicker ticker)
    {
        ticker.PriceChanged += OnPriceChanged;
    }
    
    private void OnPriceChanged(object? sender, StockPriceChangedEventArgs e)
    {
        Console.WriteLine($"📊 Display: Price changed from ${e.OldPrice} to ${e.NewPrice}");
    }
}

public class PriceAlert
{
    private readonly decimal _threshold;
    
    public PriceAlert(decimal threshold)
    {
        _threshold = threshold;
    }
    
    public void Subscribe(StockTicker ticker)
    {
        ticker.PriceChanged += OnPriceChanged;
    }
    
    private void OnPriceChanged(object? sender, StockPriceChangedEventArgs e)
    {
        if (e.NewPrice > _threshold)
        {
            Console.WriteLine($"🚨 Alert: Price ${e.NewPrice} exceeded threshold ${_threshold}!");
        }
    }
}

// Usage
var ticker = new StockTicker { Price = 100m };

var display = new PriceDisplay();
display.Subscribe(ticker);

var alert = new PriceAlert(150m);
alert.Subscribe(ticker);

ticker.Price = 120m; // Both observers notified
ticker.Price = 160m; // Both observers notified, alert triggers
```

**Command Pattern**

Encapsulates requests as objects.

```csharp
// Command interface
public interface ICommand
{
    void Execute();
    void Undo();
}

// Receiver
public class Document
{
    private StringBuilder _content = new();
    
    public void Write(string text)
    {
        _content.Append(text);
        Console.WriteLine($"Document: {_content}");
    }
    
    public void Erase(int length)
    {
        if (length > _content.Length)
            length = _content.Length;
        _content.Remove(_content.Length - length, length);
        Console.WriteLine($"Document: {_content}");
    }
    
    public string GetContent() => _content.ToString();
}

// Concrete commands
public class WriteCommand : ICommand
{
    private readonly Document _document;
    private readonly string _text;
    
    public WriteCommand(Document document, string text)
    {
        _document = document;
        _text = text;
    }
    
    public void Execute()
    {
        _document.Write(_text);
    }
    
    public void Undo()
    {
        _document.Erase(_text.Length);
    }
}

// Invoker
public class DocumentEditor
{
    private readonly Stack<ICommand> _history = new();
    
    public void ExecuteCommand(ICommand command)
    {
        command.Execute();
        _history.Push(command);
    }
    
    public void Undo()
    {
        if (_history.Count > 0)
        {
            var command = _history.Pop();
            command.Undo();
        }
    }
}

// Usage
var document = new Document();
var editor = new DocumentEditor();

editor.ExecuteCommand(new WriteCommand(document, "Hello "));
editor.ExecuteCommand(new WriteCommand(document, "World!"));
editor.Undo(); // Removes "World!"
editor.Undo(); // Removes "Hello "
```

**Chain of Responsibility**

Passes requests along a chain of handlers.

```csharp
public abstract class ValidationHandler
{
    private ValidationHandler? _nextHandler;
    
    public ValidationHandler SetNext(ValidationHandler handler)
    {
        _nextHandler = handler;
        return handler;
    }
    
    public virtual ValidationResult Handle(string input)
    {
        if (_nextHandler != null)
        {
            return _nextHandler.Handle(input);
        }
        
        return ValidationResult.Success();
    }
}

public class ValidationResult
{
    public bool IsValid { get; set; }
    public string Message { get; set; }
    
    public static ValidationResult Success() => 
        new() { IsValid = true, Message = "Valid" };
    
    public static ValidationResult Failure(string message) => 
        new() { IsValid = false, Message = message };
}

public class NotEmptyValidator : ValidationHandler
{
    public override ValidationResult Handle(string input)
    {
        if (string.IsNullOrWhiteSpace(input))
        {
            return ValidationResult.Failure("Input cannot be empty");
        }
        
        return base.Handle(input);
    }
}

public class MinLengthValidator : ValidationHandler
{
    private readonly int _minLength;
    
    public MinLengthValidator(int minLength)
    {
        _minLength = minLength;
    }
    
    public override ValidationResult Handle(string input)
    {
        if (input.Length < _minLength)
        {
            return ValidationResult.Failure($"Input must be at least {_minLength} characters");
        }
        
        return base.Handle(input);
    }
}

public class ContainsDigitValidator : ValidationHandler
{
    public override ValidationResult Handle(string input)
    {
        if (!input.Any(char.IsDigit))
        {
            return ValidationResult.Failure("Input must contain at least one digit");
        }
        
        return base.Handle(input);
    }
}

// Usage
var validator = new NotEmptyValidator();
validator
    .SetNext(new MinLengthValidator(6))
    .SetNext(new ContainsDigitValidator());

var result1 = validator.Handle("");           // Fails: empty
var result2 = validator.Handle("abc");        // Fails: too short
var result3 = validator.Handle("abcdef");     // Fails: no digit
var result4 = validator.Handle("abc123");     // Success

Console.WriteLine($"Result: {result4.IsValid} - {result4.Message}");
```

### 4. **Anti-Patterns: What to Avoid**

**God Object**

❌ A class that knows too much or does too much.

```csharp
// ❌ BAD: God object
public class OrderManager
{
    public void ProcessOrder(Order order)
    {
        // Validates order
        if (order.Items.Count == 0)
            throw new Exception("Empty order");
        
        // Calculates total
        decimal total = 0;
        foreach (var item in order.Items)
            total += item.Price * item.Quantity;
        
        // Applies discount
        if (order.CustomerId > 0)
        {
            var customer = GetCustomer(order.CustomerId);
            if (customer.IsPremium)
                total *= 0.9m;
        }
        
        // Processes payment
        var paymentGateway = new PaymentGateway();
        paymentGateway.Charge(total);
        
        // Updates inventory
        foreach (var item in order.Items)
        {
            var product = GetProduct(item.ProductId);
            product.Stock -= item.Quantity;
            SaveProduct(product);
        }
        
        // Sends email
        var emailService = new EmailService();
        emailService.Send(order.CustomerEmail, "Order confirmed");
        
        // Logs everything
        Log($"Order {order.Id} processed");
    }
    
    private Customer GetCustomer(int id) { /*...*/ }
    private Product GetProduct(int id) { /*...*/ }
    private void SaveProduct(Product p) { /*...*/ }
    private void Log(string message) { /*...*/ }
}

// ✅ GOOD: Single Responsibility Principle
public class OrderValidator
{
    public void Validate(Order order)
    {
        if (order.Items.Count == 0)
            throw new ValidationException("Empty order");
    }
}

public class OrderCalculator
{
    public decimal CalculateTotal(Order order, Customer customer)
    {
        var total = order.Items.Sum(i => i.Price * i.Quantity);
        if (customer.IsPremium)
            total *= 0.9m;
        return total;
    }
}

public class OrderProcessor
{
    private readonly IPaymentService _paymentService;
    private readonly IInventoryService _inventoryService;
    private readonly IEmailService _emailService;
    private readonly ILogger<OrderProcessor> _logger;
    
    public OrderProcessor(
        IPaymentService paymentService,
        IInventoryService inventoryService,
        IEmailService emailService,
        ILogger<OrderProcessor> logger)
    {
        _paymentService = paymentService;
        _inventoryService = inventoryService;
        _emailService = emailService;
        _logger = logger;
    }
    
    public async Task ProcessAsync(Order order)
    {
        await _paymentService.ChargeAsync(order.Total);
        await _inventoryService.UpdateStockAsync(order.Items);
        await _emailService.SendConfirmationAsync(order);
        _logger.LogInformation("Order {OrderId} processed", order.Id);
    }
}
```

**Anemic Domain Model**

❌ Objects with only properties, no behavior.

```csharp
// ❌ BAD: Anemic model
public class BankAccount
{
    public string AccountNumber { get; set; }
    public decimal Balance { get; set; }
}

public class BankAccountService
{
    public void Deposit(BankAccount account, decimal amount)
    {
        if (amount <= 0)
            throw new ArgumentException("Amount must be positive");
        account.Balance += amount;
    }
    
    public void Withdraw(BankAccount account, decimal amount)
    {
        if (amount <= 0)
            throw new ArgumentException("Amount must be positive");
        if (account.Balance < amount)
            throw new InvalidOperationException("Insufficient funds");
        account.Balance -= amount;
    }
}

// ✅ GOOD: Rich domain model
public class BankAccount
{
    public string AccountNumber { get; private set; }
    public decimal Balance { get; private set; }
    
    public BankAccount(string accountNumber, decimal initialBalance = 0)
    {
        if (string.IsNullOrWhiteSpace(accountNumber))
            throw new ArgumentException("Account number required");
        if (initialBalance < 0)
            throw new ArgumentException("Initial balance cannot be negative");
        
        AccountNumber = accountNumber;
        Balance = initialBalance;
    }
    
    public void Deposit(decimal amount)
    {
        if (amount <= 0)
            throw new ArgumentException("Deposit amount must be positive");
        
        Balance += amount;
    }
    
    public void Withdraw(decimal amount)
    {
        if (amount <= 0)
            throw new ArgumentException("Withdrawal amount must be positive");
        if (Balance < amount)
            throw new InvalidOperationException("Insufficient funds");
        
        Balance -= amount;
    }
    
    public bool CanWithdraw(decimal amount) => Balance >= amount && amount > 0;
}
```

**Pattern Overuse**

❌ Using patterns when simple code would work.

```csharp
// ❌ BAD: Unnecessary abstraction
public interface IAdditionStrategy
{
    int Add(int a, int b);
}

public class SimpleAdditionStrategy : IAdditionStrategy
{
    public int Add(int a, int b) => a + b;
}

public class Calculator
{
    private readonly IAdditionStrategy _additionStrategy;
    
    public Calculator(IAdditionStrategy additionStrategy)
    {
        _additionStrategy = additionStrategy;
    }
    
    public int Add(int a, int b) => _additionStrategy.Add(a, b);
}

// ✅ GOOD: Just use the language
public class Calculator
{
    public int Add(int a, int b) => a + b;
}
```

---

## ⚠️ Common Pitfalls

### 1. **Singleton + Dependency Injection Conflict**

```csharp
// ❌ DON'T mix classic Singleton with DI
public class CacheService
{
    private static readonly Lazy<CacheService> _instance = 
        new(() => new CacheService());
    
    public static CacheService Instance => _instance.Value;
    
    private CacheService() { }
}

// In a controller
public class ProductController : ControllerBase
{
    public IActionResult Get()
    {
        var cache = CacheService.Instance; // ❌ Can't test this!
        // ...
    }
}

// ✅ DO: Let DI container manage lifetime
public class CacheService
{
    // No singleton code!
}

// Registration
services.AddSingleton<CacheService>();

// Usage
public class ProductController : ControllerBase
{
    private readonly CacheService _cache;
    
    public ProductController(CacheService cache)
    {
        _cache = cache; // ✅ Testable
    }
}
```

### 2. **Over-Engineering with Factories**

```csharp
// ❌ Overkill for simple cases
public interface ILogger { }
public interface ILoggerFactory
{
    ILogger CreateLogger(string category);
}
public class LoggerFactory : ILoggerFactory { /*...*/ }

// ✅ DI handles this
services.AddLogging();

public class MyService
{
    private readonly ILogger<MyService> _logger;
    
    public MyService(ILogger<MyService> logger)
    {
        _logger = logger; // DI creates the right logger
    }
}
```

### 3. **Generic Repository Limiting Queries**

```csharp
// ❌ Can't express complex queries
public interface IRepository<T>
{
    Task<IEnumerable<T>> GetAllAsync();
}

// What if you need:
// - Products with price > $100
// - Products ordered by name with pagination
// - Products with related category data

// ✅ Specific repositories for complex needs
public interface IProductRepository
{
    Task<PagedResult<Product>> SearchAsync(ProductSearchCriteria criteria);
    Task<IEnumerable<Product>> GetByPriceRangeAsync(decimal min, decimal max);
    Task<Product?> GetWithCategoryAsync(int id);
}
```

### 4. **Forgetting to Unsubscribe from Events**

```csharp
// ❌ Memory leak
public class Dashboard
{
    public Dashboard(StockTicker ticker)
    {
        ticker.PriceChanged += OnPriceChanged; // ❌ Never unsubscribed!
    }
    
    private void OnPriceChanged(object? sender, EventArgs e) { }
}

// ✅ Implement IDisposable
public class Dashboard : IDisposable
{
    private readonly StockTicker _ticker;
    
    public Dashboard(StockTicker ticker)
    {
        _ticker = ticker;
        _ticker.PriceChanged += OnPriceChanged;
    }
    
    private void OnPriceChanged(object? sender, EventArgs e) { }
    
    public void Dispose()
    {
        _ticker.PriceChanged -= OnPriceChanged;
    }
}
```

### 5. **Strategy Pattern Without Validation**

```csharp
// ❌ Runtime error if strategy not set
public class PaymentProcessor
{
    private IPaymentStrategy? _strategy;
    
    public async Task ProcessAsync(decimal amount)
    {
        return await _strategy.ProcessPaymentAsync(amount); // NullReferenceException!
    }
}

// ✅ Validate in constructor
public class PaymentProcessor
{
    private readonly IPaymentStrategy _strategy;
    
    public PaymentProcessor(IPaymentStrategy strategy)
    {
        _strategy = strategy ?? throw new ArgumentNullException(nameof(strategy));
    }
    
    public async Task ProcessAsync(decimal amount)
    {
        return await _strategy.ProcessPaymentAsync(amount);
    }
}
```

### 6. **Decorator Order Matters**

```csharp
// Order affects behavior
IDataService service = new DataService();

// ❌ Caching before logging: won't log cache hits
service = new LoggingDataService(service, logger);
service = new CachingDataService(service, cache);

// ✅ Logging after caching: logs both cache hits and misses
service = new CachingDataService(service, cache);
service = new LoggingDataService(service, logger);
```

---

## ✅ Self-Check Questions

1. **When would you use Factory pattern over simple `new` keyword?**
   - When object creation is complex or depends on configuration
   - When you need to create different types based on runtime conditions
   - When you want to centralize creation logic for testing/maintenance

2. **What's the difference between Strategy and Decorator patterns?**
   - Strategy: chooses one algorithm from many (either/or)
   - Decorator: adds multiple behaviors (both/and)
   - Strategy focuses on behavior selection, Decorator on behavior extension

3. **Why is Singleton problematic with DI?**
   - Makes testing difficult (can't inject mocks)
   - DI containers manage singleton lifetime better
   - Tight coupling to concrete implementation
   - Can't have different instances in different contexts

4. **When is Repository pattern overkill?**
   - Simple CRUD apps with minimal business logic
   - When DbContext is already providing abstraction
   - When you don't need to swap data sources
   - When queries are too complex for generic repository

5. **How do you prevent memory leaks with Observer pattern?**
   - Unsubscribe from events when done (implement IDisposable)
   - Use weak event patterns for long-lived publishers
   - Consider using reactive extensions (Rx) which handle cleanup

---

## 🏋️ Practice

1. **Implement a notification system** using Strategy pattern that can send via Email, SMS, and Push, with the ability to add new channels without modifying existing code.

2. **Create a logging decorator chain** that adds: timestamp, log level filtering, and output redirection (console vs file) to a base logger.

3. **Build a document processing pipeline** using Chain of Responsibility for: virus scanning, format conversion, compression, and encryption.

4. **Design a UI form builder** using Builder pattern that constructs complex forms with validation rules, conditional fields, and custom layouts.

5. **Implement a simple ORM** using Repository and Unit of Work patterns that tracks entity changes and commits them in a transaction.

---

## 🎤 Interview Cheat Sheet

**Quick answers for common questions:**

**"What design patterns have you used?"**
> "I regularly use Repository pattern with Entity Framework for data access abstraction, Strategy pattern for implementing different payment processors or notification channels, and Decorator pattern for adding cross-cutting concerns like logging and caching. In ASP.NET Core, I use the built-in DI container which implements Factory and Singleton patterns, and the middleware pipeline is essentially Chain of Responsibility."

**"When would you use Singleton pattern?"**
> "In modern .NET with DI, I rarely implement Singleton manually—I use `AddSingleton()` in the DI container instead. This provides singleton behavior while maintaining testability. I use it for stateless services like configuration readers, caching services, or expensive-to-create objects like HttpClient factories. I avoid it for anything with per-request state or when different test contexts need different instances."

**"Explain Repository pattern. When is it useful?"**
> "Repository pattern abstracts data access, providing a collection-like interface over your data store. It's useful when you need to swap data sources, add caching layers, or isolate business logic from data access concerns. However, with Entity Framework, DbContext already provides abstraction, so I typically use specialized repositories only for complex queries or when I need additional abstraction beyond what EF provides."

**"What's the difference between Factory and Abstract Factory?"**
> "Factory creates objects of a single type, deciding which implementation to instantiate. Abstract Factory creates families of related objects—like UI components for different platforms where you need a Button, TextBox, and Menu that all match the same theme. In practice, I use simple Factory most often, and Abstract Factory only when I truly need to create coordinated groups of related objects."

**"How do you avoid pattern overuse?"**
> "I follow YAGNI (You Aren't Gonna Need It) and start with the simplest solution. I add patterns when I have a concrete problem to solve—like when I need to support multiple payment gateways (Strategy), or when I have complex object creation (Factory), or when I need to add behavior without modifying existing code (Decorator). A simple `if` statement or direct instantiation is often better than a premature abstraction."

**"Describe Decorator pattern with an example."**
> "Decorator adds behavior to objects dynamically without modifying their class. For example, I built a data service where the base implementation fetched from a database. I added a caching decorator that checked cache before calling the base service, a logging decorator that logged all calls, and a retry decorator that handled transient failures. Each decorator wrapped the previous one, creating a pipeline: Logging → Caching → Retry → Base. This let me add or remove behaviors without changing the core service code."

**"What anti-patterns should you avoid?"**
> "God Objects that do everything—violate Single Responsibility. Anemic Domain Models where objects are just data bags with no behavior—business logic should live in domain objects. Premature abstraction—adding patterns before you need them. Singleton abuse—makes testing hard and creates hidden dependencies. Over-engineering with unnecessary interfaces and abstractions when simple code would work."

**Key phrases that impress:**
- "I prefer composition over inheritance"
- "Let the DI container manage lifetimes"
- "I follow SOLID principles, especially Single Responsibility and Dependency Inversion"
- "Patterns solve specific problems; I don't use them just for the sake of it"
- "In modern C#, many patterns are built into the framework"
- "Testability is a key consideration when choosing patterns"

---

**TL;DR for interviews:**

Patterns you MUST know:
- **Singleton**: One instance (but use DI instead)
- **Factory**: Creates objects based on conditions
- **Repository**: Data access abstraction
- **Strategy**: Swap algorithms at runtime
- **Decorator**: Add behavior dynamically
- **Observer**: Event-based notification (C# events)

Real-world examples:
- Repository: Entity Framework data access
- Factory: ASP.NET Core service registration
- Singleton: Configuration, caching (via DI)
- Strategy: Payment processing, notification channels
- Decorator: Logging, caching, retry logic
- Observer: UI event handlers, pub/sub systems

Remember: Patterns are tools, not goals. Use them to solve real problems, not to make code look sophisticated.
