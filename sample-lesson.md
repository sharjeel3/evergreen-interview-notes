This will act as the *gold standard template* for all future lessons (Copilot will copy the structure and tone automatically).



# Async/Await & Task Parallel Library (TPL)

**Level:** 🚦 Intermediate  
**Tags:** `async`, `threading`, `TPL`, `performance`, `interview`

---

## Why this matters

Asynchrony is one of the most common interview topics in .NET because it touches:

- performance  
- scalability  
- concurrency  
- the runtime’s scheduling model  

A strong engineer must be able to **explain what async *actually* does**, not just use it.

---

## Core Ideas

### 1. **Async improves scalability, not speed**

Asynchronous code frees the calling thread while waiting for I/O.  
It does *not* magically make code run faster; it simply stops threads from blocking.

**Mental model:**  
“Async is about *not waiting* while you’re waiting.”

---

### 2. **A `Task` represents a promise of future completion**

`Task` = “an operation that might not be finished yet.”  
It may complete with:
- a result (`Task<T>`)
- no result (`Task`)
- an exception  
- a cancellation token request  

---

### 3. **The SynchronizationContext decides where continuation runs**

This is the root cause of many async bugs.

| Environment | Sync Context Behaviour |
|------------|-------------------------|
| ASP.NET Core | No sync context → continuations run on thread pool |
| WPF / WinForms | Continuations return to main UI thread |
| Console App | No sync context unless you add one |

This is why `ConfigureAwait(false)` matters in library code.

---

### 4. **Deadlocks occur when async waits synchronously**

Classic trap:

```csharp
// ❌ Will deadlock on UI thread or ASP.NET (full framework)
var result = SomeAsync().Result;
```

Because:

* `Result` blocks the calling thread
* but the async method tries to resume *on that same thread*
* → deadlock

---

### 5. **CPU-bound vs I/O-bound**

* **CPU-bound** → use `Task.Run`
  (calculations, encryption, image processing)

* **I/O-bound** → use `async`
  (file reads, HTTP calls, DB access)

If you get this wrong, performance collapses.

---

## Examples

### Example 1 — I/O-bound async method

```csharp
public async Task<string> GetDataAsync()
{
    using var client = new HttpClient();
    var response = await client.GetAsync("https://api.example.com");
    return await response.Content.ReadAsStringAsync();
}
```

### Example 2 — CPU-bound work on thread pool

```csharp
public async Task<int> CalculateAsync(int n)
{
    return await Task.Run(() =>
    {
        // CPU-heavy loop
        var sum = 0;
        for (int i = 0; i < n; i++) sum += i;
        return sum;
    });
}
```

### Example 3 — Using ConfigureAwait(false)

```csharp
public async Task<string> FetchAsync()
{
    var data = await File.ReadAllTextAsync("file.txt")
                         .ConfigureAwait(false);

    return data.ToUpperInvariant();
}
```

Use this in `.dll` libraries — not required in ASP.NET Core apps.

---

## Common Pitfalls

### ❌ 1. Blocking async calls

`Task.Result`, `.Wait()`, `.GetAwaiter().GetResult()` → avoid in almost all cases.

### ❌ 2. Using `async void`

Only valid for:

* event handlers
* fire-and-forget logic (rare)

### ❌ 3. Assuming async = multi-threaded

Async can use **zero** extra threads for I/O-bound operations.

### ❌ 4. Forgetting cancellation tokens

Interviewers love asking about cancellation patterns.

### ❌ 5. Overusing `Task.Run()`

`Task.Run()` is *not* a “make async” button.
Use it only for true CPU-bound work.

---

## Quick Self-Check

You should be able to answer these out loud:

1. What does async actually do under the hood?
2. Why does `.Result` cause deadlocks in UI apps?
3. Difference between CPU-bound and I/O-bound work?
4. Why is `ConfigureAwait(false)` used in libraries?
5. Does async make code faster?
6. How does ASP.NET Core handle SynchronizationContext?

If you struggle with any of these, revisit the relevant section.

---

## Further Practice

1. Convert a synchronous file-reading method to async.
2. Create an API controller that performs two HTTP calls in parallel using `Task.WhenAll`.
3. Benchmark a CPU method with and without `Task.Run`.
4. Write an async method that supports cancellation via `CancellationToken`.
5. Explain async back to someone else using your own words.

---

## Mini Interview Cheat Sheet

**Explain async in 20 seconds:**
“Async lets a thread do other work while waiting for I/O. It improves scalability by freeing up threads, not by running things faster.”

**How to avoid deadlocks:**
“Never block on async; end-to-end async all the way.”

**Rule of thumb:**

* I/O → async
* CPU → Task.Run
* Libraries → ConfigureAwait(false)

