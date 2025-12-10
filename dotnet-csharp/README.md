# .NET / C# Learning Path

A structured path through essential .NET and C# concepts, from fundamentals to advanced topics, optimized for interview preparation and quick refreshers.

---

## 🎯 Learning Goals

By working through these lessons, you'll be able to:
- Explain core .NET and C# concepts clearly in interviews
- Debug and optimize .NET applications
- Write idiomatic, performant C# code
- Understand the runtime and memory model

---

## 📚 Lesson Index

### 🏁 Fundamentals (Start Here)

Essential concepts every C# developer must know cold.

| # | Topic | Why It Matters |
|---|-------|----------------|
| [01](01-value-vs-reference-types.md) | **Value vs Reference Types** | Stack/heap, boxing, struct vs class |
| [02](02-collections-and-generics.md) | **Collections & Generics** | List, Dictionary, IEnumerable, type safety |
| [03](03-exception-handling.md) | **Exception Handling** | try-catch-finally, custom exceptions, best practices |
| [10](10-dotnet-fundamentals-and-runtime.md) | **.NET Fundamentals & Runtime** | CLR, JIT, IL, assemblies, .NET vs Framework |
| [11](11-string-manipulation-and-immutability.md) | **String Manipulation** | Immutability, StringBuilder, interning, performance |
| [12](12-oop-in-csharp.md) | **Object-Oriented Programming** | Encapsulation, inheritance, polymorphism, interfaces |

---

### 🚦 Intermediate (Core Skills)

Practical skills used daily in production code.

| # | Topic | Why It Matters |
|---|-------|----------------|
| [04](04-linq-and-delegates.md) | **LINQ & Delegates** | Functional programming, query syntax, lambda expressions |
| [05](05-async-await-and-tpl.md) | **Async/Await & TPL** | Asynchronous programming, Task, deadlocks, ConfigureAwait |
| [06](06-dependency-injection.md) | **Dependency Injection** | IoC, service lifetimes, testability |
| [13](13-threading-and-synchronization.md) | **Threading & Synchronization** | Threads, locks, race conditions, thread safety |
| [14](14-parallel-programming.md) | **Parallel Programming** | Parallel.For, PLINQ, partitioning, CPU-bound work |
| [15](15-events-delegates-and-patterns.md) | **Events & Delegates** | Event pattern, multicast delegates, memory leaks, observer |
| [16](16-testing-and-testability.md) | **Testing & Testability** | Unit tests, mocking, TDD, integration tests, xUnit |
| [17](17-configuration-and-options-pattern.md) | **Configuration & Options** | appsettings, IOptions, User Secrets, Key Vault |

---

### 🔥 Advanced (Deep Understanding)

Topics that separate senior engineers from juniors in interviews.

| # | Topic | Why It Matters |
|---|-------|----------------|
| [07](07-memory-management-and-gc.md) | **Memory & Garbage Collection** | Heap generations, GC tuning, memory leaks |
| [08](08-reflection-and-attributes.md) | **Reflection & Attributes** | Runtime type inspection, custom attributes, performance |
| [09](09-performance-optimization.md) | **Performance Optimization** | Span\<T\>, benchmarking, profiling, common pitfalls |
| [18](18-security-and-secure-coding.md) | **Security & Secure Coding** | SQL injection, XSS, authentication, encryption, OWASP |

---

### ✅ Interview Preparation

| # | Topic | Description |
|---|-------|-------------|
| [99](99-interview-checklist.md) | **Interview Checklist** | Common questions, quick answers, what to review |

---

## 🗺 Suggested Learning Paths

### Path 1: Complete Beginner
Follow lessons in order: 01 → 02 → 03 → 10 → 11 → 12 → 04 → 05 → 06 → 13 → 14 → 15 → 16 → 17

### Path 2: Interview Prep (1-2 weeks out)
1. **Fundamentals refresh** (10-12) — .NET runtime, strings, OOP
2. **Scan basics** (01-03) for any gaps
3. **Deep dive intermediate** (04-06, 13-17) — LINQ, async, DI, threading, parallel, events, testing, config
4. **Read advanced topics** (07-09, 18) for talking points
5. **Drill the interview checklist** (99)

### Path 3: Quick Refresh (night before)
1. Read "Why this matters" + "Core Ideas" in each lesson (focus on 04-09, 13-18)
2. Review "Common Pitfalls" sections
3. Go through interview checklist (99)

---

## 💡 How to Use These Lessons

Each lesson includes:
- **Why it matters** — interview relevance and real-world context
- **Core ideas** — mental models and key concepts
- **Code examples** — runnable, practical code
- **Common pitfalls** — mistakes to avoid
- **Quick self-check** — test your understanding
- **Further practice** — hands-on exercises

---

## 🔗 External Resources

- [Official C# Docs](https://learn.microsoft.com/en-us/dotnet/csharp/)
- [.NET Runtime Source](https://github.com/dotnet/runtime)
- [C# Language Design](https://github.com/dotnet/csharplang)

---

## 📝 Contributing

See the main [README](../README.MD) for lesson formatting guidelines and contribution standards.
