# Concurrent Features

**Level:** 🔥 Advanced  
**Tags:** `concurrent-mode`, `useTransition`, `useDeferredValue`, `startTransition`, `performance`, `react-18`

---

## Why this matters

Concurrent features are React 18's most significant innovation, fundamentally changing how React handles expensive updates. Interviewers use concurrent features to assess your understanding of modern React performance optimization and ability to build responsive UIs that stay interactive even during heavy computation. This knowledge separates developers who just know the API from those who understand React's rendering model.

**Why interviewers care:**
- Concurrent features represent the future of React performance optimization
- Shows understanding of React's rendering engine and scheduling
- Tests ability to identify and fix UI blocking/janky interactions
- Demonstrates knowledge of modern React (18+) vs. legacy patterns
- Senior roles expect ability to optimize complex, data-heavy UIs
- Understanding concurrency reveals deep React internals knowledge

**Real-world implications:**
- **User experience:** Keeps UI responsive during heavy updates (search, filtering)
- **Perceived performance:** Shows outdated UI while new UI loads (instant feedback)
- **Resource management:** Prioritizes important updates over less critical ones
- **Mobile performance:** Critical for low-powered devices with slow CPUs
- **Complex UIs:** Makes real-time dashboards, editors, and data-heavy apps usable
- **SEO and metrics:** Better Core Web Vitals (INP - Interaction to Next Paint)

**Common performance problems concurrent features solve:**
- Input lag (typing in search while filtering large lists)
- UI freezing during expensive renders
- Janky scrolling while data updates
- Slow navigation transitions
- Heavy computation blocking interactions

**What you must know:**
- useTransition (mark state updates as non-urgent)
- useDeferredValue (defer expensive derived values)
- startTransition (non-hook version of useTransition)
- Concurrent rendering basics
- When to use each feature
- Performance trade-offs

**Interview red flags:** Not knowing about React 18+ concurrent features, confusing useTransition with useMemo/useCallback, or using setTimeout/requestAnimationFrame hacks instead of proper concurrent APIs.

---

## Core Ideas

### 1. **Understanding Concurrent Rendering**

**Legacy (Synchronous) Rendering - React 17 and earlier:**

```jsx
// Every setState blocks the browser until render completes
function SearchResults() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);

  const handleSearch = (e) => {
    const newQuery = e.target.value;
    setQuery(newQuery);  // Blocks!
    
    // Expensive computation (filters 10,000 items)
    const filtered = items.filter(item => 
      item.name.includes(newQuery) ||
      item.description.includes(newQuery) ||
      item.tags.some(tag => tag.includes(newQuery))
    );
    setResults(filtered);  // Blocks again!
  };

  return (
    <>
      <input value={query} onChange={handleSearch} />
      {/* User can't type smoothly - input lags! */}
      {results.map(item => <Item key={item.id} {...item} />)}
    </>
  );
}
```

**Problem:** Input field freezes while filtering runs. On slower devices, typing feels laggy.

**Concurrent Rendering - React 18+:**

React can now **interrupt** low-priority work to handle high-priority updates:

```
User types → React starts filtering → User types again → React:
1. Pauses filtering work
2. Updates input immediately (high priority)
3. Resumes filtering with new value
```

**Key insight:** Not all updates are equally important. Input updates are urgent (user is waiting). Display updates can wait (user won't notice 50ms delay).

---

### 2. **useTransition: Mark Updates as Non-Urgent**

**Problem:** Updating input state also triggers expensive derived state:

```jsx
// BAD: Input lags because results update is blocking
function SearchableList({ items }) {
  const [query, setQuery] = useState('');
  
  const filteredItems = items.filter(item =>
    item.name.toLowerCase().includes(query.toLowerCase())
  );  // Expensive if items.length = 10,000

  return (
    <>
      <input 
        value={query}
        onChange={(e) => setQuery(e.target.value)}  // Lags!
      />
      <List items={filteredItems} />
    </>
  );
}
```

**Solution: useTransition**

```jsx
import { useState, useTransition } from 'react';

function SearchableList({ items }) {
  const [query, setQuery] = useState('');
  const [isPending, startTransition] = useTransition();

  const handleChange = (e) => {
    const newQuery = e.target.value;
    
    // Urgent: Update input immediately
    setQuery(newQuery);
    
    // Non-urgent: Update results in background
    startTransition(() => {
      setFilteredQuery(newQuery);
    });
  };

  const [filteredQuery, setFilteredQuery] = useState('');
  const filteredItems = items.filter(item =>
    item.name.toLowerCase().includes(filteredQuery.toLowerCase())
  );

  return (
    <>
      <input value={query} onChange={handleChange} />
      {isPending && <div>Updating results...</div>}
      <List items={filteredItems} />
    </>
  );
}
```

**What happens:**
1. User types → `query` updates immediately (input stays responsive)
2. `startTransition` marks `setFilteredQuery` as low priority
3. React renders expensive `filteredItems` in background
4. If user types again, React **interrupts** old render and starts new one
5. `isPending` = true while background render is happening

**Real-world example - Tabbed interface:**

```jsx
function TabContainer() {
  const [tab, setTab] = useState('home');
  const [isPending, startTransition] = useTransition();

  const selectTab = (nextTab) => {
    startTransition(() => {
      setTab(nextTab);  // Heavy render (thousands of components)
    });
  };

  return (
    <>
      <TabButton onClick={() => selectTab('home')} isActive={tab === 'home'}>
        Home {isPending && tab === 'home' && '⏳'}
      </TabButton>
      <TabButton onClick={() => selectTab('posts')} isActive={tab === 'posts'}>
        Posts {isPending && tab === 'posts' && '⏳'}
      </TabButton>
      <TabButton onClick={() => selectTab('profile')} isActive={tab === 'profile'}>
        Profile {isPending && tab === 'profile' && '⏳'}
      </TabButton>

      {/* Tabs stay clickable even during heavy render */}
      {tab === 'home' && <Home />}
      {tab === 'posts' && <Posts />}  {/* 10,000 posts */}
      {tab === 'profile' && <Profile />}
    </>
  );
}
```

**Without useTransition:** Clicking "Posts" freezes the UI for 200ms while React renders 10,000 posts. User can't click other tabs.

**With useTransition:** User can click tabs rapidly. React shows loading indicator and keeps UI responsive.

---

### 3. **useDeferredValue: Defer Expensive Derived Values**

**Problem:** Component re-renders on every keystroke, even though derived value is expensive:

```jsx
// BAD: SlowList re-renders on EVERY keystroke
function App() {
  const [text, setText] = useState('');
  
  return (
    <>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <SlowList text={text} />  {/* Renders 10,000 items */}
    </>
  );
}

function SlowList({ text }) {
  const items = useMemo(() => {
    // Even with useMemo, this runs on every keystroke
    return createItems(10000).filter(item => item.name.includes(text));
  }, [text]);
  
  return items.map(item => <div key={item.id}>{item.name}</div>);
}
```

**Solution: useDeferredValue**

```jsx
import { useState, useDeferredValue, memo } from 'react';

function App() {
  const [text, setText] = useState('');
  const deferredText = useDeferredValue(text);
  
  return (
    <>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      {/* SlowList uses deferredText (lags behind real text) */}
      <SlowList text={deferredText} />
    </>
  );
}

// IMPORTANT: memo() prevents re-render when text !== deferredText
const SlowList = memo(function SlowList({ text }) {
  const items = useMemo(() => {
    return createItems(10000).filter(item => item.name.includes(text));
  }, [text]);
  
  return items.map(item => <div key={item.id}>{item.name}</div>);
});
```

**What happens:**
1. User types "a" → `text = "a"`, `deferredText = ""`
2. React renders `<input>` immediately
3. `SlowList` still shows old results (`deferredText = ""`)
4. React schedules low-priority update: `deferredText = "a"`
5. If user types "ab", React **cancels** pending update
6. Repeat from step 1 with new value

**Visual indicator (optional but recommended):**

```jsx
function App() {
  const [text, setText] = useState('');
  const deferredText = useDeferredValue(text);
  const isStale = text !== deferredText;
  
  return (
    <>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <div style={{ opacity: isStale ? 0.5 : 1, transition: 'opacity 200ms' }}>
        <SlowList text={deferredText} />
      </div>
    </>
  );
}
```

---

### 4. **useTransition vs. useDeferredValue: When to Use Each**

**useTransition:**
- **You control the state update**
- Mark specific setState calls as non-urgent
- Get `isPending` flag for loading UI
- Example: Button clicks, tab changes, form submissions

```jsx
const [isPending, startTransition] = useTransition();

const handleClick = () => {
  startTransition(() => {
    setPage(nextPage);  // Your code
  });
};
```

**useDeferredValue:**
- **You don't control the state** (passed as prop)
- Defer expensive derived values
- No explicit pending flag (compare values)
- Example: Search input from parent, expensive filtering

```jsx
function Child({ query }) {  // query comes from parent
  const deferredQuery = useDeferredValue(query);
  const isStale = query !== deferredQuery;
  
  const results = expensiveFilter(deferredQuery);
  
  return <div style={{ opacity: isStale ? 0.5 : 1 }}>{results}</div>;
}
```

**Comparison:**

| Feature | useTransition | useDeferredValue |
|---------|---------------|------------------|
| **Who controls state?** | You (have setState) | Parent (receive prop) |
| **What you wrap** | setState call | Value |
| **Pending indicator** | `isPending` flag | Compare old vs. new |
| **Use case** | User actions | Expensive rendering |

**They work together:**

```jsx
function Parent() {
  const [query, setQuery] = useState('');
  const [isPending, startTransition] = useTransition();
  
  const handleChange = (e) => {
    const value = e.target.value;
    setQuery(value);  // Urgent
    
    startTransition(() => {
      setSearchQuery(value);  // Non-urgent
    });
  };
  
  const [searchQuery, setSearchQuery] = useState('');
  
  return (
    <>
      <input value={query} onChange={handleChange} />
      {isPending && '⏳'}
      <SearchResults query={searchQuery} />
    </>
  );
}

function SearchResults({ query }) {
  // Additional deferred value for extra-expensive child
  const deferredQuery = useDeferredValue(query);
  return <HeavyList query={deferredQuery} />;
}
```

---

### 5. **startTransition: Non-Hook Version**

**Use startTransition when:**
- Outside React component (utility function, event handler)
- Don't need `isPending` flag
- One-time transition (not tied to component lifecycle)

```jsx
import { startTransition } from 'react';

// Outside component
function updateGlobalState(newValue) {
  startTransition(() => {
    globalStore.set(newValue);
  });
}

// In component
function Component() {
  const handleClick = () => {
    // No isPending needed
    startTransition(() => {
      setHeavyState(computeExpensiveValue());
    });
  };
}
```

---

### 6. **Real-World Examples**

**Example 1: Autocomplete with 10,000 options**

```jsx
function Autocomplete({ options }) {
  const [input, setInput] = useState('');
  const [query, setQuery] = useState('');
  const [isPending, startTransition] = useTransition();

  const handleChange = (e) => {
    const value = e.target.value;
    setInput(value);  // Immediate
    
    startTransition(() => {
      setQuery(value);  // Deferred
    });
  };

  const filteredOptions = options.filter(option =>
    option.toLowerCase().includes(query.toLowerCase())
  );

  return (
    <>
      <input 
        value={input} 
        onChange={handleChange}
        placeholder="Type to search..."
      />
      {isPending ? (
        <div>Loading...</div>
      ) : (
        <ul>
          {filteredOptions.map(option => (
            <li key={option}>{option}</li>
          ))}
        </ul>
      )}
    </>
  );
}
```

**Example 2: Data table with filters**

```jsx
function DataTable({ data }) {
  const [filters, setFilters] = useState({});
  const deferredFilters = useDeferredValue(filters);

  const filteredData = useMemo(() => {
    return data.filter(row => {
      return Object.entries(deferredFilters).every(([key, value]) => {
        if (!value) return true;
        return row[key].toString().toLowerCase().includes(value.toLowerCase());
      });
    });
  }, [data, deferredFilters]);

  const isStale = filters !== deferredFilters;

  return (
    <>
      <FilterInputs filters={filters} setFilters={setFilters} />
      <div style={{ opacity: isStale ? 0.6 : 1 }}>
        <table>
          {filteredData.map(row => <TableRow key={row.id} data={row} />)}
        </table>
      </div>
    </>
  );
}
```

**Example 3: Router navigation**

```jsx
function Router() {
  const [page, setPage] = useState('home');
  const [isPending, startTransition] = useTransition();

  const navigate = (nextPage) => {
    startTransition(() => {
      setPage(nextPage);
    });
  };

  return (
    <>
      <nav style={{ opacity: isPending ? 0.7 : 1 }}>
        <button onClick={() => navigate('home')}>Home</button>
        <button onClick={() => navigate('posts')}>Posts</button>
        <button onClick={() => navigate('settings')}>Settings</button>
      </nav>
      
      {isPending && <LoadingBar />}
      
      {page === 'home' && <HomePage />}
      {page === 'posts' && <PostsPage />}  {/* Heavy */}
      {page === 'settings' && <SettingsPage />}
    </>
  );
}
```

---

### 7. **Performance Patterns and Best Practices**

**Pattern 1: Debouncing with useTransition (better than setTimeout)**

```jsx
// OLD: Manual debouncing
function SearchOld() {
  const [query, setQuery] = useState('');
  
  useEffect(() => {
    const timer = setTimeout(() => {
      performSearch(query);
    }, 300);
    
    return () => clearTimeout(timer);
  }, [query]);
}

// NEW: useTransition (React handles optimization)
function SearchNew() {
  const [input, setInput] = useState('');
  const [query, setQuery] = useState('');
  const [isPending, startTransition] = useTransition();
  
  const handleChange = (e) => {
    setInput(e.target.value);
    startTransition(() => {
      setQuery(e.target.value);
    });
  };
  
  // React automatically batches rapid updates
}
```

**Pattern 2: Progressive rendering**

```jsx
function HeavyList({ items }) {
  const [displayCount, setDisplayCount] = useState(50);
  const [isPending, startTransition] = useTransition();

  const loadMore = () => {
    startTransition(() => {
      setDisplayCount(count => count + 50);
    });
  };

  return (
    <>
      {items.slice(0, displayCount).map(item => <Item key={item.id} {...item} />)}
      <button onClick={loadMore} disabled={isPending}>
        {isPending ? 'Loading...' : 'Load More'}
      </button>
    </>
  );
}
```

**Pattern 3: Optimistic UI with transitions**

```jsx
function TodoList() {
  const [todos, setTodos] = useState([]);
  const [isPending, startTransition] = useTransition();

  const addTodo = async (text) => {
    const tempId = Date.now();
    
    // Optimistic update
    setTodos(prev => [...prev, { id: tempId, text, pending: true }]);
    
    startTransition(async () => {
      const newTodo = await api.createTodo(text);
      setTodos(prev => prev.map(todo => 
        todo.id === tempId ? newTodo : todo
      ));
    });
  };

  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id} style={{ opacity: todo.pending ? 0.5 : 1 }}>
          {todo.text}
        </li>
      ))}
    </ul>
  );
}
```

---

## Common Pitfalls

### ❌ **Pitfall 1: Forgetting memo() with useDeferredValue**

```jsx
// BAD: Child re-renders on every keystroke anyway!
function Parent() {
  const [text, setText] = useState('');
  const deferredText = useDeferredValue(text);
  
  return <ExpensiveChild text={deferredText} />;
}

function ExpensiveChild({ text }) {  // No memo!
  // Re-renders even when text === deferredText
}

// GOOD: memo prevents unnecessary re-renders
const ExpensiveChild = memo(function ExpensiveChild({ text }) {
  // Only re-renders when text actually changes
});
```

---

### ❌ **Pitfall 2: Using useTransition for synchronous operations**

```jsx
// BAD: useTransition doesn't help with synchronous work
function Component() {
  const [isPending, startTransition] = useTransition();
  
  const handleClick = () => {
    startTransition(() => {
      // Synchronous blocking work - still blocks!
      for (let i = 0; i < 1000000000; i++) {
        Math.random();
      }
    });
  };
}

// GOOD: Move heavy work to Web Worker or optimize algorithm
```

---

### ❌ **Pitfall 3: Not showing loading indicators**

```jsx
// BAD: User doesn't know why UI is stale
function Search() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);
  
  return (
    <>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      <Results query={deferredQuery} />  {/* No feedback! */}
    </>
  );
}

// GOOD: Show visual feedback
function Search() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);
  const isStale = query !== deferredQuery;
  
  return (
    <>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      <div style={{ opacity: isStale ? 0.5 : 1, transition: 'opacity 200ms' }}>
        <Results query={deferredQuery} />
      </div>
    </>
  );
}
```

---

## Quick Self-Check

✅ **You understand concurrent features if you can:**

1. Explain why input fields lag without concurrent features
2. Use useTransition to mark updates as non-urgent
3. Use useDeferredValue for expensive derived values
4. Know when to use useTransition vs. useDeferredValue
5. Show loading indicators during transitions
6. Combine memo() with useDeferredValue correctly
7. Use startTransition outside components
8. Identify blocking synchronous work
9. Implement progressive rendering
10. Build responsive search with large datasets

---

## Interview Questions to Practice

**Beginner:**
1. What problem do concurrent features solve?
2. What does useTransition do?
3. What is the difference between urgent and non-urgent updates?

**Intermediate:**
4. When would you use useTransition vs. useDeferredValue?
5. Why must you use memo() with useDeferredValue?
6. How do you show loading indicators with useTransition?
7. Explain how React interrupts low-priority work

**Advanced:**
8. How would you build a responsive search over 100,000 items?
9. Explain the rendering timeline with concurrent features
10. How do concurrent features relate to Suspense?

---

## Further Practice

**Build these:**
1. **Search with autocomplete** (10,000 options, stay responsive)
2. **Data table** with sortable/filterable columns (1,000 rows)
3. **Tabbed interface** with heavy tabs (measure performance)
4. **Real-time dashboard** updating every second
5. **Infinite scroll** with progressive rendering

**Measure performance:**
- Use React DevTools Profiler
- Measure interaction times (INP)
- Compare with/without concurrent features

**Resources:**
- [React Docs - useTransition](https://react.dev/reference/react/useTransition)
- [React Docs - useDeferredValue](https://react.dev/reference/react/useDeferredValue)
- [Concurrent Rendering](https://react.dev/blog/2022/03/29/react-v18#what-is-concurrent-react)

---

## Summary

**Concurrent features solve:**
- Input lag during expensive updates
- Frozen UI during heavy renders
- Poor user experience with slow interactions

**useTransition:**
- You control state update
- Mark setState as non-urgent
- Get `isPending` flag
- Use for: clicks, navigation, form submissions

**useDeferredValue:**
- You receive value from parent
- Defer expensive derived values
- Must use with `memo()`
- Use for: search input, filtering, expensive rendering

**Best practices:**
- Always show loading indicators (`isPending` or opacity)
- Use `memo()` with `useDeferredValue`
- Don't use for synchronous blocking work
- Combine with Suspense for async operations

**Remember:** Concurrent features don't make code faster—they make UIs more responsive by prioritizing what users see and interact with first.
