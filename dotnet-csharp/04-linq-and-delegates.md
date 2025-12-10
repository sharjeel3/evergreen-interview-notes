# LINQ & Delegates

**Level:** 🚦 Intermediate  
**Tags:** `LINQ`, `delegates`, `lambda`, `functional`, `interview`

---

## Why this matters

LINQ and delegates are **everywhere in modern C#**.

Interviewers test:
- Understanding of deferred execution
- Query syntax vs method syntax
- When LINQ performs well (and when it doesn't)
- Delegates, Func, Action, and lambda expressions
- IEnumerable vs IQueryable

Mastering these shows you understand functional programming in C#.

---

## Core Ideas

### 1. **Delegates are type-safe function pointers**

A delegate is a reference to a method with a specific signature.

```csharp
// Define delegate type
public delegate int MathOperation(int a, int b);

// Methods matching signature
int Add(int a, int b) => a + b;
int Multiply(int a, int b) => a * b;

// Use delegate
MathOperation operation = Add;
int result = operation(5, 3); // 8

operation = Multiply;
result = operation(5, 3); // 15
```

**Built-in delegates:**
```csharp
Func<int, int, int> add = (a, b) => a + b;       // Returns value
Action<string> log = msg => Console.WriteLine(msg); // No return
Predicate<int> isEven = x => x % 2 == 0;           // Returns bool
```

**Mental model:**  
"Delegates let you pass behavior as parameters. Func/Action are shortcuts."

---

### 2. **Lambda expressions are anonymous functions**

```csharp
// Named method
int Square(int x) => x * x;
Func<int, int> squareFunc = Square;

// Lambda expression (anonymous)
Func<int, int> squareLambda = x => x * x;

// Multi-line lambda
Func<int, int> squareVerbose = x =>
{
    var result = x * x;
    return result;
};
```

**When to use:**
- Short, one-off operations
- LINQ queries
- Event handlers
- Callbacks

---

### 3. **LINQ uses deferred execution**

```csharp
var numbers = new List<int> { 1, 2, 3, 4, 5 };

// Query is NOT executed here
var evens = numbers.Where(x => x % 2 == 0);

numbers.Add(6); // Added after query defined

// Execution happens HERE
foreach (var n in evens) // Results: 2, 4, 6
{
    Console.WriteLine(n);
}
```

**Key operators that force immediate execution:**
- `ToList()`, `ToArray()`, `ToDictionary()`
- `Count()`, `Sum()`, `Max()`, `Min()`, `Average()`
- `First()`, `Single()`, `Any()`, `All()`

**Mental model:**  
"LINQ queries are recipes, not results. Enumerate to execute."

---

### 4. **Method syntax vs Query syntax**

```csharp
var numbers = Enumerable.Range(1, 10);

// Method syntax (preferred by most)
var evenSquares = numbers
    .Where(x => x % 2 == 0)
    .Select(x => x * x)
    .OrderBy(x => x);

// Query syntax (SQL-like)
var evenSquares2 = 
    from n in numbers
    where n % 2 == 0
    orderby n
    select n * n;
```

**Rule:** Use method syntax unless query is complex with multiple `from` clauses (joins).

---

### 5. **IEnumerable<T> vs IQueryable<T>**

| Feature | `IEnumerable<T>` | `IQueryable<T>` |
|---------|------------------|-----------------|
| Execution | In-memory (LINQ to Objects) | Remote (LINQ to SQL, EF) |
| Evaluation | Client-side | Server-side (translated to SQL) |
| When to use | Collections in memory | Database queries |
| Extensibility | Delegates | Expression trees |

```csharp
// IEnumerable - filters in memory after loading
IEnumerable<User> users = context.Users; // All users loaded
var active = users.Where(u => u.IsActive); // Filtered in C#

// IQueryable - filters in database
IQueryable<User> users = context.Users; // No data loaded yet
var active = users.Where(u => u.IsActive); // Translated to SQL WHERE
```

---

## Examples

### Example 1 — Common LINQ operations

```csharp
var numbers = new[] { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

// Filter
var evens = numbers.Where(x => x % 2 == 0);

// Transform
var squares = numbers.Select(x => x * x);

// Aggregate
var sum = numbers.Sum();
var avg = numbers.Average();
var max = numbers.Max();

// Chaining
var result = numbers
    .Where(x => x > 3)
    .Select(x => x * 2)
    .OrderByDescending(x => x)
    .Take(3)
    .ToList(); // [20, 18, 16]

// Grouping
var grouped = numbers
    .GroupBy(x => x % 3)
    .Select(g => new { Key = g.Key, Values = g.ToList() });
```

---

### Example 2 — Delegates and callbacks

```csharp
public class DataProcessor
{
    // Accept behavior as parameter
    public List<T> Process<T>(List<T> items, Func<T, bool> filter)
    {
        var result = new List<T>();
        foreach (var item in items)
        {
            if (filter(item))
                result.Add(item);
        }
        return result;
    }
}

// Usage
var processor = new DataProcessor();
var numbers = new List<int> { 1, 2, 3, 4, 5 };

// Pass different behaviors
var evens = processor.Process(numbers, x => x % 2 == 0);
var greaterThan3 = processor.Process(numbers, x => x > 3);
```

---

### Example 3 — Practical use case (filtering and projection)

```csharp
public record Product(string Name, decimal Price, string Category, bool InStock);

var products = new List<Product>
{
    new("Laptop", 1200m, "Electronics", true),
    new("Phone", 800m, "Electronics", false),
    new("Desk", 300m, "Furniture", true),
    new("Chair", 150m, "Furniture", true)
};

// Get affordable in-stock products with discount
var deals = products
    .Where(p => p.InStock && p.Price < 500)
    .Select(p => new 
    { 
        p.Name, 
        OriginalPrice = p.Price, 
        DiscountPrice = p.Price * 0.9m 
    })
    .OrderBy(p => p.DiscountPrice);

foreach (var deal in deals)
{
    Console.WriteLine($"{deal.Name}: ${deal.DiscountPrice:F2}");
}
// Output:
// Chair: $135.00
// Desk: $270.00
```

---

### Example 4 — Joining data

```csharp
var orders = new[]
{
    new { OrderId = 1, CustomerId = 1, Total = 100m },
    new { OrderId = 2, CustomerId = 2, Total = 200m },
    new { OrderId = 3, CustomerId = 1, Total = 150m }
};

var customers = new[]
{
    new { CustomerId = 1, Name = "Alice" },
    new { CustomerId = 2, Name = "Bob" }
};

// Join
var customerOrders = orders
    .Join(
        customers,
        order => order.CustomerId,
        customer => customer.CustomerId,
        (order, customer) => new
        {
            customer.Name,
            order.OrderId,
            order.Total
        }
    );

// Query syntax (clearer for joins)
var customerOrders2 =
    from order in orders
    join customer in customers on order.CustomerId equals customer.CustomerId
    select new { customer.Name, order.OrderId, order.Total };
```

---

### Example 5 — Grouping and aggregation

```csharp
var sales = new[]
{
    new { Product = "Laptop", Category = "Electronics", Amount = 1200m },
    new { Product = "Phone", Category = "Electronics", Amount = 800m },
    new { Product = "Desk", Category = "Furniture", Amount = 300m },
    new { Product = "Chair", Category = "Furniture", Amount = 150m }
};

// Group by category and calculate totals
var categoryTotals = sales
    .GroupBy(s => s.Category)
    .Select(g => new
    {
        Category = g.Key,
        TotalSales = g.Sum(s => s.Amount),
        ProductCount = g.Count(),
        AveragePrice = g.Average(s => s.Amount)
    });

foreach (var cat in categoryTotals)
{
    Console.WriteLine($"{cat.Category}: {cat.TotalSales:C} " +
                     $"({cat.ProductCount} products, avg {cat.AveragePrice:C})");
}
```

---

## Common Pitfalls

### ❌ Pitfall 1: Multiple enumeration

```csharp
IEnumerable<int> GetData()
{
    Console.WriteLine("Fetching data...");
    yield return 1;
    yield return 2;
}

var data = GetData();
var count = data.Count(); // "Fetching data..."
var sum = data.Sum();     // "Fetching data..." again!

// ✅ Solution: Materialize once
var data = GetData().ToList();
var count = data.Count; // No re-execution
var sum = data.Sum();   // No re-execution
```

---

### ❌ Pitfall 2: Modifying collection during iteration

```csharp
var list = new List<int> { 1, 2, 3, 4 };

// ❌ Throws InvalidOperationException
foreach (var item in list.Where(x => x % 2 == 0))
{
    list.Remove(item);
}

// ✅ Materialize first
foreach (var item in list.Where(x => x % 2 == 0).ToList())
{
    list.Remove(item);
}
```

---

### ❌ Pitfall 3: Unnecessary ToList() in chains

```csharp
// ❌ Inefficient - multiple materializations
var result = numbers
    .Where(x => x > 5).ToList()  // ❌ Unnecessary
    .Select(x => x * 2).ToList() // ❌ Unnecessary
    .OrderBy(x => x).ToList();   // Only this one needed

// ✅ Efficient - single materialization at end
var result = numbers
    .Where(x => x > 5)
    .Select(x => x * 2)
    .OrderBy(x => x)
    .ToList(); // Materialize once
```

---

### ❌ Pitfall 4: Capturing variables in loops

```csharp
var actions = new List<Action>();

// ❌ Wrong - all lambdas capture same 'i'
for (int i = 0; i < 5; i++)
{
    actions.Add(() => Console.WriteLine(i)); // Captures reference to 'i'
}

foreach (var action in actions)
    action(); // Prints: 5, 5, 5, 5, 5

// ✅ Correct - capture loop-scoped variable
for (int i = 0; i < 5; i++)
{
    int copy = i; // New variable each iteration
    actions.Add(() => Console.WriteLine(copy));
}

foreach (var action in actions)
    action(); // Prints: 0, 1, 2, 3, 4
```

---

### ❌ Pitfall 5: Performance with large datasets

```csharp
// ❌ Slow - filters entire list multiple times
var result = largeList
    .Where(x => x.IsActive)
    .Where(x => x.Price > 100)
    .Where(x => x.Category == "Electronics");

// ✅ Better - single pass with combined predicate
var result = largeList.Where(x => 
    x.IsActive && x.Price > 100 && x.Category == "Electronics");
```

---

## Quick Self-Check

1. What's the difference between `Func<T>` and `Action<T>`?
2. When does a LINQ query execute?
3. What's the difference between `IEnumerable<T>` and `IQueryable<T>`?
4. Why might you use query syntax over method syntax?
5. What's the cost of multiple enumeration?

<details>
<summary>Answers</summary>

1. `Func<T>` returns a value; `Action<T>` returns void
2. When enumerated (foreach, ToList, Count, etc.), not when defined
3. `IEnumerable<T>` executes in-memory; `IQueryable<T>` translates to SQL/remote queries
4. For complex queries with multiple `from` clauses (joins), query syntax is often clearer
5. Re-executes the entire query pipeline each time, potentially expensive for DB queries or computed sequences

</details>

---

## Further Practice

1. **Rewrite** a foreach loop with mutations as a LINQ chain
2. **Benchmark** `Where().Select()` vs `Select().Where()` on large datasets
3. **Implement** a custom LINQ operator using `yield return`
4. **Convert** query syntax to method syntax for a multi-join query
5. **Debug** why a LINQ query re-executes multiple times

---

## Key Takeaways

✅ Delegates enable passing behavior; Func/Action are built-in shortcuts  
✅ LINQ uses deferred execution—query when enumerated  
✅ Materialize with ToList() when multiple enumerations needed  
✅ `IQueryable<T>` for database, `IEnumerable<T>` for in-memory  
✅ Avoid modifying collections during LINQ iteration  
✅ Method syntax preferred except for complex joins
