# Context API

**Level:** 🚦 Intermediate  
**Tags:** `context`, `global-state`, `prop-drilling`, `interview`, `state-management`

---

## Why this matters

The Context API solves one of React's most common pain points: prop drilling. Understanding Context is essential because it's React's built-in solution for sharing data globally, and interviewers frequently ask about it to assess your knowledge of state management strategies and when to use different patterns.

**Why interviewers care:**
- Context questions reveal whether you understand React's component tree and data flow
- Knowing when to use Context vs other solutions shows architectural judgment
- Context performance implications separate developers who understand re-rendering from those who don't
- Senior developers need to design Context architecture that scales across large applications
- Misuse of Context is a common source of performance bugs that interviewers probe

**Real-world implications:**
- **Eliminate prop drilling:** No more passing props through 5+ intermediate components
- **Theme switching:** Dark/light mode without passing theme everywhere
- **Authentication:** User data available anywhere without prop chains
- **Localization:** Language/translation access throughout app
- **Global UI state:** Modals, notifications, sidebars accessible from anywhere
- **Performance impact:** ALL consumers re-render when context changes—must optimize carefully

**Common use cases for Context:**
- Theme/styling (dark mode, color schemes)
- User authentication and permissions
- Language/localization (i18n)
- Application settings and preferences
- Current route/navigation state
- Feature flags and configuration
- Notification/toast systems
- Modal/dialog management

**When NOT to use Context:**
- Frequently changing data (causes many re-renders)
- Complex state logic (use reducer or state management library)
- Optimizing for performance-critical applications
- When prop drilling is only 2-3 levels deep (keep it simple)

**What you must know:**
- **Three parts:** createContext, Provider, useContext (consumer)
- **Provider:** Wraps component tree and provides value
- **Consumer:** Any descendant can read context with useContext
- **Re-render behavior:** ALL consumers re-render when context value changes
- **Multiple contexts:** Can use multiple contexts in same app
- **Default value:** Context has default value when no provider found
- **Optimization:** Split contexts or use useMemo to prevent unnecessary re-renders

**Interview red flag:** Using Context for all state (overkill), not understanding that all consumers re-render when value changes (performance issue), or using Context when simple prop passing would suffice (over-engineering).

---

## Core Ideas

### 1. **Context solves prop drilling**

```jsx
// ❌ Prop drilling: passing theme through every level
function App() {
  const [theme, setTheme] = useState('dark');
  return <Page theme={theme} />;
}

function Page({ theme }) {
  return <Section theme={theme} />;
}

function Section({ theme }) {
  return <Button theme={theme} />;
}

function Button({ theme }) {
  return <button className={theme}>Click</button>;
}

// ✅ Context: theme available anywhere
const ThemeContext = createContext();

function App() {
  const [theme, setTheme] = useState('dark');
  return (
    <ThemeContext.Provider value={theme}>
      <Page />
    </ThemeContext.Provider>
  );
}

function Button() {
  const theme = useContext(ThemeContext);
  return <button className={theme}>Click</button>;
}
```

**Mental model:**  
"Context = radio broadcast. Provider broadcasts value, consumers tune in."

---

### 2. **Three parts: Create, Provide, Consume**

```jsx
// 1. Create context
const MyContext = createContext(defaultValue);

// 2. Provide value
function App() {
  return (
    <MyContext.Provider value={someValue}>
      <ChildComponents />
    </MyContext.Provider>
  );
}

// 3. Consume value
function ChildComponent() {
  const value = useContext(MyContext);
  return <div>{value}</div>;
}
```

---

### 3. **All consumers re-render when value changes**

```jsx
const CountContext = createContext();

function App() {
  const [count, setCount] = useState(0);
  
  return (
    <CountContext.Provider value={count}>
      <ComponentA />  {/* Re-renders when count changes */}
      <ComponentB />  {/* Re-renders when count changes */}
      <ComponentC />  {/* Re-renders when count changes */}
    </CountContext.Provider>
  );
}
```

**Key:** Every component using `useContext(CountContext)` re-renders on value change.

---

### 4. **Default value used when no provider**

```jsx
const ThemeContext = createContext('light');  // Default: 'light'

// Without provider, useContext returns default
function Button() {
  const theme = useContext(ThemeContext);  // 'light'
  return <button className={theme}>Click</button>;
}

// With provider, uses provided value
function App() {
  return (
    <ThemeContext.Provider value="dark">
      <Button />  {/* theme = 'dark' */}
    </ThemeContext.Provider>
  );
}
```

---

### 5. **Separate contexts for different concerns**

```jsx
// ✅ Split contexts to avoid unnecessary re-renders
const UserContext = createContext();
const ThemeContext = createContext();

function App() {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('dark');
  
  return (
    <UserContext.Provider value={user}>
      <ThemeContext.Provider value={theme}>
        <Page />
      </ThemeContext.Provider>
    </UserContext.Provider>
  );
}

// Component only re-renders when theme changes, not user
function ThemedButton() {
  const theme = useContext(ThemeContext);  // Only subscribes to theme
  return <button className={theme}>Click</button>;
}
```

---

## Examples

### Example 1 — Theme Context

```jsx
import { createContext, useContext, useState } from 'react';

// Create context
const ThemeContext = createContext();

// Custom hook for consuming context
function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
}

// Provider component
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  
  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };
  
  const value = { theme, toggleTheme };
  
  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}

// Usage in App
function App() {
  return (
    <ThemeProvider>
      <Header />
      <MainContent />
    </ThemeProvider>
  );
}

// Consuming component
function Header() {
  const { theme, toggleTheme } = useTheme();
  
  return (
    <header className={theme}>
      <button onClick={toggleTheme}>
        Toggle to {theme === 'light' ? 'dark' : 'light'}
      </button>
    </header>
  );
}
```

---

### Example 2 — Auth Context

```jsx
const AuthContext = createContext();

export function useAuth() {
  return useContext(AuthContext);
}

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    // Check if user is logged in on mount
    checkAuth().then(user => {
      setUser(user);
      setLoading(false);
    });
  }, []);
  
  const login = async (email, password) => {
    const user = await loginAPI(email, password);
    setUser(user);
  };
  
  const logout = async () => {
    await logoutAPI();
    setUser(null);
  };
  
  const value = {
    user,
    loading,
    login,
    logout,
    isAuthenticated: !!user
  };
  
  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
}

// Usage
function App() {
  return (
    <AuthProvider>
      <Router>
        <Routes />
      </Router>
    </AuthProvider>
  );
}

function Dashboard() {
  const { user, logout } = useAuth();
  
  return (
    <div>
      <h1>Welcome, {user.name}!</h1>
      <button onClick={logout}>Logout</button>
    </div>
  );
}

function ProtectedRoute({ children }) {
  const { isAuthenticated, loading } = useAuth();
  
  if (loading) return <div>Loading...</div>;
  if (!isAuthenticated) return <Navigate to="/login" />;
  return children;
}
```

---

### Example 3 — Multiple Contexts

```jsx
const UserContext = createContext();
const ThemeContext = createContext();
const LanguageContext = createContext();

function App() {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');
  const [language, setLanguage] = useState('en');
  
  return (
    <UserContext.Provider value={{ user, setUser }}>
      <ThemeContext.Provider value={{ theme, setTheme }}>
        <LanguageContext.Provider value={{ language, setLanguage }}>
          <MainApp />
        </LanguageContext.Provider>
      </ThemeContext.Provider>
    </UserContext.Provider>
  );
}

// Component can use multiple contexts
function UserProfile() {
  const { user } = useContext(UserContext);
  const { theme } = useContext(ThemeContext);
  const { language } = useContext(LanguageContext);
  
  return (
    <div className={theme}>
      <h1>{translate(user.name, language)}</h1>
    </div>
  );
}
```

---

### Example 4 — Optimized Context (prevent re-renders)

```jsx
// ❌ Problem: New object every render causes all consumers to re-render
function BadProvider({ children }) {
  const [user, setUser] = useState(null);
  
  // New object on every render!
  return (
    <UserContext.Provider value={{ user, setUser }}>
      {children}
    </UserContext.Provider>
  );
}

// ✅ Solution: Memoize value
function GoodProvider({ children }) {
  const [user, setUser] = useState(null);
  
  // Same object reference unless user changes
  const value = useMemo(() => ({ user, setUser }), [user]);
  
  return (
    <UserContext.Provider value={value}>
      {children}
    </UserContext.Provider>
  );
}

// ✅ Alternative: Split contexts
const UserContext = createContext();
const UserActionsContext = createContext();

function OptimizedProvider({ children }) {
  const [user, setUser] = useState(null);
  
  // Actions don't change, only user data changes
  const actions = useMemo(() => ({ setUser }), []);
  
  return (
    <UserContext.Provider value={user}>
      <UserActionsContext.Provider value={actions}>
        {children}
      </UserActionsContext.Provider>
    </UserContext.Provider>
  );
}

// Component only re-renders when user changes, not on every parent render
function UserDisplay() {
  const user = useContext(UserContext);
  return <div>{user?.name}</div>;
}

// Component never re-renders (actions are stable)
function UserForm() {
  const { setUser } = useContext(UserActionsContext);
  // ...
}
```

---

### Example 5 — Context with Reducer

```jsx
const CartContext = createContext();

const initialState = {
  items: [],
  total: 0
};

function cartReducer(state, action) {
  switch (action.type) {
    case 'ADD_ITEM':
      return {
        ...state,
        items: [...state.items, action.payload],
        total: state.total + action.payload.price
      };
    case 'REMOVE_ITEM':
      const item = state.items.find(i => i.id === action.payload);
      return {
        ...state,
        items: state.items.filter(i => i.id !== action.payload),
        total: state.total - item.price
      };
    case 'CLEAR_CART':
      return initialState;
    default:
      return state;
  }
}

function CartProvider({ children }) {
  const [state, dispatch] = useReducer(cartReducer, initialState);
  
  const value = { state, dispatch };
  
  return (
    <CartContext.Provider value={value}>
      {children}
    </CartContext.Provider>
  );
}

function useCart() {
  const context = useContext(CartContext);
  if (!context) {
    throw new Error('useCart must be used within CartProvider');
  }
  return context;
}

// Usage
function ProductCard({ product }) {
  const { dispatch } = useCart();
  
  return (
    <div>
      <h3>{product.name}</h3>
      <button onClick={() => dispatch({ 
        type: 'ADD_ITEM', 
        payload: product 
      })}>
        Add to Cart
      </button>
    </div>
  );
}

function Cart() {
  const { state, dispatch } = useCart();
  
  return (
    <div>
      <h2>Cart ({state.items.length} items)</h2>
      <p>Total: ${state.total}</p>
      <button onClick={() => dispatch({ type: 'CLEAR_CART' })}>
        Clear Cart
      </button>
    </div>
  );
}
```

---

## Common Pitfalls

### 1. **Not memoizing context value (unnecessary re-renders)**

```jsx
// ❌ New object every render
function BadProvider({ children }) {
  const [state, setState] = useState({});
  
  return (
    <MyContext.Provider value={{ state, setState }}>
      {children}
    </MyContext.Provider>
  );
}

// ✅ Memoized value
function GoodProvider({ children }) {
  const [state, setState] = useState({});
  const value = useMemo(() => ({ state, setState }), [state]);
  
  return (
    <MyContext.Provider value={value}>
      {children}
    </MyContext.Provider>
  );
}
```

---

### 2. **Using Context without error handling**

```jsx
// ❌ Fails silently if used outside provider
function BadComponent() {
  const value = useContext(MyContext);  // undefined if no provider!
  return <div>{value.name}</div>;  // Crash!
}

// ✅ Custom hook with error handling
function useMyContext() {
  const context = useContext(MyContext);
  if (context === undefined) {
    throw new Error('useMyContext must be used within MyProvider');
  }
  return context;
}
```

---

### 3. **Putting too much in one context**

```jsx
// ❌ One giant context (everything re-renders together)
const AppContext = createContext();

function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('dark');
  const [language, setLanguage] = useState('en');
  const [cart, setCart] = useState([]);
  // ... 10 more pieces of state
  
  return (
    <AppContext.Provider value={{ 
      user, setUser, theme, setTheme, language, setLanguage, cart, setCart 
    }}>
      {children}
    </AppContext.Provider>
  );
}

// ✅ Split into focused contexts
<UserProvider>
  <ThemeProvider>
    <LanguageProvider>
      <CartProvider>
        {children}
      </CartProvider>
    </LanguageProvider>
  </ThemeProvider>
</UserProvider>
```

---

### 4. **Using Context for frequently changing data**

```jsx
// ❌ Mouse position in Context (re-renders entire app constantly!)
function MouseProvider({ children }) {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  
  useEffect(() => {
    const handleMove = (e) => setPosition({ x: e.clientX, y: e.clientY });
    window.addEventListener('mousemove', handleMove);
    return () => window.removeEventListener('mousemove', handleMove);
  }, []);
  
  return (
    <MouseContext.Provider value={position}>
      {children}
    </MouseContext.Provider>
  );
}

// ✅ Use custom hook with local state instead
function useMousePosition() {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  
  useEffect(() => {
    const handleMove = (e) => setPosition({ x: e.clientX, y: e.clientY });
    window.addEventListener('mousemove', handleMove);
    return () => window.removeEventListener('mousemove', handleMove);
  }, []);
  
  return position;
}
```

---

### 5. **Nesting too many providers (callback hell)**

```jsx
// ❌ Provider pyramid
<UserProvider>
  <ThemeProvider>
    <LanguageProvider>
      <CartProvider>
        <NotificationProvider>
          <ModalProvider>
            <App />
          </ModalProvider>
        </NotificationProvider>
      </CartProvider>
    </LanguageProvider>
  </ThemeProvider>
</UserProvider>

// ✅ Compose providers
function AppProviders({ children }) {
  return (
    <UserProvider>
      <ThemeProvider>
        <LanguageProvider>
          <CartProvider>
            {children}
          </CartProvider>
        </LanguageProvider>
      </ThemeProvider>
    </UserProvider>
  );
}

// Usage
<AppProviders>
  <App />
</AppProviders>
```

---

## FAQ

**Q: When should I use Context instead of props?**  
A: When data is needed by many components at different nesting levels. If only 2-3 levels deep, props are fine.

**Q: Does Context replace Redux?**  
A: For simple global state, yes. For complex state with many actions, Redux/Zustand may be better.

**Q: How do I prevent unnecessary re-renders?**  
A: Memoize context value with useMemo, or split contexts for different concerns.

**Q: Can I have multiple providers for same context?**  
A: Yes. Nested providers override outer ones. Useful for scoped overrides.

**Q: What's the default value for?**  
A: Used when component uses context outside any provider. Good for testing and defaults.

**Q: Should I create custom hooks for context?**  
A: Yes! `useAuth()` is cleaner than `useContext(AuthContext)` and allows error handling.

---

## Quick Self-Check

1. What three functions are needed for Context?
   (createContext, Provider, useContext)

2. What happens when context value changes?
   (All consumers re-render)

3. What's wrong with this?
   ```jsx
   <MyContext.Provider value={{ state, setState }}>
   ```
   (New object every render, causes unnecessary re-renders)

4. When should you NOT use Context?
   (Frequently changing data, simple prop passing, performance-critical apps)

5. How do you prevent Context re-render issues?
   (useMemo on value, split contexts, move state closer to where it's used)

---

## Further Practice

1. **Build:** Theme provider with dark/light mode toggle
2. **Build:** Auth provider with login/logout/protected routes
3. **Build:** Shopping cart context with add/remove items
4. **Build:** Toast/notification system with Context
5. **Optimize:** Profile and fix unnecessary re-renders in Context app
6. **Compare:** Build same feature with Context vs prop drilling vs Redux

---

## Key Takeaways

- Context solves prop drilling for global/shared state
- Three parts: createContext, Provider component, useContext hook
- ALL consumers re-render when context value changes
- Memoize context value to prevent unnecessary re-renders
- Split contexts by concern (user, theme, cart separate)
- Create custom hooks for better error handling and cleaner API
- Don't use Context for frequently changing data (performance issue)
- Default value used when no provider found (testing/fallback)
- Multiple contexts can be composed in same app
- Context is not Redux—use state management for complex logic
