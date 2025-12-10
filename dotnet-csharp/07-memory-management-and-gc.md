# Memory Management & Garbage Collection

**Level:** 🔥 Advanced  
**Tags:** `memory`, `GC`, `performance`, `interview`, `CLR`

---

## Why this matters

Understanding memory management separates **senior engineers from juniors**.

Interviewers ask about:
- Stack vs heap allocation
- GC generations and collection strategies
- Memory leaks in managed code
- `IDisposable` and finalization
- Performance optimization techniques

This knowledge is critical for diagnosing production issues and writing high-performance code.

---

## Core Ideas

### 1. **Stack vs Heap allocation**

**Stack:**
- Value types (local variables, parameters)
- Fast allocation (just move stack pointer)
- Automatic cleanup (pop when out of scope)
- Limited size (~1 MB per thread)
- Thread-specific

**Heap:**
- Reference type objects
- Slower allocation (requires GC management)
- Garbage collected
- Large size (limited by available memory)
- Shared across threads

**Mental model:**  
"Stack = fast, automatic, small. Heap = flexible, managed, large."

---

### 2. **Garbage Collection Generations**

The GC divides objects into three generations:

| Generation | Description | Collection Frequency |
|-----------|-------------|---------------------|
| Gen 0 | New objects | Very frequent |
| Gen 1 | Objects that survived one collection | Less frequent |
| Gen 2 | Long-lived objects | Rare |
| LOH (Large Object Heap) | Objects ≥ 85KB | Collected with Gen 2 |

**Why generations:**
- Most objects die young (Gen 0)
- Collecting Gen 0 is fast
- Long-lived objects (Gen 2) aren't checked often
- Improves overall GC performance

```csharp
// Check object generation
object obj = new object();
int generation = GC.GetGeneration(obj); // Typically 0
```

---

### 3. **GC Collection Modes**

```csharp
// Workstation GC (default for client apps)
// - Lower latency
// - Less throughput
// - Suitable for desktop apps

// Server GC (default for ASP.NET Core)
// - Higher throughput
// - Dedicated thread per CPU
// - Suitable for server applications

// Configure in .csproj
<ServerGarbageCollection>true</ServerGarbageCollection>
```

---

### 4. **IDisposable and Resource Management**

**Rule:** Implement `IDisposable` for **unmanaged resources** (file handles, database connections, network sockets).

```csharp
public class ResourceHolder : IDisposable
{
    private IntPtr _unmanagedResource;
    private SqlConnection _managedResource;
    private bool _disposed = false;
    
    public ResourceHolder()
    {
        _unmanagedResource = // Allocate unmanaged resource
        _managedResource = new SqlConnection();
    }
    
    // Public dispose method
    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this); // Don't call finalizer
    }
    
    // Protected dispose pattern
    protected virtual void Dispose(bool disposing)
    {
        if (_disposed) return;
        
        if (disposing)
        {
            // Dispose managed resources
            _managedResource?.Dispose();
        }
        
        // Free unmanaged resources
        if (_unmanagedResource != IntPtr.Zero)
        {
            // Free unmanaged resource
            _unmanagedResource = IntPtr.Zero;
        }
        
        _disposed = true;
    }
    
    // Finalizer (only if you have unmanaged resources)
    ~ResourceHolder()
    {
        Dispose(false);
    }
}
```

---

### 5. **Memory Leaks in Managed Code**

Yes, memory leaks are possible in .NET!

**Common causes:**
1. **Event handler leaks**
   ```csharp
   // ❌ Leak - publisher keeps subscriber alive
   publisher.SomeEvent += Handler;
   // Forgot to unsubscribe!
   ```

2. **Static references**
   ```csharp
   // ❌ Leak - static field prevents GC
   public static List<object> Cache = new();
   ```

3. **Unmanaged resources**
   ```csharp
   // ❌ Leak - forgot to dispose
   var stream = new FileStream("file.txt", FileMode.Open);
   // No disposal!
   ```

4. **Captured variables in closures**
   ```csharp
   // ❌ Leak - closure captures entire object
   public Action CreateCallback()
   {
       var largeObject = new byte[1000000];
       return () => Console.WriteLine(largeObject.Length);
   }
   ```

---

## Examples

### Example 1 — Proper Dispose pattern

```csharp
public class DatabaseConnection : IDisposable
{
    private SqlConnection _connection;
    private bool _disposed;
    
    public DatabaseConnection(string connectionString)
    {
        _connection = new SqlConnection(connectionString);
    }
    
    public void Open() => _connection.Open();
    
    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }
    
    protected virtual void Dispose(bool disposing)
    {
        if (_disposed) return;
        
        if (disposing)
        {
            _connection?.Dispose();
        }
        
        _disposed = true;
    }
}

// Usage
using var connection = new DatabaseConnection(connString);
connection.Open();
// Automatically disposed
```

---

### Example 2 — Avoiding event handler leaks

```csharp
public class Publisher
{
    public event EventHandler DataChanged;
    
    protected virtual void OnDataChanged()
    {
        DataChanged?.Invoke(this, EventArgs.Empty);
    }
}

public class Subscriber : IDisposable
{
    private readonly Publisher _publisher;
    
    public Subscriber(Publisher publisher)
    {
        _publisher = publisher;
        _publisher.DataChanged += OnDataChanged; // ✅ Subscribe
    }
    
    private void OnDataChanged(object sender, EventArgs e)
    {
        // Handle event
    }
    
    public void Dispose()
    {
        _publisher.DataChanged -= OnDataChanged; // ✅ Unsubscribe!
    }
}

// Or use weak event pattern
public class WeakEventManager<TEventArgs> where TEventArgs : EventArgs
{
    private readonly List<WeakReference<EventHandler<TEventArgs>>> _handlers = new();
    
    public void AddHandler(EventHandler<TEventArgs> handler)
    {
        _handlers.Add(new WeakReference<EventHandler<TEventArgs>>(handler));
    }
    
    public void RemoveHandler(EventHandler<TEventArgs> handler)
    {
        _handlers.RemoveAll(wr => 
            !wr.TryGetTarget(out var target) || target == handler);
    }
    
    public void RaiseEvent(object sender, TEventArgs e)
    {
        foreach (var weakRef in _handlers.ToList())
        {
            if (weakRef.TryGetTarget(out var handler))
            {
                handler(sender, e);
            }
            else
            {
                _handlers.Remove(weakRef); // Cleanup dead references
            }
        }
    }
}
```

---

### Example 3 — Understanding object lifetime

```csharp
public class ObjectLifetimeDemo
{
    public void DemonstrateGenerations()
    {
        // Create short-lived object
        var temp = new byte[100];
        Console.WriteLine($"Gen: {GC.GetGeneration(temp)}"); // 0
        
        // Force collection
        GC.Collect();
        GC.WaitForPendingFinalizers();
        
        // Create object and keep reference
        var longLived = new byte[100];
        Console.WriteLine($"Gen: {GC.GetGeneration(longLived)}"); // 0
        
        // Survive collections
        GC.Collect();
        Console.WriteLine($"Gen: {GC.GetGeneration(longLived)}"); // 1
        
        GC.Collect();
        Console.WriteLine($"Gen: {GC.GetGeneration(longLived)}"); // 2
    }
}
```

---

### Example 4 — Large Object Heap

```csharp
public class LohDemo
{
    public void DemonstrateLOH()
    {
        // Small object - goes to regular heap
        var small = new byte[1000]; // 1 KB
        Console.WriteLine($"Gen: {GC.GetGeneration(small)}"); // 0
        
        // Large object - goes to LOH
        var large = new byte[100_000]; // ~100 KB
        Console.WriteLine($"Gen: {GC.GetGeneration(large)}"); // 2 (LOH)
        
        // LOH is only collected with Gen 2
        // LOH is NOT compacted by default (can cause fragmentation)
    }
    
    // ✅ Solution for LOH fragmentation: ArrayPool
    public void UseLOHEfficiently()
    {
        var pool = ArrayPool<byte>.Shared;
        var buffer = pool.Rent(100_000); // Reuses existing array
        
        try
        {
            // Use buffer
        }
        finally
        {
            pool.Return(buffer); // Return to pool for reuse
        }
    }
}
```

---

### Example 5 — Memory profiling

```csharp
public class MemoryMonitoring
{
    public void CheckMemoryUsage()
    {
        var before = GC.GetTotalMemory(false);
        
        // Allocate memory
        var data = new byte[1_000_000]; // 1 MB
        
        var after = GC.GetTotalMemory(false);
        Console.WriteLine($"Allocated: {(after - before) / 1024} KB");
        
        // Force GC and check
        data = null;
        GC.Collect();
        GC.WaitForPendingFinalizers();
        GC.Collect();
        
        var final = GC.GetTotalMemory(true);
        Console.WriteLine($"After GC: {final / 1024} KB");
    }
    
    public void GetGCInfo()
    {
        Console.WriteLine($"Gen 0 collections: {GC.CollectionCount(0)}");
        Console.WriteLine($"Gen 1 collections: {GC.CollectionCount(1)}");
        Console.WriteLine($"Gen 2 collections: {GC.CollectionCount(2)}");
        
        var info = GC.GetGCMemoryInfo();
        Console.WriteLine($"Heap size: {info.HeapSizeBytes / 1024 / 1024} MB");
        Console.WriteLine($"Fragmented: {info.FragmentedBytes / 1024} KB");
    }
}
```

---

## Common Pitfalls

### ❌ Pitfall 1: Calling GC.Collect manually

```csharp
// ❌ DON'T DO THIS in production
GC.Collect();

// ✅ Trust the GC to do its job
// Only call GC.Collect in very specific scenarios:
// - After loading large amounts of data you won't need
// - In interactive applications during idle time
// - When profiling/benchmarking
```

---

### ❌ Pitfall 2: Not disposing IDisposable

```csharp
// ❌ Resource leak
var stream = new FileStream("file.txt", FileMode.Open);
stream.Read(buffer, 0, buffer.Length);
// Forgot to dispose!

// ✅ Always use 'using'
using var stream = new FileStream("file.txt", FileMode.Open);
stream.Read(buffer, 0, buffer.Length);
// Automatically disposed
```

---

### ❌ Pitfall 3: Finalizers are expensive

```csharp
// ❌ Unnecessary finalizer
public class MyClass
{
    ~MyClass() // Finalizer adds overhead
    {
        // Only needed if you have unmanaged resources
    }
}

// ✅ Only add finalizer if you have unmanaged resources
// And always call GC.SuppressFinalize in Dispose
```

---

### ❌ Pitfall 4: LOH fragmentation

```csharp
// ❌ Repeated large allocations cause fragmentation
for (int i = 0; i < 1000; i++)
{
    var large = new byte[100_000];
    // Use and discard
}

// ✅ Use ArrayPool to reuse buffers
var pool = ArrayPool<byte>.Shared;
for (int i = 0; i < 1000; i++)
{
    var large = pool.Rent(100_000);
    try
    {
        // Use buffer
    }
    finally
    {
        pool.Return(large);
    }
}
```

---

### ❌ Pitfall 5: Keeping references in collections

```csharp
// ❌ Static collection prevents GC
public static List<object> Cache = new();

public void AddToCache(object obj)
{
    Cache.Add(obj); // obj can never be collected!
}

// ✅ Use weak references or cache eviction
public static List<WeakReference> Cache = new();

public void AddToCache(object obj)
{
    Cache.Add(new WeakReference(obj)); // Can be collected
}

// ✅ Or use MemoryCache with expiration
private static MemoryCache _cache = new MemoryCache(new MemoryCacheOptions());

public void AddToCache(string key, object obj)
{
    _cache.Set(key, obj, TimeSpan.FromMinutes(5));
}
```

---

## Quick Self-Check

1. What's the difference between stack and heap?
2. What are GC generations and why do they exist?
3. When should you implement `IDisposable`?
4. Can you have memory leaks in .NET? How?
5. What is the Large Object Heap and when is it used?
6. Why shouldn't you call `GC.Collect()` in production?

<details>
<summary>Answers</summary>

1. Stack: fast, automatic, value types, limited size. Heap: managed, reference types, larger, GC-managed
2. Gen 0 (new), Gen 1 (survived once), Gen 2 (long-lived). Optimize performance by collecting short-lived objects frequently
3. When you have unmanaged resources (file handles, connections, etc.) that need explicit cleanup
4. Yes—event handlers not unsubscribed, static references, undisposed resources, captured closures
5. LOH stores objects ≥85KB. Collected with Gen 2, not compacted by default
6. GC is highly optimized. Manual collection disrupts heuristics, causes unnecessary work, and can hurt performance

</details>

---

## Further Practice

1. **Profile** an application with a memory profiler (dotMemory, PerfView)
2. **Create** a memory leak scenario and diagnose it
3. **Implement** a caching system with proper weak references
4. **Benchmark** ArrayPool vs repeated allocations
5. **Analyze** GC logs from a production application

---

## Key Takeaways

✅ Stack for value types, heap for reference types  
✅ GC uses generations to optimize collection  
✅ Implement `IDisposable` for unmanaged resources  
✅ Always dispose `IDisposable` with `using`  
✅ Memory leaks possible via events, static refs, undisposed resources  
✅ LOH for large objects (≥85KB), causes fragmentation  
✅ Use `ArrayPool<T>` for large temporary buffers  
✅ Trust the GC—don't call `GC.Collect()` manually  
✅ `GC.SuppressFinalize()` in Dispose if you have finalizer
