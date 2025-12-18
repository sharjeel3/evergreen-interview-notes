# React Suspense

**Level:** 🔥 Advanced  
**Tags:** `suspense`, `lazy-loading`, `code-splitting`, `data-fetching`, `streaming`, `react-18`

---

## Why this matters

Suspense is React's declarative way to handle asynchronous operations—from code splitting to data fetching. Interviewers use Suspense questions to assess your understanding of modern React patterns, loading states, and performance optimization. Suspense knowledge separates developers who manually manage loading states from those who leverage React's built-in async capabilities for cleaner, more maintainable code.

**Why interviewers care:**
- Suspense is the foundation of React Server Components and streaming SSR
- Shows understanding of modern React architecture (18+, 19+)
- Tests ability to optimize bundle size and loading performance
- Demonstrates knowledge of declarative vs. imperative loading patterns
- Senior roles expect experience with code splitting and lazy loading
- Critical for building fast, production-ready applications

**Real-world implications:**
- **Performance:** Code splitting reduces initial bundle size (faster page loads)
- **User experience:** Better loading states without manual useState juggling
- **SEO:** Streaming SSR improves Time to First Byte
- **Mobile:** Critical for slow networks and limited bandwidth
- **Architecture:** Cleaner component code (no loading state boilerplate)
- **Server Components:** Suspense enables RSC streaming and progressive hydration

**Common problems Suspense solves:**
- Large bundle sizes (load code on demand)
- Waterfall data fetching (parallel requests)
- Complex loading state management
- Flash of loading spinners (delay loading UI)
- Streaming HTML from server
- Progressive page rendering

**What you must know:**
- `<Suspense>` component for async boundaries
- `lazy()` for code splitting
- Data fetching with Suspense (React Query, Server Components)
- Error boundaries with Suspense
- Streaming SSR
- `startTransition` with Suspense

**Interview red flags:** Not knowing about code splitting, confusing Suspense with loading states, or thinking Suspense is only for lazy components.

---

## Core Ideas

### 1. **Suspense Basics: Declarative Loading States**

**Old way (imperative):**

```jsx
// Manual loading state management
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    setIsLoading(true);
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => {
        setUser(data);
        setIsLoading(false);
      })
      .catch(err => {
        setError(err);
        setIsLoading(false);
      });
  }, [userId]);

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  return <div>{user.name}</div>;
}
```

**New way (declarative with Suspense):**

```jsx
import { Suspense } from 'react';

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <UserProfile userId="123" />
    </Suspense>
  );
}

// Component throws a promise while loading
function UserProfile({ userId }) {
  const user = use(fetch(`/api/users/${userId}`));  // React 19
  return <div>{user.name}</div>;
}
```

**Key difference:**
- **Old:** Component manages its own loading state
- **New:** Parent declares loading boundary, component just "suspends"

---

### 2. **Code Splitting with lazy() - Most Common Use**

**Problem:** Initial bundle includes all code (slow page load):

```jsx
// BAD: Imports entire Dashboard even if user never sees it
import Dashboard from './Dashboard';
import Settings from './Settings';
import Profile from './Profile';

function App() {
  const [page, setPage] = useState('dashboard');
  
  return (
    <>
      {page === 'dashboard' && <Dashboard />}
      {page === 'settings' && <Settings />}
      {page === 'profile' && <Profile />}
    </>
  );
}
```

**Bundle size:** 500KB (dashboard: 200KB, settings: 150KB, profile: 150KB)  
**Initial load:** 500KB (user might never visit settings/profile!)

**Solution: lazy() + Suspense**

```jsx
import { lazy, Suspense } from 'react';

// Only load when needed
const Dashboard = lazy(() => import('./Dashboard'));
const Settings = lazy(() => import('./Settings'));
const Profile = lazy(() => import('./Profile'));

function App() {
  const [page, setPage] = useState('dashboard');
  
  return (
    <Suspense fallback={<div>Loading page...</div>}>
      {page === 'dashboard' && <Dashboard />}
      {page === 'settings' && <Settings />}
      {page === 'profile' && <Profile />}
    </Suspense>
  );
}
```

**Bundle size:** 
- Initial: 200KB (just dashboard)
- Settings: loads 150KB only when clicked
- Profile: loads 150KB only when clicked

**Real-world example - Route-based splitting:**

```jsx
import { lazy, Suspense } from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';

const Home = lazy(() => import('./pages/Home'));
const About = lazy(() => import('./pages/About'));
const Dashboard = lazy(() => import('./pages/Dashboard'));

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<PageLoader />}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/about" element={<About />} />
          <Route path="/dashboard" element={<Dashboard />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}

function PageLoader() {
  return (
    <div className="page-loader">
      <div className="spinner" />
      <p>Loading page...</p>
    </div>
  );
}
```

---

### 3. **Nested Suspense Boundaries**

**Multiple loading boundaries:**

```jsx
function App() {
  return (
    <div>
      <Header />
      
      {/* Boundary 1: Main content */}
      <Suspense fallback={<MainSkeleton />}>
        <MainContent />
      </Suspense>
      
      {/* Boundary 2: Sidebar */}
      <Suspense fallback={<SidebarSkeleton />}>
        <Sidebar />
      </Suspense>
    </div>
  );
}

function MainContent() {
  return (
    <div>
      <h1>Dashboard</h1>
      
      {/* Boundary 3: Nested component */}
      <Suspense fallback={<div>Loading chart...</div>}>
        <ExpensiveChart />
      </Suspense>
    </div>
  );
}
```

**Rendering timeline:**
1. Header renders immediately
2. MainSkeleton and SidebarSkeleton show
3. MainContent loads → shows "Loading chart..." for ExpensiveChart
4. Sidebar loads independently
5. ExpensiveChart loads

**Benefits:**
- Granular loading states (chart vs. entire page)
- Parallel loading (sidebar + main content)
- Better UX (show content as it becomes available)

---

### 4. **Suspense with Transitions**

**Problem:** Clicking navigation causes instant fallback (jarring):

```jsx
function App() {
  const [tab, setTab] = useState('home');
  
  return (
    <>
      <button onClick={() => setTab('posts')}>Posts</button>
      
      <Suspense fallback={<div>Loading...</div>}>
        {tab === 'home' && <Home />}
        {tab === 'posts' && <Posts />}  {/* Code-split */}
      </Suspense>
    </>
  );
}
```

**User experience:**
1. Click "Posts"
2. **Instant flash of "Loading..."**
3. Posts component loads

**Solution: useTransition to avoid flash**

```jsx
import { useState, useTransition, Suspense } from 'react';

function App() {
  const [tab, setTab] = useState('home');
  const [isPending, startTransition] = useTransition();
  
  const selectTab = (nextTab) => {
    startTransition(() => {
      setTab(nextTab);
    });
  };
  
  return (
    <>
      <button onClick={() => selectTab('posts')} disabled={isPending}>
        Posts {isPending && '⏳'}
      </button>
      
      <Suspense fallback={<div>Loading...</div>}>
        <div style={{ opacity: isPending ? 0.7 : 1 }}>
          {tab === 'home' && <Home />}
          {tab === 'posts' && <Posts />}
        </div>
      </Suspense>
    </>
  );
}
```

**User experience:**
1. Click "Posts"
2. **Old content stays visible** (no flash)
3. Button shows "⏳" (loading indicator)
4. Posts component loads → fades in

**Key insight:** `useTransition` prevents Suspense fallback from showing immediately. React keeps old UI until new UI is ready.

---

### 5. **Data Fetching with Suspense**

**React 19: use() hook**

```jsx
import { use, Suspense } from 'react';

function App() {
  return (
    <Suspense fallback={<div>Loading user...</div>}>
      <UserProfile userId="123" />
    </Suspense>
  );
}

function UserProfile({ userId }) {
  const user = use(fetchUser(userId));  // Suspends while loading
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}

// Helper function (returns promise)
function fetchUser(userId) {
  return fetch(`/api/users/${userId}`).then(res => res.json());
}
```

**How it works:**
1. `use()` receives a promise
2. If promise is pending → component **suspends** (throws promise)
3. React catches promise → shows Suspense fallback
4. Promise resolves → React re-renders component
5. `use()` returns resolved value

**React Query with Suspense:**

```jsx
import { useQuery } from '@tanstack/react-query';
import { Suspense } from 'react';

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <UserProfile userId="123" />
    </Suspense>
  );
}

function UserProfile({ userId }) {
  const { data: user } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetch(`/api/users/${userId}`).then(res => res.json()),
    suspense: true  // Enable Suspense mode
  });
  
  return <div>{user.name}</div>;
}
```

---

### 6. **Error Boundaries with Suspense**

**Suspense handles loading, Error Boundaries handle errors:**

```jsx
import { Suspense, Component } from 'react';

// Error Boundary (class component for now)
class ErrorBoundary extends Component {
  state = { hasError: false, error: null };
  
  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }
  
  render() {
    if (this.state.hasError) {
      return (
        <div>
          <h2>Something went wrong</h2>
          <p>{this.state.error?.message}</p>
          <button onClick={() => this.setState({ hasError: false })}>
            Retry
          </button>
        </div>
      );
    }
    
    return this.props.children;
  }
}

function App() {
  return (
    <ErrorBoundary>
      <Suspense fallback={<div>Loading...</div>}>
        <UserProfile userId="123" />
      </Suspense>
    </ErrorBoundary>
  );
}
```

**Pattern: Combined boundary**

```jsx
function AsyncBoundary({ children, fallback, errorFallback }) {
  return (
    <ErrorBoundary fallback={errorFallback}>
      <Suspense fallback={fallback}>
        {children}
      </Suspense>
    </ErrorBoundary>
  );
}

// Usage
<AsyncBoundary
  fallback={<LoadingSpinner />}
  errorFallback={<ErrorMessage />}
>
  <DataComponent />
</AsyncBoundary>
```

---

### 7. **Streaming SSR (Server-Side Rendering)**

**Traditional SSR (blocking):**

```
Server:
1. Fetch all data (wait for slowest query)
2. Render entire HTML
3. Send HTML to client

Client:
1. Receive HTML
2. Load JavaScript
3. Hydrate (make interactive)

Problem: User waits for ALL data before seeing ANYTHING
```

**Streaming SSR with Suspense:**

```jsx
// app/page.tsx (Next.js App Router with RSC)
import { Suspense } from 'react';

export default function Page() {
  return (
    <div>
      <Header />  {/* Renders immediately */}
      
      <Suspense fallback={<PostsSkeleton />}>
        <Posts />  {/* Streams when ready */}
      </Suspense>
      
      <Suspense fallback={<CommentsSkeleton />}>
        <Comments />  {/* Streams independently */}
      </Suspense>
    </div>
  );
}

async function Posts() {
  const posts = await fetchPosts();  // Async Server Component
  return posts.map(post => <Post key={post.id} {...post} />);
}

async function Comments() {
  const comments = await fetchComments();  // Can be slower
  return comments.map(c => <Comment key={c.id} {...c} />);
}
```

**Streaming timeline:**

```
Time 0ms:
  Client receives: <Header /> + <PostsSkeleton /> + <CommentsSkeleton />

Time 200ms (Posts ready):
  Client receives: <Posts /> (replaces skeleton)

Time 500ms (Comments ready):
  Client receives: <Comments /> (replaces skeleton)
```

**Benefits:**
- **Faster TTFB:** Send initial HTML immediately
- **Progressive rendering:** Show content as it loads
- **Better UX:** Users see something instantly
- **Parallel loading:** Components load independently

---

### 8. **Advanced Patterns**

**Pattern 1: Preloading with lazy()**

```jsx
import { lazy } from 'react';

const HeavyModal = lazy(() => import('./HeavyModal'));

function App() {
  // Preload on hover (before click)
  const handleMouseEnter = () => {
    import('./HeavyModal');  // Starts loading
  };
  
  return (
    <button
      onClick={() => setShowModal(true)}
      onMouseEnter={handleMouseEnter}
    >
      Open Modal
    </button>
  );
}
```

**Pattern 2: Delayed fallback (avoid flash)**

```jsx
function DelayedFallback({ children, delay = 300 }) {
  const [show, setShow] = useState(false);
  
  useEffect(() => {
    const timer = setTimeout(() => setShow(true), delay);
    return () => clearTimeout(timer);
  }, [delay]);
  
  return show ? children : null;
}

// Usage
<Suspense fallback={
  <DelayedFallback>
    <LoadingSpinner />
  </DelayedFallback>
}>
  <Component />
</Suspense>
```

**Pattern 3: Skeleton screens**

```jsx
function ProductList() {
  return (
    <Suspense fallback={<ProductListSkeleton />}>
      <ProductListContent />
    </Suspense>
  );
}

function ProductListSkeleton() {
  return (
    <div className="grid">
      {Array.from({ length: 8 }).map((_, i) => (
        <div key={i} className="skeleton-card">
          <div className="skeleton-image" />
          <div className="skeleton-text" />
          <div className="skeleton-text" />
        </div>
      ))}
    </div>
  );
}
```

---

## Common Pitfalls

### ❌ **Pitfall 1: No Suspense boundary**

```jsx
// BAD: Uncaught error - no Suspense boundary!
const LazyComponent = lazy(() => import('./Component'));

function App() {
  return <LazyComponent />;  // Crashes!
}

// GOOD: Always wrap lazy components
function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <LazyComponent />
    </Suspense>
  );
}
```

---

### ❌ **Pitfall 2: Dynamic imports in wrong place**

```jsx
// BAD: Import inside component (re-imports on every render!)
function App() {
  const LazyComponent = lazy(() => import('./Component'));
  return <Suspense><LazyComponent /></Suspense>;
}

// GOOD: Import at module level
const LazyComponent = lazy(() => import('./Component'));

function App() {
  return <Suspense><LazyComponent /></Suspense>;
}
```

---

### ❌ **Pitfall 3: Not using named exports with lazy()**

```jsx
// Component.tsx
export function MyComponent() { ... }

// BAD: Default import doesn't exist
const LazyComponent = lazy(() => import('./Component'));

// GOOD: Use named export
const LazyComponent = lazy(() => 
  import('./Component').then(module => ({ default: module.MyComponent }))
);
```

---

## Quick Self-Check

✅ **You understand Suspense if you can:**

1. Use `lazy()` for code splitting
2. Wrap lazy components in `<Suspense>`
3. Create nested Suspense boundaries
4. Combine Suspense with useTransition
5. Use Error Boundaries with Suspense
6. Implement skeleton loading screens
7. Explain streaming SSR
8. Use `use()` hook for data fetching (React 19)
9. Preload components on hover
10. Build route-based code splitting

---

## Interview Questions to Practice

**Beginner:**
1. What is Suspense used for?
2. How do you code-split a component?
3. What happens if you don't wrap lazy() in Suspense?

**Intermediate:**
4. How does Suspense work with Error Boundaries?
5. When would you use nested Suspense boundaries?
6. How do you prevent loading spinner flash with useTransition?
7. Explain streaming SSR

**Advanced:**
8. How would you implement progressive page rendering?
9. How does Suspense enable Server Components?
10. What's the difference between Suspense and loading states?

---

## Further Practice

**Build these:**
1. **Route-based code splitting** (React Router + lazy())
2. **Modal with preloading** (load on hover)
3. **Dashboard with streaming** (multiple async sections)
4. **Infinite scroll** with Suspense boundaries
5. **Image gallery** with lazy-loaded images

**Measure impact:**
- Compare bundle sizes before/after code splitting
- Use Network tab to see chunk loading
- Measure TTFB with streaming SSR

**Resources:**
- [React Docs - Suspense](https://react.dev/reference/react/Suspense)
- [React Docs - lazy](https://react.dev/reference/react/lazy)
- [Streaming SSR](https://nextjs.org/docs/app/building-your-application/routing/loading-ui-and-streaming)

---

## Summary

**Suspense enables:**
- Code splitting (smaller bundles)
- Declarative loading states
- Streaming SSR
- Progressive rendering
- Better UX

**Key APIs:**
- `<Suspense fallback={...}>` - loading boundary
- `lazy(() => import('...'))` - code splitting
- `use(promise)` - data fetching (React 19)
- `useTransition` - avoid loading flash
- Error Boundaries - handle errors

**Best practices:**
- Always wrap lazy() components in Suspense
- Use nested boundaries for granular loading
- Combine with useTransition for smooth transitions
- Implement skeleton screens for better UX
- Preload on hover for instant feel
- Use streaming SSR for faster perceived performance

**Remember:** Suspense is declarative async handling. Instead of managing loading states manually, declare boundaries and let React handle the rest.
