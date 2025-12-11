# Expression Trees and Dynamic Code

🔥 **Advanced** | ⏱ 45-55 min

---

## 📝 Why This Matters

Expression trees are one of C#'s most powerful but underutilized features. They represent code as data—turning executable C# expressions into tree structures that you can analyze, modify, and compile at runtime. This might sound abstract, but expression trees power some of the most important features you use every day.

When you write a LINQ query against Entity Framework like `db.Users.Where(u => u.Age > 18)`, that lambda expression isn't executed directly. Instead, it's converted to an expression tree that Entity Framework analyzes to generate SQL. The `u => u.Age > 18` becomes a data structure describing "a lambda with parameter u, returning a comparison where u's Age property is greater than 18." EF translates this structure to `WHERE Age > 18` in SQL. Without expression trees, LINQ to SQL/EF wouldn't be possible.

In real-world applications, expression trees enable powerful abstractions. They're used in AutoMapper for property mapping, in validation frameworks like FluentValidation for building validation rules, in testing frameworks for verifying method calls (`mock.Verify(m => m.Send(It.IsAny<string>()))`), and in serialization libraries for efficient property access. Any time a library needs to analyze your code at runtime rather than just execute it, expression trees are likely involved.

Understanding expression trees makes you a better library consumer. When you see `Expression<Func<T, bool>>` in a method signature, you know the library will analyze (not execute) your lambda, so side effects won't occur and the lambda must be translatable to the target platform (SQL, NoSQL, HTTP query strings, etc.). This explains why some LINQ methods work with `IEnumerable<T>` but fail with `IQueryable<T>`—the expression can't be translated.

Expression trees also open doors to advanced scenarios: building dynamic queries based on user input, generating code at runtime, creating DSLs (domain-specific languages), implementing business rule engines, and building compiler-like tools. If you've ever needed to build a query UI where users select fields, operators, and values to filter data, expression trees make this possible without string concatenation or SQL injection risks.

From a performance perspective, compiled expression trees can be faster than reflection for repeated operations. While reflection analyzes methods/properties every time, you can build an expression tree once, compile it to a delegate, and reuse that delegate with native performance. This pattern is used in high-performance serializers and mappers.

In interviews, expression trees are a differentiator. Most developers know lambda expressions but not the distinction between `Func<T, bool>` and `Expression<Func<T, bool>>`. Being able to explain this, show practical use cases, and demonstrate building an expression tree shows deep .NET knowledge. It's particularly relevant for senior roles where you might design APIs or build frameworks.

The learning curve is steep—expression trees involve metaprogramming (code that manipulates code), which requires thinking at a higher level of abstraction. But once you understand the basics, you'll see their patterns everywhere in modern .NET libraries and appreciate the elegance of LINQ's design.

Understanding expression trees also helps with debugging. When you get errors like "LINQ expression could not be translated," you'll understand it means the expression tree contains operations that can't be converted to SQL. When a lambda works in one context but not another, you'll know to check if one uses `Func<>` (execution) and the other uses `Expression<Func<>>` (analysis).

Expression trees bridge the gap between compile-time and runtime. They let you write type-safe code that gets analyzed at runtime, combining the best of both worlds: compile-time checking and runtime flexibility. This is why they're central to modern data access, testing, and metaprogramming in .NET.

---

## 🎯 Core Ideas

### 1. **Expression Trees vs. Lambda Expressions**

**The fundamental difference:**

```csharp
// Regular delegate - code that executes
Func<int, int, int> addFunc = (a, b) => a + b;
int result = addFunc(3, 4); // Executes: result = 7

// Expression tree - data structure describing code
Expression<Func<int, int, int>> addExpr = (a, b) => a + b;
// addExpr doesn't execute—it's a tree describing the operation

// To execute an expression tree, compile it first
Func<int, int, int> compiled = addExpr.Compile();
int result2 = compiled(3, 4); // Now it executes: result2 = 7

// Inspecting the expression tree
Console.WriteLine(addExpr.Body);           // (a + b)
Console.WriteLine(addExpr.Parameters[0]);  // a
Console.WriteLine(addExpr.Parameters[1]);  // b

var binaryExpr = (BinaryExpression)addExpr.Body;
Console.WriteLine(binaryExpr.NodeType);    // Add
Console.WriteLine(binaryExpr.Left);        // a
Console.WriteLine(binaryExpr.Right);       // b
```

**Visual representation of the expression tree:**

```
Expression<Func<int, int, int>>: (a, b) => a + b

         Lambda
           |
        Body (BinaryExpression: Add)
        /                    \
    Left (Parameter: a)   Right (Parameter: b)
```

**Why Entity Framework needs expression trees:**

```csharp
// ❌ This won't work - Func<> executes in C#, can't translate to SQL
IEnumerable<User> users = dbContext.Users.AsEnumerable();
var adults = users.Where(u => u.Age > 18); // Executes in memory after loading ALL users

// ✅ This works - Expression<Func<>> analyzed and converted to SQL
IQueryable<User> query = dbContext.Users;
var adults = query.Where(u => u.Age > 18); // Generates: SELECT * FROM Users WHERE Age > 18

// Behind the scenes, EF analyzes the expression tree:
Expression<Func<User, bool>> expr = u => u.Age > 18;
// EF walks the tree and sees:
// - Lambda with parameter 'u' of type User
// - Body is BinaryExpression (GreaterThan)
// - Left: MemberAccess to 'Age' property on 'u'
// - Right: ConstantExpression with value 18
// Translates to: WHERE Age > 18
```

### 2. **Building Expression Trees Manually**

**Simple expressions:**

```csharp
// Building: (x, y) => x + y
ParameterExpression paramX = Expression.Parameter(typeof(int), "x");
ParameterExpression paramY = Expression.Parameter(typeof(int), "y");

BinaryExpression body = Expression.Add(paramX, paramY);

Expression<Func<int, int, int>> lambda = 
    Expression.Lambda<Func<int, int, int>>(body, paramX, paramY);

var compiled = lambda.Compile();
Console.WriteLine(compiled(3, 4)); // 7

// Building: (x) => x * 2 + 1
ParameterExpression param = Expression.Parameter(typeof(int), "x");

// x * 2
BinaryExpression multiply = Expression.Multiply(
    param,
    Expression.Constant(2)
);

// (x * 2) + 1
BinaryExpression add = Expression.Add(
    multiply,
    Expression.Constant(1)
);

Expression<Func<int, int>> lambda2 = 
    Expression.Lambda<Func<int, int>>(add, param);

var compiled2 = lambda2.Compile();
Console.WriteLine(compiled2(5)); // 11 (5 * 2 + 1)
```

**Property access:**

```csharp
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
}

// Building: (p) => p.Name
ParameterExpression param = Expression.Parameter(typeof(Person), "p");
MemberExpression property = Expression.Property(param, "Name");

Expression<Func<Person, string>> lambda = 
    Expression.Lambda<Func<Person, string>>(property, param);

var compiled = lambda.Compile();
var person = new Person { Name = "Alice", Age = 30 };
Console.WriteLine(compiled(person)); // Alice

// Building: (p) => p.Age > 18
ParameterExpression param2 = Expression.Parameter(typeof(Person), "p");
MemberExpression ageProperty = Expression.Property(param2, "Age");
ConstantExpression constant = Expression.Constant(18);
BinaryExpression comparison = Expression.GreaterThan(ageProperty, constant);

Expression<Func<Person, bool>> lambda2 = 
    Expression.Lambda<Func<Person, bool>>(comparison, param2);

var compiled2 = lambda2.Compile();
Console.WriteLine(compiled2(person)); // true (30 > 18)
```

**Method calls:**

```csharp
// Building: (s) => s.ToUpper()
ParameterExpression param = Expression.Parameter(typeof(string), "s");
MethodInfo method = typeof(string).GetMethod("ToUpper", Type.EmptyTypes);
MethodCallExpression call = Expression.Call(param, method);

Expression<Func<string, string>> lambda = 
    Expression.Lambda<Func<string, string>>(call, param);

var compiled = lambda.Compile();
Console.WriteLine(compiled("hello")); // HELLO

// Building: (s) => s.Contains("test")
MethodInfo containsMethod = typeof(string).GetMethod("Contains", new[] { typeof(string) });
MethodCallExpression containsCall = Expression.Call(
    param,
    containsMethod,
    Expression.Constant("test")
);

Expression<Func<string, bool>> lambda2 = 
    Expression.Lambda<Func<string, bool>>(containsCall, param);

var compiled2 = lambda2.Compile();
Console.WriteLine(compiled2("testing")); // true
```

**Conditional expressions:**

```csharp
// Building: (x) => x > 0 ? "positive" : "non-positive"
ParameterExpression param = Expression.Parameter(typeof(int), "x");

BinaryExpression condition = Expression.GreaterThan(
    param,
    Expression.Constant(0)
);

ConstantExpression ifTrue = Expression.Constant("positive");
ConstantExpression ifFalse = Expression.Constant("non-positive");

ConditionalExpression conditional = Expression.Condition(
    condition,
    ifTrue,
    ifFalse
);

Expression<Func<int, string>> lambda = 
    Expression.Lambda<Func<int, string>>(conditional, param);

var compiled = lambda.Compile();
Console.WriteLine(compiled(5));   // positive
Console.WriteLine(compiled(-3));  // non-positive
Console.WriteLine(compiled(0));   // non-positive
```

### 3. **Practical Use Case: Dynamic Filters**

**Building filters from user input:**

```csharp
public class ProductFilter
{
    public string Property { get; set; }  // "Name", "Price", "Category"
    public string Operator { get; set; }  // "equals", "contains", "greater"
    public string Value { get; set; }
}

public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public string Category { get; set; }
}

public class DynamicFilterBuilder
{
    public static Expression<Func<Product, bool>> BuildFilter(ProductFilter filter)
    {
        var parameter = Expression.Parameter(typeof(Product), "p");
        var property = Expression.Property(parameter, filter.Property);
        
        Expression body = filter.Operator.ToLower() switch
        {
            "equals" => BuildEquals(property, filter.Value),
            "contains" => BuildContains(property, filter.Value),
            "greater" => BuildGreaterThan(property, filter.Value),
            "less" => BuildLessThan(property, filter.Value),
            _ => throw new ArgumentException($"Unknown operator: {filter.Operator}")
        };
        
        return Expression.Lambda<Func<Product, bool>>(body, parameter);
    }
    
    private static Expression BuildEquals(MemberExpression property, string value)
    {
        var constant = Expression.Constant(Convert.ChangeType(value, property.Type));
        return Expression.Equal(property, constant);
    }
    
    private static Expression BuildContains(MemberExpression property, string value)
    {
        if (property.Type != typeof(string))
            throw new InvalidOperationException("Contains only works with string properties");
        
        var method = typeof(string).GetMethod("Contains", new[] { typeof(string) });
        var constant = Expression.Constant(value);
        return Expression.Call(property, method, constant);
    }
    
    private static Expression BuildGreaterThan(MemberExpression property, string value)
    {
        var constant = Expression.Constant(Convert.ChangeType(value, property.Type));
        return Expression.Equal(property, constant);
    }
    
    private static Expression BuildLessThan(MemberExpression property, string value)
    {
        var constant = Expression.Constant(Convert.ChangeType(value, property.Type));
        return Expression.LessThan(property, constant);
    }
}

// Usage with Entity Framework
var filter = new ProductFilter
{
    Property = "Price",
    Operator = "greater",
    Value = "50"
};

var predicate = DynamicFilterBuilder.BuildFilter(filter);
var products = dbContext.Products.Where(predicate).ToList();
// Generates: SELECT * FROM Products WHERE Price > 50

// Combining multiple filters with AND
var filters = new[]
{
    new ProductFilter { Property = "Category", Operator = "equals", Value = "Electronics" },
    new ProductFilter { Property = "Price", Operator = "less", Value = "1000" }
};

Expression<Func<Product, bool>> combinedPredicate = p => true; // Start with always-true

foreach (var f in filters)
{
    var filterExpr = DynamicFilterBuilder.BuildFilter(f);
    var invokedExpr = Expression.Invoke(filterExpr, combinedPredicate.Parameters);
    var body = Expression.AndAlso(combinedPredicate.Body, invokedExpr);
    combinedPredicate = Expression.Lambda<Func<Product, bool>>(body, combinedPredicate.Parameters);
}

var results = dbContext.Products.Where(combinedPredicate).ToList();
// Generates: SELECT * FROM Products WHERE Category = 'Electronics' AND Price < 1000
```

**Better approach with PredicateBuilder:**

```csharp
public static class PredicateBuilder
{
    public static Expression<Func<T, bool>> True<T>() => x => true;
    public static Expression<Func<T, bool>> False<T>() => x => false;
    
    public static Expression<Func<T, bool>> And<T>(
        this Expression<Func<T, bool>> left,
        Expression<Func<T, bool>> right)
    {
        var parameter = Expression.Parameter(typeof(T));
        
        var leftVisitor = new ReplaceExpressionVisitor(left.Parameters[0], parameter);
        var leftBody = leftVisitor.Visit(left.Body);
        
        var rightVisitor = new ReplaceExpressionVisitor(right.Parameters[0], parameter);
        var rightBody = rightVisitor.Visit(right.Body);
        
        var body = Expression.AndAlso(leftBody, rightBody);
        return Expression.Lambda<Func<T, bool>>(body, parameter);
    }
    
    public static Expression<Func<T, bool>> Or<T>(
        this Expression<Func<T, bool>> left,
        Expression<Func<T, bool>> right)
    {
        var parameter = Expression.Parameter(typeof(T));
        
        var leftVisitor = new ReplaceExpressionVisitor(left.Parameters[0], parameter);
        var leftBody = leftVisitor.Visit(left.Body);
        
        var rightVisitor = new ReplaceExpressionVisitor(right.Parameters[0], parameter);
        var rightBody = rightVisitor.Visit(right.Body);
        
        var body = Expression.OrElse(leftBody, rightBody);
        return Expression.Lambda<Func<T, bool>>(body, parameter);
    }
    
    private class ReplaceExpressionVisitor : ExpressionVisitor
    {
        private readonly Expression _oldValue;
        private readonly Expression _newValue;
        
        public ReplaceExpressionVisitor(Expression oldValue, Expression newValue)
        {
            _oldValue = oldValue;
            _newValue = newValue;
        }
        
        public override Expression Visit(Expression node)
        {
            return node == _oldValue ? _newValue : base.Visit(node);
        }
    }
}

// Usage
var predicate = PredicateBuilder.True<Product>();

if (!string.IsNullOrEmpty(categoryFilter))
{
    predicate = predicate.And(p => p.Category == categoryFilter);
}

if (minPrice.HasValue)
{
    predicate = predicate.And(p => p.Price >= minPrice.Value);
}

if (maxPrice.HasValue)
{
    predicate = predicate.And(p => p.Price <= maxPrice.Value);
}

var products = dbContext.Products.Where(predicate).ToList();
```

### 4. **Expression Tree Visitors**

**Analyzing expression trees:**

```csharp
public class ExpressionAnalyzer : ExpressionVisitor
{
    private readonly List<string> _properties = new();
    private readonly List<object> _constants = new();
    
    public List<string> Properties => _properties;
    public List<object> Constants => _constants;
    
    protected override Expression VisitMember(MemberExpression node)
    {
        if (node.Member is PropertyInfo property)
        {
            _properties.Add(property.Name);
        }
        return base.VisitMember(node);
    }
    
    protected override Expression VisitConstant(ConstantExpression node)
    {
        _constants.Add(node.Value);
        return base.VisitConstant(node);
    }
}

// Usage
Expression<Func<Person, bool>> expr = p => p.Age > 18 && p.Name == "Alice";

var analyzer = new ExpressionAnalyzer();
analyzer.Visit(expr);

Console.WriteLine("Properties accessed: " + string.Join(", ", analyzer.Properties));
// Properties accessed: Age, Name

Console.WriteLine("Constants used: " + string.Join(", ", analyzer.Constants));
// Constants used: 18, Alice
```

**Modifying expression trees:**

```csharp
// Replace all references to one parameter with another
public class ParameterReplacer : ExpressionVisitor
{
    private readonly ParameterExpression _oldParameter;
    private readonly ParameterExpression _newParameter;
    
    public ParameterReplacer(ParameterExpression oldParameter, ParameterExpression newParameter)
    {
        _oldParameter = oldParameter;
        _newParameter = newParameter;
    }
    
    protected override Expression VisitParameter(ParameterExpression node)
    {
        return node == _oldParameter ? _newParameter : base.VisitParameter(node);
    }
}

// Combining two expressions with same parameter
public static Expression<Func<T, bool>> CombineWithAnd<T>(
    Expression<Func<T, bool>> left,
    Expression<Func<T, bool>> right)
{
    var parameter = Expression.Parameter(typeof(T), "x");
    
    var leftBody = new ParameterReplacer(left.Parameters[0], parameter).Visit(left.Body);
    var rightBody = new ParameterReplacer(right.Parameters[0], parameter).Visit(right.Body);
    
    var combined = Expression.AndAlso(leftBody, rightBody);
    return Expression.Lambda<Func<T, bool>>(combined, parameter);
}

// Usage
Expression<Func<Person, bool>> isAdult = p => p.Age >= 18;
Expression<Func<Person, bool>> hasName = p => !string.IsNullOrEmpty(p.Name);

var combined = CombineWithAnd(isAdult, hasName);
// Result: x => (x.Age >= 18) && !string.IsNullOrEmpty(x.Name)

var compiledFilter = combined.Compile();
var person1 = new Person { Age = 25, Name = "Alice" };
var person2 = new Person { Age = 30, Name = null };

Console.WriteLine(compiledFilter(person1)); // true
Console.WriteLine(compiledFilter(person2)); // false
```

### 5. **Performance: Expression Trees vs. Reflection**

**Why compiled expressions are faster than reflection:**

```csharp
public class BenchmarkExample
{
    public class Product
    {
        public string Name { get; set; }
        public decimal Price { get; set; }
    }
    
    // Direct access (baseline)
    public static string GetNameDirect(Product product)
    {
        return product.Name;
    }
    
    // Reflection (slowest)
    public static string GetNameReflection(Product product)
    {
        var property = typeof(Product).GetProperty("Name");
        return (string)property.GetValue(product);
    }
    
    // Compiled expression (fast)
    private static readonly Func<Product, string> _compiledGetter = BuildGetter();
    
    private static Func<Product, string> BuildGetter()
    {
        var parameter = Expression.Parameter(typeof(Product), "p");
        var property = Expression.Property(parameter, "Name");
        var lambda = Expression.Lambda<Func<Product, string>>(property, parameter);
        return lambda.Compile();
    }
    
    public static string GetNameCompiled(Product product)
    {
        return _compiledGetter(product);
    }
}

// Benchmark results (approximate):
// Direct:     1x (baseline)
// Compiled:   1.1x (nearly as fast as direct)
// Reflection: 50x (much slower)

// Compiled expressions amortize the compilation cost
// Build once, use many times = nearly native performance
```

**Practical usage in mapping:**

```csharp
public class PropertyMapper<TSource, TDest>
{
    private readonly Dictionary<string, Action<TSource, TDest>> _propertySetters = new();
    
    public PropertyMapper()
    {
        BuildPropertySetters();
    }
    
    private void BuildPropertySetters()
    {
        var sourceProps = typeof(TSource).GetProperties();
        var destProps = typeof(TDest).GetProperties().ToDictionary(p => p.Name);
        
        foreach (var sourceProp in sourceProps)
        {
            if (destProps.TryGetValue(sourceProp.Name, out var destProp) &&
                destProp.PropertyType == sourceProp.PropertyType)
            {
                var setter = BuildSetter(sourceProp, destProp);
                _propertySetters[sourceProp.Name] = setter;
            }
        }
    }
    
    private Action<TSource, TDest> BuildSetter(PropertyInfo sourceProp, PropertyInfo destProp)
    {
        var sourceParam = Expression.Parameter(typeof(TSource), "source");
        var destParam = Expression.Parameter(typeof(TDest), "dest");
        
        var sourceProperty = Expression.Property(sourceParam, sourceProp);
        var destProperty = Expression.Property(destParam, destProp);
        var assign = Expression.Assign(destProperty, sourceProperty);
        
        return Expression.Lambda<Action<TSource, TDest>>(assign, sourceParam, destParam).Compile();
    }
    
    public void Map(TSource source, TDest dest)
    {
        foreach (var setter in _propertySetters.Values)
        {
            setter(source, dest);
        }
    }
}

// Usage
public class PersonDto
{
    public string Name { get; set; }
    public int Age { get; set; }
}

public class PersonEntity
{
    public string Name { get; set; }
    public int Age { get; set; }
}

var mapper = new PropertyMapper<PersonDto, PersonEntity>();

var dto = new PersonDto { Name = "Alice", Age = 30 };
var entity = new PersonEntity();

mapper.Map(dto, entity);
// entity.Name = "Alice", entity.Age = 30
// Uses compiled expressions, much faster than reflection for repeated calls
```

### 6. **Common Patterns in Libraries**

**Entity Framework Core:**

```csharp
// Expression trees enable LINQ to SQL translation
var users = dbContext.Users
    .Where(u => u.Age > 18)              // Expression tree
    .OrderBy(u => u.Name)                // Expression tree
    .Select(u => new { u.Name, u.Age })  // Expression tree
    .ToList();

// EF walks these expression trees and generates:
// SELECT Name, Age FROM Users WHERE Age > 18 ORDER BY Name
```

**FluentValidation:**

```csharp
public class PersonValidator : AbstractValidator<Person>
{
    public PersonValidator()
    {
        // Expression tree: p => p.Name
        RuleFor(p => p.Name).NotEmpty().MaximumLength(100);
        
        // Expression tree: p => p.Age
        RuleFor(p => p.Age).GreaterThan(0).LessThan(150);
        
        // Library analyzes expression to get property name for error messages
    }
}
```

**Moq:**

```csharp
var mock = new Mock<IEmailService>();

// Expression tree: m => m.Send(It.IsAny<string>())
mock.Setup(m => m.Send(It.IsAny<string>())).Returns(true);

// Expression tree analyzed to identify method and arguments
mock.Verify(m => m.Send("test@example.com"), Times.Once);
```

---

## ⚠️ Common Pitfalls

### 1. **Closures in Expression Trees**

```csharp
// ❌ Closure captures variable, not value
var filters = new List<Expression<Func<Product, bool>>>();
for (int i = 0; i < 3; i++)
{
    // All expressions will see i = 3 (final value)
    filters.Add(p => p.Id == i);
}

// ✅ Capture the value explicitly
var filters2 = new List<Expression<Func<Product, bool>>>();
for (int i = 0; i < 3; i++)
{
    int capturedValue = i; // Copy to local
    filters2.Add(p => p.Id == capturedValue);
}
```

### 2. **Not All C# Features Translate to SQL**

```csharp
// ❌ Won't translate - custom method
public static bool IsValidEmail(string email) => 
    email.Contains("@");

var users = dbContext.Users
    .Where(u => IsValidEmail(u.Email)) // ❌ Exception: can't translate
    .ToList();

// ✅ Use database functions or evaluate client-side
var users2 = dbContext.Users
    .Where(u => u.Email.Contains("@")) // ✅ Translates to SQL LIKE
    .ToList();

// Or evaluate client-side:
var users3 = dbContext.Users
    .ToList() // Load all users
    .Where(u => IsValidEmail(u.Email)) // Filter in memory
    .ToList();
```

### 3. **Forgetting to Compile Expression Trees**

```csharp
// ❌ This doesn't work - expression tree isn't executable
Expression<Func<int, int>> expr = x => x * 2;
// int result = expr(5); // ❌ Compile error: can't invoke expression

// ✅ Compile first
var func = expr.Compile();
int result = func(5); // ✅ Works
```

### 4. **Performance: Compiling Every Time**

```csharp
// ❌ Compiling on every call is expensive
public string GetPropertyValue(object obj, string propertyName)
{
    var param = Expression.Parameter(typeof(object), "obj");
    var cast = Expression.Convert(param, obj.GetType());
    var property = Expression.Property(cast, propertyName);
    var lambda = Expression.Lambda<Func<object, object>>(
        Expression.Convert(property, typeof(object)), param);
    
    var func = lambda.Compile(); // ❌ Expensive, done every call
    return func(obj)?.ToString();
}

// ✅ Cache compiled expressions
private static readonly ConcurrentDictionary<string, Func<object, object>> _cache = new();

public string GetPropertyValueCached(object obj, string propertyName)
{
    var key = $"{obj.GetType().FullName}.{propertyName}";
    var func = _cache.GetOrAdd(key, _ => BuildGetter(obj.GetType(), propertyName));
    return func(obj)?.ToString();
}
```

### 5. **Incorrect Type Conversions**

```csharp
// ❌ Type mismatch
var param = Expression.Parameter(typeof(int), "x");
var constant = Expression.Constant("5"); // string, not int!
var expr = Expression.Add(param, constant); // ❌ Runtime error

// ✅ Ensure types match
var param2 = Expression.Parameter(typeof(int), "x");
var constant2 = Expression.Constant(5); // int
var expr2 = Expression.Add(param2, constant2); // ✅ Works
```

### 6. **Modifying Expression Trees Incorrectly**

```csharp
// ❌ Can't modify expression nodes (they're immutable)
Expression<Func<int, int>> expr = x => x + 1;
// expr.Body = Expression.Add(...); // ❌ No setter, expressions are immutable

// ✅ Create new expression with ExpressionVisitor
public class IncrementRewriter : ExpressionVisitor
{
    protected override Expression VisitConstant(ConstantExpression node)
    {
        if (node.Type == typeof(int))
        {
            return Expression.Constant((int)node.Value + 1);
        }
        return base.VisitConstant(node);
    }
}

var rewriter = new IncrementRewriter();
var modified = rewriter.Visit(expr);
// Original: x => x + 1
// Modified: x => x + 2 (constant incremented)
```

---

## ✅ Self-Check Questions

1. **What's the difference between `Func<T, bool>` and `Expression<Func<T, bool>>`?**
   - `Func<>`: executable delegate, runs immediately
   - `Expression<Func<>>`: data structure describing code, can be analyzed
   - EF needs `Expression<>` to translate to SQL

2. **When would you build expression trees manually instead of using lambdas?**
   - Dynamic query building from user input
   - Generating code at runtime based on conditions
   - Building DSLs or rule engines
   - When the expression structure isn't known at compile time

3. **Why are compiled expressions faster than reflection?**
   - Compilation happens once, execution is native code
   - Reflection analyzes metadata on every call
   - Compiled expressions have near-direct-access performance
   - Amortizes compilation cost over many invocations

4. **How do you combine multiple expression predicates with AND logic?**
   - Use ExpressionVisitor to replace parameters
   - Combine bodies with `Expression.AndAlso`
   - Create new lambda with unified parameter
   - Or use PredicateBuilder pattern

5. **What does "expression could not be translated" mean in EF?**
   - Expression tree contains operations that can't convert to SQL
   - Custom methods, certain LINQ operators, or complex logic
   - Solution: simplify expression or use client-side evaluation

---

## 🏋️ Practice

1. **Build a dynamic search system** where users can create filters on multiple properties with different operators (equals, contains, greater than, less than) and combine them with AND/OR logic.

2. **Create a property copier** that uses expression trees to build high-performance property mappers between two types, caching the compiled delegates.

3. **Implement a simple ORM** that converts expression trees like `db.Query<User>().Where(u => u.Age > 18).OrderBy(u => u.Name)` into SQL strings.

4. **Build an expression analyzer** that walks an expression tree and generates documentation showing all properties accessed, methods called, and constants used.

5. **Create a validation framework** where rules are defined as expression trees (`RuleFor(u => u.Email).Must(BeValidEmail)`) and validation errors include the property name extracted from the expression.

---

## 🎤 Interview Cheat Sheet

**Quick answers for common questions:**

**"What are expression trees?"**
> "Expression trees represent code as data—they're tree structures that describe code instead of executing it. When you write a LINQ query with Entity Framework like `db.Users.Where(u => u.Age > 18)`, that lambda is converted to an expression tree that EF analyzes to generate SQL. The key difference is `Func<T, bool>` executes immediately, while `Expression<Func<T, bool>>` is a data structure you can inspect, modify, and translate to other languages like SQL."

**"When have you used expression trees?"**
> "I used expression trees to build a dynamic filtering system where users could create complex queries through a UI—selecting properties, operators, and values. I built expression trees programmatically based on their selections and passed them to Entity Framework, which safely generated parameterized SQL queries. This avoided string concatenation and SQL injection risks while providing type safety and compile-time checking."

**"Why does EF need Expression<Func<>> instead of Func<>?"**
> "EF needs to translate your C# code to SQL, which requires analyzing the code structure, not executing it. An `Expression<Func<>>` is a data structure representing the code that EF can walk to extract property names, operators, and values, then convert to SQL. A regular `Func<>` is already compiled to IL—EF can't analyze it. If you use `Func<>`, the filter runs in C# after loading all data from the database, which is inefficient."

**"How do you build expression trees programmatically?"**
> "You use the `Expression` class factory methods. For example, to build `(x, y) => x + y`, you create parameter expressions for x and y, use `Expression.Add` to create the addition operation, then wrap it in `Expression.Lambda`. For property access like `p => p.Name`, you use `Expression.Property`. For method calls like `s => s.Contains('test')`, you use `Expression.Call` with the MethodInfo from reflection. You can then compile the expression to a delegate with `.Compile()` to execute it."

**"What's the performance difference between expression trees and reflection?"**
> "Reflection is slow because it analyzes metadata on every access—getting PropertyInfo, invoking GetValue, etc. Expression trees compiled with `.Compile()` generate IL code that's just-in-time compiled to native code, giving near-direct-access performance. The compilation is expensive, but once compiled, it's reusable. So expression trees are ideal when you need to build dynamic accessors once and reuse them thousands of times—like in serializers or mappers. For a single access, reflection might be simpler."

**"How do you combine multiple expression predicates?"**
> "The challenge is that each expression has its own parameter. You need to replace all parameters with a single unified parameter, then combine the bodies with `Expression.AndAlso` or `Expression.OrElse`. I typically use an ExpressionVisitor to walk the tree and replace parameter references, then create a new lambda with the combined body. There's also a PredicateBuilder pattern that handles this elegantly for building complex queries from multiple conditions."

**"What does 'could not be translated' mean in EF Core?"**
> "It means the expression tree contains operations that Entity Framework can't convert to SQL. Common causes are calling custom C# methods, using unsupported LINQ operators, or complex logic that doesn't map to SQL. The solution is either to simplify the expression to use only supported operations, or switch to client-side evaluation by calling `.AsEnumerable()` or `.ToList()` first—though that loads data into memory and may be inefficient. EF Core is more strict about this than EF6 to prevent accidental performance issues."

**Key phrases that impress:**
- "Expression trees enable LINQ providers like Entity Framework"
- "They represent code as data for analysis and translation"
- "Compiled expressions offer near-native performance"
- "ExpressionVisitor pattern for tree modification"
- "Careful about closures and client vs server evaluation"
- "Cache compiled delegates to amortize compilation cost"

---

**TL;DR for interviews:**

**What are they:**
- Data structures representing code
- `Expression<Func<>>` vs `Func<>` (data vs executable)
- Used by EF, FluentValidation, Moq, AutoMapper

**Why they matter:**
- Enable LINQ to SQL/NoSQL translation
- Allow runtime code analysis
- Build dynamic queries safely (no SQL injection)
- High performance when compiled and cached

**Key operations:**
- `Expression.Parameter` - define parameters
- `Expression.Property` - access properties
- `Expression.Call` - invoke methods
- `Expression.Lambda` - create lambda
- `.Compile()` - convert to executable delegate

**Common uses:**
- Dynamic filtering from user input
- Property mappers (AutoMapper-style)
- Validation frameworks
- Testing frameworks (Moq verification)
- ORM query translation

**Gotchas:**
- Not all C# translates to SQL
- Compilation is expensive (cache delegates!)
- Closures capture variables, not values
- Expressions are immutable (use Visitor to modify)

Remember: Expression trees bridge compile-time type safety with runtime flexibility.
