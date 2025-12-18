# Server Actions

**Level:** ⚡ React 19+ Features  
**Tags:** `react-19`, `server-actions`, `forms`, `mutations`, `progressive-enhancement`

---

## Why this matters

Server Actions are React 19's solution for handling mutations (creating, updating, deleting data) with server-side execution, type safety, and progressive enhancement. They eliminate the need for API routes, simplify form handling, and work even when JavaScript is disabled. Understanding Server Actions is essential for modern full-stack React development.

**Why interviewers care:**
- Server Actions represent modern full-stack React architecture
- Shows understanding of client-server boundaries and security
- Tests knowledge of progressive enhancement and accessibility
- Reveals ability to build production-ready forms without external libraries
- Demonstrates understanding of React 19's new form capabilities

**Real-world implications:**
- **No API routes needed:** Call server functions directly from components
- **Type safety:** End-to-end TypeScript from client to server
- **Progressive enhancement:** Forms work without JavaScript
- **Better UX:** Automatic loading states, error handling, optimistic updates
- **Security:** Sensitive logic stays on server (never exposed to client)
- **Simpler code:** Less boilerplate than traditional API routes

**Key mental model:**
```
Before: Component → fetch('/api/endpoint') → API Route → Database
With Server Actions: Component → Server Action → Database
```

**What you must know:**
- What Server Actions are and how they differ from API routes
- How to define Server Actions ('use server')
- How to call Server Actions from forms and event handlers
- How to handle loading states and errors
- Progressive enhancement patterns
- Validation (client and server)
- Integration with useActionState and useOptimistic
- Security considerations

**Interview red flags:** Not understanding security implications, exposing sensitive server logic to client, or not knowing when to use Server Actions vs. API routes.

---

## Core Ideas

### 1. **What are Server Actions?**

**Server Actions** are asynchronous functions that run on the server but can be called from Client Components.

**Basic example:**

```jsx
// app/actions.ts
'use server';  // Mark file as containing Server Actions

export async function createTodo(formData: FormData) {
  // This runs on the SERVER
  const title = formData.get('title') as string;
  
  // Direct database access (server-only)
  await db.todo.create({
    data: { title, completed: false }
  });
  
  // Revalidate cache
  revalidatePath('/todos');
}
```

```jsx
// components/TodoForm.tsx
'use client';

import { createTodo } from '@/app/actions';

export function TodoForm() {
  return (
    <form action={createTodo}>
      <input name="title" placeholder="New todo" required />
      <button type="submit">Add</button>
    </form>
  );
}
```

**How it works:**
1. User submits form
2. React serializes form data
3. Sends request to server
4. Server executes `createTodo` function
5. Server returns result
6. Client receives response and updates UI

---

### 2. **Defining Server Actions**

**Method 1: Separate file with 'use server' at top**

```js
// app/actions.ts
'use server';

// All functions in this file are Server Actions
export async function createPost(formData: FormData) {
  // Server logic
}

export async function deletePost(id: string) {
  // Server logic
}

export async function updatePost(id: string, data: any) {
  // Server logic
}
```

**Method 2: Inline with 'use server' inside function**

```jsx
// app/posts/page.tsx (Server Component)
export default function PostsPage() {
  // Define Server Action inline
  async function createPost(formData: FormData) {
    'use server';  // Mark this function as Server Action
    
    const title = formData.get('title') as string;
    await db.post.create({ data: { title } });
    revalidatePath('/posts');
  }
  
  return (
    <form action={createPost}>
      <input name="title" />
      <button type="submit">Create</button>
    </form>
  );
}
```

**Method 3: Inside Server Component (can't be exported)**

```jsx
// app/profile/page.tsx
async function ProfilePage() {
  // Inline Server Action (only usable in this component)
  async function updateProfile(formData: FormData) {
    'use server';
    
    const name = formData.get('name') as string;
    await updateUser({ name });
  }
  
  return (
    <form action={updateProfile}>
      <input name="name" />
      <button type="submit">Save</button>
    </form>
  );
}
```

---

### 3. **Calling Server Actions**

**From forms (native form action):**

```jsx
'use client';

import { createTodo } from '@/app/actions';

export function TodoForm() {
  return (
    <form action={createTodo}>
      <input name="title" required />
      <input name="description" />
      <button type="submit">Create</button>
    </form>
  );
}
```

**From event handlers:**

```jsx
'use client';

import { deleteTodo } from '@/app/actions';

export function TodoItem({ todo }) {
  const handleDelete = async () => {
    await deleteTodo(todo.id);
  };
  
  return (
    <div>
      <span>{todo.title}</span>
      <button onClick={handleDelete}>Delete</button>
    </div>
  );
}
```

**With custom arguments (bind):**

```jsx
// app/actions.ts
'use server';

export async function deleteTodo(id: string) {
  await db.todo.delete({ where: { id } });
  revalidatePath('/todos');
}
```

```jsx
// components/TodoItem.tsx
'use client';

import { deleteTodo } from '@/app/actions';

export function TodoItem({ todo }) {
  // Bind todo.id to the Server Action
  const deleteTodoWithId = deleteTodo.bind(null, todo.id);
  
  return (
    <form action={deleteTodoWithId}>
      <span>{todo.title}</span>
      <button type="submit">Delete</button>
    </form>
  );
}
```

**Programmatic call with startTransition:**

```jsx
'use client';

import { useTransition } from 'react';
import { createTodo } from '@/app/actions';

export function TodoForm() {
  const [isPending, startTransition] = useTransition();
  
  const handleSubmit = async (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    const formData = new FormData(e.currentTarget);
    
    startTransition(async () => {
      await createTodo(formData);
      e.currentTarget.reset();
    });
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input name="title" required />
      <button type="submit" disabled={isPending}>
        {isPending ? 'Creating...' : 'Create'}
      </button>
    </form>
  );
}
```

---

### 4. **Form State with useActionState (React 19)**

**useActionState** manages form state, loading, and errors automatically.

```jsx
// app/actions.ts
'use server';

export async function createTodo(prevState: any, formData: FormData) {
  const title = formData.get('title') as string;
  
  // Validation
  if (!title || title.length < 3) {
    return { error: 'Title must be at least 3 characters' };
  }
  
  try {
    await db.todo.create({ data: { title } });
    revalidatePath('/todos');
    return { success: true };
  } catch (error) {
    return { error: 'Failed to create todo' };
  }
}
```

```jsx
// components/TodoForm.tsx
'use client';

import { useActionState } from 'react';
import { createTodo } from '@/app/actions';

export function TodoForm() {
  const [state, formAction, isPending] = useActionState(
    createTodo,
    { error: null }  // Initial state
  );
  
  return (
    <form action={formAction}>
      <input name="title" required />
      
      {state?.error && (
        <p className="error">{state.error}</p>
      )}
      
      {state?.success && (
        <p className="success">Todo created!</p>
      )}
      
      <button type="submit" disabled={isPending}>
        {isPending ? 'Creating...' : 'Create'}
      </button>
    </form>
  );
}
```

**useActionState return values:**
- `state` - Current state (return value from Server Action)
- `formAction` - Function to pass to form's `action` prop
- `isPending` - Boolean indicating if action is in progress

---

### 5. **Validation**

**Server-side validation (required for security):**

```jsx
// app/actions.ts
'use server';

import { z } from 'zod';

const todoSchema = z.object({
  title: z.string().min(3, 'Title must be at least 3 characters'),
  description: z.string().optional()
});

export async function createTodo(prevState: any, formData: FormData) {
  // Parse and validate
  const result = todoSchema.safeParse({
    title: formData.get('title'),
    description: formData.get('description')
  });
  
  if (!result.success) {
    return {
      error: result.error.flatten().fieldErrors
    };
  }
  
  // Create todo with validated data
  await db.todo.create({
    data: result.data
  });
  
  revalidatePath('/todos');
  return { success: true };
}
```

```jsx
// components/TodoForm.tsx
'use client';

import { useActionState } from 'react';
import { createTodo } from '@/app/actions';

export function TodoForm() {
  const [state, formAction, isPending] = useActionState(createTodo, {});
  
  return (
    <form action={formAction}>
      <div>
        <input name="title" required />
        {state?.error?.title && (
          <span className="error">{state.error.title[0]}</span>
        )}
      </div>
      
      <div>
        <textarea name="description" />
        {state?.error?.description && (
          <span className="error">{state.error.description[0]}</span>
        )}
      </div>
      
      <button type="submit" disabled={isPending}>
        {isPending ? 'Creating...' : 'Create'}
      </button>
    </form>
  );
}
```

**Client-side validation (for UX, not security):**

```jsx
'use client';

import { useActionState } from 'react';
import { createTodo } from '@/app/actions';

export function TodoForm() {
  const [state, formAction, isPending] = useActionState(createTodo, {});
  const [clientError, setClientError] = useState('');
  
  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    const formData = new FormData(e.currentTarget);
    const title = formData.get('title') as string;
    
    // Client-side validation (instant feedback)
    if (!title || title.length < 3) {
      e.preventDefault();
      setClientError('Title must be at least 3 characters');
      return;
    }
    
    setClientError('');
    // Let form submit to Server Action
  };
  
  return (
    <form action={formAction} onSubmit={handleSubmit}>
      <input 
        name="title" 
        onChange={() => setClientError('')}
        required 
      />
      {clientError && <span className="error">{clientError}</span>}
      {state?.error && <span className="error">{state.error}</span>}
      
      <button type="submit" disabled={isPending}>Create</button>
    </form>
  );
}
```

---

### 6. **Optimistic Updates with useOptimistic**

**useOptimistic** lets you show immediate UI updates while Server Action is pending.

```jsx
'use client';

import { useOptimistic } from 'react';
import { createTodo } from '@/app/actions';

export function TodoList({ initialTodos }) {
  const [optimisticTodos, addOptimisticTodo] = useOptimistic(
    initialTodos,
    (state, newTodo) => [...state, newTodo]
  );
  
  const handleSubmit = async (formData: FormData) => {
    const title = formData.get('title') as string;
    
    // Add optimistic todo immediately (instant UI update)
    addOptimisticTodo({
      id: crypto.randomUUID(),
      title,
      completed: false,
      isPending: true  // Mark as pending
    });
    
    // Send to server (runs in background)
    await createTodo(formData);
  };
  
  return (
    <div>
      <form action={handleSubmit}>
        <input name="title" required />
        <button type="submit">Add</button>
      </form>
      
      <ul>
        {optimisticTodos.map(todo => (
          <li 
            key={todo.id}
            className={todo.isPending ? 'pending' : ''}
          >
            {todo.title}
            {todo.isPending && <span> (Saving...)</span>}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

**Complete example with delete:**

```jsx
'use client';

import { useOptimistic } from 'react';
import { deleteTodo } from '@/app/actions';

export function TodoList({ initialTodos }) {
  const [optimisticTodos, setOptimisticTodos] = useOptimistic(
    initialTodos,
    (state, { action, id }) => {
      if (action === 'delete') {
        return state.filter(todo => todo.id !== id);
      }
      return state;
    }
  );
  
  const handleDelete = async (id: string) => {
    // Optimistically remove from UI
    setOptimisticTodos({ action: 'delete', id });
    
    // Delete on server
    await deleteTodo(id);
  };
  
  return (
    <ul>
      {optimisticTodos.map(todo => (
        <li key={todo.id}>
          {todo.title}
          <button onClick={() => handleDelete(todo.id)}>Delete</button>
        </li>
      ))}
    </ul>
  );
}
```

---

### 7. **Progressive Enhancement**

**Forms work without JavaScript:**

```jsx
// app/todos/page.tsx (Server Component)
import { db } from '@/lib/db';
import { revalidatePath } from 'next/cache';

export default async function TodosPage() {
  const todos = await db.todo.findMany();
  
  // Inline Server Action
  async function createTodo(formData: FormData) {
    'use server';
    
    const title = formData.get('title') as string;
    await db.todo.create({ data: { title } });
    revalidatePath('/todos');
  }
  
  return (
    <div>
      {/* This form works WITHOUT JavaScript! */}
      <form action={createTodo}>
        <input name="title" required />
        <button type="submit">Add Todo</button>
      </form>
      
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>{todo.title}</li>
        ))}
      </ul>
    </div>
  );
}
```

**How it works:**
- If JS enabled: Form submits via fetch, React updates UI smoothly
- If JS disabled: Form submits as regular HTTP POST, page reloads with new data

**Enhanced with Client Component (for loading states):**

```jsx
// components/TodoForm.tsx
'use client';

import { useActionState } from 'react';
import { createTodo } from '@/app/actions';

export function TodoForm() {
  const [state, formAction, isPending] = useActionState(createTodo, {});
  
  return (
    <form action={formAction}>
      <input name="title" required />
      
      {/* Loading state (only shows when JS enabled) */}
      <button type="submit" disabled={isPending}>
        {isPending ? 'Adding...' : 'Add Todo'}
      </button>
      
      {/* Error message */}
      {state?.error && <p>{state.error}</p>}
    </form>
  );
}
```

---

### 8. **Security Considerations**

**✅ SECURE: Validate on server**

```jsx
'use server';

export async function deletePost(postId: string) {
  // ✅ Verify user has permission
  const session = await getSession();
  const post = await db.post.findUnique({ where: { id: postId } });
  
  if (post.authorId !== session.userId) {
    throw new Error('Unauthorized');
  }
  
  // ✅ Server-side validation
  await db.post.delete({ where: { id: postId } });
}
```

**❌ INSECURE: Trusting client data**

```jsx
'use server';

export async function deletePost(postId: string) {
  // ❌ No authorization check - anyone can delete any post!
  await db.post.delete({ where: { id: postId } });
}
```

**✅ SECURE: Sanitize input**

```jsx
'use server';

import { z } from 'zod';

const postSchema = z.object({
  title: z.string().min(1).max(200),
  content: z.string().min(1)
});

export async function createPost(formData: FormData) {
  // ✅ Validate and sanitize
  const result = postSchema.safeParse({
    title: formData.get('title'),
    content: formData.get('content')
  });
  
  if (!result.success) {
    throw new Error('Invalid data');
  }
  
  await db.post.create({ data: result.data });
}
```

**Key security rules:**
1. Always validate on server (never trust client)
2. Check authorization for every action
3. Sanitize all user input
4. Use schema validation (Zod, Yup)
5. Don't expose sensitive data in error messages
6. Rate limit actions to prevent abuse

---

## Common Pitfalls

### ❌ **Pitfall 1: Not revalidating cache**

```jsx
'use server';

export async function createTodo(formData: FormData) {
  await db.todo.create({ data: { title: formData.get('title') } });
  // ❌ Missing revalidatePath - UI won't update!
}

// ✅ CORRECT
export async function createTodo(formData: FormData) {
  await db.todo.create({ data: { title: formData.get('title') } });
  revalidatePath('/todos');  // ✅ Revalidate
}
```

---

### ❌ **Pitfall 2: Missing error handling**

```jsx
'use server';

// ❌ No error handling - crashes app on failure
export async function createTodo(formData: FormData) {
  await db.todo.create({ data: { title: formData.get('title') } });
}

// ✅ CORRECT
export async function createTodo(prevState: any, formData: FormData) {
  try {
    await db.todo.create({ data: { title: formData.get('title') } });
    return { success: true };
  } catch (error) {
    return { error: 'Failed to create todo' };
  }
}
```

---

### ❌ **Pitfall 3: Using Server Actions in Client Components without 'use client'**

```jsx
// ❌ This doesn't work - can't define Server Action in Client Component
'use client';

function MyForm() {
  async function handleSubmit(formData: FormData) {
    'use server';  // Error: Can't use 'use server' in Client Component
    // ...
  }
}

// ✅ CORRECT: Define in separate file
// app/actions.ts
'use server';
export async function handleSubmit(formData: FormData) {
  // ...
}

// components/MyForm.tsx
'use client';
import { handleSubmit } from '@/app/actions';
```

---

### ❌ **Pitfall 4: Not validating on server**

```jsx
'use server';

// ❌ Only client-side validation - insecure!
export async function createPost(formData: FormData) {
  // No server validation - trusts client
  await db.post.create({
    data: {
      title: formData.get('title'),  // Could be anything!
      content: formData.get('content')
    }
  });
}

// ✅ CORRECT: Always validate on server
export async function createPost(formData: FormData) {
  const title = formData.get('title') as string;
  const content = formData.get('content') as string;
  
  if (!title || title.length < 3) {
    return { error: 'Title too short' };
  }
  
  await db.post.create({ data: { title, content } });
}
```

---

## Quick Self-Check

✅ **You understand Server Actions if you can:**

1. Define a Server Action with 'use server'
2. Call a Server Action from a form
3. Use useActionState for form state management
4. Implement optimistic updates with useOptimistic
5. Validate input on both client and server
6. Handle errors and loading states
7. Understand security implications (authorization, validation)
8. Use revalidatePath to update cached data
9. Implement progressive enhancement
10. Know when to use Server Actions vs. API routes

---

## Interview Questions to Practice

**Beginner:**
1. What is a Server Action?
2. How do you define a Server Action?
3. How do you call a Server Action from a form?

**Intermediate:**
4. What's the difference between Server Actions and API routes?
5. How do you handle errors in Server Actions?
6. What is useActionState and how does it work?
7. How do you implement optimistic updates?

**Advanced:**
8. How would you implement multi-step form with Server Actions?
9. Explain security considerations for Server Actions
10. How do Server Actions enable progressive enhancement?

---

## Further Practice

**Build these:**
1. **Todo app** with create, update, delete Server Actions
2. **Blog CMS** with post management and validation
3. **E-commerce cart** with optimistic add/remove
4. **Multi-step wizard** with Server Actions for each step
5. **File upload** with Server Action and progress

**Resources:**
- [Server Actions docs](https://react.dev/reference/react/use-server)
- [Next.js Server Actions](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations)
- [useActionState docs](https://react.dev/reference/react/useActionState)
- [useOptimistic docs](https://react.dev/reference/react/useOptimistic)

---

## Summary

**Server Actions:**
- Async functions that run on server
- Marked with 'use server'
- Can be called from Client Components
- Replace traditional API routes
- Enable progressive enhancement

**Key patterns:**
- Use with forms (action prop)
- Use with useActionState (state management)
- Use with useOptimistic (instant UI updates)
- Always validate on server
- Revalidate cache after mutations

**Benefits:**
- Type-safe (TypeScript end-to-end)
- Simpler than API routes
- Works without JavaScript
- Better UX (loading states, optimistic updates)
- More secure (server-only code)

**Security:**
- Always validate input on server
- Check authorization for every action
- Sanitize user input
- Don't expose sensitive errors
- Rate limit to prevent abuse

**Remember:** Server Actions are for mutations (create, update, delete). For queries (read), use Server Components with async/await.
