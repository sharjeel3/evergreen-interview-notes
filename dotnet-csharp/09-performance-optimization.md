# Performance Optimization

**Level:** 🔥 Advanced  
**Tags:** `performance`, `optimization`, `benchmarking`, `Span<T>`, `interview`

---

## Why this matters

Performance optimization separates **great engineers from good ones**.

Interviewers look for:
- Ability to identify bottlenecks
- Knowledge of modern performance features (`Span<T>`, `Memory<T>`, `ValueTask`)
- Understanding of allocation and GC impact
- Benchmarking methodology
- Practical optimization strategies

Production systems demand efficient code that scales.

---

## Core Ideas

### 1. **Measure before optimizing**

**Golden rule:** "Premature optimization is the root of all evil."

Always:
1. Profile to find bottlenecks
2. Benchmark before and after
3. Optimize the hot path only
4. Verify improvement with metrics

```csharp
// Use BenchmarkDotNet for accurate measurements
[MemoryDiagnoser]
public class StringConcatBenchmarks
{
    [Benchmark]
    public string StringConcat()
    {
        var result = "";
        for (int i = 0; i < 100; i++)
            result += i.ToString();
        return result;
    }
    
    [Benchmark]
    public string StringBuilder()
    {
        var sb = new StringBuilder();
        for (int i = 0; i < 100; i++)
            sb.Append(i);
        return sb.ToString();
    }
}
```

**Mental model:**  
"Profile → Measure → Optimize → Verify. Never guess."

---

### 2. **Span<T> and Memory<T> reduce allocations**

`Span<T>` is a **ref struct** that provides a view over contiguous memory without allocations.

```csharp
// ❌ Allocates substring
string text = "Hello, World!";
string hello = text.Substring(0, 5); // Allocates new string

// ✅ No allocation with Span
ReadOnlySpan<char> span = text.AsSpan();
ReadOnlySpan<char> hello = span.Slice(0, 5); // No allocation!
```

**Benefits:**
- Zero allocations for slicing
- Works with arrays, strings, stack memory
- Safe (bounds-checked)
- High performance

---

### 3. **ValueTask for high-performance async**

Use `ValueTask<T>` instead of `Task<T>` when result is often available synchronously.

```csharp
// ❌ Always allocates Task
public async Task<int> GetCachedValueAsync(string key)
{
    if (_cache.TryGetValue(key, out int value))
        return value; // Still allocates Task!
    
    return await FetchFromDbAsync(key);
}

// ✅ No allocation when cached
public async ValueTask<int> GetCachedValueAsync(string key)
{
    if (_cache.TryGetValue(key, out int value))
        return value; // No Task allocation!
    
    return await FetchFromDbAsync(key);
}
```

**Rule:** Use `ValueTask<T>` for hot paths where synchronous completion is common.

---

### 4. **ArrayPool and MemoryPool reduce GC pressure**

Reuse arrays instead of allocating:

```csharp
// ❌ Allocates and GC collects every time
public void ProcessData()
{
    byte[] buffer = new byte[4096];
    // Use buffer
} // Buffer is now garbage

// ✅ Rent from pool
public void ProcessData()
{
    var pool = ArrayPool<byte>.Shared;
    byte[] buffer = pool.Rent(4096); // Reuses existing array
    try
    {
        // Use buffer
    }
    finally
    {
        pool.Return(buffer); // Return for reuse
    }
}
```

---

### 5. **Common performance patterns**

| Pattern | Slow | Fast |
|---------|------|------|
| String concatenation | `str += x` | `StringBuilder` |
| LINQ on hot path | `.Where().Select()` | `for` loop |
| Casting | `(Type)obj` repeatedly | Cache cast result |
| Dictionary lookup | Multiple `dict[key]` | `TryGetValue` once |
| Enumeration | `IEnumerable<T>` | `IList<T>` or array |
| Large allocations | `new byte[100000]` | `ArrayPool<byte>.Shared` |
| Async when cached | `Task<T>` | `ValueTask<T>` |
| String operations | `Substring`, `Split` | `Span<T>` |

---

## Examples

### Example 1 — Span<T> for string parsing

```csharp
// ❌ Allocates many strings
public (string, string) ParseOld(string input)
{
    var parts = input.Split(','); // Allocates array + strings
    return (parts[0].Trim(), parts[1].Trim()); // More allocations
}

// ✅ Zero allocations with Span
public (ReadOnlySpan<char>, ReadOnlySpan<char>) ParseNew(ReadOnlySpan<char> input)
{
    int commaIndex = input.IndexOf(',');
    if (commaIndex == -1)
        return (input, ReadOnlySpan<char>.Empty);
    
    var left = input.Slice(0, commaIndex).Trim();
    var right = input.Slice(commaIndex + 1).Trim();
    return (left, right);
}

// Usage
string data = "Alice, 30";
var (name, age) = ParseNew(data.AsSpan());
// No allocations!
```

---

### Example 2 — ValueTask for caching

```csharp
public class CachedDataService
{
    private readonly Dictionary<string, Data> _cache = new();
    private readonly IDataStore _store;
    
    public async ValueTask<Data> GetDataAsync(string key)
    {
        // Check cache - no Task allocation if found
        if (_cache.TryGetValue(key, out var data))
            return data;
        
        // Fetch from store - only allocates when needed
        data = await _store.FetchAsync(key);
        _cache[key] = data;
        return data;
    }
}
```

---

### Example 3 — ArrayPool for buffers

```csharp
public class FileProcessor
{
    public async Task ProcessFileAsync(string path)
    {
        var pool = ArrayPool<byte>.Shared;
        byte[] buffer = pool.Rent(4096);
        
        try
        {
            using var stream = File.OpenRead(path);
            int bytesRead;
            while ((bytesRead = await stream.ReadAsync(buffer, 0, buffer.Length)) > 0)
            {
                ProcessChunk(buffer.AsSpan(0, bytesRead));
            }
        }
        finally
        {
            pool.Return(buffer);
        }
    }
    
    private void ProcessChunk(Span<byte> data)
    {
        // Process data
    }
}
```

---

### Example 4 — Struct enumerators

```csharp
// ❌ LINQ allocates enumerator
var result = numbers.Where(x => x > 5).Select(x => x * 2).ToList();

// ✅ Struct enumerator - no allocation
public struct FilteredEnumerator<T>
{
    private T[] _array;
    private Func<T, bool> _predicate;
    private int _index;
    
    public FilteredEnumerator(T[] array, Func<T, bool> predicate)
    {
        _array = array;
        _predicate = predicate;
        _index = -1;
    }
    
    public T Current => _array[_index];
    
    public bool MoveNext()
    {
        while (++_index < _array.Length)
        {
            if (_predicate(_array[_index]))
                return true;
        }
        return false;
    }
}

// Usage - zero allocations
var numbers = new[] { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
var enumerator = new FilteredEnumerator<int>(numbers, x => x > 5);
while (enumerator.MoveNext())
{
    Console.WriteLine(enumerator.Current * 2);
}
```

---

### Example 5 — Benchmarking with BenchmarkDotNet

```csharp
using BenchmarkDotNet.Attributes;
using BenchmarkDotNet.Running;

[MemoryDiagnoser]
[RankColumn]
public class ParsingBenchmarks
{
    private const string Data = "123,456,789";
    
    [Benchmark(Baseline = true)]
    public int[] ParseWithSplit()
    {
        return Data.Split(',').Select(int.Parse).ToArray();
    }
    
    [Benchmark]
    public int[] ParseWithSpan()
    {
        var result = new int[3];
        var span = Data.AsSpan();
        int resultIndex = 0;
        
        while (!span.IsEmpty)
        {
            int commaIndex = span.IndexOf(',');
            ReadOnlySpan<char> part;
            
            if (commaIndex >= 0)
            {
                part = span.Slice(0, commaIndex);
                span = span.Slice(commaIndex + 1);
            }
            else
            {
                part = span;
                span = ReadOnlySpan<char>.Empty;
            }
            
            result[resultIndex++] = int.Parse(part);
        }
        
        return result;
    }
}

// Run benchmarks
class Program
{
    static void Main(string[] args)
    {
        BenchmarkRunner.Run<ParsingBenchmarks>();
    }
}

// Results:
// |           Method |     Mean | Allocated |
// |----------------- |---------:|----------:|
// |  ParseWithSplit | 450.0 ns |     584 B |
// |   ParseWithSpan | 120.0 ns |      72 B |
```

---

### Example 6 — Avoiding closure allocations

```csharp
// ❌ Captures 'multiplier' - allocates closure
public List<Func<int, int>> CreateMultipliersOld(int multiplier)
{
    var funcs = new List<Func<int, int>>();
    for (int i = 0; i < 10; i++)
    {
        funcs.Add(x => x * multiplier); // Closure allocation
    }
    return funcs;
}

// ✅ Pass state explicitly - no closure
public List<Multiplier> CreateMultipliersNew(int multiplier)
{
    var funcs = new List<Multiplier>();
    for (int i = 0; i < 10; i++)
    {
        funcs.Add(new Multiplier(multiplier)); // Struct, no heap allocation
    }
    return funcs;
}

public readonly struct Multiplier
{
    private readonly int _multiplier;
    
    public Multiplier(int multiplier)
    {
        _multiplier = multiplier;
    }
    
    public int Apply(int value) => value * _multiplier;
}
```

---

## Common Pitfalls

### ❌ Pitfall 1: String concatenation in loops

```csharp
// ❌ O(n²) complexity - allocates new string each iteration
string result = "";
for (int i = 0; i < 1000; i++)
{
    result += i.ToString(); // Terrible!
}

// ✅ O(n) with StringBuilder
var sb = new StringBuilder();
for (int i = 0; i < 1000; i++)
{
    sb.Append(i);
}
string result = sb.ToString();
```

---

### ❌ Pitfall 2: LINQ on hot path

```csharp
// ❌ Allocates enumerators, iterators
public int ProcessData(List<int> numbers)
{
    return numbers
        .Where(x => x > 0)
        .Select(x => x * 2)
        .Sum();
}

// ✅ Direct loop - zero allocations
public int ProcessData(List<int> numbers)
{
    int sum = 0;
    for (int i = 0; i < numbers.Count; i++)
    {
        int n = numbers[i];
        if (n > 0)
            sum += n * 2;
    }
    return sum;
}
```

---

### ❌ Pitfall 3: Boxing value types

```csharp
// ❌ Boxing on every comparison
ArrayList list = new ArrayList { 1, 2, 3, 4, 5 };
foreach (object item in list) // Boxing
{
    Console.WriteLine(item); // Boxing
}

// ✅ Generic collection - no boxing
List<int> list = new List<int> { 1, 2, 3, 4, 5 };
foreach (int item in list) // No boxing
{
    Console.WriteLine(item);
}
```

---

### ❌ Pitfall 4: Repeated Dictionary lookups

```csharp
// ❌ Looks up twice
if (dict.ContainsKey(key))
{
    var value = dict[key]; // Second lookup!
}

// ✅ Single lookup
if (dict.TryGetValue(key, out var value))
{
    // Use value
}
```

---

### ❌ Pitfall 5: Large allocations without pooling

```csharp
// ❌ Allocates 1MB each time - LOH pressure
public void ProcessRequests()
{
    for (int i = 0; i < 1000; i++)
    {
        byte[] buffer = new byte[1_000_000]; // 1 MB
        // Process
    } // GC collects 1000 MB!
}

// ✅ Rent from pool
public void ProcessRequests()
{
    var pool = ArrayPool<byte>.Shared;
    for (int i = 0; i < 1000; i++)
    {
        byte[] buffer = pool.Rent(1_000_000);
        try
        {
            // Process
        }
        finally
        {
            pool.Return(buffer);
        }
    }
}
```

---

## Quick Self-Check

1. What's the first step before optimizing code?
2. What are `Span<T>` and `Memory<T>` used for?
3. When should you use `ValueTask<T>` instead of `Task<T>`?
4. What's the performance difference between `StringBuilder` and `+=`?
5. How do you avoid allocations with large buffers?

<details>
<summary>Answers</summary>

1. Profile to identify bottlenecks, then benchmark to establish baseline
2. Zero-allocation views over contiguous memory (arrays, strings, stack)
3. When synchronous completion is common (e.g., cached results)
4. `StringBuilder` is O(n), `+=` is O(n²) due to repeated allocations
5. Use `ArrayPool<T>.Shared` to rent and return buffers

</details>

---

## Further Practice

1. **Benchmark** string concatenation vs `StringBuilder` vs `String.Concat` vs interpolation
2. **Refactor** a LINQ query to use `Span<T>` and measure improvement
3. **Profile** an application with dotTrace or PerfView
4. **Implement** a zero-allocation JSON parser for simple formats
5. **Optimize** a hot path method reducing allocations by 90%

---

## Performance Optimization Checklist

When optimizing code:

✅ **Profile first** - Use profiler to find bottlenecks  
✅ **Benchmark** - Use BenchmarkDotNet for accurate measurements  
✅ **Reduce allocations** - Use `Span<T>`, `ArrayPool<T>`, structs  
✅ **Avoid LINQ on hot path** - Use direct loops  
✅ **Cache expensive operations** - Type lookups, regex, config  
✅ **Use `StringBuilder`** - For repeated string concatenation  
✅ **Single dictionary lookup** - Use `TryGetValue` not `ContainsKey` + indexer  
✅ **ValueTask for async** - When synchronous completion is common  
✅ **Struct enumerators** - Custom iterators without allocation  
✅ **Avoid boxing** - Use generic collections and methods

---

## Key Takeaways

✅ Always profile before optimizing  
✅ `Span<T>` for zero-allocation slicing and parsing  
✅ `ValueTask<T>` when result often available synchronously  
✅ `ArrayPool<T>` for large temporary buffers  
✅ Avoid string concatenation in loops - use `StringBuilder`  
✅ Direct loops faster than LINQ on hot paths  
✅ Single `TryGetValue` better than `ContainsKey` + indexer  
✅ BenchmarkDotNet for accurate performance measurements  
✅ Struct types avoid heap allocations  
✅ Generic collections prevent boxing
