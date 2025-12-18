# React Compiler (React 19+)

**Level:** ⚡ React 19+ Features  
**Tags:** `react-19`, `compiler`, `optimization`, `memoization`, `performance`, `forget`

---

## Why this matters

The React Compiler (formerly "React Forget") is one of the most significant changes in React 19. It automatically optimizes your components without manual memoization (useMemo, useCallback, React.memo), fundamentally changing how we write performant React code. Understanding the compiler is critical for future React development and shows you're keeping up with cutting-edge features.

**Why interviewers care:**
- React Compiler represents React's future—showing knowledge demonstrates you're forward-thinking
- Understanding automatic optimization shows deep knowledge of React's rendering model
- Compiler adoption strategies reveal architectural thinking and migration planning skills
- Knowing when compiler helps vs. doesn't helps shows nuanced understanding
- Discussions about compiler limitations test critical thinking vs. hype

**Real-world implications:**
- **Less boilerplate:** No more manual useMemo/useCallback everywhere
- **Better performance by default:** Compiler optimizes automatically
- **Easier onboarding:** Junior developers don't need to understand memoization
- **Fewer bugs:** No more missing dependency arrays or stale closures
- **Faster development:** Write code naturally, compiler optimizes
- **Migration path:** Gradual adoption in existing codebases

**What React Compiler does:**
- Automatically memoizes components and values
- Optimizes re-renders without manual intervention
- Ensures correct dependency tracking (no missing dependencies)
- Transforms your code at build time
- Works with existing React code (mostly)

**What you must know:**
- What the React Compiler is and what it does
- How it differs from manual memoization
- When it's enabled and how to adopt it
- Limitations and edge cases
- How to write compiler-friendly code
- Migration strategy from manual memoization

**Interview red flags:** Thinking the compiler solves all performance issues, or dismissing it as "just automatic memoization" without understanding the implications.

---

## Core Ideas

### 1. **What is the React Compiler?**

**Before React Compiler (React ≤18):**

```jsx
// Manual optimization required
const ExpensiveList = React.memo(({ items, onSelect }) => {
  // Need useMemo to prevent recalculation
  const sortedItems = useMemo(() => {
    return items.sort((a, b) => a.name.localeCompare(b.name));
  }, [items]);

  // Need useCallback to prevent re-creating function
  const handleClick = useCallback((item) => {
    onSelect(item);
  }, [onSelect]);

  return (
    <ul>
      {sortedItems.map(item => (
        <ExpensiveItem 
          key={item.id} 
          item={item} 
          onClick={handleClick}  // Stable reference
        />
      ))}
    </ul>
  );
});

const ExpensiveItem = React.memo(({ item, onClick }) => {
  return <li onClick={() => onClick(item)}>{item.name}</li>;
});
```

**With React Compiler (React 19+):**

```jsx
// Compiler handles optimization automatically
function ExpensiveList({ items, onSelect }) {
  // No useMemo needed - compiler memoizes automatically
  const sortedItems = items.sort((a, b) => a.name.localeCompare(b.name));

  // No useCallback needed - compiler stabilizes function
  const handleClick = (item) => {
    onSelect(item);
  };

  return (
    <ul>
      {sortedItems.map(item => (
        <ExpensiveItem 
          key={item.id} 
          item={item} 
          onClick={handleClick}  // Automatically stable
        />
      ))}
    </ul>
  );
}

// No React.memo needed - compiler optimizes
function ExpensiveItem({ item, onClick }) {
  return <li onClick={() => onClick(item)}>{item.name}</li>;
}
```

**Key difference:** Write code naturally, compiler adds optimization.

---

### 2. **How the Compiler Works**

**Compilation phases:**

1. **Analysis:** Compiler analyzes your component code
2. **Dependency tracking:** Identifies which values depend on which props/state
3. **Memoization insertion:** Automatically adds memoization where needed
4. **Code transformation:** Outputs optimized JavaScript

**Example transformation:**

```jsx
// Your code (input)
function Counter({ initialCount }) {
  const [count, setCount] = useState(initialCount);
  const doubleCount = count * 2;
  
  return (
    <div>
      <p>Count: {count}</p>
      <p>Double: {doubleCount}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}

// Compiler output (conceptual - simplified)
function Counter({ initialCount }) {
  const [count, setCount] = useState(initialCount);
  
  // Compiler memoizes this calculation
  const doubleCount = useMemo(() => count * 2, [count]);
  
  // Compiler memoizes this callback
  const handleClick = useCallback(() => setCount(count + 1), [count]);
  
  // Compiler memoizes JSX
  return useMemo(() => (
    <div>
      <p>Count: {count}</p>
      <p>Double: {doubleCount}</p>
      <button onClick={handleClick}>Increment</button>
    </div>
  ), [count, doubleCount, handleClick]);
}
```

**Note:** This is conceptual. The actual compiler is more sophisticated.

---

### 3. **Enabling the React Compiler**

**Installation (React 19+):**

```bash
npm install react@19 react-dom@19
npm install babel-plugin-react-compiler --save-dev
```

**Babel configuration:**

```js
// babel.config.js
module.exports = {
  plugins: [
    ['babel-plugin-react-compiler', {
      // Options
      compilationMode: 'annotation', // or 'all'
    }]
  ]
};
```

**Compilation modes:**

1. **`all`** - Compiles all components (recommended for new projects)
   ```js
   { compilationMode: 'all' }
   ```

2. **`annotation`** - Only compile marked components (for gradual migration)
   ```jsx
   'use memo';  // Directive to enable compiler for this component
   
   function MyComponent() {
     // This component will be compiled
     return <div>Hello</div>;
   }
   ```

**Vite configuration:**

```js
// vite.config.js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [
    react({
      babel: {
        plugins: [
          ['babel-plugin-react-compiler', { compilationMode: 'all' }]
        ]
      }
    })
  ]
});
```

**Next.js configuration (App Router):**

```js
// next.config.js
module.exports = {
  experimental: {
    reactCompiler: true  // Enable React Compiler
  }
};
```

---

### 4. **What Gets Optimized**

**Automatic memoization:**

```jsx
function ProductList({ products, category }) {
  // ✅ Compiler memoizes this filter operation
  const filtered = products.filter(p => p.category === category);
  
  // ✅ Compiler memoizes this map operation
  const formatted = filtered.map(p => ({
    ...p,
    displayName: `${p.name} - $${p.price}`
  }));
  
  // ✅ Compiler stabilizes this callback
  const handleClick = (product) => {
    console.log('Clicked:', product);
  };
  
  return (
    <ul>
      {formatted.map(product => (
        <li key={product.id} onClick={() => handleClick(product)}>
          {product.displayName}
        </li>
      ))}
    </ul>
  );
}

// Before compiler: Would need useMemo for filtered/formatted, useCallback for handleClick
// With compiler: Everything is automatically optimized
```

**Component re-renders:**

```jsx
// Child component automatically optimized (no React.memo needed)
function ChildComponent({ name, count }) {
  return <div>{name}: {count}</div>;
}

function ParentComponent() {
  const [count, setCount] = useState(0);
  const name = "Counter";  // ✅ Compiler ensures stable reference
  
  return (
    <div>
      <ChildComponent name={name} count={count} />
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}

// Before: ChildComponent would re-render even if name doesn't change
// With compiler: Only re-renders when count changes
```

---

### 5. **Compiler-Friendly Code Patterns**

**✅ Good: Pure functions**

```jsx
function UserCard({ user }) {
  // Pure transformation - compiler can optimize
  const displayName = formatName(user.firstName, user.lastName);
  
  return <div>{displayName}</div>;
}

function formatName(first, last) {
  return `${first} ${last}`;
}
```

**✅ Good: Declarative logic**

```jsx
function TaskList({ tasks }) {
  // Declarative filtering - compiler optimizes
  const completedTasks = tasks.filter(t => t.completed);
  const pendingTasks = tasks.filter(t => !t.completed);
  
  return (
    <>
      <h2>Completed: {completedTasks.length}</h2>
      <h2>Pending: {pendingTasks.length}</h2>
    </>
  );
}
```

**⚠️ Caution: Side effects in render**

```jsx
function Component() {
  // Compiler may not optimize side effects correctly
  let externalValue;
  externalValue = someGlobalValue;  // Side effect - use useEffect instead
  
  return <div>{externalValue}</div>;
}
```

**⚠️ Caution: Mutations**

```jsx
function Component({ items }) {
  // BAD: Mutating input (compiler assumes immutability)
  items.sort();  // ❌ Mutates array
  
  // GOOD: Create new array
  const sorted = [...items].sort();  // ✅ Immutable
  
  return <List items={sorted} />;
}
```

---

### 6. **Migration Strategy**

**Gradual adoption:**

```jsx
// Step 1: Start with annotation mode
// babel.config.js
{
  plugins: [
    ['babel-plugin-react-compiler', { compilationMode: 'annotation' }]
  ]
}

// Step 2: Mark components individually
'use memo';
function OptimizedComponent() {
  // This component uses compiler
  return <div>Optimized</div>;
}

function UnoptimizedComponent() {
  // This component uses manual optimization (for now)
  const value = useMemo(() => expensive(), []);
  return <div>{value}</div>;
}

// Step 3: Gradually add 'use memo' to more components

// Step 4: Once confident, switch to 'all' mode
// { compilationMode: 'all' }

// Step 5: Remove manual useMemo/useCallback/React.memo
```

**Removing manual memoization:**

```jsx
// Before migration
const Component = React.memo(({ data }) => {
  const processed = useMemo(() => {
    return data.map(d => d.value * 2);
  }, [data]);
  
  const handleClick = useCallback(() => {
    console.log('clicked');
  }, []);
  
  return <div onClick={handleClick}>{processed.join(',')}</div>;
});

// After migration (with compiler enabled)
function Component({ data }) {
  const processed = data.map(d => d.value * 2);  // No useMemo
  
  const handleClick = () => {
    console.log('clicked');  // No useCallback
  };
  
  return <div onClick={handleClick}>{processed.join(',')}</div>;
}
// No React.memo wrapper needed
```

---

### 7. **Limitations and Edge Cases**

**What compiler DOESN'T optimize:**

```jsx
// ❌ Non-React code (plain JS)
function notAComponent() {
  return { value: 123 };  // Not compiled
}

// ❌ Dynamic code evaluation
function Component() {
  const code = 'return 42';
  const result = eval(code);  // Compiler can't optimize
  return <div>{result}</div>;
}

// ❌ Refs with mutations
function Component() {
  const ref = useRef({ count: 0 });
  ref.current.count++;  // Mutation - compiler can't track
  return <div>{ref.current.count}</div>;
}
```

**When manual optimization is still needed:**

```jsx
// Expensive computation at module level
const EXPENSIVE_CONSTANT = computeExpensiveValue();  // Runs on import

// Better: Lazy initialization
let cachedValue;
function getExpensiveValue() {
  if (!cachedValue) {
    cachedValue = computeExpensiveValue();
  }
  return cachedValue;
}

// Or use React cache (React 19)
import { cache } from 'react';

const getExpensiveValue = cache(() => {
  return computeExpensiveValue();
});
```

**Compiler opt-out:**

```jsx
'use no memo';  // Disable compiler for this component

function ManuallyOptimizedComponent() {
  // Use manual optimization here if needed
  const value = useMemo(() => expensive(), []);
  return <div>{value}</div>;
}
```

---

## Common Pitfalls

### ❌ **Pitfall 1: Mixing manual and automatic optimization**

```jsx
// BAD: Compiler optimizes, but you also add useMemo
function Component({ data }) {
  // Compiler already memoizes this, useMemo is redundant
  const processed = useMemo(() => data.map(d => d * 2), [data]);
  return <div>{processed.join(',')}</div>;
}

// GOOD: Let compiler handle it
function Component({ data }) {
  const processed = data.map(d => d * 2);
  return <div>{processed.join(',')}</div>;
}
```

---

### ❌ **Pitfall 2: Assuming compiler fixes all performance issues**

```jsx
// BAD: Compiler can't fix algorithmic inefficiency
function Component({ items }) {
  // O(n²) - slow even with compiler
  const duplicates = items.filter((item, i) => 
    items.findIndex(x => x.id === item.id) !== i
  );
  
  return <div>{duplicates.length} duplicates</div>;
}

// GOOD: Fix algorithm first
function Component({ items }) {
  // O(n) - efficient algorithm
  const seen = new Set();
  const duplicates = items.filter(item => {
    if (seen.has(item.id)) return true;
    seen.add(item.id);
    return false;
  });
  
  return <div>{duplicates.length} duplicates</div>;
}
```

---

### ❌ **Pitfall 3: Mutating props or state**

```jsx
// BAD: Compiler assumes immutability
function Component({ items }) {
  items.sort();  // ❌ Mutates prop - breaks compiler assumptions
  return <List items={items} />;
}

// GOOD: Immutable operations
function Component({ items }) {
  const sorted = [...items].sort();  // ✅ New array
  return <List items={sorted} />;
}
```

---

## Quick Self-Check

✅ **You understand React Compiler if you can:**

1. Explain what the React Compiler does
2. Describe how it differs from manual memoization
3. Enable the compiler in a React project
4. Identify code patterns that are compiler-friendly
5. Explain when manual optimization is still needed
6. Plan a migration strategy from manual to automatic optimization
7. Understand compiler limitations
8. Write code that the compiler can optimize effectively

---

## Interview Questions to Practice

**Beginner:**
1. What is the React Compiler?
2. What manual optimization hooks does it replace?
3. How do you enable the React Compiler in a project?

**Intermediate:**
4. How does the React Compiler decide what to memoize?
5. What's the difference between 'all' and 'annotation' compilation modes?
6. When would you still need manual optimization with the compiler?
7. How would you migrate an existing app to use the compiler?

**Advanced:**
8. What are the limitations of the React Compiler?
9. How does the compiler handle side effects?
10. When would you use 'use no memo' to opt out of compilation?

---

## Further Practice

**Experiments:**
1. **Enable compiler** in a small project and measure performance
2. **Migrate** a component from manual memoization to compiler
3. **Compare** bundle size with and without compiler
4. **Test** edge cases (refs, side effects, mutations)
5. **Profile** components before and after compiler

**Resources:**
- [React Compiler documentation](https://react.dev/learn/react-compiler)
- [React Compiler playground](https://playground.react.dev/)
- [Babel plugin docs](https://www.npmjs.com/package/babel-plugin-react-compiler)

---

## Summary

**What it is:**
- Build-time optimization tool (Babel plugin)
- Automatically adds memoization where needed
- Analyzes dependencies and optimizes re-renders
- Replaces manual useMemo, useCallback, React.memo

**How to enable:**
- Install `babel-plugin-react-compiler`
- Configure Babel/Vite/Next.js
- Choose 'all' or 'annotation' mode
- Gradually migrate existing code

**Benefits:**
- Less boilerplate (no manual optimization)
- Better performance by default
- Fewer bugs (automatic dependency tracking)
- Easier code maintenance

**Limitations:**
- Doesn't optimize non-React code
- Assumes immutability (no mutations)
- Can't fix bad algorithms
- Doesn't handle all edge cases

**Best practices:**
- Write pure, immutable code
- Let compiler handle optimization
- Remove manual memoization after enabling compiler
- Use 'annotation' mode for gradual adoption
- Profile to verify improvements

**Remember:** The compiler is a tool, not magic. Write good code first, let the compiler optimize it.
