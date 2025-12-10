# Threading & Synchronization

**Level:** 🚦 Intermediate  
**Tags:** `threading`, `concurrency`, `synchronization`, `locks`, `thread-safety`, `interview`

---

## Why this matters

Threading and synchronization are among the hardest topics in .NET, yet they're essential for writing correct, performant concurrent code. Most developers use async/await without understanding the underlying threading model, leading to subtle race conditions, deadlocks, and data corruption that only appear in production under load.

**Threading bugs are silent killers in production.** I've debugged race conditions that occurred once per million requests, caused data corruption in financial systems, and only manifested under specific timing conditions impossible to reproduce locally. One team had a critical race condition in their payment processing system that existed for 18 months before it caused duplicate charges totaling $250,000. The bug was a classic: shared mutable state accessed by multiple threads without synchronization. Threading bugs don't crash immediately—they silently corrupt data, cause intermittent failures, and create Heisenbugs that disappear when you try to debug them.

**Interviews heavily test concurrency because it reveals systems thinking.** When an interviewer asks "How would you make this class thread-safe?" or "What's the difference between lock and Monitor?", they're not testing memorization—they're assessing whether you understand the fundamental challenge of concurrent programming: coordinating access to shared mutable state. Senior engineers must understand the performance implications of different synchronization primitives, when to use lock-free algorithms, and how to design systems that minimize shared state. I've seen candidates solve complex algorithm problems but fail on basic threading questions, immediately disqualifying them for senior roles.

**Choosing the wrong synchronization primitive kills performance.** Lock is simple but coarse-grained. ReaderWriterLockSlim allows concurrent reads but is complex. SemaphoreSlim controls resource pooling. Interlocked provides lock-free atomicity. Using a Mutex when you need a lock, or a lock when you need Interlocked, can make code 10-100x slower. I've seen APIs timeout because developers used synchronous locks in async code (blocking thread pool threads), or wrapped every property access in locks (massive contention overhead), or failed to use ReaderWriterLockSlim for read-heavy scenarios (sequential reads that should have been parallel).

**Thread safety vs performance is a constant tradeoff.** Making everything thread-safe with locks is easy but slow. Lock-free algorithms are fast but complex and bug-prone. The art is knowing when each approach is appropriate. I've reviewed code where developers made singleton initialization thread-safe using double-checked locking but got the memory barrier wrong, causing subtle bugs on ARM processors. Or used ConcurrentDictionary everywhere "to be safe" but destroyed performance with unnecessary synchronization overhead. Thread safety isn't binary—it's a spectrum of tradeoffs.

**Async/await doesn't eliminate threading concerns.** Many developers think async makes threading problems disappear. It doesn't. You still need to synchronize access to shared state. I've seen codebases where developers assumed async operations were somehow atomic, leading to race conditions in "async-first" code. The relationship between async and threading is subtle: async/await is about not blocking threads during I/O, but it doesn't provide atomicity or isolation. You still need locks, semaphores, or lock-free techniques for shared mutable state.

**Deadlocks are surprisingly easy to create.** Classic scenario: Thread A locks resource 1 then tries to lock resource 2; Thread B locks resource 2 then tries to lock resource 1. Deadlock. I've debugged production deadlocks in web servers where request threads were completely frozen, requiring process restarts every few hours. The fix was simple (consistent lock ordering), but finding the bug required production profiling because it only happened under specific load patterns. Understanding deadlock causes—mutual exclusion, hold and wait, no preemption, circular wait—is essential for prevention.

**ThreadPool vs Thread vs Task is frequently misunderstood.** Threads are expensive (1MB stack, OS overhead). ThreadPool reuses threads efficiently. Task is a higher-level abstraction over ThreadPool. Using `new Thread()` for short-lived work is wasteful and doesn't scale. I've seen applications create thousands of threads under load, causing memory exhaustion and OS scheduler thrashing, when they should have used ThreadPool or Task.Run(). Conversely, long-running background work shouldn't starve the ThreadPool—it needs dedicated threads. Knowing when to use each shows understanding of resource management.

**Memory visibility and happens-before relationships are subtle.** Without proper synchronization, one thread's writes may not be visible to other threads due to CPU caching and compiler optimizations. The `volatile` keyword, `Interlocked` operations, and locks all establish memory barriers ensuring visibility. I've debugged issues where a flag set by one thread wasn't seen by another thread for seconds because it wasn't marked `volatile`. This is advanced, but senior engineers need to understand it because these bugs are nearly impossible to diagnose without this knowledge.

**Race conditions manifest as flaky tests and "impossible" production bugs.** Tests that pass 99.9% of the time but occasionally fail? Race condition. Production bugs where "impossible" state combinations occur? Race condition. I've spent weeks debugging race conditions in caching layers, where two threads simultaneously checked if a cache entry exists (both see it doesn't), both fetch from database, both write to cache, causing duplicate work and inconsistent state. The fix—a SemaphoreSlim to ensure only one thread fetches per cache key—was 3 lines. Finding it took 100 hours.

**Synchronization primitives have non-obvious costs and limitations.** Lock creates a Monitor object (heap allocation). SemaphoreSlim is more efficient but requires disposal. ReaderWriterLockSlim is fast for read-heavy but has write starvation issues. Mutex works cross-process but is slow. These details matter at scale. I've seen production performance issues where replacing locks with Interlocked operations eliminated 95% of contention overhead, cutting P99 latencies from 200ms to 10ms.

**Career impact is substantial.** Threading expertise is rare and highly valued. Many developers avoid concurrency, hoping async/await is enough. Those who master threading—who can debug race conditions, choose appropriate synchronization, and design lock-free algorithms—command premium salaries and respect. When you can walk into a production incident where threads are deadlocked, analyze the thread dump, identify the lock ordering issue, and propose a fix in minutes, you're operating at staff+ level.

---

## Core Ideas

### 1. **Thread vs Task vs ThreadPool**

**.NET has multiple levels of abstraction for concurrency:**

**Thread:**
- OS-level thread (1MB stack, expensive to create)
- Full control but manual management
- Use for: long-running background work

```csharp
var thread = new Thread(() => { /* work */ });
thread.IsBackground = true;
thread.Start();
```

**ThreadPool:**
- Reusable pool of worker threads
- Efficient for short-lived work
- Use for: CPU-bound work that completes quickly

```csharp
ThreadPool.QueueUserWorkItem(_ => { /* work */ });
```

**Task:**
- High-level abstraction over ThreadPool
- Composable, supports async/await, cancellation
- Use for: modern code, default choice

```csharp
await Task.Run(() => { /* CPU work */ });
```

**Mental model:**  
"Task/ThreadPool for most work; dedicated Thread only for long-running background jobs."

---

### 2. **Race Conditions: The Core Concurrency Problem**

A **race condition** occurs when correctness depends on the relative timing of threads.

```csharp
// ❌ RACE CONDITION
private int _counter = 0;

public void Increment()
{
    _counter++;  // Read-Modify-Write: NOT atomic!
}

// Two threads execute simultaneously:
// Thread 1: read 0 → calculate 1 → (context switch)
// Thread 2: read 0 → calculate 1 → write 1
// Thread 1: write 1
// Result: 1 (expected 2) — lost update!
```

**How to fix:**
- Use synchronization primitives (lock, Interlocked)
- Eliminate shared mutable state
- Use thread-safe collections

---

### 3. **Lock: Mutual Exclusion**

**`lock`** ensures only one thread executes a critical section at a time.

```csharp
private readonly object _lock = new object();
private int _counter = 0;

public void Increment()
{
    lock (_lock)
    {
        _counter++;  // Now thread-safe
    }
}
```

**How it works:**
- `lock (obj)` is syntactic sugar for `Monitor.Enter(obj)` / `Monitor.Exit(obj)`
- Only one thread can hold the lock at a time
- Other threads block (wait) until the lock is released

**Rules:**
- Lock on a **private object** (not `this`, not `typeof(MyClass)`)
- Keep critical sections **short** (minimize lock contention)
- **Never** lock on a value type (gets boxed, different instance each time!)

---

### 4. **Monitor: Low-Level Lock Control**

**`Monitor`** is what `lock` compiles to, offering more control.

```csharp
private readonly object _lock = new object();

public void DoWork()
{
    bool lockTaken = false;
    try
    {
        Monitor.Enter(_lock, ref lockTaken);
        
        // Critical section
        _counter++;
    }
    finally
    {
        if (lockTaken)
            Monitor.Exit(_lock);
    }
}
```

**Advanced Monitor features:**
- `Monitor.TryEnter(obj, timeout)` — try to acquire lock with timeout
- `Monitor.Wait(obj)` — release lock and wait for signal
- `Monitor.Pulse(obj)` — signal one waiting thread
- `Monitor.PulseAll(obj)` — signal all waiting threads

**Producer-Consumer pattern:**
```csharp
private readonly object _lock = new object();
private Queue<int> _queue = new();

public void Produce(int item)
{
    lock (_lock)
    {
        _queue.Enqueue(item);
        Monitor.Pulse(_lock);  // Wake up consumer
    }
}

public int Consume()
{
    lock (_lock)
    {
        while (_queue.Count == 0)
            Monitor.Wait(_lock);  // Release lock and wait
        
        return _queue.Dequeue();
    }
}
```

---

### 5. **Interlocked: Lock-Free Atomicity**

**`Interlocked`** provides atomic operations without locks (uses CPU-level atomic instructions).

```csharp
private int _counter = 0;

public void Increment()
{
    Interlocked.Increment(ref _counter);  // Atomic, lock-free
}

public int Read()
{
    return Interlocked.CompareExchange(ref _counter, 0, 0);  // Atomic read
}
```

**Available operations:**
- `Interlocked.Increment/Decrement` — atomic +1/-1
- `Interlocked.Add` — atomic addition
- `Interlocked.CompareExchange` — atomic compare-and-swap (CAS)
- `Interlocked.Exchange` — atomic write

**Why use Interlocked:**
- **Faster** than locks (no context switch, no blocking)
- **Scalable** (no contention on uncontended paths)
- **Simple** atomicity for counters, flags, references

**When NOT to use:**
Complex operations (multiple variables) still need locks.

---

### 6. **SemaphoreSlim: Limiting Concurrent Access**

**`SemaphoreSlim`** limits how many threads can access a resource concurrently.

```csharp
// Allow max 3 concurrent database connections
private readonly SemaphoreSlim _semaphore = new SemaphoreSlim(3, 3);

public async Task<string> QueryDatabase()
{
    await _semaphore.WaitAsync();  // Acquire slot (block if 3 already in use)
    try
    {
        // Perform database query
        return "result";
    }
    finally
    {
        _semaphore.Release();  // Release slot
    }
}
```

**Use cases:**
- **Throttling** — limit concurrent API calls
- **Resource pooling** — limit database connections
- **Rate limiting** — control request throughput

**SemaphoreSlim vs Semaphore:**
- `SemaphoreSlim` — lightweight, async-friendly, NOT cross-process
- `Semaphore` — cross-process, heavier, no async support

---

### 7. **ReaderWriterLockSlim: Concurrent Reads, Exclusive Writes**

**`ReaderWriterLockSlim`** allows multiple concurrent readers OR one exclusive writer.

```csharp
private readonly ReaderWriterLockSlim _lock = new ReaderWriterLockSlim();
private Dictionary<string, int> _cache = new();

public int Read(string key)
{
    _lock.EnterReadLock();
    try
    {
        return _cache[key];  // Multiple readers allowed
    }
    finally
    {
        _lock.ExitReadLock();
    }
}

public void Write(string key, int value)
{
    _lock.EnterWriteLock();
    try
    {
        _cache[key] = value;  // Exclusive access
    }
    finally
    {
        _lock.ExitWriteLock();
    }
}
```

**When to use:**
- Read-heavy scenarios (90%+ reads)
- Shared data structures (caches, registries)

**Trade-off:**
More complex than `lock`, slight overhead. Only beneficial when reads vastly outnumber writes.

---

### 8. **Mutex: Cross-Process Synchronization**

**`Mutex`** synchronizes threads across processes.

```csharp
// Only one instance of application can run
using var mutex = new Mutex(true, "Global\\MyAppMutex", out bool createdNew);

if (!createdNew)
{
    Console.WriteLine("Another instance is already running");
    return;
}

// Run application
```

**Mutex vs Lock:**
- `lock` — in-process only, fast
- `Mutex` — cross-process, slow (kernel mode)

**Use cases:**
- Single-instance applications
- Cross-process resource locking

---

### 9. **Thread Safety Patterns**

**Pattern 1: Immutability**
```csharp
// ✅ Immutable = thread-safe
public record UserInfo(string Name, int Age);
```

**Pattern 2: Thread-Local Storage**
```csharp
// Each thread has its own instance
private static ThreadLocal<Random> _random = new(() => new Random());
```

**Pattern 3: Concurrent Collections**
```csharp
// Thread-safe without external locks
private ConcurrentDictionary<string, int> _cache = new();
```

**Pattern 4: Double-Checked Locking (Lazy Singleton)**
```csharp
private static readonly Lazy<Singleton> _instance = 
    new Lazy<Singleton>(() => new Singleton());

public static Singleton Instance => _instance.Value;
```

---

## Examples

### Example 1 — Lock vs Interlocked performance

```csharp
// ❌ Slower (lock overhead)
private readonly object _lock = new();
private int _counter = 0;

public void IncrementWithLock()
{
    lock (_lock) { _counter++; }
}

// ✅ Faster (lock-free)
private int _counter = 0;

public void IncrementLockFree()
{
    Interlocked.Increment(ref _counter);
}
```

---

### Example 2 — Deadlock scenario

```csharp
// ❌ DEADLOCK: inconsistent lock ordering
private readonly object _lock1 = new();
private readonly object _lock2 = new();

public void Method1()
{
    lock (_lock1)
    {
        Thread.Sleep(10);
        lock (_lock2) { /* work */ }
    }
}

public void Method2()
{
    lock (_lock2)  // ❌ Different order!
    {
        Thread.Sleep(10);
        lock (_lock1) { /* work */ }
    }
}

// ✅ FIX: consistent lock ordering
public void Method2Fixed()
{
    lock (_lock1)  // ✅ Same order as Method1
    {
        lock (_lock2) { /* work */ }
    }
}
```

---

### Example 3 — SemaphoreSlim for throttling

```csharp
private readonly SemaphoreSlim _semaphore = new(10);  // Max 10 concurrent

public async Task<string> CallExternalApi(string url)
{
    await _semaphore.WaitAsync();
    try
    {
        using var client = new HttpClient();
        return await client.GetStringAsync(url);
    }
    finally
    {
        _semaphore.Release();
    }
}

// Ensures at most 10 concurrent API calls
```

---

### Example 4 — Thread-safe lazy initialization

```csharp
// ✅ Thread-safe, lazy initialization
private readonly Lazy<ExpensiveObject> _expensive = 
    new Lazy<ExpensiveObject>(() => new ExpensiveObject());

public ExpensiveObject GetExpensive()
{
    return _expensive.Value;  // Created once, thread-safe
}
```

---

## Common Pitfalls

### ❌ 1. Locking on wrong objects

```csharp
// ❌ BAD: locking on `this` or type
lock (this) { }  // External code can lock on same object
lock (typeof(MyClass)) { }  // Global lock, huge contention

// ✅ GOOD: private lock object
private readonly object _lock = new();
lock (_lock) { }
```

---

### ❌ 2. Not releasing locks in async methods

```csharp
// ❌ DOESN'T WORK: lock across await
lock (_lock)
{
    await SomeAsync();  // Compile error
}

// ✅ Use SemaphoreSlim for async
await _semaphore.WaitAsync();
try
{
    await SomeAsync();
}
finally
{
    _semaphore.Release();
}
```

---

### ❌ 3. Long-running work in ThreadPool

```csharp
// ❌ BAD: blocks ThreadPool thread for hours
Task.Run(() => 
{
    while (true) { /* background work */ }
});

// ✅ GOOD: dedicated thread for long-running work
var thread = new Thread(() => 
{
    while (true) { /* background work */ }
});
thread.IsBackground = true;
thread.Start();
```

---

### ❌ 4. Forgetting volatile for flags

```csharp
// ❌ BAD: compiler/CPU may cache, other threads don't see update
private bool _stopRequested = false;

// ✅ GOOD: volatile ensures visibility
private volatile bool _stopRequested = false;
```

---

### ❌ 5. Using locks for everything

```csharp
// ❌ OVERKILL: locking every property access
public int Count
{
    get { lock (_lock) { return _count; } }
    set { lock (_lock) { _count = value; } }
}

// ✅ BETTER: use Interlocked for simple reads/writes
public int Count
{
    get => Interlocked.CompareExchange(ref _count, 0, 0);
    set => Interlocked.Exchange(ref _count, value);
}
```

---

## Quick Self-Check

You should be able to answer these out loud:

1. What's a race condition and how do you prevent it?
2. When would you use `lock` vs `Interlocked` vs `SemaphoreSlim`?
3. What's the difference between a Thread and a Task?
4. How do you cause a deadlock and how do you prevent it?
5. Why can't you use `lock` in an async method?
6. What's the purpose of ReaderWriterLockSlim?
7. When should you use a Mutex instead of lock?
8. What does `volatile` do?

If you struggle with any of these, revisit the relevant section.

---

## Further Practice

1. Write a thread-safe counter using lock, then optimize with Interlocked.
2. Create a producer-consumer queue using Monitor.Wait/Pulse.
3. Implement a resource pool using SemaphoreSlim (max 5 connections).
4. Write code that deadlocks, then fix it with consistent lock ordering.
5. Benchmark lock vs Interlocked for incrementing a counter 1 million times.

---

## Mini Interview Cheat Sheet

**Explain threading in 30 seconds:**
"Threading enables concurrent execution but requires synchronization to prevent race conditions. Use lock for mutual exclusion, Interlocked for atomic operations, SemaphoreSlim for throttling, and avoid shared mutable state when possible."

**Thread safety strategies:**
1. **Immutability** — no shared mutable state
2. **Synchronization** — lock, Monitor, Semaphore
3. **Atomics** — Interlocked operations
4. **Concurrent collections** — ConcurrentDictionary, etc.

**When to use what:**
- **lock** — general-purpose mutual exclusion
- **Interlocked** — simple atomic operations (counters)
- **SemaphoreSlim** — limiting concurrency, async-friendly
- **ReaderWriterLockSlim** — read-heavy scenarios
- **Mutex** — cross-process synchronization

**Avoid deadlocks:**
"Always acquire locks in the same order. Use timeouts. Minimize lock scope. Prefer lock-free designs."

---
