# Next.js and React Frameworks

**Level:** 🔥 Advanced  
**Tags:** `nextjs`, `app-router`, `ssr`, `ssg`, `isr`, `server-components`, `routing`, `middleware`, `deployment`

---

## Why this matters

Next.js is the most popular React framework, used by companies like Netflix, TikTok, Twitch, and Nike. Interviewers use Next.js questions to assess your understanding of production-grade React applications, server-side rendering, performance optimization, and full-stack development. Next.js knowledge separates frontend developers from full-stack engineers who can build scalable, SEO-friendly, performant web applications.

**Why interviewers care:**
- Next.js is the industry standard for production React apps (most jobs use it)
- Shows understanding of SSR, SSG, ISR, and when to use each
- Tests knowledge of React Server Components and modern React patterns
- Demonstrates SEO and performance optimization skills
- Senior roles expect experience with routing, middleware, and deployment
- Full-stack roles expect API routes and backend integration knowledge

**Real-world implications:**
- **SEO:** Server-side rendering makes content crawlable by search engines
- **Performance:** Static generation delivers pre-built HTML (instant loads)
- **User experience:** Streaming SSR shows content as it loads
- **Developer experience:** File-based routing, automatic code splitting, TypeScript
- **Production readiness:** Built-in optimizations, image optimization, caching
- **Scalability:** Edge runtime, middleware, incremental static regeneration

**Common problems Next.js solves:**
- SEO (content hidden in client-side JavaScript)
- Slow page loads (large client bundles)
- Complex routing (manual route configuration)
- Image optimization (lazy loading, responsive images, modern formats)
- API routes (no separate backend needed)
- Deployment complexity (zero-config deployments)

**What you must know:**
- App Router vs. Pages Router (new vs. old)
- Server Components vs. Client Components
- SSR (Server-Side Rendering)
- SSG (Static Site Generation)
- ISR (Incremental Static Regeneration)
- File-based routing
- Data fetching patterns
- Middleware
- Deployment (Vercel, self-hosting)

**Interview red flags:** Only knowing Pages Router, not understanding Server Components, confusing SSR/SSG/ISR, or not knowing deployment strategies.

---

## Core Ideas

### 1. **Next.js App Router (New - React 18+)**

**File-based routing:**

```
app/
├── layout.tsx          → Root layout (wraps all pages)
├── page.tsx            → Home page (/)
├── about/
│   └── page.tsx        → /about
├── blog/
│   ├── page.tsx        → /blog
│   └── [slug]/
│       └── page.tsx    → /blog/:slug (dynamic route)
└── dashboard/
    ├── layout.tsx      → Dashboard layout
    ├── page.tsx        → /dashboard
    └── settings/
        └── page.tsx    → /dashboard/settings
```

**Basic page:**

```tsx
// app/page.tsx
export default function HomePage() {
  return (
    <div>
      <h1>Welcome to Next.js</h1>
      <p>This is the home page</p>
    </div>
  );
}
```

**Layouts (shared UI):**

```tsx
// app/layout.tsx (Root layout - wraps entire app)
export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <body>
        <header>My Website</header>
        <nav>
          <a href="/">Home</a>
          <a href="/about">About</a>
        </nav>
        <main>{children}</main>
        <footer>© 2024</footer>
      </body>
    </html>
  );
}
```

```tsx
// app/dashboard/layout.tsx (Nested layout)
export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <div className="dashboard">
      <aside>
        <DashboardNav />
      </aside>
      <main>{children}</main>
    </div>
  );
}
```

**Dynamic routes:**

```tsx
// app/blog/[slug]/page.tsx
interface PageProps {
  params: { slug: string }
}

export default function BlogPost({ params }: PageProps) {
  return <h1>Blog Post: {params.slug}</h1>;
}

// URLs:
// /blog/hello-world → params.slug = "hello-world"
// /blog/react-tips → params.slug = "react-tips"
```

**Catch-all routes:**

```tsx
// app/docs/[...slug]/page.tsx
interface PageProps {
  params: { slug: string[] }
}

export default function DocsPage({ params }: PageProps) {
  return <h1>Docs: {params.slug.join('/')}</h1>;
}

// URLs:
// /docs/getting-started → params.slug = ["getting-started"]
// /docs/api/reference → params.slug = ["api", "reference"]
```

---

### 2. **Server Components vs. Client Components**

**Server Components (default in App Router):**

```tsx
// app/posts/page.tsx
// This runs ONLY on the server!
export default async function PostsPage() {
  // Direct database access (no API needed)
  const posts = await db.query('SELECT * FROM posts');
  
  return (
    <div>
      <h1>Posts</h1>
      {posts.map(post => (
        <article key={post.id}>
          <h2>{post.title}</h2>
          <p>{post.content}</p>
        </article>
      ))}
    </div>
  );
}
```

**Benefits:**
- Zero JavaScript sent to client (smaller bundles)
- Direct database/API access (no need for API routes)
- Automatic code splitting
- Better SEO (fully rendered HTML)

**Limitations:**
- No useState, useEffect, event handlers
- Can't use browser APIs
- Can't use React hooks (except use() in React 19)

**Client Components (interactive):**

```tsx
'use client';  // This directive marks it as Client Component

import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

**When to use Client Components:**
- Need interactivity (onClick, onChange, etc.)
- Need React hooks (useState, useEffect, etc.)
- Need browser APIs (localStorage, window, etc.)
- Need to subscribe to events

**Composition pattern (mix Server + Client):**

```tsx
// app/dashboard/page.tsx (Server Component)
import { getUser } from '@/lib/auth';
import { ClientCounter } from './ClientCounter';

export default async function DashboardPage() {
  // Runs on server
  const user = await getUser();
  const stats = await fetchStats();
  
  return (
    <div>
      <h1>Welcome, {user.name}</h1>
      <p>Total visits: {stats.visits}</p>
      
      {/* Interactive component */}
      <ClientCounter initialCount={stats.clicks} />
    </div>
  );
}
```

```tsx
// app/dashboard/ClientCounter.tsx (Client Component)
'use client';

import { useState } from 'react';

export function ClientCounter({ initialCount }: { initialCount: number }) {
  const [count, setCount] = useState(initialCount);
  
  return (
    <button onClick={() => setCount(count + 1)}>
      Clicks: {count}
    </button>
  );
}
```

**Key principle:** Use Server Components by default, Client Components only when needed.

---

### 3. **Data Fetching Patterns**

**SSG (Static Site Generation) - Default:**

```tsx
// app/blog/page.tsx
// Renders at build time
export default async function BlogPage() {
  const posts = await fetch('https://api.example.com/posts').then(r => r.json());
  
  return (
    <div>
      {posts.map(post => <Post key={post.id} {...post} />)}
    </div>
  );
}

// Build time:
// Next.js fetches data → renders HTML → saves to disk
// Runtime: Serves pre-built HTML (instant!)
```

**When to use SSG:**
- Content doesn't change often (blog, docs, marketing pages)
- Same content for all users
- SEO is critical
- Best performance (pre-built HTML)

**SSR (Server-Side Rendering) - Dynamic:**

```tsx
// app/profile/page.tsx
// Renders on every request
export default async function ProfilePage() {
  const user = await getUser();  // Dynamic data
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>Last login: {user.lastLogin}</p>
    </div>
  );
}

// Opt out of caching (force SSR)
export const dynamic = 'force-dynamic';

// Or use cookies/headers (automatically SSR)
import { cookies } from 'next/headers';

export default async function Page() {
  const token = cookies().get('token');  // Uses cookies → SSR
}
```

**When to use SSR:**
- Personalized content (user-specific data)
- Real-time data (stock prices, live scores)
- Content changes frequently
- Need request data (cookies, headers)

**ISR (Incremental Static Regeneration):**

```tsx
// app/products/[id]/page.tsx
export default async function ProductPage({ params }: { params: { id: string } }) {
  const product = await fetch(`https://api.example.com/products/${params.id}`, {
    next: { revalidate: 60 }  // Revalidate every 60 seconds
  }).then(r => r.json());
  
  return (
    <div>
      <h1>{product.name}</h1>
      <p>${product.price}</p>
    </div>
  );
}

// Build time: Pre-renders first 100 products
export async function generateStaticParams() {
  const products = await fetch('https://api.example.com/products?limit=100')
    .then(r => r.json());
  
  return products.map(product => ({
    id: product.id.toString()
  }));
}
```

**How ISR works:**
1. Build time: Pre-renders popular pages
2. First request: Serves cached page (fast!)
3. After 60s: Next request triggers regeneration in background
4. Subsequent requests: Still serve old page (fast!)
5. Regeneration completes: New page cached
6. Next request: Serves new page

**When to use ISR:**
- E-commerce (product pages change occasionally)
- News sites (articles update periodically)
- Large sites (can't pre-render all pages)
- Balance between SSG speed and SSR freshness

---

### 4. **Client-Side Fetching (SWR / React Query)**

**For data that changes frequently or is user-specific:**

```tsx
'use client';

import useSWR from 'swr';

const fetcher = (url: string) => fetch(url).then(r => r.json());

export default function Dashboard() {
  const { data, error, isLoading } = useSWR('/api/user', fetcher, {
    refreshInterval: 3000  // Revalidate every 3s
  });
  
  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error loading data</div>;
  
  return (
    <div>
      <h1>Hello, {data.name}</h1>
      <p>Balance: ${data.balance}</p>
    </div>
  );
}
```

**When to use client-side fetching:**
- Real-time updates (chat, notifications)
- User interactions (infinite scroll, search)
- Frequently changing data
- No SEO needed

---

### 5. **API Routes (Server Functions)**

**REST API:**

```tsx
// app/api/posts/route.ts
import { NextRequest, NextResponse } from 'next/server';

// GET /api/posts
export async function GET(request: NextRequest) {
  const posts = await db.posts.findMany();
  return NextResponse.json(posts);
}

// POST /api/posts
export async function POST(request: NextRequest) {
  const body = await request.json();
  const post = await db.posts.create({ data: body });
  return NextResponse.json(post, { status: 201 });
}
```

**Dynamic routes:**

```tsx
// app/api/posts/[id]/route.ts
export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const post = await db.posts.findUnique({ where: { id: params.id } });
  
  if (!post) {
    return NextResponse.json({ error: 'Not found' }, { status: 404 });
  }
  
  return NextResponse.json(post);
}

export async function DELETE(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  await db.posts.delete({ where: { id: params.id } });
  return NextResponse.json({ success: true });
}
```

**Server Actions (React 19 - better than API routes for mutations):**

```tsx
// app/actions.ts
'use server';

export async function createPost(formData: FormData) {
  const title = formData.get('title') as string;
  const content = formData.get('content') as string;
  
  const post = await db.posts.create({
    data: { title, content }
  });
  
  revalidatePath('/blog');  // Revalidate blog page
  return { success: true, post };
}
```

```tsx
// app/blog/new/page.tsx
import { createPost } from '@/app/actions';

export default function NewPostPage() {
  return (
    <form action={createPost}>
      <input name="title" placeholder="Title" required />
      <textarea name="content" placeholder="Content" required />
      <button type="submit">Create Post</button>
    </form>
  );
}
```

---

### 6. **Middleware (Edge Runtime)**

**Run code before request completes:**

```tsx
// middleware.ts (root of project)
import { NextRequest, NextResponse } from 'next/server';

export function middleware(request: NextRequest) {
  const token = request.cookies.get('token');
  
  // Redirect if not authenticated
  if (!token && request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.redirect(new URL('/login', request.url));
  }
  
  // Add custom header
  const response = NextResponse.next();
  response.headers.set('x-custom-header', 'my-value');
  
  return response;
}

// Run middleware on specific paths
export const config = {
  matcher: ['/dashboard/:path*', '/api/:path*']
};
```

**Use cases:**
- Authentication/authorization
- Redirects (A/B testing, localization)
- Rewriting URLs
- Setting headers/cookies
- Bot detection
- Rate limiting

---

### 7. **Image Optimization**

```tsx
import Image from 'next/image';

export default function ProductPage() {
  return (
    <div>
      <Image
        src="/product.jpg"
        alt="Product image"
        width={800}
        height={600}
        priority  // Load immediately (above the fold)
      />
      
      <Image
        src="/thumbnail.jpg"
        alt="Thumbnail"
        width={200}
        height={200}
        loading="lazy"  // Lazy load (below the fold)
      />
      
      {/* Remote images */}
      <Image
        src="https://example.com/image.jpg"
        alt="Remote image"
        width={800}
        height={600}
        // next.config.js must allow domain
      />
    </div>
  );
}
```

```js
// next.config.js
module.exports = {
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'example.com',
      },
    ],
  },
};
```

**Benefits:**
- Automatic lazy loading
- Responsive images (srcset)
- Modern formats (WebP, AVIF)
- Prevents layout shift
- Optimizes on-demand

---

### 8. **Deployment**

**Vercel (zero-config):**

```bash
# Install Vercel CLI
npm i -g vercel

# Deploy
vercel

# Production
vercel --prod
```

**Self-hosting (Node.js):**

```bash
# Build
npm run build

# Start production server
npm run start
```

**Docker:**

```dockerfile
# Dockerfile
FROM node:18-alpine

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY . .
RUN npm run build

EXPOSE 3000
CMD ["npm", "start"]
```

**Environment variables:**

```bash
# .env.local (development)
DATABASE_URL=postgresql://localhost/mydb
API_KEY=abc123

# Vercel (production)
# Add in Vercel dashboard → Settings → Environment Variables
```

```tsx
// Access in code
const dbUrl = process.env.DATABASE_URL;
```

---

### 9. **Performance Optimization**

**Route Groups (organize without affecting URL):**

```
app/
├── (marketing)/
│   ├── layout.tsx     → Marketing layout
│   ├── page.tsx       → / (home)
│   └── about/
│       └── page.tsx   → /about
└── (shop)/
    ├── layout.tsx     → Shop layout
    └── products/
        └── page.tsx   → /products

// Parentheses don't appear in URL!
```

**Parallel Routes (multiple sections on same page):**

```
app/
└── dashboard/
    ├── @analytics/
    │   └── page.tsx
    ├── @notifications/
    │   └── page.tsx
    └── layout.tsx
```

```tsx
// app/dashboard/layout.tsx
export default function DashboardLayout({
  children,
  analytics,
  notifications,
}: {
  children: React.ReactNode
  analytics: React.ReactNode
  notifications: React.ReactNode
}) {
  return (
    <div>
      {children}
      <aside>
        {analytics}
        {notifications}
      </aside>
    </div>
  );
}
```

**Streaming with Suspense:**

```tsx
import { Suspense } from 'react';

export default function DashboardPage() {
  return (
    <div>
      <h1>Dashboard</h1>
      
      <Suspense fallback={<AnalyticsSkeleton />}>
        <Analytics />  {/* Async Server Component */}
      </Suspense>
      
      <Suspense fallback={<NotificationsSkeleton />}>
        <Notifications />  {/* Loads independently */}
      </Suspense>
    </div>
  );
}

async function Analytics() {
  const data = await fetchAnalytics();  // Slow query
  return <div>{/* ... */}</div>;
}
```

---

## Common Pitfalls

### ❌ **Pitfall 1: Forgetting 'use client' directive**

```tsx
// BAD: Server Component can't use useState
export default function Counter() {
  const [count, setCount] = useState(0);  // Error!
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}

// GOOD: Mark as Client Component
'use client';

export default function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

---

### ❌ **Pitfall 2: Passing Server Components to Client Components**

```tsx
// BAD: Can't pass Server Component as children
'use client';

export function ClientWrapper({ children }: { children: React.ReactNode }) {
  const [state, setState] = useState();
  return <div>{children}</div>;  // children is Server Component - breaks!
}

// GOOD: Composition in parent (Server Component)
export default function Page() {
  return (
    <ClientWrapper>
      <ServerComponent />  {/* Works! */}
    </ClientWrapper>
  );
}
```

---

### ❌ **Pitfall 3: Not configuring remote images**

```tsx
// BAD: Remote image without config
<Image src="https://example.com/image.jpg" width={800} height={600} />
// Error: Invalid src prop

// GOOD: Add domain to next.config.js
// next.config.js
module.exports = {
  images: {
    remotePatterns: [{ protocol: 'https', hostname: 'example.com' }]
  }
};
```

---

## Quick Self-Check

✅ **You understand Next.js if you can:**

1. Explain SSG vs. SSR vs. ISR
2. Use Server Components and Client Components correctly
3. Implement file-based routing with dynamic routes
4. Fetch data in Server Components
5. Create API routes
6. Use middleware for authentication
7. Optimize images with next/image
8. Deploy to Vercel and self-host
9. Implement Server Actions
10. Stream content with Suspense

---

## Interview Questions to Practice

**Beginner:**
1. What is Next.js and why use it over Create React App?
2. What's the difference between SSG and SSR?
3. How does file-based routing work?

**Intermediate:**
4. When would you use ISR over SSG?
5. Explain Server Components vs. Client Components
6. How do you handle authentication in Next.js?
7. What is middleware used for?

**Advanced:**
8. How would you architect a large e-commerce site in Next.js?
9. Explain the rendering timeline with streaming SSR
10. How do you optimize Next.js for Core Web Vitals?

---

## Further Practice

**Build these:**
1. **Blog** with SSG + ISR (markdown posts)
2. **E-commerce store** (product pages, cart, checkout)
3. **Dashboard** with authentication (middleware + Server Actions)
4. **Multi-tenant app** (different layouts per tenant)
5. **Real-time chat** (Server Components + client-side updates)

**Resources:**
- [Next.js Docs](https://nextjs.org/docs)
- [Next.js Learn](https://nextjs.org/learn)
- [Vercel Examples](https://github.com/vercel/next.js/tree/canary/examples)

---

## Summary

**Next.js solves:**
- SEO (server-rendered HTML)
- Performance (static generation, streaming)
- Developer experience (file-based routing, zero config)
- Full-stack development (API routes, Server Actions)

**Key concepts:**
- **App Router:** File-based routing, layouts, Server Components
- **SSG:** Pre-render at build time (best performance)
- **SSR:** Render on each request (dynamic content)
- **ISR:** Hybrid (static + periodic updates)
- **Server Components:** Zero client JS, direct backend access
- **Client Components:** Interactive, React hooks, browser APIs

**When to use what:**
- Marketing pages → SSG
- User dashboards → SSR
- Product pages → ISR
- Real-time data → Client-side fetching

**Remember:** Start with Server Components (default), use Client Components only when needed for interactivity. Choose rendering strategy based on data freshness requirements.
