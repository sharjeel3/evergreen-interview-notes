# Async/Await & Task Parallel Library (TPL)

**Level:** 🚦 Intermediate  
**Tags:** `async`, `threading`, `TPL`, `performance`, `interview`

---

## Why this matters

Asynchrony is **one of the most common interview topics** in .NET because it touches:
- Performance and scalability
- Concurrency and thread management  
- The runtime's scheduling model
- Common deadlock scenarios

A strong engineer must explain **what async actually does**, not just use the keywords.

---

## Core Ideas

### 1. **Async improves scalability, not speed**

Asynchronous code frees the calling thread while waiting for I/O.  
It does *not* magically make code run faster; it stops threads from blocking.

**Mental model:**  
"Async is about *not waiting* while you're waiting."

**Example:** Web server handling 1000 concurrent requests
- **Synchronous:** 1000 blocked threads waiting for I/O → thread pool exhaustion
- **Asynchronous:** A few threads handle 1000 requests → scalable

---

### 2. **A Task represents a promise of future completion**

`Task` = "an operation that might not be finished yet."

It may complete with:
- A result (`Task<T>`)
- No result (`Task`)
- An exception
- Cancellation

```csharp
Task<string> task = FetchDataAsync(); // Returns immediately
// Do other work...
string result = await task; // Waits for completion
```

---

### 3. **The SynchronizationContext decides where continuations run**

This is the root cause of many async bugs.

| Environment | Sync Context Behavior |
|------------|------------------------|
| ASP.NET Core | No sync context → continuations run on thread pool |
| WPF / WinForms | Continuations return to main UI thread |
| Console App | No sync context unless you add one |

**Why `ConfigureAwait(false)` matters:**

```csharp
// In library code
public async Task<string> GetDataAsync()
{
    var data = await httpClient.GetStringAsync(url)
        .ConfigureAwait(false); // Don't capture context
    
    return data.ToUpper(); // Runs on thread pool
}
```

**Rule:** Use `ConfigureAwait(false)` in library code to avoid context capture overhead.

---

### 4. **Deadlocks occur when async waits synchronously**

Classic trap:

```csharp
// ❌ Will deadlock on UI thread or ASP.NET (full framework)
var result = SomeAsync().Result;
```

**Why:**
1. `Result` blocks the calling thread
2. The async method tries to resume *on that same thread*
3. → Deadlock

**Solution:** Never block on async—go async all the way.

```csharp
// ✅ Correct
var result = await SomeAsync();
```

---

### 5. **CPU-bound vs I/O-bound**

**I/O-bound** → use `async`  
(file reads, HTTP calls, database queries)

```csharp
public async Task<string> ReadFileAsync()
{
    return await File.ReadAllTextAsync("data.txt");
}
```

**CPU-bound** → use `Task.Run`  
(calculations, encryption, image processing)

```csharp
public async Task<int> CalculateAsync(int n)
{
    return await Task.Run(() =>
    {
        int sum = 0;
        for (int i = 0; i < n; i++) sum += i;
        return sum;
    });
}
```

**If you get this wrong, performance collapses.**

---

## Examples

### Example 1 — I/O-bound async method

```csharp
public async Task<string> GetDataAsync()
{
    using var client = new HttpClient();
    var response = await client.GetAsync("https://api.example.com/data");
    return await response.Content.ReadAsStringAsync();
}

// Usage
var data = await GetDataAsync();
Console.WriteLine(data);
```

---

### Example 2 — CPU-bound work on thread pool

```csharp
public async Task<int> ComputeHashAsync(string input)
{
    return await Task.Run(() =>
    {
        using var sha256 = SHA256.Create();
        byte[] bytes = Encoding.UTF8.GetBytes(input);
        byte[] hash = sha256.ComputeHash(bytes);
        return BitConverter.ToInt32(hash, 0);
    });
}
```

---

### Example 3 — Parallel async operations

```csharp
public async Task<(string, string, string)> FetchAllDataAsync()
{
    // Start all requests concurrently
    Task<string> task1 = httpClient.GetStringAsync("https://api1.com");
    Task<string> task2 = httpClient.GetStringAsync("https://api2.com");
    Task<string> task3 = httpClient.GetStringAsync("https://api3.com");
    
    // Wait for all to complete
    await Task.WhenAll(task1, task2, task3);
    
    // All results now available
    return (task1.Result, task2.Result, task3.Result);
}

// Or with array
public async Task<string[]> FetchMultipleAsync(string[] urls)
{
    var tasks = urls.Select(url => httpClient.GetStringAsync(url));
    return await Task.WhenAll(tasks);
}
```

---

### Example 4 — Cancellation support

```csharp
public async Task<string> ProcessDataAsync(CancellationToken cancellationToken)
{
    // Pass token to async operations
    var data = await httpClient.GetStringAsync(url, cancellationToken);
    
    // Check for cancellation in CPU-bound sections
    for (int i = 0; i < 1000000; i++)
    {
        cancellationToken.ThrowIfCancellationRequested();
        // Process...
    }
    
    return data;
}

// Usage
using var cts = new CancellationTokenSource();
cts.CancelAfter(TimeSpan.FromSeconds(5)); // Auto-cancel after 5s

try
{
    var result = await ProcessDataAsync(cts.Token);
}
catch (OperationCanceledException)
{
    Console.WriteLine("Operation was cancelled");
}
```

---

### Example 5 — ConfigureAwait in library code

```csharp
// Library method - use ConfigureAwait(false)
public async Task<User> GetUserAsync(int id)
{
    // Don't need to return to original context
    var json = await httpClient.GetStringAsync($"/users/{id}")
        .ConfigureAwait(false);
    
    var user = JsonSerializer.Deserialize<User>(json);
    return user;
}

// Application code - usually omit ConfigureAwait
public async Task OnButtonClickAsync()
{
    var user = await GetUserAsync(123);
    // Need to be on UI thread to update controls
    userNameLabel.Text = user.Name;
}
```

---

## Common Pitfalls

### ❌ Pitfall 1: Blocking async calls

```csharp
// ❌ Deadlock risk
var result = SomeAsync().Result;
var result = SomeAsync().GetAwaiter().GetResult();
SomeAsync().Wait();

// ✅ Async all the way
var result = await SomeAsync();
```

---

### ❌ Pitfall 2: Using async void

```csharp
// ❌ Bad - exceptions can't be caught
public async void ProcessDataAsync()
{
    await Task.Delay(100);
    throw new Exception("Boom"); // Crashes app!
}

// ✅ Good - return Task
public async Task ProcessDataAsync()
{
    await Task.Delay(100);
    throw new Exception("Boom"); // Can be caught
}

// ⚠️ Only valid use: event handlers
private async void Button_Click(object sender, EventArgs e)
{
    await ProcessDataAsync();
}
```

---

### ❌ Pitfall 3: Assuming async = multi-threaded

```csharp
// Async does NOT create threads for I/O
public async Task ReadFileAsync()
{
    // Zero extra threads used
    var data = await File.ReadAllTextAsync("data.txt");
}
```

**Async uses the OS's asynchronous I/O capabilities, not thread pool threads.**

---

### ❌ Pitfall 4: Forgetting await

```csharp
// ❌ Returns Task, doesn't wait
public Task DoWorkAsync()
{
    ProcessDataAsync(); // Fire and forget - probably wrong!
    return Task.CompletedTask;
}

// ✅ Await the task
public async Task DoWorkAsync()
{
    await ProcessDataAsync();
}
```

---

### ❌ Pitfall 5: Overusing Task.Run

```csharp
// ❌ Don't wrap I/O in Task.Run
public Task<string> ReadFileAsync()
{
    return Task.Run(() => File.ReadAllText("data.txt")); // Wrong!
}

// ✅ Use async I/O directly
public async Task<string> ReadFileAsync()
{
    return await File.ReadAllTextAsync("data.txt");
}
```

**Rule:** `Task.Run` is for CPU-bound work, not I/O.

---

## Quick Self-Check

1. What does async actually do under the hood?
2. Why does `.Result` cause deadlocks in UI apps?
3. Difference between CPU-bound and I/O-bound work?
4. Why use `ConfigureAwait(false)` in libraries?
5. Does async make code faster?
6. When is `async void` acceptable?

<details>
<summary>Answers</summary>

1. Async allows method to return before completion, using state machine to resume later
2. `.Result` blocks calling thread while async tries to resume on same thread → deadlock
3. CPU-bound = calculations; I/O-bound = waiting for external resources (disk, network)
4. Avoids capturing SynchronizationContext, improving performance and avoiding deadlocks
5. No—async improves scalability by freeing threads, doesn't make operations faster
6. Only for event handlers; otherwise exceptions can't be caught

</details>

---

## Further Practice

1. **Convert** a synchronous HTTP call to async
2. **Benchmark** synchronous vs async file I/O with 100 concurrent operations
3. **Implement** exponential backoff retry with cancellation support
4. **Create** a method that fetches 10 URLs in parallel with timeout
5. **Debug** a deadlock caused by `.Result` in a WinForms app

---

## Key Takeaways

✅ Async improves scalability, not speed  
✅ Never block on async—await all the way down  
✅ Use `Task.Run` for CPU-bound work, not I/O  
✅ `ConfigureAwait(false)` in library code  
✅ `async void` only for event handlers  
✅ Async doesn't create threads for I/O  
✅ Always support cancellation with `CancellationToken`  
✅ Use `Task.WhenAll` for parallel operations
