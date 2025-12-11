# Nullable Types and Modern C#

🚦 **Intermediate** | ⏱ 45-55 min

---

## 📝 Why This Matters

Null reference exceptions have been called the "billion-dollar mistake" by their inventor, Tony Hoare. In C#, they've been a constant source of runtime crashes—`NullReferenceException` is consistently one of the most common exceptions in production applications. Modern C# addresses this with nullable reference types (NRT), one of the most significant language improvements in recent years.

Starting with C# 8.0, the compiler can help you track which references might be null and warn you before runtime errors occur. Instead of discovering null issues when users click a button and your app crashes, you get warnings at compile time. This shifts null handling from a runtime concern to a compile-time concern, catching bugs earlier in the development cycle when they're cheaper to fix.

In real-world projects, nullable reference types have a dramatic impact on code quality. Teams report significant reductions in `NullReferenceException` crashes after enabling NRT. The feature forces you to be explicit about nullability: if a property can be null, you mark it with `?`; if it can't, the compiler warns when you might assign null. This makes your intent clear to both the compiler and other developers.

Beyond nullable reference types, modern C# (versions 8-12) has introduced features that make code more concise, safer, and expressive. Records provide immutable data types with value semantics. Pattern matching lets you write cleaner conditionals and type checks. Init-only properties enable immutable object initialization. Top-level statements reduce boilerplate in simple programs. Target-typed new expressions eliminate redundant type names. Required members ensure objects are properly initialized.

These features aren't just syntactic sugar—they enable better architectures. Records make it easier to implement immutable domain models, which simplify concurrent programming and debugging (immutable objects can't change under you). Pattern matching reduces complex if-else chains into readable expressions. Init-only properties let you create immutable objects without verbose constructors. Required members catch initialization mistakes at compile time instead of runtime.

From a practical standpoint, modern C# features improve productivity. Less boilerplate means you write and read less code to accomplish the same task. Null-safety means fewer defensive null checks cluttering your code. Records eliminate the need to write equality comparisons, hash codes, and `ToString()` implementations. Pattern matching makes data transformation more declarative.

For interviews, understanding modern C# shows you're current with the language. Employers want developers who know the latest features and use them appropriately. Being able to explain when to use records vs classes, how nullable reference types work, and what problems pattern matching solves demonstrates practical knowledge of contemporary C# development.

The migration path also matters. Most codebases can't adopt these features overnight. Understanding how to gradually enable nullable reference types in a large codebase, when to refactor classes to records, and how to balance modern features with team familiarity is a valuable skill. Knowing the trade-offs shows maturity beyond just knowing the syntax.

Modern C# also reflects broader industry trends toward functional programming concepts: immutability, pattern matching, expression-based code, and explicit handling of absence (null). While C# remains fundamentally object-oriented, these functional influences make code safer and more maintainable. Understanding this philosophy helps you write better C# and prepares you for other modern languages.

The compiler's static analysis for nullable reference types is also a fascinating technical achievement. It performs data flow analysis to track nullability through your code, understanding that after `if (x != null)`, x can't be null in the if-block. This level of analysis was previously only in specialized tools; now it's built into the compiler and runs on every build.

---

## 🎯 Core Ideas

### 1. **Nullable Reference Types (NRT)**

**Before C# 8.0: All references could be null**

```csharp
// ❌ Old C# - everything is nullable by default
public class Person
{
    public string Name { get; set; }
    public string Email { get; set; }
}

// This compiles but crashes at runtime!
var person = new Person();
Console.WriteLine(person.Name.Length); // NullReferenceException!
```

**C# 8.0+: Explicit nullability**

```csharp
// Enable nullable reference types in csproj:
// <Nullable>enable</Nullable>

// ✅ Modern C# - explicit about nullability
public class Person
{
    // Non-nullable: must never be null
    public string Name { get; set; }
    
    // Nullable: might be null
    public string? Email { get; set; }
    
    // Constructor ensures non-nullable properties are set
    public Person(string name)
    {
        Name = name; // ✅ Required
        // Email is optional (nullable)
    }
}

var person = new Person("Alice");

// Compiler knows Name is never null
Console.WriteLine(person.Name.Length); // ✅ Safe

// Compiler warns: Email might be null
Console.WriteLine(person.Email.Length); // ⚠️ Warning CS8602

// ✅ Proper null check
if (person.Email != null)
{
    Console.WriteLine(person.Email.Length); // ✅ Safe after null check
}

// ✅ Null-conditional operator
Console.WriteLine(person.Email?.Length); // ✅ Safe, returns null if Email is null
```

**Nullable annotations and warnings:**

```csharp
public class UserService
{
    // Return type: User? means might return null
    public User? FindById(int id)
    {
        // ... might return null if not found
        return null; // ✅ OK because return type is nullable
    }
    
    // Return type: User means never returns null
    public User GetById(int id)
    {
        var user = FindById(id);
        
        // ⚠️ Warning: Converting nullable to non-nullable
        return user; // Compiler warns: might be null!
        
        // ✅ Correct: throw if null
        return user ?? throw new KeyNotFoundException($"User {id} not found");
    }
    
    // Parameter: string means can't be null
    public void SendEmail(string email)
    {
        // Compiler assumes email is never null
        Console.WriteLine($"Sending to: {email.ToLower()}");
    }
    
    // Parameter: string? means can be null
    public void SendEmailIfProvided(string? email)
    {
        // ⚠️ Warning if you don't check
        // Console.WriteLine(email.ToLower());
        
        // ✅ Proper handling
        if (!string.IsNullOrEmpty(email))
        {
            Console.WriteLine($"Sending to: {email.ToLower()}");
        }
    }
}
```

**Null-forgiving operator (!):**

```csharp
// When YOU know something isn't null but compiler doesn't
public class PersonRepository
{
    private Person? _cachedPerson;
    
    public void LoadCache()
    {
        _cachedPerson = GetPersonFromDatabase();
    }
    
    public string GetCachedName()
    {
        // You know LoadCache was called, but compiler doesn't
        // ⚠️ Warning: _cachedPerson might be null
        // return _cachedPerson.Name;
        
        // ✅ Use null-forgiving operator to suppress warning
        return _cachedPerson!.Name;
        
        // ⚠️ Be careful! If you're wrong, NullReferenceException at runtime
        // Better: add validation
        if (_cachedPerson == null)
            throw new InvalidOperationException("Cache not loaded");
        return _cachedPerson.Name;
    }
}
```

### 2. **Nullable Value Types**

**Value types are non-nullable by default:**

```csharp
int x = 5;      // Never null (value type)
// int y = null; // ❌ Compile error

int? z = null;  // ✅ Nullable<int>, can be null
```

**Working with nullable value types:**

```csharp
int? nullableInt = null;

// Checking for value
if (nullableInt.HasValue)
{
    int value = nullableInt.Value; // Safe to access .Value
    Console.WriteLine(value);
}

// Null-coalescing operator
int result = nullableInt ?? 0; // Use 0 if null
Console.WriteLine(result); // 0

// Null-coalescing assignment (C# 8.0)
nullableInt ??= 10; // Assign 10 only if null
Console.WriteLine(nullableInt); // 10

// GetValueOrDefault
int defaultValue = nullableInt.GetValueOrDefault(); // Returns 0 (default) if null
int customDefault = nullableInt.GetValueOrDefault(100); // Returns 100 if null
```

**Common patterns:**

```csharp
public class Order
{
    public DateTime OrderDate { get; set; }
    
    // Nullable: order might not be shipped yet
    public DateTime? ShippedDate { get; set; }
    
    public int DaysToShip()
    {
        // If not shipped, return -1
        if (!ShippedDate.HasValue)
            return -1;
        
        return (ShippedDate.Value - OrderDate).Days;
    }
    
    // Alternative with null-coalescing
    public string GetShippedDateDisplay()
    {
        return ShippedDate?.ToString("yyyy-MM-dd") ?? "Not shipped";
    }
}
```

### 3. **Records (C# 9.0+)**

**Immutable data classes with built-in functionality:**

```csharp
// Traditional class - lots of boilerplate
public class PersonClass
{
    public string Name { get; }
    public int Age { get; }
    
    public PersonClass(string name, int age)
    {
        Name = name;
        Age = age;
    }
    
    // Must manually implement for value semantics
    public override bool Equals(object? obj)
    {
        if (obj is PersonClass other)
            return Name == other.Name && Age == other.Age;
        return false;
    }
    
    public override int GetHashCode() => HashCode.Combine(Name, Age);
    public override string ToString() => $"Person {{ Name = {Name}, Age = {Age} }}";
}

// ✅ Record - concise and automatic
public record Person(string Name, int Age);

// Usage
var person1 = new Person("Alice", 30);
var person2 = new Person("Alice", 30);
var person3 = new Person("Bob", 25);

// Value-based equality (automatic!)
Console.WriteLine(person1 == person2);  // True (same values)
Console.WriteLine(person1 == person3);  // False (different values)

// Reference equality for classes
var class1 = new PersonClass("Alice", 30);
var class2 = new PersonClass("Alice", 30);
Console.WriteLine(class1 == class2);    // False (different objects)

// Automatic ToString
Console.WriteLine(person1); // Person { Name = Alice, Age = 30 }

// With-expressions: non-destructive mutation
var person4 = person1 with { Age = 31 };
Console.WriteLine(person1.Age); // 30 (original unchanged)
Console.WriteLine(person4.Age); // 31 (new instance with modified age)
```

**Record with additional members:**

```csharp
public record Product(string Name, decimal Price)
{
    // Additional properties
    public string? Category { get; init; }
    
    // Methods
    public decimal GetPriceWithTax(decimal taxRate) => Price * (1 + taxRate);
    
    // Validation in constructor
    public Product(string Name, decimal Price) : this(Name, Price)
    {
        if (Price < 0)
            throw new ArgumentException("Price cannot be negative");
    }
}

var product = new Product("Laptop", 999.99m) { Category = "Electronics" };
Console.WriteLine(product.GetPriceWithTax(0.08m)); // 1079.99
```

**Record structs (C# 10.0):**

```csharp
// Value type record (struct)
public record struct Point(int X, int Y);

// Mutable record struct
public record struct MutablePoint(int X, int Y)
{
    public int X { get; set; } = X;
    public int Y { get; set; } = Y;
}
```

**When to use records vs classes:**

```csharp
// ✅ Use records for:
// - DTOs (Data Transfer Objects)
public record UserDto(int Id, string Name, string Email);

// - Value objects in domain models
public record Money(decimal Amount, string Currency);

// - Immutable configuration
public record AppSettings(string ConnectionString, int Timeout);

// - Event payloads
public record OrderCreatedEvent(int OrderId, DateTime CreatedAt);

// ❌ Use classes for:
// - Entities with identity (not value-based equality)
public class User // Identity = Id, not all properties
{
    public int Id { get; set; }
    public string Name { get; set; }
}

// - Mutable objects with behavior
public class ShoppingCart // Mutable state, complex behavior
{
    private List<CartItem> _items = new();
    public void AddItem(CartItem item) { _items.Add(item); }
}

// - When inheritance is needed (though records support inheritance too)
public class Animal { }
public class Dog : Animal { }
```

### 4. **Init-Only Properties (C# 9.0)**

**Immutable objects without constructor bloat:**

```csharp
// Before C# 9: verbose constructors or mutable properties
public class PersonOld
{
    // Option 1: Mutable (not safe)
    public string Name { get; set; }
    public int Age { get; set; }
    
    // Option 2: Immutable via constructor (verbose)
    public PersonOld(string name, int age, string email, string phone)
    {
        Name = name;
        Age = age;
        // ... 20 more parameters
    }
}

// ✅ C# 9+: init-only properties
public class Person
{
    public string Name { get; init; }
    public int Age { get; init; }
    public string Email { get; init; }
    public string Phone { get; init; }
    
    // Can only be set during object initialization
}

// Object initializer syntax works
var person = new Person
{
    Name = "Alice",
    Age = 30,
    Email = "alice@example.com",
    Phone = "555-1234"
};

// ❌ Can't modify after initialization
// person.Name = "Bob"; // Compile error: init-only property

// ✅ Create modified copy with 'with' if it's a record
public record PersonRecord
{
    public string Name { get; init; }
    public int Age { get; init; }
}

var person2 = new PersonRecord { Name = "Alice", Age = 30 };
var person3 = person2 with { Age = 31 }; // New instance
```

### 5. **Required Members (C# 11.0)**

**Ensure properties are initialized:**

```csharp
// Problem: compiler doesn't enforce initialization
public class PersonOld
{
    public string Name { get; init; }  // Might forget to set!
}

var person = new PersonOld(); // ❌ Name is null, no warning

// ✅ Solution: required members
public class Person
{
    public required string Name { get; init; }
    public required string Email { get; init; }
    public int Age { get; init; } // Optional, not required
}

// ❌ Compile error: required members not set
// var person1 = new Person();

// ✅ Must initialize required members
var person2 = new Person
{
    Name = "Alice",
    Email = "alice@example.com"
    // Age is optional
};

// ✅ Works with constructors too
public class PersonWithConstructor
{
    public required string Name { get; init; }
    public required string Email { get; init; }
    
    // Constructor satisfies requirements
    [SetsRequiredMembers] // Tells compiler this sets required members
    public PersonWithConstructor(string name, string email)
    {
        Name = name;
        Email = email;
    }
}

var person3 = new PersonWithConstructor("Bob", "bob@example.com");
```

### 6. **Pattern Matching (C# 7.0-12.0)**

**Type patterns:**

```csharp
object obj = "Hello";

// Old way
if (obj is string)
{
    string str = (string)obj;
    Console.WriteLine(str.Length);
}

// ✅ Pattern matching with declaration
if (obj is string str)
{
    Console.WriteLine(str.Length); // str is already string
}

// Switch expressions (C# 8.0)
string GetDescription(object obj) => obj switch
{
    string s => $"String of length {s.Length}",
    int i => $"Integer: {i}",
    null => "It's null",
    _ => "Unknown type"
};
```

**Property patterns (C# 8.0):**

```csharp
public record Person(string Name, int Age);

string Classify(Person person) => person switch
{
    { Age: < 18 } => "Minor",
    { Age: >= 18 and < 65 } => "Adult",
    { Age: >= 65 } => "Senior",
    _ => "Unknown"
};

// More complex property patterns
string GetStatus(Person person) => person switch
{
    { Name: "Admin" } => "Administrator",
    { Age: < 18, Name: var name } => $"{name} is a minor",
    { Age: >= 65, Name: var name } => $"{name} is a senior",
    _ => "Regular user"
};
```

**Relational patterns (C# 9.0):**

```csharp
int GetGrade(int score) => score switch
{
    >= 90 => 'A',
    >= 80 and < 90 => 'B',
    >= 70 and < 80 => 'C',
    >= 60 and < 70 => 'D',
    < 60 => 'F',
    _ => '?'
};

// Combining patterns
bool IsValidAge(int age) => age is >= 0 and <= 120;
```

**List patterns (C# 11.0):**

```csharp
int[] numbers = { 1, 2, 3, 4, 5 };

string Describe(int[] array) => array switch
{
    [] => "Empty",
    [1] => "Single element: 1",
    [1, 2] => "Two elements: 1 and 2",
    [1, 2, ..] => "Starts with 1, 2",
    [.., 5] => "Ends with 5",
    [1, .., 5] => "Starts with 1, ends with 5",
    _ => "Other"
};

Console.WriteLine(Describe(numbers)); // "Starts with 1, ends with 5"
```

### 7. **Other Modern Features**

**Target-typed new (C# 9.0):**

```csharp
// Before: redundant type names
List<string> oldList = new List<string>();
Person oldPerson = new Person("Alice", 30);

// ✅ After: type inferred from target
List<string> newList = new(); // Type inferred as List<string>
Person newPerson = new("Alice", 30);

// Especially useful in complex scenarios
Dictionary<string, List<int>> dict = new()
{
    ["key1"] = new() { 1, 2, 3 },
    ["key2"] = new() { 4, 5, 6 }
};
```

**Top-level statements (C# 9.0):**

```csharp
// Before: boilerplate Program class
namespace MyApp
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Hello World!");
        }
    }
}

// ✅ After: minimal code (perfect for simple programs/scripts)
Console.WriteLine("Hello World!");
// Compiler generates the class and Main method
```

**Global using directives (C# 10.0):**

```csharp
// In a GlobalUsings.cs file or any .cs file:
global using System;
global using System.Collections.Generic;
global using System.Linq;

// Now these namespaces are available in ALL files without 'using' statements
```

**File-scoped namespaces (C# 10.0):**

```csharp
// Before: extra indentation level
namespace MyApp.Services
{
    public class UserService
    {
        // ...
    }
}

// ✅ After: less indentation
namespace MyApp.Services;

public class UserService
{
    // ...
}
```

**Raw string literals (C# 11.0):**

```csharp
// Before: escape hell
string json = "{\"name\": \"Alice\", \"age\": 30}";
string path = "C:\\Users\\Alice\\Documents\\file.txt";

// ✅ After: raw strings
string json = """{"name": "Alice", "age": 30}""";
string path = """C:\Users\Alice\Documents\file.txt""";

// Multi-line with proper indentation
string html = """
    <div>
        <h1>Hello</h1>
        <p>World</p>
    </div>
    """;
```

---

## ⚠️ Common Pitfalls

### 1. **Nullable Reference Types Don't Prevent Null at Runtime**

```csharp
// ⚠️ Nullable reference types are compile-time only!
public class UserService
{
    public User GetUser(int id)
    {
        // ⚠️ Compiler thinks this returns non-null User
        // But at runtime, might return null from database/API
        return database.Users.FirstOrDefault(u => u.Id == id);
        // FirstOrDefault returns null if not found!
    }
}

// ✅ Be explicit about nullability
public class UserService
{
    public User? GetUser(int id) // ✅ Correctly marked as nullable
    {
        return database.Users.FirstOrDefault(u => u.Id == id);
    }
}
```

### 2. **Init-Only Properties Are Only Shallow Immutable**

```csharp
public class Order
{
    public List<OrderItem> Items { get; init; } = new();
}

var order = new Order();

// ❌ Can't reassign property (good)
// order.Items = new List<OrderItem>();

// ⚠️ But CAN mutate the list contents!
order.Items.Add(new OrderItem()); // This works!

// ✅ For true immutability, use ImmutableList
public class Order
{
    public ImmutableList<OrderItem> Items { get; init; } = ImmutableList<OrderItem>.Empty;
}
```

### 3. **Records Equality Can Be Surprising**

```csharp
public record Person(string Name, List<string> Hobbies);

var person1 = new Person("Alice", new List<string> { "Reading" });
var person2 = new Person("Alice", new List<string> { "Reading" });

// ⚠️ False! Reference equality for collections
Console.WriteLine(person1 == person2);

// Collections are compared by reference, not contents
// ✅ Use ImmutableList or implement custom Equals
```

### 4. **Null-Forgiving Operator Abuse**

```csharp
public class UserService
{
    private User? _currentUser;
    
    public string GetCurrentUserName()
    {
        // ❌ BAD: Suppressing warnings without ensuring safety
        return _currentUser!.Name; // NullReferenceException if null!
    }
    
    // ✅ GOOD: Proper null handling
    public string GetCurrentUserName()
    {
        if (_currentUser == null)
            throw new InvalidOperationException("No user logged in");
        return _currentUser.Name;
    }
}
```

### 5. **Pattern Matching Order Matters**

```csharp
// ❌ Unreachable pattern
int Classify(int value) => value switch
{
    < 0 => -1,
    _ => 0,      // Catches everything
    > 0 => 1,    // ⚠️ Warning: unreachable
};

// ✅ Specific before general
int Classify(int value) => value switch
{
    < 0 => -1,
    > 0 => 1,
    _ => 0,      // Default last
};
```

### 6. **Required Members with Inheritance**

```csharp
public class Base
{
    public required string Name { get; init; }
}

public class Derived : Base
{
    public required string Email { get; init; }
}

// ❌ Must set BOTH required members
// var obj = new Derived { Name = "Alice" };

// ✅ Set all required from hierarchy
var obj = new Derived
{
    Name = "Alice",
    Email = "alice@example.com"
};
```

---

## ✅ Self-Check Questions

1. **What's the difference between nullable value types and nullable reference types?**
   - Nullable value types: `int?` wraps value in `Nullable<int>` struct
   - Nullable reference types: compiler feature, no runtime overhead
   - NRT is compile-time only, doesn't prevent null at runtime

2. **When should you use records instead of classes?**
   - Records: immutable data with value equality (DTOs, value objects)
   - Classes: entities with identity, mutable objects with behavior
   - Records better for data that's compared by value

3. **What does `required` do differently than constructor parameters?**
   - `required`: compiler ensures property is set during initialization
   - Constructor: forces specific initialization order
   - `required` allows object initializer syntax with enforcement

4. **How do nullable reference types help catch bugs?**
   - Compiler warns when you might dereference null
   - Forces explicit handling of nullable values
   - Shifts null bugs from runtime to compile-time

5. **What's the purpose of the null-forgiving operator (!)?**
   - Tells compiler "I know this isn't null" when it can't tell
   - Suppresses nullability warnings
   - Use sparingly—you're disabling safety checks

---

## 🏋️ Practice

1. **Enable nullable reference types** in an existing project. Fix all warnings by adding appropriate null checks and nullable annotations.

2. **Refactor a DTO class hierarchy** to use records with inheritance, demonstrating value equality and with-expressions.

3. **Build a configuration system** using init-only properties and required members to ensure all config values are set at startup.

4. **Implement a domain model** using records for value objects (Money, Address, Email) and classes for entities (User, Order).

5. **Create a query builder** using pattern matching to handle different filter types (equals, contains, range) and generate SQL safely.

---

## 🎤 Interview Cheat Sheet

**Quick answers for common questions:**

**"What are nullable reference types?"**
> "Starting in C# 8, you can enable nullable reference types which make the compiler track whether references might be null. By default, reference types become non-nullable, and you use `?` to mark nullable ones like `string?`. The compiler warns when you might dereference null without checking. This catches potential `NullReferenceException` bugs at compile time instead of runtime. It's purely a compile-time feature—no runtime overhead—but it significantly improves code safety."

**"When do you use records vs classes?"**
> "I use records for immutable data where equality is based on values, like DTOs, configuration objects, or value objects in domain models. Records automatically implement value-based equality, `ToString()`, and with-expressions for non-destructive mutation. I use classes for entities with identity where equality is based on ID, or when I need complex mutable state and behavior. For example, a `UserDto` record but a `ShoppingCart` class."

**"Explain init-only properties."**
> "Init-only properties can only be set during object initialization—in object initializers or constructors—but not after. This gives you immutability without verbose constructors. You write `public string Name { get; init; }` and can set it like `new Person { Name = 'Alice' }`, but can't modify it later. Combined with required members in C# 11, you get compiler-enforced initialization of immutable properties without constructor bloat."

**"What's the difference between `??` and `?.`?"**
> "The null-coalescing operator `??` provides a default value: `name ?? 'Unknown'` returns 'Unknown' if name is null. The null-conditional operator `?.` safely accesses members: `person?.Name` returns null if person is null, avoiding `NullReferenceException`. You can combine them: `person?.Name ?? 'Unknown'` gets the name or provides a default if person or Name is null."

**"How do you handle nullable reference types with legacy code?"**
> "I enable NRT gradually. First, enable it for new code only. Then address warnings in critical paths. Use `#nullable enable` at the file level for incremental adoption. For external libraries without NRT annotations, I add `[NotNull]` attributes or wrap them in my own null-safe APIs. The key is not trying to fix everything at once—focus on new code and high-risk areas first."

**"What's pattern matching and why use it?"**
> "Pattern matching lets you test values and extract data in one operation. Instead of `if (obj is string) { var s = (string)obj; }` you write `if (obj is string s)`. Switch expressions are more powerful: `obj switch { string s => s.Length, int i => i, _ => 0 }`. Property patterns let you match on object properties: `person switch { { Age: < 18 } => 'Minor' }`. It makes conditional logic more declarative and less verbose than chained if-else statements."

**"What are the benefits of records?"**
> "Records provide value-based equality automatically—two records with the same property values are equal. They implement `Equals()`, `GetHashCode()`, and `ToString()` for you. The with-expression creates modified copies: `person with { Age = 31 }`. This makes them perfect for immutable data like DTOs, event payloads, or value objects. Less boilerplate, safer code, and clearer intent—when you see a record, you know it's immutable data with value semantics."

**Key phrases that impress:**
- "Nullable reference types shift null safety from runtime to compile-time"
- "Records provide value semantics for immutable data"
- "Init-only properties enable immutability without constructor bloat"
- "Pattern matching makes conditional logic more declarative"
- "Required members catch initialization errors at compile time"
- "Modern C# borrows functional programming concepts for safer code"

---

**TL;DR for interviews:**

**Nullable Reference Types (C# 8+):**
- `string?` = nullable, `string` = non-nullable
- Compile-time only, prevents null dereference bugs
- `??` = null-coalescing, `?.` = null-conditional, `!` = null-forgiving

**Records (C# 9+):**
- Immutable data with value equality
- Use for DTOs, value objects, events
- `with` expressions for non-destructive mutation

**Init-only properties (C# 9+):**
- Set during initialization only: `{ get; init; }`
- Immutability without verbose constructors

**Required members (C# 11+):**
- `required` ensures properties are initialized
- Compiler error if not set

**Pattern matching (C# 7-12):**
- `is` with declaration: `if (obj is string s)`
- Switch expressions: `obj switch { ... }`
- Property patterns: `{ Age: < 18 }`
- List patterns: `[1, .., 5]`

**Other modern features:**
- Target-typed new: `List<int> list = new();`
- File-scoped namespaces: `namespace App;`
- Raw strings: `"""multi-line string"""`
- Top-level statements: no Program class needed

Remember: These features make code safer, more concise, and more expressive. Use them!
