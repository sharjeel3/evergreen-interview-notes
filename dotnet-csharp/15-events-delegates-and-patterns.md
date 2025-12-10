# Events, Delegates & Event Patterns

**Level:** 🚦 Intermediate  
**Tags:** `events`, `delegates`, `event-pattern`, `memory-leaks`, `observer`, `callbacks`, `interview`

---

## Why this matters

Events and delegates are fundamental to .NET's event-driven architecture, yet they're misunderstood more often than mastered. From Windows Forms to ASP.NET Core, from UI frameworks to domain events, the event pattern appears everywhere—and so do the bugs. Memory leaks from unsubscribed events, race conditions from event handlers, and architectural coupling from misused events plague production systems. Understanding events deeply separates developers who cargo-cult patterns from those who design clean, maintainable event-driven systems.

**Event-related memory leaks are among the most common and hardest to debug.** I've investigated production applications with mysteriously growing memory usage—GBs of leaked objects—traced to event subscriptions never removed. A UI component subscribes to a service's event, the component is destroyed, but the subscription keeps it alive indefinitely. Multiply by thousands of components over hours of runtime, and you have a memory leak that crashes the application. One team's WPF application leaked 50MB per form opened because form closed events weren't unsubscribed, keeping entire object graphs in memory. The fix was trivial—unsubscribe in Dispose—but finding it required heap analysis and understanding how event subscriptions create GC roots.

**Interviews test events because they reveal understanding of reference semantics and lifecycle management.** When an interviewer asks "What happens when you subscribe to an event but don't unsubscribe?", they're checking if you understand that the event publisher holds a reference to the subscriber, preventing garbage collection. Can you explain why `+=` creates a strong reference? Do you know the weak event pattern for avoiding leaks? Have you debugged real event-driven systems? These aren't trivia questions—they predict whether your code will leak memory in production.

**Delegates are the mechanism underlying events, yet many developers use them without understanding them.** A delegate is a type-safe function pointer—an object that references methods. This enables callbacks, event handlers, and LINQ's functional style. I've seen code where developers couldn't explain the difference between `Action`, `Func`, and custom delegates, or when to use multicast delegates versus single-cast. Understanding delegates is prerequisite to understanding events, async callbacks, dependency injection, and functional programming patterns. When you can explain how delegates enable inversion of control and why they're reference types, you demonstrate foundational C# knowledge.

**The standard event pattern (sender, EventArgs) exists for good reasons but is often violated.** The pattern—`public event EventHandler<TEventArgs> EventName`—provides consistency, tooling support, and weak reference opportunities. But I've reviewed codebases with inconsistent event signatures, events that don't follow naming conventions, and events that leak implementation details. One team exposed `Action` callbacks as events, losing type safety and convention. Another used custom event signatures incompatible with standard handlers, preventing use of += operator. Following the standard pattern isn't pedantry—it's professional practice that improves maintainability and enables frameworks to work with your events.

**Multicast delegates (events with multiple subscribers) introduce subtle ordering and exception handling issues.** When multiple handlers subscribe to an event, they execute sequentially in subscription order—but if one throws an exception, subsequent handlers don't run. I've debugged systems where critical cleanup handlers never executed because an earlier handler threw. The event publisher sees the exception, but other subscribers are silently skipped. Understanding this behavior and techniques for exception-safe multicast invocation (try-catch per handler) is essential for reliable event-driven systems.

**Async event handlers are a trap that even experienced developers fall into.** Declaring an event handler as `async void` seems natural for async work, but it breaks error handling—exceptions in async void handlers crash the application. I've investigated production crashes where async event handlers threw exceptions that weren't caught anywhere, terminating the process. The correct pattern—return Task from handlers and await invocation—is non-obvious and requires understanding the interaction between events and async. This topic appears in senior interviews because it requires deep understanding of both features.

**Event sourcing and domain events are architectural patterns increasingly common in modern systems.** Beyond UI events, domain events represent state changes in business logic. "Order placed," "Payment processed," "Inventory depleted"—these events enable decoupled, observable systems. I've helped teams refactor monolithic services into event-driven microservices using domain events for communication. Understanding when to use events for architectural communication versus direct calls, how to make events immutable, and patterns for guaranteed delivery requires systems thinking beyond basic event syntax.

**The observer pattern, implemented via events, is one of the Gang of Four patterns tested in interviews.** Events are C#'s native implementation of observer pattern—separating subjects (event publishers) from observers (event subscribers). Interviewers ask you to implement observer pattern to see if you recognize that events are the .NET solution. Can you articulate the benefits (decoupling, one-to-many notification) and drawbacks (memory leaks, implicit dependencies)? Understanding design patterns through actual language features demonstrates practical knowledge versus academic memorization.

**Custom event accessors (add/remove) enable advanced scenarios but are rarely taught.** Most events use automatic add/remove, but custom accessors let you control subscription behavior—thread safety, subscription limits, weak references, logging. I've implemented custom event accessors to enforce single-subscriber invariants, integrate with message buses, and prevent memory leaks through weak references. This is advanced, but knowing it's possible and when you'd need it shows depth of C# expertise that senior roles value.

**Event-driven architectures scale well but are harder to debug and reason about.** Versus direct method calls with clear stack traces, events create indirect coupling that's harder to trace. When a button click triggers an event that triggers another event that triggers a background operation that fails... debugging requires understanding the entire event chain. I've spent days debugging event-driven systems where the connection between cause and effect was obscured by multiple event layers. Tools like event tracing and proper logging become essential. Understanding these debugging challenges prevents creating event spaghetti that's impossible to maintain.

**Career impact: Event expertise is expected at senior+ levels.** You'll design pub-sub systems, implement observable domain models, integrate with message brokers, and debug event-driven microservices. When you can architect an event-driven system that doesn't leak memory, handles errors gracefully, and remains debuggable at scale, you're operating at a high level. Conversely, not understanding events limits you to simpler codebases and prevents you from participating in architectural decisions about event-driven design.

---

## Core Ideas

### 1. **Delegates: Type-Safe Function Pointers**

A **delegate** is a type that represents references to methods with a specific signature.

```csharp
// Define a delegate type
public delegate int MathOperation(int a, int b);

// Methods that match the signature
public int Add(int a, int b) => a + b;
public int Multiply(int a, int b) => a * b;

// Use delegate
MathOperation operation = Add;
int result = operation(5, 3);  // Calls Add => 8

operation = Multiply;
result = operation(5, 3);  // Calls Multiply => 15
```

**Built-in delegate types:**
```csharp
// Action: void return, 0-16 parameters
Action action = () => Console.WriteLine("Hello");
Action<int> actionWithParam = (x) => Console.WriteLine(x);

// Func: returns T, 0-16 parameters
Func<int> getNumber = () => 42;
Func<int, int, int> add = (a, b) => a + b;

// Predicate: returns bool, 1 parameter
Predicate<int> isEven = (x) => x % 2 == 0;
```

**Mental model:**  
"A delegate is an object that knows how to call a method."

---

### 2. **Multicast Delegates: Multiple Subscribers**

Delegates can reference multiple methods (multicast):

```csharp
Action action = Method1;
action += Method2;  // Add second method
action += Method3;  // Add third method

action();  // Calls Method1, then Method2, then Method3

action -= Method2;  // Remove Method2
action();  // Calls Method1, then Method3
```

**Important behavior:**
- Methods invoked in subscription order
- If one throws exception, subsequent methods don't run
- Return value is from last method only (other returns are lost)

---

### 3. **Events: Publisher-Subscriber Pattern**

An **event** is a special delegate that enforces publisher-subscriber pattern.

```csharp
public class Button
{
    // Event declaration
    public event EventHandler Clicked;
    
    // Raise event
    public void OnClick()
    {
        Clicked?.Invoke(this, EventArgs.Empty);
    }
}

// Subscriber
var button = new Button();
button.Clicked += (sender, e) => Console.WriteLine("Button clicked!");
button.OnClick();  // Triggers event
```

**Event vs Delegate:**
```csharp
// ❌ Delegate: anyone can invoke or reassign
public Action MyAction;
MyAction = null;  // External code can clear all subscribers!

// ✅ Event: only owner can invoke, subscribers can only += or -=
public event Action MyEvent;
// MyEvent = null;  // ❌ Compile error outside declaring class
```

**Mental model:**  
"Events are protected delegates that prevent external code from invoking or clearing subscribers."

---

### 4. **Standard Event Pattern**

**.NET convention for events:**

```csharp
// Standard EventArgs
public class OrderEventArgs : EventArgs
{
    public int OrderId { get; set; }
    public decimal Total { get; set; }
}

// Publisher
public class OrderService
{
    // Standard event signature: EventHandler<TEventArgs>
    public event EventHandler<OrderEventArgs> OrderPlaced;
    
    protected virtual void OnOrderPlaced(OrderEventArgs e)
    {
        OrderPlaced?.Invoke(this, e);
    }
    
    public void PlaceOrder(int orderId, decimal total)
    {
        // Business logic
        
        // Raise event
        OnOrderPlaced(new OrderEventArgs 
        { 
            OrderId = orderId, 
            Total = total 
        });
    }
}

// Subscriber
var service = new OrderService();
service.OrderPlaced += (sender, e) =>
{
    Console.WriteLine($"Order {e.OrderId} placed: ${e.Total}");
};
```

**Why this pattern:**
- **Consistency** — all .NET events follow same structure
- **Tooling** — Visual Studio understands this pattern
- **Extensibility** — easy to add more event data
- **Sender reference** — handler knows who raised the event

---

### 5. **Memory Leaks from Event Subscriptions**

**Problem: Event subscriptions create strong references**

```csharp
public class Publisher
{
    public event EventHandler SomeEvent;
}

public class Subscriber
{
    public Subscriber(Publisher publisher)
    {
        publisher.SomeEvent += HandleEvent;  // Publisher now holds reference to this
    }
    
    private void HandleEvent(object sender, EventArgs e) { }
}

// ❌ Memory leak:
var publisher = new Publisher();
var subscriber = new Subscriber(publisher);
subscriber = null;  // subscriber object NOT garbage collected!
// Why? publisher.SomeEvent still holds reference to subscriber.HandleEvent
```

**Solution: Always unsubscribe**

```csharp
public class Subscriber : IDisposable
{
    private readonly Publisher _publisher;
    
    public Subscriber(Publisher publisher)
    {
        _publisher = publisher;
        _publisher.SomeEvent += HandleEvent;
    }
    
    public void Dispose()
    {
        _publisher.SomeEvent -= HandleEvent;  // ✅ Remove subscription
    }
    
    private void HandleEvent(object sender, EventArgs e) { }
}

// Usage
using (var subscriber = new Subscriber(publisher))
{
    // Use subscriber
}  // Dispose automatically unsubscribes
```

**Alternative: Weak event pattern (advanced)**
```csharp
// Use WeakEventManager to avoid strong references
WeakEventManager<Publisher, EventArgs>
    .AddHandler(publisher, nameof(Publisher.SomeEvent), HandleEvent);
```

---

### 6. **Thread Safety in Event Invocation**

**Problem: Race condition when invoking events**

```csharp
// ❌ NOT THREAD-SAFE: event could become null between check and invoke
if (MyEvent != null)
{
    MyEvent(this, EventArgs.Empty);  // Another thread might unsubscribe here
}

// ✅ THREAD-SAFE: copy reference
var handler = MyEvent;
if (handler != null)
{
    handler(this, EventArgs.Empty);
}

// ✅ BETTER: null-conditional operator (does the copy automatically)
MyEvent?.Invoke(this, EventArgs.Empty);
```

---

### 7. **Multicast Exception Handling**

**Problem: First exception stops subsequent handlers**

```csharp
public event EventHandler MyEvent;

MyEvent += (s, e) => Console.WriteLine("Handler 1");
MyEvent += (s, e) => throw new Exception("Handler 2 fails");
MyEvent += (s, e) => Console.WriteLine("Handler 3");  // Never runs!

MyEvent?.Invoke(this, EventArgs.Empty);
// Output: "Handler 1", then exception thrown
```

**Solution: Invoke handlers individually**

```csharp
protected virtual void OnMyEvent(EventArgs e)
{
    var handler = MyEvent;
    if (handler == null) return;
    
    foreach (Delegate d in handler.GetInvocationList())
    {
        try
        {
            d.DynamicInvoke(this, e);
        }
        catch (Exception ex)
        {
            // Log exception but continue invoking other handlers
            Console.WriteLine($"Handler exception: {ex.Message}");
        }
    }
}
```

---

### 8. **Async Event Handlers**

**❌ WRONG: async void event handlers**
```csharp
// Exceptions crash the application!
myEvent += async (sender, e) =>
{
    await SomeAsyncWork();  // If this throws, app crashes
};
```

**✅ RIGHT: Return Task and await invocation**
```csharp
// Define event with Func<Task>
public event Func<object, EventArgs, Task> AsyncEvent;

// Invoke and await all handlers
protected virtual async Task OnAsyncEvent(EventArgs e)
{
    var handler = AsyncEvent;
    if (handler != null)
    {
        await Task.WhenAll(
            handler.GetInvocationList()
                .Cast<Func<object, EventArgs, Task>>()
                .Select(h => h(this, e))
        );
    }
}

// Subscribe
AsyncEvent += async (sender, e) =>
{
    await SomeAsyncWork();  // Exceptions handled properly
};
```

---

## Examples

### Example 1 — Simple event implementation

```csharp
public class TemperatureSensor
{
    public event EventHandler<TemperatureEventArgs> TemperatureChanged;
    
    private double _temperature;
    
    public double Temperature
    {
        get => _temperature;
        set
        {
            if (_temperature != value)
            {
                _temperature = value;
                OnTemperatureChanged(new TemperatureEventArgs { Temperature = value });
            }
        }
    }
    
    protected virtual void OnTemperatureChanged(TemperatureEventArgs e)
    {
        TemperatureChanged?.Invoke(this, e);
    }
}

public class TemperatureEventArgs : EventArgs
{
    public double Temperature { get; set; }
}

// Usage
var sensor = new TemperatureSensor();
sensor.TemperatureChanged += (sender, e) =>
{
    Console.WriteLine($"Temperature changed to {e.Temperature}°C");
};

sensor.Temperature = 25.5;  // Triggers event
```

---

### Example 2 — Proper subscription/unsubscription

```csharp
public class DataMonitor : IDisposable
{
    private readonly DataSource _source;
    
    public DataMonitor(DataSource source)
    {
        _source = source;
        _source.DataReceived += OnDataReceived;  // Subscribe
    }
    
    private void OnDataReceived(object sender, DataEventArgs e)
    {
        Console.WriteLine($"Received: {e.Data}");
    }
    
    public void Dispose()
    {
        _source.DataReceived -= OnDataReceived;  // ✅ Unsubscribe
    }
}

// Usage
using var monitor = new DataMonitor(dataSource);
// Monitor automatically unsubscribes when disposed
```

---

### Example 3 — Custom event accessor for logging

```csharp
public class Button
{
    private EventHandler _clicked;
    
    // Custom event accessor
    public event EventHandler Clicked
    {
        add
        {
            Console.WriteLine("Subscriber added");
            _clicked += value;
        }
        remove
        {
            Console.WriteLine("Subscriber removed");
            _clicked -= value;
        }
    }
    
    public void Click()
    {
        _clicked?.Invoke(this, EventArgs.Empty);
    }
}
```

---

### Example 4 — Domain events pattern

```csharp
// Domain event
public class OrderPlacedEvent
{
    public int OrderId { get; set; }
    public string CustomerId { get; set; }
    public decimal Total { get; set; }
    public DateTime PlacedAt { get; set; }
}

// Event publisher
public class Order
{
    public event EventHandler<OrderPlacedEvent> OrderPlaced;
    
    public void Place(string customerId, decimal total)
    {
        var orderId = SaveToDatabase();
        
        // Raise domain event
        OrderPlaced?.Invoke(this, new OrderPlacedEvent
        {
            OrderId = orderId,
            CustomerId = customerId,
            Total = total,
            PlacedAt = DateTime.UtcNow
        });
    }
}

// Event subscribers (different concerns)
public class EmailNotifier
{
    public EmailNotifier(Order order)
    {
        order.OrderPlaced += async (sender, e) =>
        {
            await SendConfirmationEmail(e.CustomerId, e.OrderId);
        };
    }
}

public class InventoryUpdater
{
    public InventoryUpdater(Order order)
    {
        order.OrderPlaced += (sender, e) =>
        {
            UpdateInventory(e.OrderId);
        };
    }
}
```

---

## Common Pitfalls

### ❌ 1. Not unsubscribing from events

```csharp
// ❌ Memory leak: subscriber never GC'd
publisher.SomeEvent += HandleEvent;

// ✅ Always unsubscribe
publisher.SomeEvent -= HandleEvent;
// Or implement IDisposable
```

---

### ❌ 2. Using async void event handlers

```csharp
// ❌ Exceptions crash app
myEvent += async (s, e) => await DoWork();

// ✅ Use event that returns Task
public event Func<object, EventArgs, Task> AsyncEvent;
```

---

### ❌ 3. Not checking for null before invoking

```csharp
// ❌ NullReferenceException if no subscribers
MyEvent(this, EventArgs.Empty);

// ✅ Null-conditional operator
MyEvent?.Invoke(this, EventArgs.Empty);
```

---

### ❌ 4. Exposing delegates instead of events

```csharp
// ❌ External code can invoke or clear
public Action MyCallback;

// ✅ Use event
public event Action MyEvent;
```

---

### ❌ 5. Assuming event handlers run on specific thread

```csharp
// ❌ Event may be raised on background thread
myEvent += (s, e) =>
{
    // Don't assume this runs on UI thread!
    myTextBox.Text = "Updated";  // May throw cross-thread exception
};

// ✅ Marshal to UI thread if needed
myEvent += (s, e) =>
{
    Dispatcher.Invoke(() => myTextBox.Text = "Updated");
};
```

---

## Quick Self-Check

You should be able to answer these out loud:

1. What's the difference between a delegate and an event?
2. How do event subscriptions cause memory leaks?
3. What's the standard .NET event pattern?
4. How do you safely invoke an event that might have no subscribers?
5. What happens if one event handler throws an exception?
6. Why is `async void` dangerous in event handlers?
7. What's a multicast delegate?
8. How do you properly unsubscribe from events?

If you struggle with any of these, revisit the relevant section.

---

## Further Practice

1. Implement a simple pub-sub system using events.
2. Create a memory leak with event subscriptions, then fix it.
3. Build a domain events system for an order processing workflow.
4. Implement custom event accessors with logging or validation.
5. Profile memory usage before and after fixing event subscription leaks.

---

## Mini Interview Cheat Sheet

**Explain events in 30 seconds:**
"Events implement the observer pattern—publishers raise events, subscribers handle them. Events are protected delegates that prevent external code from invoking or clearing subscribers. Always unsubscribe to prevent memory leaks."

**Delegate vs Event:**
- **Delegate** — can be invoked, reassigned, cleared by anyone
- **Event** — only owner can invoke, subscribers only += or -=

**Memory leak prevention:**
"Event subscriptions create strong references. Always unsubscribe in Dispose or use weak event pattern. Publisher holds reference to subscriber's handler, preventing GC."

**Standard pattern:**
```csharp
public event EventHandler<TEventArgs> EventName;
protected virtual void OnEventName(TEventArgs e)
{
    EventName?.Invoke(this, e);
}
```

**Common types:**
- **Action** — void return
- **Func<T>** — returns T
- **EventHandler<T>** — standard event delegate
- **Predicate<T>** — returns bool

---
