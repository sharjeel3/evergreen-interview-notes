# Performance Optimization

**Level:** 🚦 Intermediate  
**Tags:** `performance`, `memo`, `useMemo`, `useCallback`, `optimization`, `interview`

---

## Why this matters

Performance optimization is a favorite interview topic because it tests:

- understanding of React's rendering behavior
- when and why to optimize
- trade-offs between optimization and code complexity

**Interview red flag:** premature optimization or not knowing when React re-renders

---

## Core Ideas

### 1. **React re-renders when state or props change**

**Re-render triggers:**
- Component's own state changes (`setState`)
- Props passed to component change
- Parent component re-renders (by default, all children re-render)
- Context value changes (all consumers re-render)

**Mental model:**  
"Re-render ≠ slow. Only optimize when profiling shows problems."

---

### 2. **React.memo prevents unnecessary re-renders**

```jsx
const MyComponent = React.memo(function MyComponent(props) {
  return <div>{props.value}</div>;
});
```

**How it works:**
- Shallow compares props
- If props haven't changed, skips re-render
- Similar to `PureComponent` for class components

**When to use:** Component renders often with same props.

---

### 3. **useMemo caches expensive calculations**

```jsx
const expensiveResult = useMemo(() => {
  return expensiveCalculation(a, b);
}, [a, b]);
```

**Purpose:** Avoid recalculating on every render.

**When to use:**
- Expensive computations (filtering large lists, complex math)
- Creating objects/arrays passed to memoized children

---

### 4. **useCallback caches function references**

```jsx
const handleClick = useCallback(() => {
  doSomething(a, b);
}, [a, b]);
```

**Purpose:** Keep same function reference across renders.

**When to use:**
- Passing callbacks to memoized children
- Dependencies in useEffect
- Optimization for expensive event handlers

---

### 5. **Performance measurement over guessing**

```jsx
import { Profiler } from 'react';

<Profiler id="MyComponent" onRender={onRenderCallback}>
  <MyComponent />
</Profiler>
```

**Tools:**
- React DevTools Profiler
- Chrome DevTools Performance tab
- `console.time()` / `console.timeEnd()`

**Rule:** Measure first, optimize second.

---

## Examples

### Example 1 — React.memo

```jsx
// ❌ Re-renders even if count didn't change
function ExpensiveChild({ count }) {
  console.log('Rendering ExpensiveChild');
  return <div>{count}</div>;
}

function Parent() {
  const [text, setText] = useState('');
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <input value={text} onChange={e => setText(e.target.value)} />
      <ExpensiveChild count={count} />
    </div>
  );
}
// Every keystroke re-renders ExpensiveChild (even though count unchanged)

// ✅ Memoized: only re-renders when count changes
const ExpensiveChild = React.memo(function ExpensiveChild({ count }) {
  console.log('Rendering ExpensiveChild');
  return <div>{count}</div>;
});
```

---

### Example 2 — useMemo for expensive calculations

```jsx
function FilterList({ items, filterText }) {
  // ❌ Filters on every render (even if items/filterText unchanged)
  const filteredItems = items.filter(item => 
    item.name.includes(filterText)
  );
  
  return <ul>{filteredItems.map(...)}</ul>;
}

// ✅ Memoized: only recalculates when items or filterText change
function FilterList({ items, filterText }) {
  const filteredItems = useMemo(() => {
    console.log('Filtering...');
    return items.filter(item => item.name.includes(filterText));
  }, [items, filterText]);
  
  return <ul>{filteredItems.map(...)}</ul>;
}
```

---

### Example 3 — useCallback with memo child

```jsx
// ❌ New function every render → memo doesn't help
function Parent() {
  const [count, setCount] = useState(0);
  
  const handleClick = () => {  // New reference every render
    console.log('Clicked');
  };
  
  return <MemoizedChild onClick={handleClick} />;
}
// MemoizedChild re-renders every time (handleClick changes)

// ✅ Memoized callback → stable reference
function Parent() {
  const [count, setCount] = useState(0);
  
  const handleClick = useCallback(() => {
    console.log('Clicked');
  }, []);  // Same reference across renders
  
  return <MemoizedChild onClick={handleClick} />;
}

const MemoizedChild = React.memo(function Child({ onClick }) {
  console.log('Rendering Child');
  return <button onClick={onClick}>Click</button>;
});
```

---

### Example 4 — Code splitting with lazy loading

```jsx
import { lazy, Suspense } from 'react';

// ❌ Loads all components upfront
import HeavyComponent from './HeavyComponent';

// ✅ Loads only when needed
const HeavyComponent = lazy(() => import('./HeavyComponent'));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <HeavyComponent />
    </Suspense>
  );
}
```

---

### Example 5 — List virtualization

```jsx
// ❌ Renders 10,000 items (slow!)
function HugeList({ items }) {
  return (
    <ul>
      {items.map(item => <li key={item.id}>{item.name}</li>)}
    </ul>
  );
}

// ✅ Only renders visible items
import { FixedSizeList } from 'react-window';

function VirtualizedList({ items }) {
  return (
    <FixedSizeList
      height={600}
      itemCount={items.length}
      itemSize={35}
      width='100%'
    >
      {({ index, style }) => (
        <div style={style}>{items[index].name}</div>
      )}
    </FixedSizeList>
  );
}
```

---

## Common Pitfalls

### 1. **Premature optimization**

```jsx
// ❌ Unnecessary memoization for cheap components
const SimpleText = React.memo(({ text }) => <span>{text}</span>);

// ✅ Just render it—React is fast
const SimpleText = ({ text }) => <span>{text}</span>;
```

**Rule:** Optimize when profiling shows a problem, not by default.

---

### 2. **Memoizing with unstable deps**

```jsx
// ❌ New object every render → useMemo useless
function Bad({ items }) {
  const options = { sortKey: 'name' };  // New reference every render
  
  const sorted = useMemo(() => {
    return sortItems(items, options);
  }, [items, options]);  // options always different
}

// ✅ Stable dependency
function Good({ items }) {
  const sortKey = 'name';
  
  const sorted = useMemo(() => {
    return sortItems(items, sortKey);
  }, [items, sortKey]);
}
```

---

### 3. **Forgetting keys in lists**

```jsx
// ❌ React can't optimize re-renders
{items.map(item => <Item item={item} />)}

// ✅ Stable keys help React optimize
{items.map(item => <Item key={item.id} item={item} />)}

// ❌ Using index as key (breaks on reorder/filter)
{items.map((item, i) => <Item key={i} item={item} />)}
```

---

### 4. **Inline object/array props**

```jsx
// ❌ New object every render → memo useless
<MemoChild style={{ color: 'red' }} />
<MemoChild items={[1, 2, 3]} />

// ✅ Define outside or use useMemo
const style = { color: 'red' };
const items = useMemo(() => [1, 2, 3], []);

<MemoChild style={style} items={items} />
```

---

### 5. **Over-using useCallback/useMemo**

```jsx
// ❌ Overkill: adds complexity without benefit
function Bad() {
  const handleClick = useCallback(() => {
    console.log('Clicked');
  }, []);
  
  const value = useMemo(() => 2 + 2, []);
  
  return <div onClick={handleClick}>{value}</div>;
}

// ✅ Simple is better
function Good() {
  const handleClick = () => console.log('Clicked');
  const value = 4;
  
  return <div onClick={handleClick}>{value}</div>;
}
```

---

## FAQ

**Q: Should I wrap everything in React.memo?**  
A: No. Only optimize components that:
- Render frequently
- Are expensive to render
- Often receive same props

**Q: What's the cost of useMemo/useCallback?**  
A: Memory + comparison overhead. Only use when benefit > cost.

**Q: When does React re-render?**  
A: When state/props/context change, or parent re-renders.

**Q: What's the difference between useMemo and useCallback?**  
A: `useMemo` returns memoized value. `useCallback` returns memoized function.  
`useCallback(fn, deps)` = `useMemo(() => fn, deps)`

**Q: How do I identify performance issues?**  
A: Use React DevTools Profiler. Look for:
- Frequent renders
- Long render times
- Unnecessary renders

**Q: Does React.memo do deep comparison?**  
A: No, shallow only. Pass custom comparison function for deep checks.

---

## Quick Self-Check

1. When does React re-render a component?

2. What does React.memo do?
   (Skips re-render if props unchanged)

3. When should you use useMemo?
   (Expensive calculations, object identity for memo children)

4. What's wrong with this?
   ```jsx
   <MemoChild options={{ sort: 'name' }} />
   ```
   (New object every render → memo ineffective)

5. What's the #1 rule for optimization?
   (Measure first, optimize second)

---

## Further Practice

1. **Profile:** Use React DevTools to find slow components
2. **Build:** Virtualized list with react-window
3. **Optimize:** Refactor app with unnecessary re-renders
4. **Experiment:** Measure impact of memo/useMemo/useCallback
5. **Debug:** Create and fix memo with unstable deps

---

## Key Takeaways

- Measure before optimizing (use React DevTools Profiler)
- React re-renders when state/props/context change or parent re-renders
- `React.memo` prevents re-renders when props unchanged
- `useMemo` caches expensive calculations
- `useCallback` caches function references
- Avoid inline objects/arrays as props to memoized components
- Don't over-optimize—clarity > premature optimization
- Use stable keys for lists
- Code splitting with `React.lazy` for large components
- Virtualize long lists with react-window/react-virtualized
