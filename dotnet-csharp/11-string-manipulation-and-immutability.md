# String Manipulation & Immutability

**Level:** 🏁 Beginner  
**Tags:** `string`, `StringBuilder`, `performance`, `immutability`, `memory`, `interview`

---

## Why this matters

String handling is one of the most common sources of performance bugs and memory waste in .NET applications, yet it's often overlooked because strings "just work." Understanding string immutability and when to use `StringBuilder` is a fundamental skill that separates developers who write correct code from those who write efficient code.

**Interviews test this relentlessly because it's a proxy for memory awareness.** When an interviewer asks "What's wrong with concatenating strings in a loop?", they're not just testing string knowledge—they're checking if you understand object allocation, garbage collection pressure, and performance implications. I've seen candidates fail senior-level interviews not because they couldn't write the code, but because they couldn't explain why their string concatenation approach would kill production performance.

**Production memory leaks often trace back to string misuse.** One team I worked with had an API endpoint timing out under load. The culprit? A logging method that concatenated 50+ fields using `+` operator in a loop, generating thousands of intermediate string objects per request. The GC couldn't keep up, leading to Gen 2 collections every few seconds and response times climbing from 100ms to 3+ seconds. Changing to `StringBuilder` cut memory allocations by 95% and brought response times back to normal. The bug had been in production for months because it only manifested under real-world load.

**String interning causes mysterious memory behavior.** I've debugged applications where developers thought they were saving memory by interning strings, only to discover they'd created a memory leak because interned strings live until AppDomain unload. One application was "leaking" 2GB of string data in the intern pool because developers were interning dynamically generated user input. Understanding when the runtime automatically interns strings (literals) versus when you explicitly intern them is crucial for memory-sensitive applications.

**Performance differences are dramatic and non-obvious.** Concatenating 10,000 strings with `+` operator: **O(n²)** complexity, seconds to complete. Same operation with `StringBuilder`: **O(n)** complexity, milliseconds to complete. But concatenating 3 strings with `+`? Faster than `StringBuilder` due to overhead. The performance characteristics are counter-intuitive, and interviews love to probe whether you know when to use which approach.

**Modern C# features change the rules.** String interpolation (`$"{x}"`) looks like concatenation but behaves differently. `Span<char>` and `stackalloc` enable zero-allocation string operations. `string.Create()` offers low-level control. Developers who learned C# years ago often don't know these newer, more efficient approaches exist, and it shows in their code reviews and performance profiles.

**Encoding and Unicode issues still bite production systems.** I've seen critical bugs where customer names were truncated (multi-byte characters mishandled), financial data was corrupted (wrong encoding during serialization), and security vulnerabilities were introduced (XSS attacks via unescaped strings). Understanding that .NET strings are UTF-16, how to work with different encodings, and when to use `Span<byte>` versus `string` is essential for any application handling international data or binary protocols.

**String comparison is deceptively complex.** Code that works perfectly in English fails in production when Turkish users log in because "i".ToUpper() != "I" in Turkish locale (`StringComparison.InvariantCultureIgnoreCase` vs `StringComparison.CurrentCultureIgnoreCase`). Security bugs emerge when case-insensitive comparisons use the wrong overload. One team had a critical vulnerability because their URL routing logic used culture-sensitive string comparison, allowing attackers to bypass authorization checks with specially crafted URLs.

**Memory allocations compound in high-throughput scenarios.** In a REST API handling 10,000 requests/second, even tiny string operations matter. If each request allocates 10 unnecessary string objects (5KB each), that's 50MB/second of garbage, 180GB/hour, forcing aggressive GC pauses that impact latency. I've seen P99 latencies drop from 500ms to 50ms just by eliminating unnecessary string allocations in hot paths using `Span<char>` and `ArrayPool<char>`.

**Interview questions escalate quickly.** Entry-level: "What's string immutability?" Mid-level: "When should you use StringBuilder?" Senior-level: "How would you optimize this JSON serialization code that's allocating 500MB/minute?" The string questions start simple but rapidly become proxies for understanding memory, performance profiling, and systems thinking.

**Defensive copying causes invisible performance costs.** Because strings are immutable, methods like `Substring()` in .NET Framework actually reference the original string's internal array (in some versions), causing memory retention bugs. In .NET Core+, `Substring()` copies data, avoiding retention but increasing allocations. Knowing which version of .NET changed this behavior, and why, demonstrates deep platform knowledge that senior roles require.

**Career impact is real.** I've interviewed dozens of candidates who could solve LeetCode problems but couldn't explain why their solution allocated 100MB for a string processing task that should use 100 bytes. The ability to write memory-efficient string code, explain your choices, and profile the results is what gets you promoted from mid-level to senior, from senior to staff. It's not glamorous, but it's fundamental.

---

## Core Ideas

### 1. **Strings are immutable**

Once a string is created, **it cannot be changed**.

Every operation that appears to modify a string actually creates a **new string object**.

```csharp
string s = "hello";
s = s + " world";  // Creates NEW string, old "hello" is now garbage
```

**Why immutability:**
- **Thread safety** — multiple threads can safely read the same string
- **Security** — strings can't be modified after validation
- **Performance** — strings can be interned and shared
- **Predictability** — no spooky action at a distance

**Mental model:**  
"Every string modification creates a new string; the old string becomes garbage."

---

### 2. **String concatenation with `+` is expensive in loops**

Each `+` operation creates a new string object.

```csharp
// ❌ BAD: O(n²) complexity
string result = "";
for (int i = 0; i < 10000; i++)
{
    result += i.ToString();  // Creates 10,000 intermediate strings
}
```

**Why this is slow:**
- Each iteration creates a new string
- Each new string copies all previous characters
- Total allocations: ~50 million characters copied
- GC pressure: thousands of short-lived objects

---

### 3. **Use StringBuilder for repeated concatenation**

`StringBuilder` uses a **mutable character buffer** that grows as needed.

```csharp
// ✅ GOOD: O(n) complexity
var sb = new StringBuilder();
for (int i = 0; i < 10000; i++)
{
    sb.Append(i.ToString());
}
string result = sb.ToString();
```

**When to use StringBuilder:**
- Concatenating in loops
- Building strings dynamically
- Unknown final size
- More than ~5 concatenations

**When NOT to use StringBuilder:**
- Simple concatenation of 2-3 strings (overhead not worth it)
- String interpolation already does it efficiently

---

### 4. **String interning pools identical literals**

The CLR automatically **interns** string literals to save memory.

```csharp
string a = "hello";
string b = "hello";

Console.WriteLine(ReferenceEquals(a, b));  // True — same object!
```

**How it works:**
- String literals are stored in the **intern pool** (special heap)
- Multiple references to same literal point to same object
- Intern pool lives until AppDomain unload

**Manual interning:**
```csharp
string dynamic = GetUserInput();  // Not interned
string interned = string.Intern(dynamic);  // Now interned
```

**⚠️ Warning:**  
Interning dynamic strings can cause memory leaks because they're never collected.

---

### 5. **String interpolation is efficient**

String interpolation (`$"..."`) is compiled to efficient code (often `string.Format` or `StringBuilder` internally).

```csharp
string name = "Alice";
int age = 30;

// ✅ Modern, readable, efficient
string message = $"Name: {name}, Age: {age}";

// Older equivalent
string message = string.Format("Name: {0}, Age: {1}", name, age);
```

**Compiled to:**
```csharp
// Compiler generates efficient code, often:
DefaultInterpolatedStringHandler handler = ...;
handler.AppendLiteral("Name: ");
handler.AppendFormatted(name);
handler.AppendLiteral(", Age: ");
handler.AppendFormatted(age);
string message = handler.ToStringAndClear();
```

---

### 6. **String comparison has cultural implications**

Strings can be compared in multiple ways:

```csharp
string a = "hello";
string b = "HELLO";

// Ordinal (fastest, byte-by-byte)
bool same1 = a.Equals(b, StringComparison.Ordinal);  // False

// Ordinal ignore case (fast, safe)
bool same2 = a.Equals(b, StringComparison.OrdinalIgnoreCase);  // True

// Culture-sensitive (slower, locale-aware)
bool same3 = a.Equals(b, StringComparison.CurrentCultureIgnoreCase);  // True
```

**Rules of thumb:**
- **Ordinal** — file paths, URLs, identifiers
- **OrdinalIgnoreCase** — case-insensitive identifiers
- **CurrentCulture** — user-facing text
- **InvariantCulture** — data interchange (JSON, XML)

**⚠️ Famous bug:**  
In Turkish, `"i".ToUpper() != "I"` (returns "İ"). Always use explicit comparison for security/correctness.

---

### 7. **Modern string performance tools**

**.NET provides zero-allocation string tools:**

**Span\<char\>** — stack-allocated string slice
```csharp
ReadOnlySpan<char> span = "hello world".AsSpan();
ReadOnlySpan<char> word = span.Slice(0, 5);  // "hello" — no allocation
```

**string.Create()** — low-level construction
```csharp
string result = string.Create(10, state, (span, s) =>
{
    // Write directly to string's internal buffer
    "hello".AsSpan().CopyTo(span);
});
```

**ArrayPool\<char\>** — reusable buffers
```csharp
char[] buffer = ArrayPool<char>.Shared.Rent(1024);
try
{
    // Use buffer
}
finally
{
    ArrayPool<char>.Shared.Return(buffer);
}
```

---

## Examples

### Example 1 — String immutability demonstration

```csharp
string original = "hello";
string modified = original.ToUpper();

Console.WriteLine(original);   // "hello" — unchanged!
Console.WriteLine(modified);   // "HELLO" — new string
Console.WriteLine(ReferenceEquals(original, modified));  // False
```

---

### Example 2 — StringBuilder capacity planning

```csharp
// ✅ Pre-allocate capacity if you know final size
var sb = new StringBuilder(capacity: 10000);

for (int i = 0; i < 1000; i++)
{
    sb.Append(i.ToString());
}

string result = sb.ToString();
```

**Why capacity matters:**
- Default capacity: 16 characters
- When exceeded, buffer is reallocated (expensive)
- Pre-allocating avoids reallocations

---

### Example 3 — String interning

```csharp
string a = "hello";
string b = "hel" + "lo";  // Compile-time constant folding
string c = string.Concat("hel", "lo");  // Runtime concatenation

Console.WriteLine(ReferenceEquals(a, b));  // True — both interned
Console.WriteLine(ReferenceEquals(a, c));  // False — c not interned

// Manual interning
string d = string.Intern(c);
Console.WriteLine(ReferenceEquals(a, d));  // True — now same object
```

---

### Example 4 — String comparison pitfalls

```csharp
string url1 = "http://example.com/Path";
string url2 = "http://example.com/PATH";

// ❌ BAD: Culture-sensitive comparison
bool same1 = url1.Equals(url2, StringComparison.CurrentCultureIgnoreCase);

// ✅ GOOD: Ordinal comparison for URLs
bool same2 = url1.Equals(url2, StringComparison.OrdinalIgnoreCase);

// In most cultures: same1 == same2
// In Turkish: same1 might differ!
```

---

### Example 5 — Zero-allocation string parsing with Span

```csharp
string data = "Alice,30,Engineer";
ReadOnlySpan<char> span = data.AsSpan();

int firstComma = span.IndexOf(',');
ReadOnlySpan<char> name = span.Slice(0, firstComma);  // "Alice" — no allocation

int secondComma = span.Slice(firstComma + 1).IndexOf(',') + firstComma + 1;
ReadOnlySpan<char> ageSpan = span.Slice(firstComma + 1, secondComma - firstComma - 1);

int age = int.Parse(ageSpan);  // Parses directly from span
```

---

## Common Pitfalls

### ❌ 1. Concatenating in loops with `+`

**Never do this:**
```csharp
string result = "";
foreach (var item in items)
{
    result += item.ToString();  // O(n²) disaster
}
```

**Do this instead:**
```csharp
var sb = new StringBuilder();
foreach (var item in items)
{
    sb.Append(item.ToString());
}
string result = sb.ToString();
```

---

### ❌ 2. Using `StringBuilder` for 2-3 concatenations

```csharp
// ❌ Overkill — StringBuilder has overhead
var sb = new StringBuilder();
sb.Append("Hello ");
sb.Append(name);
string result = sb.ToString();

// ✅ Simpler and faster
string result = "Hello " + name;

// ✅ Or use interpolation
string result = $"Hello {name}";
```

---

### ❌ 3. Interning dynamic user input

```csharp
// ❌ MEMORY LEAK — interned strings never collected
string userInput = GetUserInput();
string interned = string.Intern(userInput);  // Lives forever!
```

**Only intern:**
- Known finite set of strings
- Frequently compared strings
- Not user-generated content

---

### ❌ 4. Culture-sensitive comparison for identifiers

```csharp
// ❌ BAD: Locale-dependent
if (filename.ToLower() == "readme.txt")  

// ✅ GOOD: Ordinal comparison
if (filename.Equals("readme.txt", StringComparison.OrdinalIgnoreCase))
```

---

### ❌ 5. Forgetting UTF-16 encoding

.NET strings are **UTF-16** internally.

```csharp
string emoji = "😀";  // 1 character...
Console.WriteLine(emoji.Length);  // ...but Length = 2! (surrogate pair)

// ✅ Use StringInfo for correct character count
var info = new StringInfo(emoji);
Console.WriteLine(info.LengthInTextElements);  // 1
```

---

## Quick Self-Check

You should be able to answer these out loud:

1. Why are strings immutable?
2. What's the time complexity of concatenating strings with `+` in a loop?
3. When should you use `StringBuilder` vs `+` operator?
4. What is string interning and when is it automatic?
5. What's the risk of manually interning dynamic strings?
6. What's the difference between `Ordinal` and `CurrentCulture` string comparison?
7. How does `Span<char>` help performance?
8. Why is `string.Length` sometimes wrong for emoji?

If you struggle with any of these, revisit the relevant section.

---

## Further Practice

1. Write a benchmark comparing `+` vs `StringBuilder` for 1000 concatenations.
2. Profile memory allocations using a tool like dotMemory or PerfView.
3. Implement a CSV parser using `Span<char>` with zero allocations.
4. Write a method that uses `string.Create()` to build a formatted string.
5. Explain string immutability to someone else using your own words.

---

## Mini Interview Cheat Sheet

**Explain string immutability in 20 seconds:**
"Strings are immutable—every modification creates a new string. This ensures thread safety but makes concatenation in loops expensive because it's O(n²)."

**When to use StringBuilder:**
"Use StringBuilder when concatenating in loops or building strings dynamically. For 2-3 concatenations, use `+` or interpolation—StringBuilder has overhead."

**String comparison rule:**
"Use `StringComparison.Ordinal` for identifiers, paths, URLs. Use `CurrentCulture` for user-facing text. Never use `ToLower()` for comparison—use `Equals()` with explicit comparison."

**Key terms:**
- **Immutable** — strings can't be changed after creation
- **Intern pool** — special heap for string literals
- **StringBuilder** — mutable buffer for efficient concatenation
- **Span\<char\>** — zero-allocation string slice

---
