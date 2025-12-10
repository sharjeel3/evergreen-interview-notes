# Value Types vs Reference Types

**Level:** 🏁 Beginner  
**Tags:** `fundamentals`, `memory`, `interview`, `types`

---

## Why this matters

This is **the #1 most common C# interview question**.

Understanding value vs reference types is essential because it affects:
- How data is stored (stack vs heap)
- How data is passed to methods (copy vs reference)
- Performance characteristics
- When boxing/unboxing occurs

Interviewers use this to assess if you understand how .NET actually works under the hood.

---

## Core Ideas

### 1. **Value types live on the stack, reference types live on the heap**

**Value types:**
- `int`, `double`, `bool`, `char`, `DateTime`, `struct`, `enum`
- Stored directly where declared (usually stack for local variables)
- Contain the actual data

**Reference types:**
- `class`, `interface`, `delegate`, `object`, `string`, arrays
- Stored on the heap
- Variable contains a pointer/reference to the heap location

**Mental model:**  
"Value types ARE the data. Reference types POINT TO the data."

---

### 2. **Passing behavior differs**

```csharp
// Value type - copy is passed
void Increment(int x) 
{
    x++; // Changes local copy only
}

int a = 5;
Increment(a);
Console.WriteLine(a); // Still 5

// Reference type - reference is passed
void AddItem(List<int> list) 
{
    list.Add(42); // Modifies original object
}

var myList = new List<int>();
AddItem(myList);
Console.WriteLine(myList.Count); // 1
```

---

### 3. **Boxing and Unboxing**

**Boxing** = converting a value type to `object` (heap allocation)  
**Unboxing** = extracting value type from `object`

```csharp
int i = 123;
object o = i;        // Boxing - allocates on heap
int j = (int)o;      // Unboxing - extracts value
```

**Why it matters:**
- Boxing allocates memory on heap
- Causes GC pressure
- Performance killer in tight loops

**Modern alternative:** Use generics to avoid boxing

```csharp
// ❌ Boxing occurs
ArrayList list = new ArrayList();
list.Add(42); // int boxed to object

// ✅ No boxing
List<int> list = new List<int>();
list.Add(42); // No conversion needed
```

---

### 4. **Structs vs Classes**

| Feature | `struct` (value type) | `class` (reference type) |
|---------|---------------------|-------------------------|
| Storage | Stack | Heap |
| Default | Cannot be null | Can be null |
| Inheritance | No inheritance | Supports inheritance |
| When to use | Small, immutable data | Complex objects with behavior |

**Rule of thumb:** Use `struct` when:
- Less than 16 bytes
- Immutable
- Short-lived
- You don't need reference semantics

---

### 5. **Nullable Value Types**

```csharp
int? nullableInt = null;  // Nullable<int>
bool hasValue = nullableInt.HasValue;
int value = nullableInt.GetValueOrDefault();

// Null-coalescing
int result = nullableInt ?? 0;
```

---

## Examples

### Example 1 — Understanding the difference

```csharp
// Value type
struct Point 
{
    public int X;
    public int Y;
}

var p1 = new Point { X = 1, Y = 2 };
var p2 = p1; // Copies the entire struct
p2.X = 10;

Console.WriteLine(p1.X); // 1 (unchanged)
Console.WriteLine(p2.X); // 10

// Reference type
class Circle 
{
    public int Radius { get; set; }
}

var c1 = new Circle { Radius = 5 };
var c2 = c1; // Copies the reference, not the object
c2.Radius = 10;

Console.WriteLine(c1.Radius); // 10 (both point to same object)
Console.WriteLine(c2.Radius); // 10
```

---

### Example 2 — ref and out keywords

```csharp
// 'ref' - pass value type by reference
void Increment(ref int x) 
{
    x++; // Modifies original
}

int a = 5;
Increment(ref a);
Console.WriteLine(a); // 6

// 'out' - must assign before returning
bool TryParse(string s, out int result)
{
    result = 0; // Must assign
    return int.TryParse(s, out result);
}
```

---

### Example 3 — String is a reference type (but acts like value type)

```csharp
string s1 = "hello";
string s2 = s1;
s2 = "world";

Console.WriteLine(s1); // "hello" (unchanged)
```

Why? Strings are **immutable**. Reassignment creates a new string object.

---

## Common Pitfalls

### ❌ Pitfall 1: Forgetting structs are copied

```csharp
struct Config 
{
    public int Setting;
}

var config = new Config { Setting = 10 };
var list = new List<Config> { config };
list[0].Setting = 20; // ❌ Compile error!
```

**Why:** `list[0]` returns a copy. Use a class or access via index with assignment.

---

### ❌ Pitfall 2: Boxing in collections

```csharp
// ❌ Bad - boxing on every Add
ArrayList numbers = new ArrayList();
for (int i = 0; i < 1000; i++)
    numbers.Add(i); // Boxing!

// ✅ Good - no boxing
List<int> numbers = new List<int>();
for (int i = 0; i < 1000; i++)
    numbers.Add(i);
```

---

### ❌ Pitfall 3: Mutable structs

```csharp
// ❌ Bad - mutable struct
struct MutablePoint 
{
    public int X { get; set; }
}

// ✅ Good - immutable struct
readonly struct ImmutablePoint 
{
    public int X { get; init; }
    public int Y { get; init; }
}
```

**Why:** Mutable structs cause confusion because modifications affect copies, not originals.

---

## Quick Self-Check

1. What's stored on the stack vs the heap?
2. What happens when you pass an `int` to a method?
3. What is boxing? When does it occur?
4. Why are strings immutable?
5. When should you use `struct` vs `class`?

<details>
<summary>Answers</summary>

1. Stack: value types (local variables), method parameters, references. Heap: reference type objects
2. A copy of the int value is passed
3. Boxing converts value type to object (heap allocation). Occurs when assigning to object, non-generic collections, or interfaces
4. Immutability enables string interning, thread safety, and security
5. Use struct for small (<16 bytes), immutable data with no inheritance needed

</details>

---

## Further Practice

1. **Write a benchmark** comparing `List<int>` vs `ArrayList` for 10,000 additions
2. **Create a readonly struct** representing a 3D point with distance calculation
3. **Debug this:**
   ```csharp
   struct Counter { public int Count; }
   var list = new List<Counter> { new Counter() };
   list[0].Count++; // Why does this fail?
   ```
4. **Explain** why `string s = "test"; object o = s;` doesn't box

---

## Key Takeaways

✅ Value types copy data, reference types copy references  
✅ Avoid boxing by using generics  
✅ Use structs for small, immutable data  
✅ Strings are reference types but immutable  
✅ Use `ref`/`out` to pass value types by reference
