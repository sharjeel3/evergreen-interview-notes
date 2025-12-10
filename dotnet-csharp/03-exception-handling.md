# Exception Handling

**Level:** 🏁 Beginner  
**Tags:** `fundamentals`, `exceptions`, `error-handling`, `interview`, `best-practices`

---

## Why this matters

Exception handling is **critical for production code** and a common interview topic.

Interviewers assess:
- Do you understand when to use exceptions vs return codes?
- Do you know the performance cost of exceptions?
- Can you write defensive code without over-catching?
- Do you understand finally blocks and IDisposable?

Poor exception handling crashes apps, leaks resources, and hides bugs.

---

## Core Ideas

### 1. **Exceptions are for exceptional conditions**

**Use exceptions for:**
- Unexpected errors (file not found, network timeout)
- Programmer mistakes (null reference, invalid argument)
- Unrecoverable errors

**Don't use exceptions for:**
- Control flow
- Expected conditions (validation failures)
- Performance-critical paths

**Mental model:**  
"Exceptions are expensive. Use them when you can't continue normally."

---

### 2. **The try-catch-finally pattern**

```csharp
try
{
    // Code that might throw
}
catch (SpecificException ex)
{
    // Handle specific exception
}
catch (Exception ex)
{
    // Handle all other exceptions
    throw; // Re-throw to preserve stack trace
}
finally
{
    // Always executes (cleanup)
}
```

**Key rules:**
- Catch specific exceptions first (most derived to least derived)
- Use `throw;` not `throw ex;` to preserve stack trace
- finally block runs even if exception occurs

---

### 3. **Built-in Exception Hierarchy**

```
Exception (base)
├── SystemException
│   ├── ArgumentException
│   │   ├── ArgumentNullException
│   │   └── ArgumentOutOfRangeException
│   ├── InvalidOperationException
│   ├── NullReferenceException
│   └── NotImplementedException
├── ApplicationException (obsolete, don't use)
└── Custom exceptions
```

**Common exceptions you'll catch:**
- `ArgumentNullException` — null argument
- `ArgumentOutOfRangeException` — invalid range
- `InvalidOperationException` — object in wrong state
- `FileNotFoundException` — file doesn't exist
- `HttpRequestException` — HTTP request failed

---

### 4. **Custom Exceptions**

```csharp
// ✅ Good custom exception
public class InsufficientFundsException : Exception
{
    public decimal Balance { get; }
    public decimal Requested { get; }
    
    public InsufficientFundsException(decimal balance, decimal requested)
        : base($"Insufficient funds. Balance: {balance}, Requested: {requested}")
    {
        Balance = balance;
        Requested = requested;
    }
    
    // Required for serialization
    protected InsufficientFundsException(
        SerializationInfo info, 
        StreamingContext context) : base(info, context)
    {
    }
}
```

**When to create custom exceptions:**
- Domain-specific errors
- Need to pass additional data
- Caller needs to catch specifically

---

### 5. **Exception Filters (C# 6+)**

```csharp
try
{
    await MakeHttpRequestAsync();
}
catch (HttpRequestException ex) when (ex.StatusCode == HttpStatusCode.NotFound)
{
    // Handle 404 specifically
}
catch (HttpRequestException ex) when (ex.StatusCode == HttpStatusCode.Unauthorized)
{
    // Handle 401 specifically
}
```

**Benefit:** Stack doesn't unwind unless filter returns true (better debugging).

---

## Examples

### Example 1 — Proper exception handling

```csharp
public class AccountService
{
    public void Withdraw(Account account, decimal amount)
    {
        // ✅ Guard clauses with ArgumentException
        if (account == null)
            throw new ArgumentNullException(nameof(account));
        
        if (amount <= 0)
            throw new ArgumentOutOfRangeException(nameof(amount), 
                "Amount must be positive");
        
        // ✅ Business rule exception
        if (account.Balance < amount)
            throw new InsufficientFundsException(account.Balance, amount);
        
        account.Balance -= amount;
    }
}

// Usage
try
{
    service.Withdraw(account, 100);
}
catch (ArgumentNullException ex)
{
    // Log and show user error
    _logger.LogError(ex, "Account was null");
}
catch (InsufficientFundsException ex)
{
    // User-friendly message
    Console.WriteLine($"Cannot withdraw {ex.Requested}. Balance: {ex.Balance}");
}
```

---

### Example 2 — try-finally and using statements

```csharp
// ❌ Manual cleanup is error-prone
FileStream fs = null;
try
{
    fs = new FileStream("file.txt", FileMode.Open);
    // Use file
}
finally
{
    if (fs != null)
        fs.Dispose(); // Might forget this!
}

// ✅ using statement guarantees disposal
using (var fs = new FileStream("file.txt", FileMode.Open))
{
    // Use file
} // Dispose() called automatically

// ✅ Even better: using declaration (C# 8+)
using var fs = new FileStream("file.txt", FileMode.Open);
// Use file
// Dispose() called at end of scope
```

---

### Example 3 — Re-throwing exceptions correctly

```csharp
// ❌ WRONG - loses original stack trace
catch (Exception ex)
{
    LogError(ex);
    throw ex; // DON'T DO THIS!
}

// ✅ Correct - preserves stack trace
catch (Exception ex)
{
    LogError(ex);
    throw; // Just 'throw' with no variable
}

// ✅ Wrap with more context
catch (Exception ex)
{
    throw new InvalidOperationException("Failed to process order", ex);
}
```

---

### Example 4 — Exception filters for logging

```csharp
try
{
    ProcessData();
}
catch (Exception ex) when (LogException(ex))
{
    // Never executes because LogException returns false
}

bool LogException(Exception ex)
{
    _logger.LogError(ex, "Error processing data");
    return false; // Continue unwinding
}
```

This logs exceptions without catching them (stack unwinds normally).

---

### Example 5 — TryParse pattern (avoiding exceptions)

```csharp
// ❌ Slow: exceptions for control flow
try
{
    int value = int.Parse(input);
}
catch (FormatException)
{
    // Handle invalid input
}

// ✅ Fast: TryParse pattern
if (int.TryParse(input, out int value))
{
    // Use value
}
else
{
    // Handle invalid input
}
```

**Rule:** If failure is expected, use Try* pattern instead of exceptions.

---

## Common Pitfalls

### ❌ Pitfall 1: Catching and swallowing exceptions

```csharp
// ❌ NEVER do this
try
{
    ProcessImportantData();
}
catch (Exception ex)
{
    // Silently swallowed - bug is hidden!
}

// ✅ At minimum, log it
catch (Exception ex)
{
    _logger.LogError(ex, "Error processing data");
    throw; // Or handle appropriately
}
```

---

### ❌ Pitfall 2: Catching too broadly

```csharp
// ❌ Catches everything, including OutOfMemoryException!
catch (Exception ex)
{
    Retry(); // Can't recover from OOM
}

// ✅ Catch only what you can handle
catch (HttpRequestException ex)
{
    await RetryAsync();
}
```

---

### ❌ Pitfall 3: Using exceptions for validation

```csharp
// ❌ Slow and unclear
public void ProcessAge(int age)
{
    if (age < 0)
        throw new Exception("Invalid age");
}

// ✅ Clear and fast
public void ProcessAge(int age)
{
    if (age < 0)
        throw new ArgumentOutOfRangeException(nameof(age), 
            "Age must be non-negative");
}

// ✅ Or return result object for validation
public Result<User> ValidateUser(User user)
{
    if (user.Age < 0)
        return Result<User>.Failure("Age must be non-negative");
    
    return Result<User>.Success(user);
}
```

---

### ❌ Pitfall 4: Not disposing resources

```csharp
// ❌ Resource leak if exception occurs
var connection = new SqlConnection(connString);
connection.Open();
// If exception here, connection never disposed
connection.Dispose();

// ✅ Always use 'using'
using var connection = new SqlConnection(connString);
connection.Open();
// Connection disposed even if exception
```

---

### ❌ Pitfall 5: Exceptions are expensive

```csharp
// ❌ Terrible performance
for (int i = 0; i < 10000; i++)
{
    try
    {
        int.Parse(items[i]);
    }
    catch
    {
        // Exception thrown 10000 times if all invalid
    }
}

// ✅ Much faster
for (int i = 0; i < 10000; i++)
{
    if (int.TryParse(items[i], out int value))
    {
        // Use value
    }
}
```

**Cost:** Throwing exception can be 1000x slower than normal path.

---

## Quick Self-Check

1. When should you use exceptions vs return values?
2. What's the difference between `throw;` and `throw ex;`?
3. What does a finally block guarantee?
4. When should you create a custom exception?
5. Why is `catch (Exception)` dangerous?

<details>
<summary>Answers</summary>

1. Use exceptions for unexpected errors; return values for expected failures (validation)
2. `throw;` preserves stack trace; `throw ex;` replaces it
3. Finally block always executes, even if exception thrown or caught
4. For domain-specific errors, when you need additional context, or when callers need to catch specifically
5. Catches everything including system exceptions (OutOfMemoryException, StackOverflowException) you can't recover from

</details>

---

## Further Practice

1. **Write a custom exception** for a domain error (e.g., `OrderAlreadyShippedException`)
2. **Refactor** code that uses exceptions for control flow to use Result<T> pattern
3. **Benchmark** `int.Parse` vs `int.TryParse` in a loop
4. **Debug** a stack trace and identify the original throw location
5. **Implement** a retry policy with exponential backoff for transient exceptions

---

## Key Takeaways

✅ Exceptions are for exceptional conditions, not control flow  
✅ Always use `using` for IDisposable resources  
✅ Use `throw;` to preserve stack trace  
✅ Catch specific exceptions, not `Exception`  
✅ Prefer Try* patterns over exceptions for expected failures  
✅ Custom exceptions for domain-specific errors  
✅ Log exceptions before swallowing them (if you must)
