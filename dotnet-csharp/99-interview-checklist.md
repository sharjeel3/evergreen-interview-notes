# .NET / C# Interview Checklist

**Quick reference for interview preparation**

Use this checklist to verify you can confidently discuss each topic. Focus on the "Must Know" items first, then expand to "Nice to Have" as time permits.

---

## 🎯 How to Use This Checklist

**1-2 weeks out:**
- Review all "Must Know" topics
- Practice explaining concepts out loud
- Code at least 3 examples per topic

**2-3 days out:**
- Scan "Common Interview Questions"
- Review "Red Flags to Avoid"
- Practice whiteboard coding

**Night before:**
- Read through "Quick Talking Points"
- Review your weak areas
- Get good sleep!

---

## ✅ Core Concepts (Must Know)

### Value vs Reference Types
- [ ] Explain stack vs heap allocation
- [ ] Describe value type vs reference type behavior
- [ ] What is boxing and why is it expensive?
- [ ] When to use `struct` vs `class`
- [ ] How `ref` and `out` parameters work

**Quick talking point:**  
"Value types store data directly on the stack (or inline), reference types store a pointer to heap memory. Boxing converts value types to objects on the heap, causing allocations and GC pressure."

---

### Collections & Generics
- [ ] Difference between `IEnumerable<T>`, `ICollection<T>`, `IList<T>`
- [ ] When to use `List<T>` vs `Dictionary<TKey, TValue>` vs `HashSet<T>`
- [ ] What is deferred execution in LINQ?
- [ ] Why use generics over `ArrayList`?
- [ ] Big O complexity of common collections

**Quick talking point:**  
"Generics provide type safety and eliminate boxing. `IEnumerable<T>` enables deferred execution and LINQ. Choose `List<T>` for ordered data, `Dictionary` for fast lookup, `HashSet` for unique items."

---

### Exception Handling
- [ ] When to use exceptions vs return values
- [ ] Difference between `throw` and `throw ex`
- [ ] Purpose of `finally` block
- [ ] How to implement custom exceptions
- [ ] Exception filters (C# 6+)
- [ ] Try-Catch best practices

**Quick talking point:**  
"Exceptions are for exceptional conditions, not control flow. Always use `throw;` to preserve stack trace. Finally block guarantees cleanup. Use `using` for `IDisposable` resources."

---

### LINQ & Delegates
- [ ] What is a delegate and how does it work?
- [ ] Difference between `Func<T>`, `Action<T>`, `Predicate<T>`
- [ ] Lambda expressions vs anonymous methods
- [ ] Deferred execution vs immediate execution
- [ ] Query syntax vs method syntax
- [ ] `IEnumerable<T>` vs `IQueryable<T>`

**Quick talking point:**  
"Delegates are type-safe function pointers. LINQ uses deferred execution—queries execute when enumerated. Use `IQueryable<T>` for database queries (translated to SQL), `IEnumerable<T>` for in-memory collections."

---

### Async/Await & TPL
- [ ] What problem does async/await solve?
- [ ] Difference between async and multi-threading
- [ ] I/O-bound vs CPU-bound operations
- [ ] What is `SynchronizationContext`?
- [ ] Why does `.Result` cause deadlocks?
- [ ] When to use `ConfigureAwait(false)`
- [ ] `Task` vs `ValueTask<T>`

**Quick talking point:**  
"Async improves scalability by freeing threads during I/O, not speed. Never block on async with `.Result`. Use `Task.Run` for CPU-bound work, native async APIs for I/O. `ConfigureAwait(false)` in library code to avoid context capture."

---

### Dependency Injection
- [ ] What is IoC and why use it?
- [ ] Difference between Transient, Scoped, Singleton
- [ ] Constructor vs property injection
- [ ] What is captive dependency?
- [ ] How does DI improve testability?
- [ ] Service locator anti-pattern

**Quick talking point:**  
"DI inverts control of object creation to a container. Transient creates new instances, Scoped one per request, Singleton one for lifetime. Constructor injection makes dependencies explicit and enables testing with mocks."

---

## 🔥 Advanced Topics (Nice to Have)

### Memory Management & GC
- [ ] GC generations (Gen 0, 1, 2, LOH)
- [ ] When and why GC collects
- [ ] `IDisposable` pattern implementation
- [ ] How to cause memory leaks in .NET
- [ ] Workstation vs Server GC
- [ ] What triggers LOH allocation?

**Quick talking point:**  
"GC uses generations to optimize collection. Most objects die young (Gen 0). Implement `IDisposable` for unmanaged resources. Memory leaks possible via event handlers, static refs, or undisposed resources. LOH for objects ≥85KB, not compacted by default."

---

### Reflection & Attributes
- [ ] When to use reflection
- [ ] Performance implications
- [ ] How to create custom attributes
- [ ] Caching reflection objects
- [ ] Expression trees for compiled delegates
- [ ] Assembly loading and plugins

**Quick talking point:**  
"Reflection enables runtime type inspection—useful for DI, serialization, plugins. 50-100x slower than direct calls. Always cache `Type`, `MethodInfo`, `PropertyInfo`. Use compiled expressions for repeated calls."

---

### Performance Optimization
- [ ] `Span<T>` and `Memory<T>` benefits
- [ ] `ArrayPool<T>` for buffer reuse
- [ ] ValueTask for async optimization
- [ ] String concatenation best practices
- [ ] LINQ vs direct loops performance
- [ ] How to use BenchmarkDotNet

**Quick talking point:**  
"`Span<T>` provides zero-allocation slicing. `ArrayPool<T>` reuses buffers to reduce GC. `ValueTask<T>` avoids allocation when result is cached. Always profile before optimizing—never guess."

---

## 💡 Common Interview Questions

### Beginner Level

**Q: What's the difference between `string` and `StringBuilder`?**  
A: `string` is immutable—each concatenation creates a new string. `StringBuilder` is mutable and efficient for repeated concatenations. Use `StringBuilder` in loops, `string` for few operations.

**Q: What is boxing?**  
A: Converting a value type to `object` (heap allocation). Occurs with non-generic collections or assigning to `object`. Avoid with generics.

**Q: Explain `using` statement.**  
A: Syntactic sugar for `try-finally` with `Dispose()`. Ensures `IDisposable` resources are cleaned up, even if exception occurs.

---

### Intermediate Level

**Q: What is deferred execution in LINQ?**  
A: Queries don't execute when defined, only when enumerated (foreach, ToList, Count, etc.). Enables query composition and optimization.

**Q: Why does this deadlock?**
```csharp
public void Method()
{
    var result = GetDataAsync().Result; // Deadlock!
}

public async Task<string> GetDataAsync()
{
    return await httpClient.GetStringAsync(url);
}
```
A: `Result` blocks calling thread. Async method tries to resume on same thread (UI/ASP.NET context). Solution: `await` all the way or use `.ConfigureAwait(false)`.

**Q: What's the difference between Transient and Scoped lifetime?**  
A: Transient creates new instance every time. Scoped creates one instance per request/scope. Use Scoped for `DbContext`, Transient for stateless services.

---

### Advanced Level

**Q: How does GC decide when to collect?**  
A: Triggers on Gen 0 budget exhausted, explicit `GC.Collect()`, low memory, or application idle. Gen 0 collected frequently, Gen 2 rarely.

**Q: Explain captive dependency problem.**  
A: When longer-lived service (Singleton) captures shorter-lived dependency (Scoped), keeping it alive inappropriately. Fix by injecting `IServiceProvider` and creating scopes.

**Q: How would you optimize this?**
```csharp
for (int i = 0; i < 1000; i++)
{
    result += i.ToString();
}
```
A: Use `StringBuilder` to avoid O(n²) allocations. Or `String.Join` with range. String concatenation in loops creates new string each iteration.

---

## 🚨 Red Flags to Avoid

### Don't Say These

❌ "Async makes code faster"  
✅ "Async improves scalability by freeing threads during I/O"

❌ "I use reflection whenever I need dynamic behavior"  
✅ "I use reflection sparingly due to performance cost and cache reflection objects"

❌ "Just call `GC.Collect()` to free memory"  
✅ "Trust the GC—manual collection disrupts its heuristics"

❌ "Exceptions are expensive, never use them"  
✅ "Exceptions are for exceptional conditions. Use Try* patterns for expected failures"

❌ "I prefer property injection for flexibility"  
✅ "I use constructor injection to make dependencies explicit and required"

---

## 🎤 Behavioral Interview Tips

### When Asked: "Tell me about a time you optimized performance"

Good answer structure:
1. **Situation:** Describe the problem (slow API, high memory, etc.)
2. **Action:** Profiled with tool X, identified bottleneck Y
3. **Result:** Reduced response time by Z%, improved throughput by N%
4. **Learning:** What you'd do differently, insights gained

Example:
> "Our API was timing out under load. I used dotTrace to profile and found LINQ queries in a tight loop causing excessive allocations. I refactored to use direct loops and `Span<T>` for parsing, reducing allocations by 90% and cutting response time from 800ms to 120ms. This taught me to always profile before optimizing and that micro-optimizations matter in hot paths."

---

### When Asked: "What's your approach to learning new tech?"

Good answer:
> "I follow a three-step approach: First, I read official docs to understand core concepts. Second, I build small projects to apply the knowledge hands-on. Third, I teach others or write notes to solidify understanding. For example, when learning async/await, I built a sample app, read Stephen Cleary's blog, and explained it to my team."

---

## 🔧 Practical Coding Tips

### Whiteboard Coding

1. **Clarify requirements** before writing code
2. **Talk through your thought process** 
3. **Start with simple solution**, then optimize
4. **Test with edge cases** (null, empty, large input)
5. **Explain Big O complexity**

### Common Whiteboard Questions

**Reverse a string:**
```csharp
public static string Reverse(string input)
{
    if (string.IsNullOrEmpty(input)) return input;
    
    char[] chars = input.ToCharArray();
    Array.Reverse(chars);
    return new string(chars);
    
    // Or with Span<T>:
    // Span<char> span = stackalloc char[input.Length];
    // for (int i = 0; i < input.Length; i++)
    //     span[i] = input[input.Length - 1 - i];
    // return new string(span);
}
```

**Find duplicates in array:**
```csharp
public static List<int> FindDuplicates(int[] numbers)
{
    var seen = new HashSet<int>();
    var duplicates = new HashSet<int>();
    
    foreach (int num in numbers)
    {
        if (!seen.Add(num))
            duplicates.Add(num);
    }
    
    return duplicates.ToList();
}
```

**Implement simple cache with expiration:**
```csharp
public class SimpleCache<TKey, TValue>
{
    private readonly Dictionary<TKey, (TValue Value, DateTime Expiry)> _cache = new();
    private readonly TimeSpan _defaultExpiration;
    
    public SimpleCache(TimeSpan defaultExpiration)
    {
        _defaultExpiration = defaultExpiration;
    }
    
    public void Set(TKey key, TValue value)
    {
        _cache[key] = (value, DateTime.UtcNow.Add(_defaultExpiration));
    }
    
    public bool TryGet(TKey key, out TValue value)
    {
        if (_cache.TryGetValue(key, out var entry) && entry.Expiry > DateTime.UtcNow)
        {
            value = entry.Value;
            return true;
        }
        
        value = default;
        _cache.Remove(key); // Cleanup expired
        return false;
    }
}
```

---

## 📋 Pre-Interview Day Checklist

**24 hours before:**
- [ ] Review this checklist
- [ ] Scan all lesson "Core Ideas" sections
- [ ] Practice explaining 3 topics out loud
- [ ] Code 2-3 whiteboard problems
- [ ] Get 8 hours of sleep

**Morning of:**
- [ ] Review "Quick Talking Points"
- [ ] Scan "Red Flags to Avoid"
- [ ] Eat a good breakfast
- [ ] Stay hydrated
- [ ] Arrive early / test tech setup

---

## 🎯 Final Tips

✅ **It's okay to say "I don't know"** - Follow with how you'd find out  
✅ **Ask clarifying questions** - Shows thoughtfulness  
✅ **Explain your reasoning** - Process matters more than perfect answer  
✅ **Admit mistakes** - "Good catch, I'd fix that in code review"  
✅ **Be honest about experience** - Better to admit gaps than fake it

---

## 📚 Quick Reference Links

- [C# Docs](https://learn.microsoft.com/en-us/dotnet/csharp/)
- [.NET Performance Tips](https://learn.microsoft.com/en-us/dotnet/framework/performance/)
- [Async Best Practices](https://learn.microsoft.com/en-us/archive/msdn-magazine/2013/march/async-await-best-practices-in-asynchronous-programming)
- [BenchmarkDotNet](https://benchmarkdotnet.org/)

---

**Good luck! 🚀**

Remember: Interviewers want to see how you think, not just what you know. Be curious, be honest, and be confident in your ability to learn.
