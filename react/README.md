# React Learning Path

A structured path through essential React concepts, from fundamentals to advanced topics, optimized for interview preparation and quick refreshers.

---

## 🎯 Learning Goals

By working through these lessons, you'll be able to:
- Explain React's core concepts and mental models clearly in interviews
- Build performant, maintainable React applications
- Debug common React issues and anti-patterns
- Understand modern React patterns (hooks, composition, state management)

---

## 📚 Lesson Index

### 🏁 Fundamentals (Start Here)

Essential concepts every React developer must know cold.

| # | Topic | Why It Matters |
|---|-------|----------------|
| [01](01-jsx-and-rendering.md) | **JSX & Rendering** | Virtual DOM, JSX syntax, elements vs components |
| [02](02-components-and-props.md) | **Components & Props** | Function vs class components, props flow, composition |
| [03](03-state-and-lifecycle.md) | **State & Lifecycle** | useState, component lifecycle, when to use state |
| [04](04-hooks-basics.md) | **Hooks Basics** | Rules of hooks, built-in hooks, why hooks exist |
| [05](05-event-handling.md) | **Event Handling** | Synthetic events, event binding, common patterns |

---

### 🚦 Intermediate (Core Skills)

Practical skills used daily in production React applications.

| # | Topic | Why It Matters |
|---|-------|----------------|
| [06](06-useeffect-and-side-effects.md) | **useEffect & Side Effects** | Dependencies, cleanup, async effects, common pitfalls |
| [07](07-custom-hooks.md) | **Custom Hooks** | Reusable logic, hook composition, best practices |
| [08](08-context-api.md) | **Context API** | Global state, prop drilling, when to use Context |
| [09](09-performance-optimization.md) | **Performance Optimization** | useMemo, useCallback, React.memo, profiling |
| [10](10-forms-and-controlled-components.md) | **Forms & Controlled Components** | Controlled vs uncontrolled, validation, form libraries |
| [11](11-lists-and-keys.md) | **Lists & Keys** | Rendering lists, key prop, reconciliation algorithm |
| [12](12-routing.md) | **Routing** | React Router, navigation, protected routes, lazy loading |
| [13](13-error-boundaries.md) | **Error Boundaries** | Error handling, fallback UI, logging errors |
| [14](14-refs-and-dom.md) | **Refs & DOM Access** | useRef, forwardRef, when to access DOM directly |
| [15](15-component-composition.md) | **Component Composition** | children prop, render props, HOCs vs hooks |
| [16](16-portals-and-rendering-patterns.md) | **Portals & Rendering Patterns** | ReactDOM.createPortal, modals, tooltips, escape hatches |

---

### 🔥 Advanced (Deep Dive)

Advanced patterns, architecture, and production concerns.

| # | Topic | Why It Matters |
|---|-------|----------------|
| [17](17-state-management-patterns.md) | **State Management Patterns** | Redux, Zustand, Jotai, Recoil, when to use what |
| [18](18-testing-react-applications.md) | **Testing React Applications** | Jest, React Testing Library, unit & integration tests |
| [19](19-typescript-with-react.md) | **TypeScript with React** | Typing props, hooks, generics, common patterns |
| [20](20-advanced-hooks.md) | **Advanced Hooks** | useReducer, useImperativeHandle, useLayoutEffect, useDebugValue |

---

### ⚡ React 18/19+ Features

Cutting-edge features and modern React development.

| # | Topic | Why It Matters |
|---|-------|----------------|
| [21](21-concurrent-features.md) | **Concurrent Features** | Concurrent rendering, transitions, useTransition, startTransition |
| [22](22-react-suspense.md) | **React Suspense** | Suspense for data fetching, lazy loading, boundaries |
| [23](23-react-compiler.md) | **React Compiler (React 19+)** | Automatic memoization, forget.js, optimization without useMemo |
| [24](24-server-components.md) | **Server Components** | RSC architecture, server vs client components, streaming |
| [25](25-server-actions.md) | **Server Actions** | Form actions, mutations, progressive enhancement |
| [26](26-modern-react-patterns.md) | **Modern React Patterns (2024+)** | use hook, Actions, Optimistic UI, Form status |
| [27](27-react-apis-reference.md) | **React APIs Reference** | createContext, forwardRef, memo, lazy, Suspense, StrictMode |
| [28](28-react-internals.md) | **React Internals** | Fiber, reconciliation, scheduler, work loop |
| [29](29-next-js-and-frameworks.md) | **Next.js & React Frameworks** | App Router, SSR, SSG, ISR, middleware, deployment |
| [30](30-performance-and-production.md) | **Performance & Production** | Bundle optimization, code splitting, caching, monitoring |

---

### 📝 Interview Prep

| # | Topic |
|---|-------|
| [99](99-interview-checklist.md) | **Interview Checklist** |

---

## 🚀 Suggested Learning Paths

### Path 1: Interview Prep (1-2 days)
Focus on fundamentals and commonly asked questions:
1. Lessons 01-05 (Fundamentals)
2. Lessons 06, 09, 11 (useEffect, Performance, Keys)
3. Lesson 20 (Advanced Hooks - useReducer)
4. Review `99-interview-checklist.md`

### Path 2: Production-Ready (1-2 weeks)
Build strong practical skills:
1. All Fundamentals (01-05)
2. All Intermediate (06-16)
3. Lessons 17, 18, 19 (State management, Testing, TypeScript)
4. Lesson 29 (Next.js basics)

### Path 3: Modern React Mastery (2-4 weeks)
Focus on React 18/19+ features:
1. Fundamentals (01-05)
2. Core Intermediate (06-12)
3. Advanced Core (17-20)
4. Concurrent Features (21)
5. React Suspense (22)
6. React Compiler (23)
7. Server Components & Actions (24-25)
8. Modern Patterns (26)

### Path 4: Deep Mastery (1-2 months)
Complete understanding of React:
- Work through lessons 01-20 thoroughly
- Deep dive into React 18/19+ features (21-26)
- Master React internals (28)
- Master Next.js and production deployment (29-30)
- Build real-world projects applying all concepts
- Read official React docs alongside

---

## 💡 Tips for Using These Lessons

- **Stay current**: React 18/19 introduced major changes (Suspense, Server Components, Compiler)
- **Code along**: Don't just read—type out examples
- **Break things**: Change code and observe what happens
- **Ask "why"**: Understand the reasoning behind patterns
- **Practice explaining**: Teach concepts back in your own words
- **Check yourself**: Use the self-check questions at the end of each lesson
- **Build projects**: Apply concepts in real applications

---

## 🆕 What's New in Modern React

### React 19 (2024)
- **React Compiler**: Automatic memoization without useMemo/useCallback
- **Actions**: Built-in support for async form handling
- **use() Hook**: Read promises and context in render
- **Optimistic Updates**: Built-in UI patterns for pending states
- **Server Actions**: Direct server function calls from client

### React 18 (2022)
- **Concurrent Rendering**: Non-blocking rendering with priorities
- **Transitions**: Mark updates as non-urgent (useTransition)
- **Suspense for Data Fetching**: Beyond lazy loading
- **Automatic Batching**: All updates batched, not just events
- **Streaming SSR**: Improved server-side rendering performance

---

## 📖 Additional Resources

- [Official React Docs](https://react.dev)
- [React 19 Release Notes](https://react.dev/blog/2024/12/05/react-19)
- [React Compiler Documentation](https://react.dev/learn/react-compiler)
- [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/)
- [React DevTools](https://react.dev/learn/react-developer-tools)
- [Next.js Documentation](https://nextjs.org/docs)
- [Kent C. Dodds Blog](https://kentcdodds.com/blog)
- [Dan Abramov's Overreacted](https://overreacted.io/)
