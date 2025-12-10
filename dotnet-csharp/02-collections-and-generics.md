# Collections & Generics

**Level:** 🏁 Beginner  
**Tags:** `fundamentals`, `collections`, `generics`, `LINQ`, `interview`

---

## Why this matters

Collections and generics form the backbone of **virtually every .NET application**—from simple data storage to complex algorithms. Your ability to choose and use them correctly directly impacts application performance, maintainability, and correctness.

**Type safety and maintainability:** Before generics (pre-C# 2.0), collections stored objects, requiring casts everywhere and losing all compile-time type checking. A single wrong cast could crash production code. Generics eliminated an entire class of runtime errors by moving type checking to compile time. Understanding this evolution helps you appreciate why generic collections are non-negotiable in modern C# code.

**Performance characteristics:** Choosing the wrong collection can destroy application performance. Using `List<T>.Contains()` in a loop is O(n²), while `HashSet<T>.Contains()` is O(n). For 10,000 items, that's the difference between 100 million operations and 10,000 operations—potentially seconds vs milliseconds. Understanding Big O complexity of operations (lookup, insertion, deletion) is crucial for writing scalable code.

**Memory efficiency:** Collections have different memory footprints. A `Dictionary<TKey, TValue>` uses more memory than a `List<T>` due to hash buckets. A `LinkedList<T>` has significant overhead per node. Choosing appropriately can save megabytes in memory-constrained environments.

**Deferred execution mastery:** Understanding `IEnumerable<T>` and deferred execution is critical for writing efficient LINQ queries, especially when working with databases via Entity Framework. Without this knowledge, you might accidentally load millions of records into memory when you only needed to count them.

**Interview expectations:** When interviewers ask about collections and generics, they're evaluating:
- Can you choose the right data structure for the problem?
- Do you understand time/space complexity trade-offs?
- Can you optimize code that's performing poorly?
- Do you know when LINQ is appropriate vs when to use direct loops?
- Can you design APIs that are both flexible and performant?

This knowledge demonstrates that you can write production-quality code that scales and performs well under real-world conditions.

---

## Core Ideas

### 1. **Generic collections provide type safety without boxing**

**Before generics (C# 1.0):**
```csharp
ArrayList list = new ArrayList();
list.Add(42);           // Boxing
list.Add("hello");      // No type safety!
int x = (int)list[0];   // Unboxing + cast
```

**With generics (C# 2.0+):**
```csharp
List<int> list = new List<int>();
list.Add(42);          // No boxing
// list.Add("hello");  // ❌ Compile error
int x = list[0];       // No cast needed
```

**Mental model:**  
"Generics = type safety + performance. No more boxing, no more casting."

---

### 2. **Common Collection Interfaces**

| Interface | Features | When to use |
|-----------|----------|-------------|
| `IEnumerable<T>` | foreach only, deferred execution | Read-only iteration, LINQ queries |
| `ICollection<T>` | + Count, Add, Remove, Contains | When you need count or modify |
| `IList<T>` | + indexer, Insert, RemoveAt | Random access by index |
| `IDictionary<TKey, TValue>` | Key-value pairs | Fast lookup by key |

**Key insight:** Return `IEnumerable<T>` from methods to hide implementation and enable deferred execution.

---

### 3. **Common Generic Collections**

| Collection | Underlying Structure | Access Time | When to use |
|------------|---------------------|-------------|-------------|
| `List<T>` | Dynamic array | O(1) index, O(n) search | Default choice, ordered data |
| `Dictionary<TKey, TValue>` | Hash table | O(1) average | Fast key-based lookup |
| `HashSet<T>` | Hash table | O(1) average | Unique items, fast contains |
| `Queue<T>` | FIFO | O(1) enqueue/dequeue | First-in-first-out |
| `Stack<T>` | LIFO | O(1) push/pop | Last-in-first-out |
| `LinkedList<T>` | Doubly-linked list | O(1) add/remove at ends | Frequent insertions/deletions |
| `SortedList<TKey, TValue>` | Sorted array | O(log n) | Sorted key-value pairs, infrequent updates |
| `SortedSet<T>` | Red-black tree | O(log n) | Sorted unique items |

---

### 4. **IEnumerable<T> and Deferred Execution**

```csharp
IEnumerable<int> GetNumbers()
{
    Console.WriteLine("Getting numbers...");
    yield return 1;
    yield return 2;
    yield return 3;
}

var numbers = GetNumbers(); // Nothing printed yet!
foreach (var n in numbers)  // "Getting numbers..." printed now
{
    Console.WriteLine(n);
}
```

**Why it matters:**
- Enables LINQ to work efficiently
- Avoids loading entire collections into memory
- Composition of queries without intermediate allocations

---

### 5. **Generic Type Constraints**

```csharp
// Constrain T to reference types
public class Repository<T> where T : class
{
    public T Find(int id) => default(T); // Can return null
}

// Multiple constraints
public class Service<T> where T : IDisposable, new()
{
    public void Use()
    {
        using var instance = new T(); // new() allows parameterless constructor
        // T implements IDisposable, so 'using' works
    }
}

// Common constraints:
// where T : struct          // Value type
// where T : class           // Reference type
// where T : new()           // Has parameterless constructor
// where T : BaseClass       // Inherits from BaseClass
// where T : IInterface      // Implements IInterface
// where T : U               // T must be or derive from U
```

---

## Examples

### Example 1 — Choosing the right collection

```csharp
// ✅ List<T> - Ordered collection, index access
var names = new List<string> { "Alice", "Bob", "Charlie" };
Console.WriteLine(names[1]); // "Bob"

// ✅ Dictionary<K,V> - Fast lookup by key
var ages = new Dictionary<string, int>
{
    ["Alice"] = 30,
    ["Bob"] = 25
};
Console.WriteLine(ages["Alice"]); // 30

// ✅ HashSet<T> - Unique items, fast Contains
var uniqueIds = new HashSet<int> { 1, 2, 3, 2, 1 };
Console.WriteLine(uniqueIds.Count); // 3 (duplicates removed)
Console.WriteLine(uniqueIds.Contains(2)); // True, O(1)

// ✅ Queue<T> - FIFO processing
var queue = new Queue<string>();
queue.Enqueue("First");
queue.Enqueue("Second");
Console.WriteLine(queue.Dequeue()); // "First"
```

---

### Example 2 — IEnumerable<T> for deferred execution

```csharp
IEnumerable<int> GetEvenNumbers(int max)
{
    Console.WriteLine("Starting...");
    for (int i = 0; i <= max; i++)
    {
        if (i % 2 == 0)
            yield return i;
    }
}

// Deferred - nothing executes yet
var evens = GetEvenNumbers(10);

// Execution happens here
foreach (var n in evens.Take(3)) // Only processes 0, 2, 4
{
    Console.WriteLine(n);
}
```

---

### Example 3 — Generic constraints in action

```csharp
// Repository pattern with constraints
public interface IEntity
{
    int Id { get; set; }
}

public class Repository<T> where T : class, IEntity, new()
{
    private readonly List<T> _data = new();
    
    public void Add(T entity)
    {
        if (entity.Id == 0)
            entity.Id = _data.Count + 1;
        _data.Add(entity);
    }
    
    public T FindById(int id) => _data.FirstOrDefault(e => e.Id == id);
    
    public T CreateNew()
    {
        return new T(); // new() constraint allows this
    }
}

public class User : IEntity
{
    public int Id { get; set; }
    public string Name { get; set; }
}

// Usage
var repo = new Repository<User>();
repo.Add(new User { Name = "Alice" });
```

---

### Example 4 — Dictionary patterns

```csharp
var inventory = new Dictionary<string, int>
{
    ["Apple"] = 10,
    ["Banana"] = 5
};

// ✅ Safe lookup
if (inventory.TryGetValue("Apple", out int count))
{
    Console.WriteLine($"We have {count} apples");
}

// ✅ Get or add default
int oranges = inventory.TryGetValue("Orange", out int o) ? o : 0;

// ✅ Update or add (C# 9+)
inventory["Apple"] = inventory.TryGetValue("Apple", out var apples) ? apples + 1 : 1;

// Or use CollectionsMarshal for performance (C# 11+)
```

---

## Common Pitfalls

### ❌ Pitfall 1: Modifying collection while iterating

```csharp
var list = new List<int> { 1, 2, 3, 4, 5 };

// ❌ Throws InvalidOperationException
foreach (var item in list)
{
    if (item % 2 == 0)
        list.Remove(item);
}

// ✅ Solution 1: ToList() to iterate over copy
foreach (var item in list.ToList())
{
    if (item % 2 == 0)
        list.Remove(item);
}

// ✅ Solution 2: Use RemoveAll
list.RemoveAll(x => x % 2 == 0);

// ✅ Solution 3: Iterate backwards (for Lists)
for (int i = list.Count - 1; i >= 0; i--)
{
    if (list[i] % 2 == 0)
        list.RemoveAt(i);
}
```

---

### ❌ Pitfall 2: Multiple enumeration of IEnumerable<T>

```csharp
IEnumerable<int> GetExpensiveData()
{
    // Expensive operation
    yield return FetchFromDatabase();
    yield return FetchFromDatabase();
}

var data = GetExpensiveData();
var count = data.Count();  // Enumerates once
var sum = data.Sum();      // Enumerates again! 💸

// ✅ Solution: Materialize once
var data = GetExpensiveData().ToList(); // Single enumeration
var count = data.Count;    // No re-enumeration
var sum = data.Sum();      // No re-enumeration
```

---

### ❌ Pitfall 3: Dictionary KeyNotFoundException

```csharp
var dict = new Dictionary<string, int>();

// ❌ Throws KeyNotFoundException
int value = dict["missing"];

// ✅ Always use TryGetValue
if (dict.TryGetValue("missing", out int val))
{
    Console.WriteLine(val);
}

// ✅ Or provide default
int value = dict.GetValueOrDefault("missing", -1);
```

---

### ❌ Pitfall 4: List<T> vs LinkedList<T> misuse

```csharp
// ❌ Slow: List with frequent insertions at beginning
var list = new List<int>();
for (int i = 0; i < 10000; i++)
    list.Insert(0, i); // O(n) each time!

// ✅ Fast: LinkedList for frequent insertions
var linked = new LinkedList<int>();
for (int i = 0; i < 10000; i++)
    linked.AddFirst(i); // O(1)
```

**Rule:** Use `List<T>` 95% of the time. Use `LinkedList<T>` only when profiling shows it helps.

---

## Quick Self-Check

1. What's the difference between `IEnumerable<T>` and `IList<T>`?
2. When would you use `HashSet<T>` over `List<T>`?
3. What is deferred execution? Give an example.
4. What are generic constraints and why use them?
5. What's the Big O of Dictionary lookup?

<details>
<summary>Answers</summary>

1. `IEnumerable<T>` only supports iteration; `IList<T>` adds indexing, insertion, and removal
2. When you need unique items and fast O(1) `Contains` checks
3. Query/operation doesn't execute until enumeration. Example: LINQ queries with `Where`, `Select`
4. Constraints like `where T : class` restrict generic types to ensure certain capabilities. Enables compile-time safety
5. O(1) average case, O(n) worst case (hash collision)

</details>

---

## Further Practice

1. **Implement a generic LRU cache** using `Dictionary<TKey, LinkedListNode<T>>`
2. **Benchmark** `List<int>` vs `LinkedList<int>` for 10,000 insertions at beginning
3. **Write a method** that takes `IEnumerable<T>` and returns unique items without using `HashSet`
4. **Create a generic method** with constraint `where T : IComparable<T>` that finds max element

---

## Key Takeaways

✅ Use `List<T>` as default, `Dictionary<TKey, TValue>` for fast lookup  
✅ Return `IEnumerable<T>` to enable deferred execution and hide implementation  
✅ Avoid multiple enumeration—materialize with `ToList()` when needed  
✅ Use `TryGetValue` for dictionaries, not indexer  
✅ Generic constraints enable type-safe, reusable code
