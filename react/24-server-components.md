# React Server Components

**Level:** ⚡ React 19+ Features  
**Tags:** `react-19`, `server-components`, `rsc`, `streaming`, `data-fetching`, `architecture`

---

## Why this matters

React Server Components (RSC) represent the biggest architectural shift in React since hooks. They fundamentally change how we build React applications by enabling components to run on the server, send minimal JavaScript to the client, and stream content progressively. Understanding RSC is critical for modern React development, especially with Next.js App Router adoption.

**Why interviewers care:**
- RSC demonstrates understanding of modern React architecture
- Shows knowledge of client-server boundaries and data fetching patterns
- Tests ability to reason about performance (bundle size, waterfall, streaming)
- Reveals understanding of React's future direction
- Senior positions increasingly expect RSC knowledge

**Real-world implications:**
- **Smaller bundles:** Server components don't ship JavaScript to browser
- **Faster initial load:** Server-rendered content appears instantly
- **Direct database access:** Query databases directly in components (no API layer)
- **Better SEO:** Content is server-rendered and crawlable
- **Streaming:** Show content progressively as it's ready
- **Zero client JavaScript:** Some components need no JS on client

**Key architectural shift:**
```
Before RSC: All components run on client → fetch data → re-render
With RSC: Server components run on server → stream to client → minimal re-rendering
```

**What you must know:**
- Difference between Server Components and Client Components
- When to use each type
- How to pass data between server and client
- Server component limitations (no useState, useEffect, etc.)
- Client component limitations (can't be async, can't access server APIs)
- How streaming works
- Data fetching patterns with RSC

**Interview red flags:** Confusing RSC with SSR/SSG, not understanding client-server boundaries, or claiming to know RSC without Next.js App Router experience.

---

## Core Ideas

### 1. **Server Components vs. Client Components**

**Server Components (default in App Router):**

```jsx
// app/users/page.tsx
// This is a Server Component (default)
// Runs ONLY on the server

async function UsersPage() {
  // ✅ Can access server-only APIs
  const users = await db.query('SELECT * FROM users');
  
  // ✅ Can use environment variables
  const apiKey = process.env.SECRET_API_KEY;
  
  // ✅ Can import server-only libraries
  const bcrypt = require('bcrypt');
  
  return (
    <div>
      <h1>Users</h1>
      {users.map(user => (
        <div key={user.id}>{user.name}</div>
      ))}
    </div>
  );
}

export default UsersPage;
```

**Server Component capabilities:**
- ✅ Async/await (can be async functions)
- ✅ Direct database access
- ✅ Access server-only APIs (file system, environment variables)
- ✅ Import large libraries without affecting bundle size
- ❌ NO hooks (useState, useEffect, useContext, etc.)
- ❌ NO browser APIs (window, document, etc.)
- ❌ NO event handlers (onClick, onChange, etc.)

**Client Components (opt-in with 'use client'):**

```jsx
// components/Counter.tsx
'use client';  // Mark as Client Component

import { useState } from 'react';

function Counter() {
  // ✅ Can use hooks
  const [count, setCount] = useState(0);
  
  // ✅ Can use event handlers
  const handleClick = () => setCount(count + 1);
  
  // ✅ Can access browser APIs
  const handleSave = () => {
    localStorage.setItem('count', String(count));
  };
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={handleClick}>Increment</button>
      <button onClick={handleSave}>Save</button>
    </div>
  );
}

export default Counter;
```

**Client Component capabilities:**
- ✅ Hooks (useState, useEffect, etc.)
- ✅ Event handlers (onClick, etc.)
- ✅ Browser APIs (window, localStorage, etc.)
- ✅ Interactivity
- ❌ NO async/await in component function
- ❌ NO direct server access (database, file system)
- ❌ Increases bundle size (ships JavaScript)

---

### 2. **Composing Server and Client Components**

**Pattern 1: Server Component → Client Component (common)**

```jsx
// app/page.tsx (Server Component)
import ClientCounter from '@/components/ClientCounter';

async function HomePage() {
  // Fetch data on server
  const initialData = await fetchData();
  
  return (
    <div>
      <h1>Welcome</h1>
      {/* Pass server data to client component */}
      <ClientCounter initialCount={initialData.count} />
    </div>
  );
}
```

```jsx
// components/ClientCounter.tsx (Client Component)
'use client';

import { useState } from 'react';

function ClientCounter({ initialCount }: { initialCount: number }) {
  const [count, setCount] = useState(initialCount);
  
  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  );
}
```

**Pattern 2: Client Component wrapping Server Component (via children)**

```jsx
// components/ClientWrapper.tsx (Client Component)
'use client';

import { useState } from 'react';

function ClientWrapper({ children }: { children: React.ReactNode }) {
  const [isOpen, setIsOpen] = useState(false);
  
  return (
    <div>
      <button onClick={() => setIsOpen(!isOpen)}>Toggle</button>
      {isOpen && children}  {/* children can be Server Components */}
    </div>
  );
}
```

```jsx
// app/page.tsx (Server Component)
import ClientWrapper from '@/components/ClientWrapper';

async function HomePage() {
  const data = await fetchData();
  
  return (
    <ClientWrapper>
      {/* This is still a Server Component! */}
      <ServerContent data={data} />
    </ClientWrapper>
  );
}

async function ServerContent({ data }) {
  // Server logic here
  return <div>{data.message}</div>;
}
```

**❌ WRONG: Importing Server Component into Client Component**

```jsx
// ❌ This doesn't work!
'use client';

import ServerComponent from './ServerComponent';  // Server Component

function ClientComponent() {
  return <ServerComponent />;  // Error: Can't import Server into Client
}
```

**✅ CORRECT: Pass Server Component as children**

```jsx
// ✅ This works!
'use client';

function ClientComponent({ children }) {
  return <div>{children}</div>;  // children can be Server Component
}
```

---

### 3. **Data Fetching in Server Components**

**Direct database access:**

```jsx
// app/posts/page.tsx
import { db } from '@/lib/db';

async function PostsPage() {
  // Direct database query (no API route needed)
  const posts = await db.post.findMany({
    orderBy: { createdAt: 'desc' },
    include: { author: true }
  });
  
  return (
    <div>
      {posts.map(post => (
        <article key={post.id}>
          <h2>{post.title}</h2>
          <p>By {post.author.name}</p>
          <p>{post.content}</p>
        </article>
      ))}
    </div>
  );
}
```

**Parallel data fetching:**

```jsx
// app/dashboard/page.tsx
async function DashboardPage() {
  // Fetch in parallel
  const [user, posts, stats] = await Promise.all([
    fetchUser(),
    fetchPosts(),
    fetchStats()
  ]);
  
  return (
    <div>
      <UserProfile user={user} />
      <PostsList posts={posts} />
      <StatsWidget stats={stats} />
    </div>
  );
}
```

**Sequential data fetching (when needed):**

```jsx
async function UserPosts({ userId }: { userId: string }) {
  // First, fetch user
  const user = await fetchUser(userId);
  
  // Then, fetch posts (depends on user)
  const posts = await fetchUserPosts(user.id);
  
  return (
    <div>
      <h1>{user.name}'s Posts</h1>
      {posts.map(post => <Post key={post.id} post={post} />)}
    </div>
  );
}
```

**Caching and revalidation:**

```jsx
// app/posts/page.tsx

// Cache for 1 hour
async function getPosts() {
  const res = await fetch('https://api.example.com/posts', {
    next: { revalidate: 3600 }  // Cache for 3600 seconds
  });
  return res.json();
}

// Never cache (always fresh)
async function getLiveData() {
  const res = await fetch('https://api.example.com/live', {
    cache: 'no-store'  // No caching
  });
  return res.json();
}

// Cache forever (static)
async function getStaticData() {
  const res = await fetch('https://api.example.com/static', {
    cache: 'force-cache'  // Cache indefinitely
  });
  return res.json();
}
```

---

### 4. **Streaming with Suspense**

**Problem:** Slow data fetching blocks entire page.

**Solution:** Stream content as it's ready with Suspense.

```jsx
// app/page.tsx
import { Suspense } from 'react';

function HomePage() {
  return (
    <div>
      <h1>Dashboard</h1>
      
      {/* Fast content renders immediately */}
      <QuickStats />
      
      {/* Slow content streams in when ready */}
      <Suspense fallback={<LoadingSkeleton />}>
        <SlowContent />
      </Suspense>
      
      {/* Another slow component */}
      <Suspense fallback={<div>Loading more...</div>}>
        <AnotherSlowComponent />
      </Suspense>
    </div>
  );
}

// Fast component
function QuickStats() {
  return <div>Quick stats here (no async)</div>;
}

// Slow component (async)
async function SlowContent() {
  // Simulates slow API/database call
  await new Promise(resolve => setTimeout(resolve, 3000));
  const data = await fetchSlowData();
  
  return <div>{data.message}</div>;
}
```

**Timeline:**
1. Server sends initial HTML with `<QuickStats />` and `<LoadingSkeleton />`
2. Client sees content immediately
3. Server finishes `<SlowContent />` → streams update to client
4. Client replaces skeleton with real content (no full page reload)

**Nested Suspense boundaries:**

```jsx
function DashboardPage() {
  return (
    <div>
      <h1>Dashboard</h1>
      
      <Suspense fallback={<div>Loading user...</div>}>
        <UserProfile>
          {/* Nested boundary for even finer control */}
          <Suspense fallback={<div>Loading posts...</div>}>
            <UserPosts />
          </Suspense>
        </UserProfile>
      </Suspense>
    </div>
  );
}
```

---

### 5. **Server Actions (Mutations)**

Server Actions let you call server functions from Client Components (covered more in lesson 25).

**Basic example:**

```jsx
// app/actions.ts (Server Action)
'use server';

export async function createPost(formData: FormData) {
  const title = formData.get('title');
  const content = formData.get('content');
  
  // Server-side logic (database, validation, etc.)
  await db.post.create({
    data: { title, content }
  });
}
```

```jsx
// components/PostForm.tsx (Client Component)
'use client';

import { createPost } from '@/app/actions';

function PostForm() {
  return (
    <form action={createPost}>
      <input name="title" placeholder="Title" />
      <textarea name="content" placeholder="Content" />
      <button type="submit">Create Post</button>
    </form>
  );
}
```

---

### 6. **When to Use Server vs. Client Components**

**Use Server Components when:**
- ✅ Fetching data (databases, APIs)
- ✅ Accessing backend resources (environment variables, file system)
- ✅ Large dependencies (markdown parsers, syntax highlighters)
- ✅ Sensitive logic (API keys, authentication)
- ✅ Static content that doesn't need interactivity

**Use Client Components when:**
- ✅ Interactivity (buttons, forms, modals)
- ✅ React hooks (useState, useEffect, etc.)
- ✅ Browser APIs (localStorage, window, etc.)
- ✅ Event handlers (onClick, onChange, etc.)
- ✅ Third-party libraries that use hooks

**Decision tree:**

```
Does it need interactivity? 
  ├─ YES → Client Component
  └─ NO → Server Component
  
Does it use hooks?
  ├─ YES → Client Component
  └─ NO → Server Component
  
Does it fetch data?
  ├─ YES → Server Component (preferred)
  └─ NO → Either (default to Server)
  
Does it need browser APIs?
  ├─ YES → Client Component
  └─ NO → Server Component
```

**Example: Splitting a component:**

```jsx
// ❌ Before: Everything is a Client Component
'use client';

function ProductPage({ productId }) {
  const [quantity, setQuantity] = useState(1);
  const [product, setProduct] = useState(null);
  
  useEffect(() => {
    fetch(`/api/products/${productId}`)
      .then(res => res.json())
      .then(setProduct);
  }, [productId]);
  
  if (!product) return <div>Loading...</div>;
  
  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
      <p>${product.price}</p>
      
      <input 
        type="number" 
        value={quantity}
        onChange={(e) => setQuantity(Number(e.target.value))}
      />
      <button onClick={() => addToCart(product, quantity)}>
        Add to Cart
      </button>
    </div>
  );
}
```

```jsx
// ✅ After: Split into Server + Client Components

// app/products/[id]/page.tsx (Server Component)
import AddToCartButton from '@/components/AddToCartButton';

async function ProductPage({ params }: { params: { id: string } }) {
  // Fetch on server (no useEffect needed)
  const product = await db.product.findUnique({
    where: { id: params.id }
  });
  
  return (
    <div>
      {/* Static content (Server Component) */}
      <h1>{product.name}</h1>
      <p>{product.description}</p>
      <p>${product.price}</p>
      
      {/* Interactive part (Client Component) */}
      <AddToCartButton product={product} />
    </div>
  );
}

// components/AddToCartButton.tsx (Client Component)
'use client';

import { useState } from 'react';

function AddToCartButton({ product }) {
  const [quantity, setQuantity] = useState(1);
  
  return (
    <div>
      <input 
        type="number" 
        value={quantity}
        onChange={(e) => setQuantity(Number(e.target.value))}
      />
      <button onClick={() => addToCart(product, quantity)}>
        Add to Cart
      </button>
    </div>
  );
}
```

**Benefits of splitting:**
- Smaller bundle (product info doesn't ship JS)
- Better SEO (product info is server-rendered)
- Faster initial load (less JS to parse)
- Interactive parts still work (Client Component)

---

### 7. **Common Patterns**

**Pattern: Layout with Client and Server Components**

```jsx
// app/layout.tsx (Server Component)
import Header from '@/components/Header';
import Sidebar from '@/components/Sidebar';

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <Header />  {/* Client Component (has nav menu) */}
        <div className="container">
          <Sidebar />  {/* Client Component (collapsible) */}
          <main>{children}</main>  {/* Server Component (pages) */}
        </div>
      </body>
    </html>
  );
}
```

**Pattern: Loading and error boundaries**

```jsx
// app/posts/loading.tsx
export default function Loading() {
  return <div>Loading posts...</div>;
}

// app/posts/error.tsx
'use client';  // Error boundaries must be Client Components

export default function Error({ error, reset }) {
  return (
    <div>
      <h2>Something went wrong!</h2>
      <p>{error.message}</p>
      <button onClick={reset}>Try again</button>
    </div>
  );
}

// app/posts/page.tsx
async function PostsPage() {
  const posts = await fetchPosts();  // Can throw error
  return <div>{/* posts */}</div>;
}
```

**Pattern: Conditional rendering with streaming**

```jsx
function DashboardPage() {
  return (
    <div>
      <h1>Dashboard</h1>
      
      {/* Show premium content only for premium users */}
      <Suspense fallback={<div>Checking access...</div>}>
        <PremiumContent />
      </Suspense>
    </div>
  );
}

async function PremiumContent() {
  const user = await getCurrentUser();
  
  if (!user.isPremium) {
    return <div>Upgrade to premium to see this content</div>;
  }
  
  const premiumData = await fetchPremiumData();
  return <div>{premiumData.content}</div>;
}
```

---

## Common Pitfalls

### ❌ **Pitfall 1: Using hooks in Server Components**

```jsx
// ❌ This doesn't work!
async function ServerComponent() {
  const [state, setState] = useState(0);  // Error: Can't use hooks
  return <div>{state}</div>;
}

// ✅ Make it a Client Component
'use client';

function ClientComponent() {
  const [state, setState] = useState(0);
  return <div>{state}</div>;
}
```

---

### ❌ **Pitfall 2: Importing Server Component into Client Component**

```jsx
// ❌ This doesn't work!
'use client';

import ServerComp from './ServerComp';  // Error!

function ClientComp() {
  return <ServerComp />;
}

// ✅ Pass as children instead
'use client';

function ClientComp({ children }) {
  return <div>{children}</div>;
}
```

---

### ❌ **Pitfall 3: Serialization errors (passing non-serializable props)**

```jsx
// ❌ Can't pass functions from Server to Client
async function ServerComponent() {
  const handleClick = () => console.log('clicked');
  
  return <ClientComponent onClick={handleClick} />;  // Error!
}

// ✅ Define function in Client Component
async function ServerComponent() {
  return <ClientComponent />;
}

'use client';
function ClientComponent() {
  const handleClick = () => console.log('clicked');
  return <button onClick={handleClick}>Click</button>;
}
```

---

### ❌ **Pitfall 4: Not using Suspense for async Server Components**

```jsx
// ❌ Entire page waits for slow data
async function Page() {
  const slowData = await fetchSlowData();  // Blocks everything
  return <div>{slowData}</div>;
}

// ✅ Use Suspense to stream slow parts
function Page() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <SlowComponent />
    </Suspense>
  );
}

async function SlowComponent() {
  const slowData = await fetchSlowData();
  return <div>{slowData}</div>;
}
```

---

## Quick Self-Check

✅ **You understand Server Components if you can:**

1. Explain the difference between Server and Client Components
2. Know when to use 'use client' directive
3. Fetch data directly in Server Components
4. Compose Server and Client Components correctly
5. Use Suspense for streaming
6. Understand serialization constraints (can't pass functions, etc.)
7. Split components to minimize client bundle
8. Use loading.tsx and error.tsx files
9. Explain how streaming improves UX
10. Understand the benefits and limitations of RSC

---

## Interview Questions to Practice

**Beginner:**
1. What's the difference between Server Components and Client Components?
2. How do you make a component a Client Component?
3. Can you use useState in a Server Component?

**Intermediate:**
4. How do you pass data from a Server Component to a Client Component?
5. How does streaming work with Suspense?
6. When should you use a Server Component vs. a Client Component?
7. What props can you pass from Server to Client Components?

**Advanced:**
8. How would you optimize a page with fast and slow data?
9. Explain how to compose Client Components with Server Component children
10. How does RSC reduce bundle size compared to traditional client-side React?

---

## Further Practice

**Build these:**
1. **Blog with RSC** - Server-rendered posts, client-side comments
2. **Dashboard** - Streaming data with multiple Suspense boundaries
3. **E-commerce product page** - Server content + client cart functionality
4. **Search with filters** - Server results + client filter UI

**Resources:**
- [React Server Components docs](https://react.dev/reference/react/use-server)
- [Next.js App Router docs](https://nextjs.org/docs/app)
- [Server Components RFC](https://github.com/reactjs/rfcs/blob/main/text/0188-server-components.md)

---

## Summary

**Server Components:**
- Run only on server
- Can be async
- Direct database/API access
- No hooks, no interactivity
- Don't ship JavaScript to client
- Default in Next.js App Router

**Client Components:**
- Run on client (and server for SSR)
- Use 'use client' directive
- Can use hooks and event handlers
- Ship JavaScript to client
- Need browser APIs

**Key patterns:**
- Server Components for data fetching
- Client Components for interactivity
- Pass Server Components as children to Client Components
- Use Suspense for streaming
- Split components to minimize client bundle

**Benefits:**
- Smaller bundles (Server Components ship no JS)
- Faster loads (streaming + server rendering)
- Better SEO (server-rendered content)
- Direct server access (no API layer needed)

**Remember:** Default to Server Components, add 'use client' only when you need hooks, events, or browser APIs.
