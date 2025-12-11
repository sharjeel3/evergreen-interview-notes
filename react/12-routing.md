# Routing

**Level:** 🚦 Intermediate  
**Tags:** `routing`, `react-router`, `navigation`, `spa`, `interview`

---

## Why this matters

Client-side routing is fundamental to building Single Page Applications (SPAs). React Router is the de facto routing library for React, and understanding navigation patterns is essential for any professional React application. Interviewers expect you to know how to implement multi-page experiences without full page reloads, handle protected routes, manage navigation state, and optimize loading with code splitting.

**Why interviewers care:**
- Routing knowledge shows you can build real multi-page applications
- Questions about protected routes assess authentication/authorization understanding
- Navigation patterns reveal how you handle user experience and state management
- Code splitting with routing demonstrates performance optimization knowledge
- Deep linking and URL management are critical for SEO and user experience
- Senior roles require designing routing architecture for large applications

**Real-world implications:**
- **User experience:** Instant navigation without page reloads
- **SEO:** URLs for each view enable search engine indexing
- **Deep linking:** Users can bookmark and share specific app states
- **Navigation state:** Browser back/forward buttons work correctly
- **Code splitting:** Load only code needed for current route (performance)
- **Protected routes:** Prevent unauthorized access to certain pages
- **Analytics:** Track page views and user journeys through the app

**Common routing scenarios:**
- Public pages (landing, about, pricing) vs protected pages (dashboard, settings)
- Nested routes (layouts with changing sub-content)
- Dynamic routes (user profiles, product details with ID in URL)
- Redirects (after login, form submission, authorization checks)
- 404 pages and error handling
- Lazy-loaded routes for code splitting
- Programmatic navigation (after actions complete)

**What you must know:**
- **React Router basics:** BrowserRouter, Routes, Route, Link, NavLink
- **Navigation:** Link vs programmatic navigation (useNavigate)
- **Route params:** Dynamic segments in URLs (`:id`)
- **Nested routes:** Layouts with Outlet for child routes
- **Protected routes:** Redirect unauthenticated users
- **Lazy loading:** React.lazy + Suspense for code splitting routes
- **useLocation, useParams, useNavigate hooks:** Access routing state
- **URL state:** Query params and search strings

**Interview red flag:** Not knowing the difference between Link and `<a>` tags, or inability to implement protected routes shows you haven't built real SPAs. Saying "just use window.location" bypasses React Router entirely and breaks SPA behavior.

---

## Core Ideas

### 1. **React Router enables client-side navigation**

```jsx
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      <nav>
        {/* Link prevents full page reload */}
        <Link to="/">Home</Link>
        <Link to="/about">About</Link>
      </nav>
      
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
      </Routes>
    </BrowserRouter>
  );
}

// BrowserRouter: Wraps app, provides routing context
// Routes: Container for all Route definitions
// Route: Maps path to component
// Link: Navigation without page reload
```

**Mental model:**  
"React Router swaps components based on URL without browser refresh."

---

### 2. **Dynamic routes with parameters**

```jsx
<Routes>
  <Route path="/users/:userId" element={<UserProfile />} />
  <Route path="/posts/:postId/comments/:commentId" element={<Comment />} />
</Routes>

function UserProfile() {
  const { userId } = useParams();  // Extract URL params
  // Fetch user data based on userId
  return <div>User {userId}</div>;
}
```

**Key:** `:paramName` defines dynamic segments; `useParams()` extracts them.

---

### 3. **Nested routes and layouts**

```jsx
function App() {
  return (
    <Routes>
      <Route path="/" element={<Layout />}>
        {/* Nested routes render inside Layout's Outlet */}
        <Route index element={<Home />} />
        <Route path="about" element={<About />} />
        <Route path="contact" element={<Contact />} />
      </Route>
    </Routes>
  );
}

function Layout() {
  return (
    <div>
      <header>
        <nav>
          <Link to="/">Home</Link>
          <Link to="/about">About</Link>
          <Link to="/contact">Contact</Link>
        </nav>
      </header>
      
      {/* Child route renders here */}
      <Outlet />
      
      <footer>© 2024</footer>
    </div>
  );
}
```

**Key:** `Outlet` is where nested routes render. Layout stays, content swaps.

---

### 4. **Programmatic navigation**

```jsx
import { useNavigate } from 'react-router-dom';

function LoginForm() {
  const navigate = useNavigate();
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    await loginUser();
    
    // Navigate after action completes
    navigate('/dashboard');
    
    // Or with state
    navigate('/welcome', { state: { from: 'login' } });
    
    // Or go back
    navigate(-1);  // Like browser back button
  };
  
  return <form onSubmit={handleSubmit}>...</form>;
}
```

---

### 5. **Protected routes**

```jsx
function ProtectedRoute({ children }) {
  const { isAuthenticated } = useAuth();
  const location = useLocation();
  
  if (!isAuthenticated) {
    // Redirect to login, preserve attempted location
    return <Navigate to="/login" state={{ from: location }} replace />;
  }
  
  return children;
}

// Usage
<Routes>
  <Route path="/login" element={<Login />} />
  
  <Route 
    path="/dashboard" 
    element={
      <ProtectedRoute>
        <Dashboard />
      </ProtectedRoute>
    } 
  />
</Routes>
```

---

## Examples

### Example 1 — Complete routing setup

```jsx
import { 
  BrowserRouter, 
  Routes, 
  Route, 
  Link, 
  Navigate 
} from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/users/:id" element={<UserProfile />} />
        <Route path="/404" element={<NotFound />} />
        <Route path="*" element={<Navigate to="/404" replace />} />
      </Routes>
    </BrowserRouter>
  );
}

function Home() {
  return (
    <div>
      <h1>Home</h1>
      <Link to="/about">About Us</Link>
      <Link to="/users/123">User 123</Link>
    </div>
  );
}

function UserProfile() {
  const { id } = useParams();
  return <h1>User Profile: {id}</h1>;
}

function NotFound() {
  return <h1>404 - Page Not Found</h1>;
}
```

---

### Example 2 — Nested routes with layout

```jsx
function App() {
  return (
    <Routes>
      {/* Dashboard layout with nested routes */}
      <Route path="/dashboard" element={<DashboardLayout />}>
        <Route index element={<Overview />} />
        <Route path="analytics" element={<Analytics />} />
        <Route path="settings" element={<Settings />} />
      </Route>
    </Routes>
  );
}

function DashboardLayout() {
  return (
    <div className="dashboard">
      <aside>
        <nav>
          <Link to="/dashboard">Overview</Link>
          <Link to="/dashboard/analytics">Analytics</Link>
          <Link to="/dashboard/settings">Settings</Link>
        </nav>
      </aside>
      
      <main>
        {/* Nested routes render here */}
        <Outlet />
      </main>
    </div>
  );
}
```

---

### Example 3 — Protected routes with redirect

```jsx
// Auth context
const AuthContext = createContext();

function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  
  const login = async (credentials) => {
    const userData = await loginAPI(credentials);
    setUser(userData);
  };
  
  const logout = () => setUser(null);
  
  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

function useAuth() {
  return useContext(AuthContext);
}

// Protected route component
function ProtectedRoute({ children }) {
  const { user } = useAuth();
  const location = useLocation();
  
  if (!user) {
    // Save attempted location to redirect after login
    return <Navigate to="/login" state={{ from: location }} replace />;
  }
  
  return children;
}

// Routes
function App() {
  return (
    <AuthProvider>
      <BrowserRouter>
        <Routes>
          <Route path="/login" element={<Login />} />
          <Route path="/signup" element={<Signup />} />
          
          <Route 
            path="/dashboard" 
            element={
              <ProtectedRoute>
                <Dashboard />
              </ProtectedRoute>
            } 
          />
          
          <Route 
            path="/profile" 
            element={
              <ProtectedRoute>
                <Profile />
              </ProtectedRoute>
            } 
          />
        </Routes>
      </BrowserRouter>
    </AuthProvider>
  );
}

// Login with redirect back to attempted page
function Login() {
  const { login } = useAuth();
  const navigate = useNavigate();
  const location = useLocation();
  
  const from = location.state?.from?.pathname || '/dashboard';
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    await login(credentials);
    navigate(from, { replace: true });  // Go to attempted page
  };
  
  return <form onSubmit={handleSubmit}>...</form>;
}
```

---

### Example 4 — Lazy loading routes

```jsx
import { lazy, Suspense } from 'react';

// Lazy load components
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Analytics = lazy(() => import('./pages/Analytics'));
const Settings = lazy(() => import('./pages/Settings'));

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<LoadingSpinner />}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/dashboard" element={<Dashboard />} />
          <Route path="/analytics" element={<Analytics />} />
          <Route path="/settings" element={<Settings />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}

function LoadingSpinner() {
  return <div>Loading...</div>;
}
```

---

### Example 5 — Search params and filters

```jsx
import { useSearchParams } from 'react-router-dom';

function ProductList() {
  const [searchParams, setSearchParams] = useSearchParams();
  
  // Read query params
  const category = searchParams.get('category') || 'all';
  const sort = searchParams.get('sort') || 'name';
  
  const updateFilter = (key, value) => {
    // Update URL query params
    setSearchParams({ ...Object.fromEntries(searchParams), [key]: value });
  };
  
  return (
    <div>
      <select 
        value={category} 
        onChange={e => updateFilter('category', e.target.value)}
      >
        <option value="all">All</option>
        <option value="electronics">Electronics</option>
        <option value="books">Books</option>
      </select>
      
      <select 
        value={sort} 
        onChange={e => updateFilter('sort', e.target.value)}
      >
        <option value="name">Name</option>
        <option value="price">Price</option>
      </select>
      
      {/* URL: /products?category=electronics&sort=price */}
      <ProductGrid category={category} sort={sort} />
    </div>
  );
}
```

---

### Example 6 — NavLink for active styling

```jsx
import { NavLink } from 'react-router-dom';

function Navigation() {
  return (
    <nav>
      {/* NavLink adds 'active' class to current route */}
      <NavLink 
        to="/" 
        className={({ isActive }) => isActive ? 'nav-link active' : 'nav-link'}
      >
        Home
      </NavLink>
      
      <NavLink 
        to="/about"
        style={({ isActive }) => ({
          color: isActive ? 'red' : 'black'
        })}
      >
        About
      </NavLink>
    </nav>
  );
}
```

---

## Common Pitfalls

### 1. **Using `<a>` instead of `<Link>`**

```jsx
// ❌ Full page reload (breaks SPA)
<a href="/about">About</a>

// ✅ Client-side navigation
<Link to="/about">About</Link>
```

---

### 2. **Not wrapping app in BrowserRouter**

```jsx
// ❌ Router hooks won't work
function App() {
  return (
    <Routes>
      <Route path="/" element={<Home />} />
    </Routes>
  );
}

// ✅ Wrap in BrowserRouter
function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
      </Routes>
    </BrowserRouter>
  );
}
```

---

### 3. **Forgetting Outlet in layouts**

```jsx
// ❌ Nested routes won't render
function Layout() {
  return (
    <div>
      <nav>...</nav>
      {/* Missing Outlet! */}
    </div>
  );
}

// ✅ Add Outlet
function Layout() {
  return (
    <div>
      <nav>...</nav>
      <Outlet />  {/* Child routes render here */}
    </div>
  );
}
```

---

### 4. **Incorrect path matching**

```jsx
// ❌ Paths must match exactly
<Routes>
  <Route path="users" element={<Users />} />  {/* Matches /users */}
</Routes>

// Navigating to "/users/" (with trailing slash) won't match!

// ✅ Be consistent with paths
<Route path="/users" element={<Users />} />
<Link to="/users">Users</Link>
```

---

### 5. **Using window.location for navigation**

```jsx
// ❌ Bypasses React Router (full reload)
<button onClick={() => window.location.href = '/dashboard'}>
  Go to Dashboard
</button>

// ✅ Use navigate or Link
const navigate = useNavigate();
<button onClick={() => navigate('/dashboard')}>
  Go to Dashboard
</button>
```

---

## FAQ

**Q: Link vs navigate—when to use each?**  
A: `Link` for navigation elements (menus, buttons that look like links). `navigate` for programmatic navigation (after form submit, after API call).

**Q: How do I redirect after login?**  
A: Save attempted location in `Navigate` state, then redirect there after successful login.

**Q: What's the difference between BrowserRouter and HashRouter?**  
A: BrowserRouter uses HTML5 history API (`/about`). HashRouter uses hash (`/#/about`). Use BrowserRouter unless you need hash for legacy servers.

**Q: How do I handle 404s?**  
A: Add catch-all route: `<Route path="*" element={<NotFound />} />`

**Q: Can I have multiple Routes components?**  
A: Yes, but usually one Routes at top level with nested routes inside.

**Q: How do I get query parameters?**  
A: `useSearchParams()` hook: `const [params] = useSearchParams(); params.get('q')`

**Q: How do I pass state during navigation?**  
A: `navigate('/path', { state: { data } })` then access with `useLocation().state`

---

## Quick Self-Check

1. What's wrong with this?
   ```jsx
   <a href="/about">About</a>
   ```
   (Should use `<Link to="/about">` to avoid full page reload)

2. How do you get URL params from `/users/:id`?
   (useParams hook: `const { id } = useParams()`)

3. How do you implement protected routes?
   (Check auth state, render `<Navigate to="/login" />` if not authenticated)

4. What is `Outlet` used for?
   (Renders nested child routes inside parent layout)

5. How do you navigate programmatically after an action?
   (useNavigate hook: `const navigate = useNavigate(); navigate('/path')`)

---

## Further Practice

1. **Build:** Multi-page app with public and protected routes
2. **Build:** Nested dashboard layout with sidebar navigation
3. **Implement:** 404 page and catch-all route
4. **Add:** Lazy loading to routes with code splitting
5. **Build:** Search page with URL query params for filters
6. **Implement:** Protected routes that redirect back after login
7. **Build:** Breadcrumbs component using useLocation

---

## Key Takeaways

- React Router enables client-side navigation without page reloads
- Use `Link` for navigation elements, `navigate()` for programmatic navigation
- Dynamic routes use `:paramName` syntax; extract with `useParams()`
- Nested routes render inside parent's `Outlet` component
- Protected routes check auth state and redirect with `Navigate`
- Lazy load routes with `React.lazy` and `Suspense` for code splitting
- `useSearchParams` manages URL query strings (filters, search)
- `NavLink` provides active styling for current route
- Never use `<a href>` or `window.location` for internal navigation
- Always wrap app in `BrowserRouter` to enable routing context
