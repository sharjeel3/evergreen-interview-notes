# Error Boundaries

**Level:** 🚦 Intermediate  
**Tags:** `error-handling`, `error-boundaries`, `production`, `debugging`, `interview`

---

## Why this matters

Error boundaries are React's mechanism for gracefully handling JavaScript errors in component trees. Without error boundaries, a single error anywhere in your component tree can crash the entire application, showing users a blank screen. Interviewers ask about error boundaries because they assess your understanding of production-grade error handling, user experience design, and React's lifecycle methods.

**Why interviewers care:**
- Error boundaries prevent complete app crashes from single component failures
- Shows you understand production requirements (graceful degradation)
- Demonstrates knowledge of React class components (still required for error boundaries)
- Reveals understanding of error propagation in component trees
- Senior roles require designing fault-tolerant UI architectures
- Error logging integration shows production monitoring awareness

**Real-world implications:**
- **Resilience:** Isolated component errors don't crash entire app
- **User experience:** Show helpful error UI instead of blank screen
- **Debugging:** Catch and log errors for monitoring services (Sentry, LogRocket)
- **Partial functionality:** Rest of app continues working when one part fails
- **Recovery:** Provide "Try Again" actions to recover from errors
- **SEO/Accessibility:** Error UI is better than white screen for crawlers/users
- **Production stability:** Prevent cascading failures in large applications

**Common scenarios requiring error boundaries:**
- Third-party library errors (unexpected API changes, network failures)
- Data shape mismatches (API returns unexpected format)
- Rendering errors (null reference, type errors)
- Lazy-loaded component failures (network issues during code splitting)
- User-generated content rendering (potentially malformed data)
- Experimental features or unstable code paths
- Cross-cutting concerns (wrap entire sections of app)

**What you must know:**
- **Error boundaries are class components:** Can't be implemented with hooks yet
- **Two lifecycle methods:** `getDerivedStateFromError` and `componentDidCatch`
- **What they catch:** Rendering errors, lifecycle errors, constructor errors in children
- **What they don't catch:** Event handlers, async code, SSR, errors in error boundary itself
- **Granularity:** Can wrap entire app or individual components
- **Fallback UI:** Show user-friendly error message instead of crash
- **Error logging:** Send errors to monitoring services
- **Recovery:** Allow users to retry or navigate away

**Interview red flag:** Not knowing error boundaries exist, or saying "just use try/catch" shows lack of production React experience. Try/catch doesn't work for rendering errors—you must use error boundaries.

---

## Core Ideas

### 1. **Error boundaries catch errors in component trees**

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }
  
  static getDerivedStateFromError(error) {
    // Update state so next render shows fallback UI
    return { hasError: true };
  }
  
  componentDidCatch(error, errorInfo) {
    // Log error to monitoring service
    console.error('Error caught:', error, errorInfo);
  }
  
  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong.</h1>;
    }
    
    return this.props.children;
  }
}

// Usage
<ErrorBoundary>
  <MyComponent />
</ErrorBoundary>
```

**Mental model:**  
"Error boundaries are try/catch for React component rendering."

---

### 2. **Two lifecycle methods handle errors**

```jsx
class ErrorBoundary extends React.Component {
  // 1. getDerivedStateFromError: Update state for fallback UI
  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }
  
  // 2. componentDidCatch: Side effects (logging)
  componentDidCatch(error, errorInfo) {
    // errorInfo.componentStack: Where error occurred
    logErrorToService(error, errorInfo);
  }
}
```

**Key:**  
- `getDerivedStateFromError`: Render fallback (must be static, pure)
- `componentDidCatch`: Log error (side effects allowed)

---

### 3. **What error boundaries catch**

```jsx
// ✅ Caught by error boundary:
// - Rendering errors
function Component() {
  return <div>{data.user.name}</div>;  // data is undefined
}

// - Lifecycle errors
componentDidMount() {
  throw new Error('Failed to mount');
}

// - Constructor errors
constructor(props) {
  throw new Error('Failed to initialize');
}

// ❌ NOT caught by error boundary:
// - Event handlers
<button onClick={() => { throw new Error('Click error'); }}>
  {/* Use try/catch in handler */}
</button>

// - Async code
useEffect(() => {
  fetch('/api/data')
    .then(res => {
      throw new Error('Fetch failed');  // Not caught
    });
}, []);

// - Server-side rendering
// - Errors in error boundary itself
```

---

### 4. **Granularity: Wrap entire app or specific sections**

```jsx
// Option 1: Wrap entire app
<ErrorBoundary>
  <App />
</ErrorBoundary>

// Option 2: Wrap specific sections
<div>
  <ErrorBoundary fallback={<SidebarError />}>
    <Sidebar />
  </ErrorBoundary>
  
  <ErrorBoundary fallback={<ContentError />}>
    <MainContent />
  </ErrorBoundary>
</div>
```

**Trade-off:** Fine-grained boundaries = better isolation but more boilerplate.

---

### 5. **Provide recovery options**

```jsx
class ErrorBoundary extends React.Component {
  state = { hasError: false };
  
  static getDerivedStateFromError() {
    return { hasError: true };
  }
  
  resetError = () => {
    this.setState({ hasError: false });
  };
  
  render() {
    if (this.state.hasError) {
      return (
        <div>
          <h1>Something went wrong</h1>
          <button onClick={this.resetError}>Try Again</button>
        </div>
      );
    }
    
    return this.props.children;
  }
}
```

---

## Examples

### Example 1 — Basic error boundary

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }
  
  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }
  
  componentDidCatch(error, errorInfo) {
    console.error('Error boundary caught:', error, errorInfo);
  }
  
  render() {
    if (this.state.hasError) {
      return (
        <div role="alert">
          <h2>Oops! Something went wrong.</h2>
          <details style={{ whiteSpace: 'pre-wrap' }}>
            {this.state.error && this.state.error.toString()}
          </details>
        </div>
      );
    }
    
    return this.props.children;
  }
}

// Usage
function App() {
  return (
    <ErrorBoundary>
      <Dashboard />
    </ErrorBoundary>
  );
}
```

---

### Example 2 — Error boundary with logging

```jsx
class ErrorBoundary extends React.Component {
  state = { hasError: false };
  
  static getDerivedStateFromError(error) {
    return { hasError: true };
  }
  
  componentDidCatch(error, errorInfo) {
    // Send to monitoring service
    logErrorToSentry({
      error,
      errorInfo,
      componentStack: errorInfo.componentStack,
      userId: getCurrentUserId(),
      timestamp: new Date().toISOString()
    });
  }
  
  render() {
    if (this.state.hasError) {
      return <ErrorFallback />;
    }
    
    return this.props.children;
  }
}

function logErrorToSentry({ error, errorInfo, userId, timestamp }) {
  // Integration with Sentry, LogRocket, etc.
  Sentry.captureException(error, {
    contexts: {
      react: {
        componentStack: errorInfo.componentStack
      }
    },
    user: { id: userId },
    tags: { timestamp }
  });
}
```

---

### Example 3 — Reusable error boundary with custom fallback

```jsx
class ErrorBoundary extends React.Component {
  state = { hasError: false, error: null };
  
  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }
  
  componentDidCatch(error, errorInfo) {
    console.error('Error:', error, errorInfo);
  }
  
  resetError = () => {
    this.setState({ hasError: false, error: null });
  };
  
  render() {
    if (this.state.hasError) {
      // Use custom fallback or default
      if (this.props.fallback) {
        return this.props.fallback({
          error: this.state.error,
          resetError: this.resetError
        });
      }
      
      return (
        <div>
          <h2>Something went wrong</h2>
          <button onClick={this.resetError}>Try Again</button>
        </div>
      );
    }
    
    return this.props.children;
  }
}

// Usage with custom fallback
function App() {
  return (
    <ErrorBoundary
      fallback={({ error, resetError }) => (
        <div className="error-container">
          <h1>Oops!</h1>
          <p>{error.message}</p>
          <button onClick={resetError}>Reload Component</button>
        </div>
      )}
    >
      <Dashboard />
    </ErrorBoundary>
  );
}
```

---

### Example 4 — Multiple granular error boundaries

```jsx
function App() {
  return (
    <div className="app">
      <Header />
      
      <div className="content">
        {/* Sidebar failure doesn't crash main content */}
        <ErrorBoundary fallback={<SidebarError />}>
          <Sidebar />
        </ErrorBoundary>
        
        {/* Main content failure doesn't crash sidebar */}
        <ErrorBoundary fallback={<MainContentError />}>
          <MainContent />
        </ErrorBoundary>
      </div>
      
      <Footer />
    </div>
  );
}

function SidebarError() {
  return (
    <aside className="sidebar-error">
      <p>Unable to load sidebar</p>
      <button onClick={() => window.location.reload()}>
        Refresh Page
      </button>
    </aside>
  );
}
```

---

### Example 5 — Error boundary with lazy loading

```jsx
const LazyComponent = React.lazy(() => import('./LazyComponent'));

function App() {
  return (
    <ErrorBoundary
      fallback={({ resetError }) => (
        <div>
          <h2>Failed to load component</h2>
          <p>Check your network connection</p>
          <button onClick={resetError}>Retry</button>
        </div>
      )}
    >
      <Suspense fallback={<Spinner />}>
        <LazyComponent />
      </Suspense>
    </ErrorBoundary>
  );
}
```

---

### Example 6 — Error boundary with reset on route change

```jsx
import { useLocation } from 'react-router-dom';

function App() {
  const location = useLocation();
  
  return (
    <ErrorBoundary key={location.pathname}>
      {/* Key change resets error boundary on navigation */}
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
      </Routes>
    </ErrorBoundary>
  );
}
```

---

## Common Pitfalls

### 1. **Assuming error boundaries catch event handler errors**

```jsx
// ❌ Error boundary won't catch this
function Button() {
  const handleClick = () => {
    throw new Error('Button error');  // Not caught!
  };
  
  return <button onClick={handleClick}>Click</button>;
}

// ✅ Use try/catch in event handlers
function Button() {
  const handleClick = () => {
    try {
      riskyOperation();
    } catch (error) {
      console.error('Error:', error);
      // Handle or rethrow
    }
  };
  
  return <button onClick={handleClick}>Click</button>;
}
```

---

### 2. **Not providing recovery mechanism**

```jsx
// ❌ Users stuck on error screen
class ErrorBoundary extends React.Component {
  render() {
    if (this.state.hasError) {
      return <h1>Error!</h1>;  // No way to recover
    }
    return this.props.children;
  }
}

// ✅ Provide reset button
class ErrorBoundary extends React.Component {
  resetError = () => this.setState({ hasError: false });
  
  render() {
    if (this.state.hasError) {
      return (
        <div>
          <h1>Error!</h1>
          <button onClick={this.resetError}>Try Again</button>
        </div>
      );
    }
    return this.props.children;
  }
}
```

---

### 3. **Error boundary too broad**

```jsx
// ❌ One error crashes entire app
<ErrorBoundary>
  <App />
</ErrorBoundary>

// ✅ Isolate critical sections
<App>
  <ErrorBoundary fallback={<HeaderError />}>
    <Header />
  </ErrorBoundary>
  
  <ErrorBoundary fallback={<MainError />}>
    <Main />
  </ErrorBoundary>
</App>
```

---

### 4. **Not logging errors**

```jsx
// ❌ Errors go unnoticed
class ErrorBoundary extends React.Component {
  componentDidCatch(error, errorInfo) {
    // Nothing logged!
  }
}

// ✅ Log to monitoring service
class ErrorBoundary extends React.Component {
  componentDidCatch(error, errorInfo) {
    logToSentry(error, errorInfo);
    console.error('Error:', error, errorInfo);
  }
}
```

---

### 5. **Errors in error boundary itself**

```jsx
// ❌ Error in error boundary is uncaught
class ErrorBoundary extends React.Component {
  render() {
    if (this.state.hasError) {
      // Bug in fallback UI!
      return <div>{this.props.undefined.property}</div>;
    }
    return this.props.children;
  }
}

// ✅ Keep fallback UI simple and safe
class ErrorBoundary extends React.Component {
  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong</h1>;  // Simple, safe
    }
    return this.props.children;
  }
}
```

---

## FAQ

**Q: Can I use hooks to create error boundaries?**  
A: No (as of React 18). Error boundaries require class components with `getDerivedStateFromError` and `componentDidCatch`.

**Q: Do error boundaries catch async errors?**  
A: No. Wrap async code in try/catch and update state manually.

**Q: What about event handler errors?**  
A: Not caught. Use try/catch inside handlers.

**Q: Should I have one error boundary or many?**  
A: Multiple. Isolate critical sections so one error doesn't crash entire app.

**Q: How do I reset error boundary after error?**  
A: Provide reset button that sets `hasError: false` in state, or change error boundary's `key` prop.

**Q: Can I show different fallbacks for different errors?**  
A: Yes. Check error type/message in render and return appropriate fallback.

**Q: Do error boundaries work with Suspense?**  
A: Yes. Use error boundary outside Suspense to catch lazy loading failures.

---

## Quick Self-Check

1. What methods do you need to implement an error boundary?
   (`getDerivedStateFromError` and `componentDidCatch`)

2. Do error boundaries catch event handler errors?
   (No, use try/catch in handlers)

3. Can you create error boundaries with hooks?
   (No, must use class components)

4. What's the difference between the two lifecycle methods?
   (`getDerivedStateFromError` for state/render, `componentDidCatch` for side effects/logging)

5. Where should you place error boundaries?
   (Multiple boundaries around critical sections, not just one at root)

---

## Further Practice

1. **Build:** Error boundary with custom fallback UI and retry button
2. **Implement:** Error logging integration with Sentry or similar service
3. **Create:** Multiple error boundaries for different app sections
4. **Test:** Deliberately throw errors in components to test boundaries
5. **Build:** Error boundary that shows different messages based on error type
6. **Implement:** Error boundary that resets on route change
7. **Create:** Error boundary for lazy-loaded components

---

## Key Takeaways

- Error boundaries catch rendering errors, lifecycle errors, and constructor errors in children
- Implemented as class components with `getDerivedStateFromError` and `componentDidCatch`
- `getDerivedStateFromError`: Update state to show fallback UI (pure, no side effects)
- `componentDidCatch`: Log errors to monitoring services (side effects allowed)
- Don't catch: event handlers, async code, SSR, errors in error boundary itself
- Use multiple boundaries to isolate sections (better fault tolerance)
- Provide recovery mechanisms (reset button, retry action)
- Always log errors to monitoring services in production
- Keep fallback UI simple to avoid errors in error boundary itself
- Reset error boundary by changing its `key` prop or updating state
