# Component Composition

**Level:** 🚦 Intermediate  
**Tags:** `composition`, `children`, `render-props`, `HOC`, `patterns`, `interview`

---

## Why this matters

Component composition is the cornerstone of React's architecture—how you combine smaller components to build complex UIs. Instead of inheritance (class-based OOP), React embraces composition patterns like children props, render props, and Higher-Order Components (HOCs). Interviewers ask about composition to assess your ability to design reusable, flexible component APIs and avoid prop drilling. Understanding composition patterns separates junior developers from senior engineers who can architect maintainable systems.

**Why interviewers care:**
- Composition patterns show you understand React's philosophy (composition over inheritance)
- Children prop questions assess component API design skills
- Render props vs HOCs vs hooks reveals evolution of React patterns
- Compound components demonstrate advanced API design knowledge
- Senior roles require designing flexible, reusable component libraries
- Composition patterns solve cross-cutting concerns without tight coupling

**Real-world implications:**
- **Reusability:** Build generic wrappers (Layout, Card, Modal) used throughout app
- **Flexibility:** Components accept any children without knowing structure in advance
- **Separation of concerns:** Logic (behavior) separated from presentation (UI)
- **API design:** Clean, intuitive component interfaces
- **Maintenance:** Changes to wrapper don't affect children and vice versa
- **Avoid prop drilling:** Share state/logic without passing props through layers
- **Design systems:** Build consistent, composable component libraries

**Common composition scenarios:**
- Layout components (Grid, Stack, Container) that arrange children
- Wrapper components (Card, Panel) that add styling/behavior
- Modals, tooltips, popovers with flexible content
- Lists that render different item types
- Forms with flexible field arrangements
- Tabs, accordions, carousels with dynamic content
- Conditional rendering wrappers (authentication, feature flags)

**What you must know:**
- **Children prop:** Most basic composition pattern (render anything inside)
- **Render props:** Pass function as prop to control what gets rendered
- **Higher-Order Components (HOCs):** Functions that take component, return enhanced component
- **Compound components:** Multiple components work together (Tabs, Select, Radio Group)
- **Hooks replaced most patterns:** Custom hooks often cleaner than HOCs/render props
- **Composition vs inheritance:** React prefers composition for code reuse
- **Slots pattern:** Named children for specific positions
- **When to use what:** Children for simple, render props for flexibility, hooks for logic reuse

**Interview red flag:** Relying on inheritance, not knowing children prop, or inability to explain trade-offs between composition patterns shows lack of React experience. Overusing HOCs when hooks would be simpler is outdated practice.

---

## Core Ideas

### 1. **Children prop: Most basic composition**

```jsx
function Card({ children }) {
  return (
    <div className="card">
      {children}
    </div>
  );
}

// Usage: Can pass anything as children
<Card>
  <h2>Title</h2>
  <p>Content</p>
  <button>Action</button>
</Card>
```

**Mental model:**  
"Children prop makes components generic containers—they don't care what goes inside."

---

### 2. **Named slots for specific positions**

```jsx
function Layout({ header, sidebar, children, footer }) {
  return (
    <div className="layout">
      <header>{header}</header>
      <div className="main">
        <aside>{sidebar}</aside>
        <main>{children}</main>
      </div>
      <footer>{footer}</footer>
    </div>
  );
}

// Usage
<Layout
  header={<Header />}
  sidebar={<Sidebar />}
  footer={<Footer />}
>
  <MainContent />
</Layout>
```

---

### 3. **Render props for flexible rendering**

```jsx
function MouseTracker({ render }) {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  
  const handleMouseMove = (e) => {
    setPosition({ x: e.clientX, y: e.clientY });
  };
  
  return (
    <div onMouseMove={handleMouseMove}>
      {render(position)}
    </div>
  );
}

// Usage: Parent controls rendering
<MouseTracker 
  render={({ x, y }) => (
    <div>Mouse at ({x}, {y})</div>
  )}
/>
```

**Key:** Component provides data/behavior, parent controls presentation.

---

### 4. **Higher-Order Components (HOCs)**

```jsx
// HOC: Function that takes component, returns enhanced component
function withAuth(Component) {
  return function AuthenticatedComponent(props) {
    const { isAuthenticated } = useAuth();
    
    if (!isAuthenticated) {
      return <Redirect to="/login" />;
    }
    
    return <Component {...props} />;
  };
}

// Usage
const ProtectedDashboard = withAuth(Dashboard);
const ProtectedProfile = withAuth(Profile);
```

---

### 5. **Compound components**

```jsx
function Tabs({ children }) {
  const [activeIndex, setActiveIndex] = useState(0);
  
  return (
    <div>
      {React.Children.map(children, (child, index) =>
        React.cloneElement(child, {
          isActive: index === activeIndex,
          onActivate: () => setActiveIndex(index)
        })
      )}
    </div>
  );
}

function Tab({ isActive, onActivate, children }) {
  return (
    <button 
      className={isActive ? 'active' : ''}
      onClick={onActivate}
    >
      {children}
    </button>
  );
}

// Usage
<Tabs>
  <Tab>Tab 1</Tab>
  <Tab>Tab 2</Tab>
  <Tab>Tab 3</Tab>
</Tabs>
```

---

## Examples

### Example 1 — Card composition with children

```jsx
function Card({ children, variant = 'default' }) {
  return (
    <div className={`card card--${variant}`}>
      {children}
    </div>
  );
}

function CardHeader({ children }) {
  return <div className="card-header">{children}</div>;
}

function CardBody({ children }) {
  return <div className="card-body">{children}</div>;
}

function CardFooter({ children }) {
  return <div className="card-footer">{children}</div>;
}

// Usage
<Card variant="primary">
  <CardHeader>
    <h3>Product Title</h3>
  </CardHeader>
  <CardBody>
    <p>Product description goes here.</p>
    <img src="/product.jpg" alt="Product" />
  </CardBody>
  <CardFooter>
    <button>Add to Cart</button>
    <span>$99.99</span>
  </CardFooter>
</Card>
```

---

### Example 2 — Render props for data fetching

```jsx
function DataFetcher({ url, render }) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    fetch(url)
      .then(res => res.json())
      .then(data => {
        setData(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err);
        setLoading(false);
      });
  }, [url]);
  
  return render({ data, loading, error });
}

// Usage: Parent controls rendering
function UserList() {
  return (
    <DataFetcher 
      url="/api/users"
      render={({ data, loading, error }) => {
        if (loading) return <Spinner />;
        if (error) return <Error message={error.message} />;
        return (
          <ul>
            {data.map(user => <li key={user.id}>{user.name}</li>)}
          </ul>
        );
      }}
    />
  );
}

// Same logic, different rendering
function UserCards() {
  return (
    <DataFetcher 
      url="/api/users"
      render={({ data, loading, error }) => {
        if (loading) return <Spinner />;
        if (error) return <Error message={error.message} />;
        return (
          <div className="cards">
            {data.map(user => <UserCard key={user.id} user={user} />)}
          </div>
        );
      }}
    />
  );
}
```

---

### Example 3 — HOC for loading state

```jsx
function withLoading(Component) {
  return function LoadingComponent({ isLoading, ...props }) {
    if (isLoading) {
      return <Spinner />;
    }
    
    return <Component {...props} />;
  };
}

// Original components
function UserList({ users }) {
  return (
    <ul>
      {users.map(user => <li key={user.id}>{user.name}</li>)}
    </ul>
  );
}

function ProductGrid({ products }) {
  return (
    <div className="grid">
      {products.map(product => <ProductCard key={product.id} product={product} />)}
    </div>
  );
}

// Enhanced with loading state
const UserListWithLoading = withLoading(UserList);
const ProductGridWithLoading = withLoading(ProductGrid);

// Usage
<UserListWithLoading isLoading={loading} users={users} />
<ProductGridWithLoading isLoading={loading} products={products} />
```

---

### Example 4 — Compound components (Tabs)

```jsx
// Tabs component with context for state sharing
const TabsContext = createContext();

function Tabs({ children, defaultIndex = 0 }) {
  const [activeIndex, setActiveIndex] = useState(defaultIndex);
  
  return (
    <TabsContext.Provider value={{ activeIndex, setActiveIndex }}>
      <div className="tabs">{children}</div>
    </TabsContext.Provider>
  );
}

function TabList({ children }) {
  return <div className="tab-list">{children}</div>;
}

function Tab({ index, children }) {
  const { activeIndex, setActiveIndex } = useContext(TabsContext);
  const isActive = activeIndex === index;
  
  return (
    <button
      className={`tab ${isActive ? 'active' : ''}`}
      onClick={() => setActiveIndex(index)}
    >
      {children}
    </button>
  );
}

function TabPanels({ children }) {
  return <div className="tab-panels">{children}</div>;
}

function TabPanel({ index, children }) {
  const { activeIndex } = useContext(TabsContext);
  
  if (activeIndex !== index) return null;
  
  return <div className="tab-panel">{children}</div>;
}

// Usage
function ProductInfo() {
  return (
    <Tabs defaultIndex={0}>
      <TabList>
        <Tab index={0}>Description</Tab>
        <Tab index={1}>Specifications</Tab>
        <Tab index={2}>Reviews</Tab>
      </TabList>
      
      <TabPanels>
        <TabPanel index={0}>
          <Description />
        </TabPanel>
        <TabPanel index={1}>
          <Specifications />
        </TabPanel>
        <TabPanel index={2}>
          <Reviews />
        </TabPanel>
      </TabPanels>
    </Tabs>
  );
}
```

---

### Example 5 — Layout composition with named slots

```jsx
function DashboardLayout({ 
  header, 
  sidebar, 
  content, 
  footer 
}) {
  return (
    <div className="dashboard-layout">
      <header className="header">{header}</header>
      
      <div className="main">
        <aside className="sidebar">{sidebar}</aside>
        <main className="content">{content}</main>
      </div>
      
      <footer className="footer">{footer}</footer>
    </div>
  );
}

// Usage
<DashboardLayout
  header={
    <Header>
      <Logo />
      <Navigation />
      <UserMenu />
    </Header>
  }
  sidebar={
    <Sidebar>
      <Menu items={menuItems} />
    </Sidebar>
  }
  content={
    <Content>
      <Outlet />  {/* Router nested routes */}
    </Content>
  }
  footer={
    <Footer>
      <Copyright />
      <Links />
    </Footer>
  }
/>
```

---

### Example 6 — Composition vs inheritance

```jsx
// ❌ Bad: Using inheritance (not React way)
class Button extends React.Component {
  render() {
    return <button>{this.props.children}</button>;
  }
}

class PrimaryButton extends Button {
  render() {
    return (
      <button className="primary">
        {this.props.children}
      </button>
    );
  }
}

// ✅ Good: Using composition
function Button({ variant = 'default', children, ...props }) {
  return (
    <button 
      className={`btn btn--${variant}`}
      {...props}
    >
      {children}
    </button>
  );
}

// Usage
<Button variant="primary">Primary</Button>
<Button variant="secondary">Secondary</Button>
<Button variant="danger">Delete</Button>
```

---

## Common Pitfalls

### 1. **Not passing props down in HOCs**

```jsx
// ❌ Original props not forwarded
function withAuth(Component) {
  return function AuthenticatedComponent() {
    const { isAuthenticated } = useAuth();
    if (!isAuthenticated) return <Redirect to="/login" />;
    return <Component />;  // Missing props!
  };
}

// ✅ Forward all props
function withAuth(Component) {
  return function AuthenticatedComponent(props) {
    const { isAuthenticated } = useAuth();
    if (!isAuthenticated) return <Redirect to="/login" />;
    return <Component {...props} />;
  };
}
```

---

### 2. **Overusing render props when hooks would be simpler**

```jsx
// ❌ Render props (outdated for this use case)
<DataFetcher 
  url="/api/users"
  render={({ data, loading }) => (
    loading ? <Spinner /> : <UserList users={data} />
  )}
/>

// ✅ Custom hook (cleaner)
function useDataFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    fetch(url)
      .then(res => res.json())
      .then(data => {
        setData(data);
        setLoading(false);
      });
  }, [url]);
  
  return { data, loading };
}

function UserList() {
  const { data, loading } = useDataFetch('/api/users');
  if (loading) return <Spinner />;
  return <ul>{data.map(user => <li key={user.id}>{user.name}</li>)}</ul>;
}
```

---

### 3. **Prop collision in HOCs**

```jsx
// ❌ HOC prop collides with component prop
function withUser(Component) {
  return function(props) {
    const user = useUser();
    return <Component user={user} {...props} />;  // 'user' might be in props!
  };
}

// ✅ Namespace HOC props
function withUser(Component) {
  return function(props) {
    const currentUser = useUser();
    return <Component currentUser={currentUser} {...props} />;
  };
}
```

---

### 4. **Not handling children properly**

```jsx
// ❌ Children as array breaks when single child
function Tabs({ children }) {
  return children.map((child, index) => (  // Error if single child!
    <div key={index}>{child}</div>
  ));
}

// ✅ Use React.Children utilities
function Tabs({ children }) {
  return React.Children.map(children, (child, index) => (
    <div key={index}>{child}</div>
  ));
}
```

---

### 5. **Complex HOC composition**

```jsx
// ❌ Hard to read, debug, and reason about
const EnhancedComponent = 
  withAuth(
    withLoading(
      withData(
        withTheme(Component)
      )
    )
  );

// ✅ Use custom hooks instead
function Component(props) {
  const { isAuthenticated } = useAuth();
  const { data, loading } = useData();
  const theme = useTheme();
  
  // All logic in one place, easy to debug
}
```

---

## FAQ

**Q: When should I use children vs props?**  
A: Children for content, props for configuration/data.

**Q: Render props vs HOCs vs hooks?**  
A: Hooks for logic reuse (preferred), render props for render control, HOCs are mostly legacy.

**Q: What are compound components?**  
A: Multiple components that work together (share state via context). Example: Tabs, Select, Radio Group.

**Q: When to use composition vs Context?**  
A: Composition for parent-child relationships, Context for deeply nested data.

**Q: Can I use multiple children slots?**  
A: Yes, use named props instead of children prop.

**Q: How do I manipulate children?**  
A: Use `React.Children` utilities (`map`, `forEach`, `count`, `only`, `toArray`).

**Q: Is composition better than inheritance?**  
A: Yes in React. Composition is more flexible and avoids tight coupling.

---

## Quick Self-Check

1. What's the most basic composition pattern?
   (Children prop—allows any content inside component)

2. When would you use render props?
   (When parent needs control over what/how children render based on component's data)

3. Are HOCs still recommended?
   (Generally no—hooks are cleaner for logic reuse)

4. What are compound components?
   (Multiple components that work together, sharing state via context)

5. How do you avoid prop collision in HOCs?
   (Namespace HOC props with unique names)

---

## Further Practice

1. **Build:** Card component with Header, Body, Footer sub-components
2. **Create:** Render props component for data fetching
3. **Build:** HOC that adds loading state to any component
4. **Implement:** Compound component (Tabs, Accordion, or Select)
5. **Refactor:** Convert HOC to custom hook
6. **Build:** Layout component with named slots (header, sidebar, content, footer)
7. **Create:** Generic Modal component with flexible content

---

## Key Takeaways

- Component composition is React's answer to code reuse (not inheritance)
- Children prop enables generic container components
- Named slots (props) provide specific positions for content
- Render props give parents control over rendering based on child's data
- HOCs wrap components to add behavior (mostly legacy now)
- Custom hooks are preferred over HOCs/render props for logic reuse
- Compound components work together, sharing state via context
- React.Children utilities safely manipulate children arrays
- Composition separates concerns: logic vs presentation
- Prefer composition over inheritance in React applications
