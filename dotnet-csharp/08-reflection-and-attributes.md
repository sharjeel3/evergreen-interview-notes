# Reflection & Attributes

**Level:** 🔥 Advanced  
**Tags:** `reflection`, `attributes`, `metadata`, `interview`, `runtime`

---

## Why this matters

Reflection is **the foundation of nearly every advanced .NET framework and tool** you use daily. Understanding it reveals how the .NET ecosystem actually works under the hood and enables you to build sophisticated, dynamic systems.

**Framework internals:** Major .NET frameworks and libraries are built on reflection:
- **ASP.NET Core:** Uses reflection to discover controllers, bind request data to parameters, and invoke action methods
- **Entity Framework:** Uses reflection to map database columns to object properties, generate queries, and materialize entities
- **Dependency Injection:** All DI containers use reflection to discover constructors, resolve dependencies, and create instances
- **JSON Serializers:** (Newtonsoft.Json, System.Text.Json) Use reflection to discover properties and serialize/deserialize objects
- **Unit Test Frameworks:** (xUnit, NUnit, MSTest) Use reflection to discover test methods and invoke them
- **AutoMapper:** Uses reflection to map properties between objects
- **Validation Frameworks:** Use reflection to find validation attributes and apply rules

Without understanding reflection, you can't understand how these frameworks work or debug when they don't work as expected.

**When you absolutely need reflection:** Certain problems can only be solved with reflection:
- **Plugin architectures:** Loading assemblies at runtime and discovering types that implement interfaces
- **Configuration frameworks:** Binding configuration to strongly-typed objects without knowing types at compile time
- **ORM implementations:** Mapping database results to arbitrary types
- **Generic serialization:** Converting arbitrary objects to/from JSON, XML, binary formats
- **Validation frameworks:** Applying validation rules defined via attributes
- **Code generation tools:** Analyzing code and generating boilerplate
- **Debugging tools:** Inspecting object state at runtime

**The performance trap:** Reflection is 50-100x slower than direct code:
- Getting a `Type` object: ~20ns
- Calling `GetMethod`: ~200ns
- Invoking method via reflection: ~100ns (vs 2ns direct)
- Creating instance via `Activator.CreateInstance`: ~500ns (vs 5ns with `new`)

In tight loops (millions of iterations), this overhead is catastrophic. I've seen applications that were unusably slow because developers used reflection in hot paths. After refactoring to use compiled expressions or direct calls, performance improved 50x.

**When reflection goes wrong:** Common production issues:
- **Performance disasters:** Reflection in request processing causes API timeout under load
- **Security vulnerabilities:** Improper use can allow arbitrary code execution
- **Missing type errors:** Types not found at runtime cause production failures
- **Memory leaks:** Caching reflection objects improperly causes memory buildup
- **Breaking changes:** Reflection-based code breaks when internal implementations change

**The optimization strategy:** Professional developers use reflection wisely:
1. **Cache reflection objects:** Cache `Type`, `MethodInfo`, `PropertyInfo` (not the invocation)
2. **Use compiled expressions:** Compile reflection to delegates for repeated calls (10x faster)
3. **Consider source generators:** (C# 9+) Generate code at compile time instead
4. **Limit scope:** Only use reflection at initialization, not in hot paths

**Attributes enable declarative programming:** Custom attributes let you:
- Add metadata without changing code structure
- Implement cross-cutting concerns (validation, authorization, caching)
- Configure behavior declaratively
- Enable framework extensibility

This pattern is everywhere in .NET: `[HttpGet]`, `[Required]`, `[Authorize]`, `[DataMember]`, `[TestMethod]`. Understanding how to create and read custom attributes lets you build similarly elegant APIs.

**Real-world impact stories:**
- **Dependency injection containers:** All use reflection to discover constructors and create instances. Understanding this helps you debug "No parameterless constructor" errors.
- **EF Core migrations:** Use reflection to discover DbContext properties and generate database schemas
- **API versioning:** Uses reflection to discover API controllers and versions
- **Swagger/OpenAPI:** Uses reflection to generate API documentation from code
- **GraphQL:** Uses reflection to discover types and generate schemas

**Advanced scenarios:**
- **Dynamic proxy generation:** Create types at runtime for AOP, lazy loading, remote procedure calls
- **Expression tree compilation:** Convert runtime expressions to executable code
- **Assembly scanning:** Discover all types implementing an interface for registration
- **Convention-based configuration:** Apply configuration based on naming patterns

**Interview expectations:** When interviewers ask about reflection, they're evaluating:
- Do you understand how major frameworks work internally?
- Can you make informed decisions about when to use reflection?
- Do you know how to optimize reflection-heavy code?
- Can you design plugin architectures or extensible systems?
- Do you understand the trade-offs between flexibility and performance?
- Can you debug issues in reflection-based frameworks?
- Do you know alternatives (source generators, compiled expressions)?

Reflection questions separate candidates who just use frameworks from those who understand them. It's often discussed in senior and architect interviews because it requires system-level thinking about the runtime, type system, and performance characteristics.

Understanding reflection deeply allows you to:
- Build frameworks and libraries used by other developers
- Debug complex issues in third-party libraries
- Design flexible, extensible architectures
- Optimize performance-critical code paths
- Make informed architectural decisions about trade-offs

---

## Core Ideas

### 1. **Reflection inspects types at runtime**

Reflection allows you to:
- Discover type information (properties, methods, fields)
- Create instances dynamically
- Invoke methods at runtime
- Read and write properties/fields
- Load assemblies

```csharp
Type type = typeof(string);
// Or from instance
object obj = "hello";
Type type2 = obj.GetType();

// Get type information
Console.WriteLine(type.FullName);    // System.String
Console.WriteLine(type.IsClass);     // True
Console.WriteLine(type.IsSealed);    // True
```

**Mental model:**  
"Reflection = inspecting and manipulating code as data at runtime."

---

### 2. **Attributes are metadata attached to code**

Attributes annotate types, methods, properties with additional information.

```csharp
// Built-in attribute
[Obsolete("Use NewMethod instead")]
public void OldMethod() { }

// Custom attribute
[AttributeUsage(AttributeTargets.Class | AttributeTargets.Method)]
public class AuthorAttribute : Attribute
{
    public string Name { get; }
    public string Date { get; set; }
    
    public AuthorAttribute(string name)
    {
        Name = name;
    }
}

// Usage
[Author("Alice", Date = "2024-01-15")]
public class MyClass
{
    [Author("Bob")]
    public void MyMethod() { }
}
```

---

### 3. **Reflection is expensive**

**Performance cost:**
- Type inspection: moderate
- Creating instances: slow (vs `new`)
- Invoking methods: very slow (vs direct call)
- Property access: slow

**Benchmark (relative cost):**
```
Direct call:       1x
Compiled lambda:   1.2x
Reflection:        50-100x
```

**Solutions:**
1. Cache `Type`, `MethodInfo`, `PropertyInfo` objects
2. Use compiled expressions for repeated calls
3. Avoid reflection in hot paths

---

### 4. **Common reflection operations**

```csharp
Type type = typeof(MyClass);

// Get members
PropertyInfo[] properties = type.GetProperties();
MethodInfo[] methods = type.GetMethods();
FieldInfo[] fields = type.GetFields(BindingFlags.NonPublic | BindingFlags.Instance);

// Create instance
object instance = Activator.CreateInstance(type);

// Get and set property
PropertyInfo prop = type.GetProperty("Name");
prop.SetValue(instance, "Alice");
string value = (string)prop.GetValue(instance);

// Invoke method
MethodInfo method = type.GetMethod("DoWork");
method.Invoke(instance, new object[] { arg1, arg2 });
```

---

### 5. **Expression trees for performance**

For repeated reflection calls, compile expression trees:

```csharp
// ❌ Slow reflection
PropertyInfo prop = typeof(Person).GetProperty("Name");
prop.SetValue(person, "Alice"); // Slow!

// ✅ Fast compiled expression
var param = Expression.Parameter(typeof(Person));
var property = Expression.Property(param, "Name");
var value = Expression.Constant("Alice");
var assign = Expression.Assign(property, value);
var lambda = Expression.Lambda<Action<Person>>(assign, param);
var compiled = lambda.Compile(); // Compile once

compiled(person); // Fast!
```

---

## Examples

### Example 1 — Custom attribute for validation

```csharp
[AttributeUsage(AttributeTargets.Property)]
public class RequiredAttribute : Attribute
{
}

[AttributeUsage(AttributeTargets.Property)]
public class MaxLengthAttribute : Attribute
{
    public int Length { get; }
    
    public MaxLengthAttribute(int length)
    {
        Length = length;
    }
}

// Model with attributes
public class User
{
    [Required]
    [MaxLength(50)]
    public string Name { get; set; }
    
    [Required]
    public string Email { get; set; }
}

// Validator using reflection
public class Validator
{
    public static List<string> Validate(object obj)
    {
        var errors = new List<string>();
        var type = obj.GetType();
        
        foreach (var prop in type.GetProperties())
        {
            var value = prop.GetValue(obj);
            
            // Check [Required]
            if (prop.GetCustomAttribute<RequiredAttribute>() != null)
            {
                if (value == null || (value is string str && string.IsNullOrEmpty(str)))
                {
                    errors.Add($"{prop.Name} is required");
                }
            }
            
            // Check [MaxLength]
            var maxLength = prop.GetCustomAttribute<MaxLengthAttribute>();
            if (maxLength != null && value is string strValue)
            {
                if (strValue.Length > maxLength.Length)
                {
                    errors.Add($"{prop.Name} exceeds max length of {maxLength.Length}");
                }
            }
        }
        
        return errors;
    }
}

// Usage
var user = new User { Name = "", Email = null };
var errors = Validator.Validate(user);
foreach (var error in errors)
{
    Console.WriteLine(error);
}
// Output:
// Name is required
// Email is required
```

---

### Example 2 — Simple dependency injection container

```csharp
public class SimpleContainer
{
    private readonly Dictionary<Type, Type> _registrations = new();
    private readonly Dictionary<Type, object> _singletons = new();
    
    public void Register<TInterface, TImplementation>() where TImplementation : TInterface
    {
        _registrations[typeof(TInterface)] = typeof(TImplementation);
    }
    
    public void RegisterSingleton<TInterface>(TInterface instance)
    {
        _singletons[typeof(TInterface)] = instance;
    }
    
    public T Resolve<T>()
    {
        return (T)Resolve(typeof(T));
    }
    
    private object Resolve(Type type)
    {
        // Check for singleton
        if (_singletons.TryGetValue(type, out var singleton))
        {
            return singleton;
        }
        
        // Get implementation type
        if (_registrations.TryGetValue(type, out var implType))
        {
            type = implType;
        }
        
        // Get constructor
        var constructor = type.GetConstructors()[0];
        var parameters = constructor.GetParameters();
        
        // Resolve dependencies recursively
        var args = parameters.Select(p => Resolve(p.ParameterType)).ToArray();
        
        // Create instance
        return Activator.CreateInstance(type, args);
    }
}

// Usage
interface ILogger { void Log(string msg); }
class ConsoleLogger : ILogger 
{ 
    public void Log(string msg) => Console.WriteLine(msg); 
}

interface IService { void DoWork(); }
class MyService : IService
{
    private readonly ILogger _logger;
    
    public MyService(ILogger logger)
    {
        _logger = logger;
    }
    
    public void DoWork()
    {
        _logger.Log("Working...");
    }
}

var container = new SimpleContainer();
container.Register<ILogger, ConsoleLogger>();
container.Register<IService, MyService>();

var service = container.Resolve<IService>();
service.DoWork(); // "Working..."
```

---

### Example 3 — Object mapper

```csharp
public class ObjectMapper
{
    public static TDestination Map<TSource, TDestination>(TSource source)
        where TDestination : new()
    {
        var destination = new TDestination();
        var sourceType = typeof(TSource);
        var destType = typeof(TDestination);
        
        // Get matching properties
        foreach (var sourceProp in sourceType.GetProperties())
        {
            var destProp = destType.GetProperty(sourceProp.Name);
            
            // If property exists in destination and types match
            if (destProp != null && destProp.CanWrite && 
                destProp.PropertyType == sourceProp.PropertyType)
            {
                var value = sourceProp.GetValue(source);
                destProp.SetValue(destination, value);
            }
        }
        
        return destination;
    }
}

// Usage
public class UserDto
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
}

public class UserViewModel
{
    public int Id { get; set; }
    public string Name { get; set; }
    // No Email property
}

var dto = new UserDto { Id = 1, Name = "Alice", Email = "alice@example.com" };
var viewModel = ObjectMapper.Map<UserDto, UserViewModel>(dto);
Console.WriteLine($"{viewModel.Id}, {viewModel.Name}"); // "1, Alice"
```

---

### Example 4 — Plugin system with assembly loading

```csharp
// Plugin interface
public interface IPlugin
{
    string Name { get; }
    void Execute();
}

// Plugin loader
public class PluginLoader
{
    public List<IPlugin> LoadPlugins(string directory)
    {
        var plugins = new List<IPlugin>();
        var dllFiles = Directory.GetFiles(directory, "*.dll");
        
        foreach (var dllFile in dllFiles)
        {
            try
            {
                // Load assembly
                var assembly = Assembly.LoadFrom(dllFile);
                
                // Find types implementing IPlugin
                var pluginTypes = assembly.GetTypes()
                    .Where(t => typeof(IPlugin).IsAssignableFrom(t) && !t.IsInterface);
                
                // Create instances
                foreach (var type in pluginTypes)
                {
                    var plugin = (IPlugin)Activator.CreateInstance(type);
                    plugins.Add(plugin);
                }
            }
            catch (Exception ex)
            {
                Console.WriteLine($"Failed to load {dllFile}: {ex.Message}");
            }
        }
        
        return plugins;
    }
}

// Usage
var loader = new PluginLoader();
var plugins = loader.LoadPlugins("./plugins");

foreach (var plugin in plugins)
{
    Console.WriteLine($"Executing {plugin.Name}");
    plugin.Execute();
}
```

---

### Example 5 — Cached property accessor

```csharp
public class PropertyAccessor<T>
{
    private static readonly Dictionary<string, Func<T, object>> _getters = new();
    private static readonly Dictionary<string, Action<T, object>> _setters = new();
    
    static PropertyAccessor()
    {
        // Build cached accessors for all properties
        foreach (var prop in typeof(T).GetProperties())
        {
            // Build getter
            var param = Expression.Parameter(typeof(T));
            var property = Expression.Property(param, prop);
            var convert = Expression.Convert(property, typeof(object));
            var lambda = Expression.Lambda<Func<T, object>>(convert, param);
            _getters[prop.Name] = lambda.Compile();
            
            // Build setter
            if (prop.CanWrite)
            {
                var instance = Expression.Parameter(typeof(T));
                var value = Expression.Parameter(typeof(object));
                var convertValue = Expression.Convert(value, prop.PropertyType);
                var assign = Expression.Assign(Expression.Property(instance, prop), convertValue);
                var setLambda = Expression.Lambda<Action<T, object>>(assign, instance, value);
                _setters[prop.Name] = setLambda.Compile();
            }
        }
    }
    
    public static object GetValue(T instance, string propertyName)
    {
        return _getters[propertyName](instance);
    }
    
    public static void SetValue(T instance, string propertyName, object value)
    {
        _setters[propertyName](instance, value);
    }
}

// Usage - much faster than reflection
var person = new Person { Name = "Alice", Age = 30 };
var name = PropertyAccessor<Person>.GetValue(person, "Name"); // Fast!
PropertyAccessor<Person>.SetValue(person, "Age", 31); // Fast!
```

---

## Common Pitfalls

### ❌ Pitfall 1: Not caching Type/MethodInfo

```csharp
// ❌ Slow - gets type every call
public void SlowMethod(object obj)
{
    Type type = obj.GetType();
    MethodInfo method = type.GetMethod("DoWork");
    method.Invoke(obj, null);
}

// ✅ Fast - cache reflection objects
private static readonly Dictionary<Type, MethodInfo> _methodCache = new();

public void FastMethod(object obj)
{
    Type type = obj.GetType();
    if (!_methodCache.TryGetValue(type, out var method))
    {
        method = type.GetMethod("DoWork");
        _methodCache[type] = method;
    }
    method.Invoke(obj, null);
}
```

---

### ❌ Pitfall 2: Ignoring BindingFlags

```csharp
// ❌ Only gets public instance members
var methods = type.GetMethods();

// ✅ Specify what you want
var allMethods = type.GetMethods(
    BindingFlags.Public | 
    BindingFlags.NonPublic | 
    BindingFlags.Instance | 
    BindingFlags.Static);
```

---

### ❌ Pitfall 3: Using reflection in hot paths

```csharp
// ❌ Terrible performance
for (int i = 0; i < 1000000; i++)
{
    var method = type.GetMethod("Process");
    method.Invoke(instance, null);
}

// ✅ Use direct call or compiled expression
for (int i = 0; i < 1000000; i++)
{
    instance.Process(); // Direct call
}
```

---

### ❌ Pitfall 4: Not handling exceptions from Invoke

```csharp
// ❌ Inner exception is wrapped
try
{
    method.Invoke(instance, null);
}
catch (Exception ex)
{
    // ex is TargetInvocationException, not the actual exception!
}

// ✅ Unwrap inner exception
try
{
    method.Invoke(instance, null);
}
catch (TargetInvocationException ex)
{
    throw ex.InnerException; // Actual exception
}
```

---

## Quick Self-Check

1. What is reflection and when would you use it?
2. What's the performance cost of reflection?
3. How do you create a custom attribute?
4. What are BindingFlags and why are they important?
5. How can you improve reflection performance?

<details>
<summary>Answers</summary>

1. Runtime type inspection/manipulation. Use for: DI, serialization, plugins, testing frameworks
2. 50-100x slower than direct calls. Type inspection moderate, method invocation very slow
3. Inherit from `Attribute`, add `AttributeUsage`, define properties/constructor
4. Flags specifying which members to retrieve (Public, NonPublic, Instance, Static, etc.)
5. Cache Type/MethodInfo objects, use compiled expressions, avoid in hot paths

</details>

---

## Further Practice

1. **Build** a simple JSON serializer using reflection
2. **Create** a fluent validation library with custom attributes
3. **Implement** a property change notifier using reflection
4. **Benchmark** reflection vs direct call vs compiled expression
5. **Build** an auto-mapper that handles nested objects

---

## Key Takeaways

✅ Reflection inspects and manipulates types at runtime  
✅ Attributes add metadata to code elements  
✅ Reflection is 50-100x slower than direct calls  
✅ Cache Type, MethodInfo, PropertyInfo objects  
✅ Use compiled expressions for repeated reflection  
✅ Specify BindingFlags to get correct members  
✅ Unwrap TargetInvocationException for actual exceptions  
✅ Common uses: DI, serialization, validation, plugins
