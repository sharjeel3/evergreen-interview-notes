# Object-Oriented Programming in C#

**Level:** 🏁 Beginner  
**Tags:** `OOP`, `inheritance`, `polymorphism`, `interfaces`, `encapsulation`, `design`, `interview`

---

## Why this matters

Object-oriented programming is the architectural foundation of C# and .NET, yet many developers use OOP features without truly understanding the principles behind them. Interviews heavily test OOP because it reveals your ability to design maintainable, extensible systems—a core skill that distinguishes senior engineers from code writers.

**OOP questions are gatekeepers in technical interviews.** When an interviewer asks "Explain the difference between an interface and an abstract class," they're not testing memorization—they're assessing your design thinking. Can you choose the right abstraction? Do you understand when composition beats inheritance? Have you actually designed systems, or just written code? I've seen brilliant algorithm experts fail interviews because they couldn't articulate basic OOP design principles. The ability to explain *why* you'd use an interface over inheritance, or when to seal a class, demonstrates architectural maturity.

**Production code quality directly correlates with OOP mastery.** Poor OOP design creates codebases that become increasingly expensive to maintain. I've inherited systems where everything was in one giant class (no encapsulation), where inheritance hierarchies were 7 levels deep (fragile base class problem), and where interfaces had 40 methods (interface segregation violation). These aren't academic problems—they cost companies millions in technical debt. One team spent 18 months refactoring a poorly designed inheritance hierarchy that made adding features take 10x longer than it should. Good OOP design prevents these disasters.

**Polymorphism enables extensibility without modification.** The Open/Closed Principle—"open for extension, closed for modification"—isn't just theory. In production, this means you can add new payment processors, notification channels, or data sources without touching existing tested code. I've seen systems where adding a new feature required changing 30 files because the original design lacked proper abstraction. Versus systems where adding a feature meant implementing one interface in one new file. That's the difference between polymorphism done right and wrong.

**Encapsulation prevents entire classes of bugs.** When state is properly encapsulated with properties and validation, invalid states become impossible. I debugged a production issue where a negative balance was possible in a banking system because fields were public instead of properly encapsulated with validation. The bug had existed for years, causing incorrect interest calculations totaling thousands of dollars. Proper encapsulation with a validated property setter would have prevented this entirely. Interviews test encapsulation because it's a proxy for defensive programming and design by contract.

**Interface design impacts testability and dependency injection.** Modern .NET development relies heavily on dependency injection, which requires programming to interfaces rather than concrete implementations. Teams that don't understand interfaces write untestable code—classes with hardcoded dependencies that can't be mocked. I've reviewed codebases where unit testing was nearly impossible because everything was tightly coupled to concrete types. The shift from `new DatabaseService()` to injecting `IDataService` seems small but fundamentally changes architecture and testability.

**Inheritance hierarchies become maintenance nightmares when misused.** The "fragile base class problem" is real: change a base class method, break 20 derived classes. I've seen production outages caused by well-intentioned changes to base classes that had unexpected ripple effects. Modern wisdom favors composition over inheritance, but you need to understand *why*. Interviews love asking "When would you use inheritance vs composition?" because the answer reveals whether you've been burned by inheritance hell and learned the lesson.

**Virtual, override, and sealed keywords control extensibility contracts.** These modifiers define which methods can be overridden, which must be overridden, and which are locked down. Getting this wrong causes runtime bugs. I've debugged issues where derived classes override methods but forget to call `base.Method()`, breaking invariants. Or where `sealed` would have prevented dangerous overrides but wasn't used. Understanding these keywords shows you think about class contracts and API design.

**Abstract classes vs interfaces is a design choice with real implications.** Interfaces allow multiple inheritance (implementation), abstract classes provide shared behavior. Choosing wrong makes refactoring expensive. I've seen codebases where interfaces should have been abstract classes (to share common logic) and abstract classes that should have been interfaces (to allow multiple implementation). One team spent 3 months refactoring an abstract class hierarchy to interfaces because they needed classes to implement multiple abstractions. That work could have been avoided with better initial design.

**Records, structs, and classes serve different purposes.** Modern C# offers records for immutable data DTOs, structs for small value types, and classes for mutable reference types. Mixing these up causes performance issues and unexpected behavior. I've seen codebases where large structs caused massive memory copies, or where mutable records violated immutability contracts, or where classes were used for DTOs causing unnecessary heap allocations. Knowing when to use which type shows understanding of memory models and performance.

**SOLID principles are interview staples because they encode best practices.** Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, Dependency Inversion—these aren't academic. They're learned through painful production experiences. Every SOLID principle solves real problems: SRP prevents god classes, LSP prevents contract violations, ISP prevents bloated interfaces. Interviews ask about SOLID because it's shorthand for "have you designed real systems and learned from mistakes?"

**Career progression requires OOP design skills.** Junior developers write methods. Mid-level developers write classes. Senior developers design systems of classes that are maintainable, testable, and extensible. Staff+ engineers architect systems that last years and handle changing requirements gracefully. That progression is impossible without mastering OOP. When you can explain the tradeoffs between inheritance and composition, design a clean interface hierarchy, and apply SOLID principles naturally, you're ready for senior roles.

---

## Core Ideas

### 1. **Encapsulation: Hiding Implementation Details**

**Encapsulation** means bundling data and methods that operate on that data, and restricting direct access to internal state.

**Goal:** Control how state is accessed and modified.

```csharp
public class BankAccount
{
    private decimal _balance;  // Private field

    public decimal Balance => _balance;  // Read-only public property

    public void Deposit(decimal amount)
    {
        if (amount <= 0)
            throw new ArgumentException("Amount must be positive");
        
        _balance += amount;
    }

    public void Withdraw(decimal amount)
    {
        if (amount > _balance)
            throw new InvalidOperationException("Insufficient funds");
        
        _balance -= amount;
    }
}
```

**Why it matters:**
- **Validation** — enforce business rules (no negative deposits)
- **Invariants** — maintain valid state (balance never negative after withdrawal check)
- **Change isolation** — change internal implementation without breaking callers

**Mental model:**  
"Encapsulation is about controlling access, not just hiding data."

---

### 2. **Inheritance: Reusing Code Through Hierarchies**

**Inheritance** allows a class to reuse code from a base class.

```csharp
public class Animal
{
    public string Name { get; set; }
    
    public virtual void MakeSound()
    {
        Console.WriteLine("Some generic sound");
    }
}

public class Dog : Animal
{
    public override void MakeSound()
    {
        Console.WriteLine("Woof!");
    }
}

public class Cat : Animal
{
    public override void MakeSound()
    {
        Console.WriteLine("Meow!");
    }
}
```

**Key concepts:**
- **Base class** — parent class (`Animal`)
- **Derived class** — child class (`Dog`, `Cat`)
- **Virtual** — method can be overridden in derived classes
- **Override** — provides new implementation in derived class
- **Base** — calls base class implementation

**When to use inheritance:**
- Clear "is-a" relationship (`Dog` *is an* `Animal`)
- Need to share common behavior
- Polymorphism through base class references

**⚠️ Caution:**  
Favor composition over inheritance in most cases. Inheritance creates tight coupling.

---

### 3. **Polymorphism: One Interface, Many Implementations**

**Polymorphism** allows objects of different types to be treated uniformly through a common interface or base class.

```csharp
List<Animal> animals = new List<Animal>
{
    new Dog { Name = "Buddy" },
    new Cat { Name = "Whiskers" }
};

foreach (var animal in animals)
{
    animal.MakeSound();  // Calls correct implementation at runtime
}

// Output:
// Woof!
// Meow!
```

**How it works:**
- Method calls are resolved at **runtime** (late binding)
- The actual type of the object determines which method runs
- Enables writing code that works with base types but operates on derived types

**Mental model:**  
"Polymorphism means the same code can work with different types without knowing their specifics."

---

### 4. **Interfaces: Contracts Without Implementation**

An **interface** defines a contract—what methods/properties a type must implement, but not *how*.

```csharp
public interface IPaymentProcessor
{
    Task<bool> ProcessPayment(decimal amount);
    Task<bool> RefundPayment(string transactionId);
}

public class StripeProcessor : IPaymentProcessor
{
    public async Task<bool> ProcessPayment(decimal amount)
    {
        // Stripe-specific implementation
        return true;
    }

    public async Task<bool> RefundPayment(string transactionId)
    {
        // Stripe-specific refund logic
        return true;
    }
}

public class PayPalProcessor : IPaymentProcessor
{
    public async Task<bool> ProcessPayment(decimal amount)
    {
        // PayPal-specific implementation
        return true;
    }

    public async Task<bool> RefundPayment(string transactionId)
    {
        // PayPal-specific refund logic
        return true;
    }
}
```

**Why interfaces matter:**
- **Dependency injection** — program to abstractions, not implementations
- **Testability** — mock interfaces in unit tests
- **Multiple inheritance** — a class can implement many interfaces
- **Decoupling** — change implementation without changing callers

---

### 5. **Abstract Classes: Shared Behavior + Enforced Contracts**

An **abstract class** provides shared implementation *and* enforces that derived classes implement certain members.

```csharp
public abstract class Shape
{
    public string Color { get; set; }  // Shared property

    public abstract double CalculateArea();  // Must be implemented

    public void Display()  // Shared method
    {
        Console.WriteLine($"Shape color: {Color}, Area: {CalculateArea()}");
    }
}

public class Circle : Shape
{
    public double Radius { get; set; }

    public override double CalculateArea()
    {
        return Math.PI * Radius * Radius;
    }
}

public class Rectangle : Shape
{
    public double Width { get; set; }
    public double Height { get; set; }

    public override double CalculateArea()
    {
        return Width * Height;
    }
}
```

**Abstract class vs Interface:**

| Feature | Interface | Abstract Class |
|---------|-----------|----------------|
| Multiple inheritance | ✅ Yes (implement many) | ❌ No (inherit one) |
| Shared implementation | ❌ No (C# 8+ allows default) | ✅ Yes |
| Fields | ❌ No | ✅ Yes |
| Constructors | ❌ No | ✅ Yes |
| When to use | Contract-only | Shared behavior + contract |

---

### 6. **Sealed Classes: Preventing Inheritance**

**Sealed** prevents a class from being inherited.

```csharp
public sealed class Configuration
{
    public string ApiKey { get; set; }
    public string DatabaseConnection { get; set; }
}

// ❌ Compile error
public class ExtendedConfig : Configuration { }  // Can't inherit from sealed class
```

**When to seal:**
- Class wasn't designed for inheritance
- Security-sensitive classes
- Performance (sealed methods can be devirtualized by JIT)

---

### 7. **Composition Over Inheritance**

**Composition** means building complex objects by combining simpler ones, rather than inheriting.

```csharp
// ❌ Inheritance approach (tight coupling)
public class FlyingCar : Car, IFlyable  // Can't do this—C# doesn't support multiple inheritance
{
}

// ✅ Composition approach (flexible)
public class FlyingCar
{
    private readonly Car _car;
    private readonly Airplane _airplane;

    public FlyingCar()
    {
        _car = new Car();
        _airplane = new Airplane();
    }

    public void Drive() => _car.Drive();
    public void Fly() => _airplane.Fly();
}
```

**Why composition wins:**
- **Flexibility** — mix and match behaviors
- **No fragile base class problem**
- **Easier to test** (mock composed dependencies)
- **Avoids deep inheritance hierarchies**

**Rule of thumb:**  
"Prefer composition unless there's a clear 'is-a' relationship."

---

### 8. **Struct vs Class vs Record**

| Type | Memory | Mutability | Use Case |
|------|--------|------------|----------|
| **Class** | Heap (reference) | Mutable | Complex objects, inheritance |
| **Struct** | Stack (value) | Mutable (avoid) | Small data (<16 bytes), high-performance |
| **Record** | Heap (reference) | Immutable | DTOs, value objects, immutable data |

```csharp
// Class: mutable reference type
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
}

// Struct: value type (copy semantics)
public struct Point
{
    public int X { get; init; }
    public int Y { get; init; }
}

// Record: immutable reference type
public record PersonDto(string Name, int Age);
```

---

## Examples

### Example 1 — Polymorphism with interfaces

```csharp
public interface ILogger
{
    void Log(string message);
}

public class ConsoleLogger : ILogger
{
    public void Log(string message) => Console.WriteLine(message);
}

public class FileLogger : ILogger
{
    public void Log(string message) => File.AppendAllText("log.txt", message);
}

// ✅ Client code works with abstraction
public class OrderService
{
    private readonly ILogger _logger;

    public OrderService(ILogger logger)
    {
        _logger = logger;  // Can be ConsoleLogger, FileLogger, or TestLogger
    }

    public void ProcessOrder()
    {
        _logger.Log("Order processed");
    }
}
```

---

### Example 2 — Abstract class with shared behavior

```csharp
public abstract class Repository<T>
{
    protected string ConnectionString { get; set; }

    // Shared implementation
    protected async Task<SqlConnection> GetConnection()
    {
        var connection = new SqlConnection(ConnectionString);
        await connection.OpenAsync();
        return connection;
    }

    // Enforced contract
    public abstract Task<T> GetById(int id);
    public abstract Task<bool> Save(T entity);
}

public class UserRepository : Repository<User>
{
    public override async Task<User> GetById(int id)
    {
        using var connection = await GetConnection();  // Reuses base class logic
        // User-specific query logic
        return new User();
    }

    public override async Task<bool> Save(User entity)
    {
        using var connection = await GetConnection();
        // User-specific save logic
        return true;
    }
}
```

---

### Example 3 — Virtual, override, and sealed

```csharp
public class BaseClass
{
    public virtual void Method1()  // Can be overridden
    {
        Console.WriteLine("Base Method1");
    }

    public void Method2()  // Cannot be overridden (not virtual)
    {
        Console.WriteLine("Base Method2");
    }
}

public class DerivedClass : BaseClass
{
    public override void Method1()  // Overrides base implementation
    {
        Console.WriteLine("Derived Method1");
        base.Method1();  // Optionally call base
    }

    public sealed override void Method1() { }  // Can't be overridden further
}
```

---

## Common Pitfalls

### ❌ 1. Deep inheritance hierarchies

```csharp
// ❌ BAD: Fragile, hard to maintain
Animal → Mammal → Carnivore → Feline → DomesticCat → PersianCat

// ✅ GOOD: Shallow hierarchy + composition
Animal → Cat (composed with Traits: IDomestic, IPersian)
```

---

### ❌ 2. Exposing mutable collections

```csharp
// ❌ BAD: Callers can modify internal state
public class Team
{
    public List<Player> Players { get; set; }  // Mutable!
}

// ✅ GOOD: Return read-only view
public class Team
{
    private readonly List<Player> _players = new();
    
    public IReadOnlyCollection<Player> Players => _players.AsReadOnly();
}
```

---

### ❌ 3. Not using interfaces for dependencies

```csharp
// ❌ BAD: Tightly coupled to concrete type
public class OrderService
{
    private readonly SqlDatabase _database = new();  // Can't test, can't swap
}

// ✅ GOOD: Depend on abstraction
public class OrderService
{
    private readonly IDatabase _database;
    
    public OrderService(IDatabase database)
    {
        _database = database;  // Testable, swappable
    }
}
```

---

### ❌ 4. Forgetting to call base methods

```csharp
public class Base
{
    public virtual void Initialize()
    {
        // Critical setup logic
    }
}

public class Derived : Base
{
    public override void Initialize()
    {
        // ❌ Forgot to call base.Initialize() — breaks base class invariants!
        // Custom logic
    }
}

// ✅ Always call base unless explicitly replacing behavior
public override void Initialize()
{
    base.Initialize();  // ✅
    // Custom logic
}
```

---

### ❌ 5. Using struct for large data

```csharp
// ❌ BAD: Struct is 100+ bytes, copied on every assignment
public struct LargeData
{
    public string Name;
    public string Address;
    public DateTime BirthDate;
    // ... 20 more fields
}

// ✅ GOOD: Use class for large data
public class LargeData { /* ... */ }
```

---

## Quick Self-Check

You should be able to answer these out loud:

1. What are the four pillars of OOP? (Encapsulation, Inheritance, Polymorphism, Abstraction)
2. When should you use an interface vs an abstract class?
3. What's the difference between `virtual`, `override`, and `sealed`?
4. Why favor composition over inheritance?
5. When should you use a struct instead of a class?
6. What's polymorphism and why does it matter?
7. How do interfaces enable dependency injection and testability?
8. What's the difference between a class and a record?

If you struggle with any of these, revisit the relevant section.

---

## Further Practice

1. Design a shape hierarchy (Circle, Rectangle, Triangle) using inheritance and polymorphism.
2. Refactor a class with deep inheritance (5+ levels) to use composition instead.
3. Create an interface for a plugin system (IPlugin) and implement 2-3 plugins.
4. Write a sealed class and explain when sealing is appropriate.
5. Explain the SOLID principles using your own code examples.

---

## Mini Interview Cheat Sheet

**Explain OOP in 30 seconds:**
"OOP structures code around objects that encapsulate state and behavior. The four pillars are: encapsulation (data hiding), inheritance (code reuse), polymorphism (one interface, many implementations), and abstraction (hiding complexity)."

**Interface vs Abstract Class:**
"Use interfaces for contracts without shared code. Use abstract classes when you need shared implementation. Interfaces allow multiple inheritance; abstract classes don't."

**When to use inheritance:**
"Only for clear 'is-a' relationships. Favor composition for 'has-a' relationships. Avoid deep hierarchies—they're fragile and hard to maintain."

**SOLID in one line each:**
- **S**: One class, one responsibility
- **O**: Open for extension, closed for modification
- **L**: Derived classes must be substitutable for base classes
- **I**: Many small interfaces > one large interface
- **D**: Depend on abstractions, not concretions

---
