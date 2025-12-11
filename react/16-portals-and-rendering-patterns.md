# Portals & Rendering Patterns

**Level:** 🚦 Intermediate  
**Tags:** `portals`, `createPortal`, `modals`, `tooltips`, `rendering`, `interview`

---

## Why this matters

Portals solve a critical problem in React: rendering components outside their parent's DOM hierarchy while maintaining the React component tree structure. This is essential for UI elements like modals, tooltips, dropdowns, and notifications that need to "break out" of their parent containers to avoid z-index issues, overflow clipping, and positioning constraints. Interviewers ask about portals to assess your understanding of the DOM vs React tree, and your ability to build accessible, properly-positioned UI overlays.

**Why interviewers care:**
- Portals show you understand React's rendering model and the DOM
- Modal/overlay questions assess ability to handle z-index and positioning
- Event bubbling through portals tests deep React knowledge
- Accessibility concerns (focus trapping, ARIA) demonstrate production experience
- Senior roles require designing component libraries with proper overlay management
- Understanding portals reveals knowledge of React's escape hatches

**Real-world implications:**
- **Modals:** Render at document root to avoid z-index stacking issues
- **Tooltips/Popovers:** Position relative to viewport, not parent container
- **Dropdowns:** Prevent overflow clipping by parent elements
- **Notifications:** Toast messages that appear above all content
- **Context menus:** Right-click menus that overlay entire page
- **Accessibility:** Proper focus management and keyboard navigation
- **CSS isolation:** Avoid parent styles affecting overlays

**Common scenarios requiring portals:**
- Modal dialogs (confirmation, forms, images)
- Tooltips and popovers (help text, additional info)
- Dropdowns and select menus (autocomplete, filters)
- Toast notifications (success/error messages)
- Context menus (right-click actions)
- Lightboxes (image galleries)
- Floating action buttons or widgets

**What you must know:**
- **ReactDOM.createPortal:** Renders children into DOM node outside parent
- **React tree vs DOM tree:** Events bubble through React tree, not DOM tree
- **Accessibility:** Focus trapping, keyboard navigation, ARIA attributes
- **Mounting targets:** Usually `document.body` or dedicated div
- **Event bubbling:** Events propagate through React tree despite DOM separation
- **Use cases:** Modals, tooltips, dropdowns, notifications
- **Cleanup:** Remove portal root when component unmounts
- **Z-index management:** Portals help avoid stacking context issues

**Interview red flag:** Not knowing portals exist, or unable to explain when/why to use them shows lack of production UI component experience. Saying "just use absolute positioning" ignores z-index and overflow problems that portals solve.

---

## Core Ideas

### 1. **Portals render outside parent DOM hierarchy**

```jsx
import { createPortal } from 'react-dom';

function Modal({ children }) {
  return createPortal(
    <div className="modal">{children}</div>,
    document.body  // Render as child of body, not parent component
  );
}

// Usage
function App() {
  return (
    <div className="app">
      <h1>My App</h1>
      <Modal>
        <h2>Modal Title</h2>
        <p>Modal content</p>
      </Modal>
    </div>
  );
}

/*
DOM structure:
<div class="app">
  <h1>My App</h1>
</div>
<div class="modal">    ← Rendered at body level
  <h2>Modal Title</h2>
  <p>Modal content</p>
</div>
*/
```

**Mental model:**  
"Portals teleport components to different DOM locations while keeping React tree intact."

---

### 2. **Events bubble through React tree, not DOM tree**

```jsx
function Parent() {
  const handleClick = () => {
    console.log('Clicked in Parent');  // This WILL fire!
  };
  
  return (
    <div onClick={handleClick}>
      <Modal>
        <button>Click Me</button>
      </Modal>
    </div>
  );
}

/*
Even though Modal renders outside Parent in DOM,
clicking button triggers Parent's onClick because
events bubble through React component tree.
*/
```

**Key:** React maintains component tree structure for event bubbling.

---

### 3. **Common pattern: Portal with state**

```jsx
function App() {
  const [isOpen, setIsOpen] = useState(false);
  
  return (
    <div>
      <button onClick={() => setIsOpen(true)}>Open Modal</button>
      
      {isOpen && (
        <Modal onClose={() => setIsOpen(false)}>
          <h2>Modal Content</h2>
        </Modal>
      )}
    </div>
  );
}

function Modal({ children, onClose }) {
  return createPortal(
    <div className="modal-overlay" onClick={onClose}>
      <div className="modal-content" onClick={e => e.stopPropagation()}>
        {children}
        <button onClick={onClose}>Close</button>
      </div>
    </div>,
    document.body
  );
}
```

---

### 4. **Portal mount point**

```jsx
// Create portal root in HTML
// <div id="modal-root"></div>

const modalRoot = document.getElementById('modal-root');

function Modal({ children }) {
  return createPortal(
    children,
    modalRoot  // Render into specific div, not body
  );
}
```

---

### 5. **Why portals solve z-index issues**

```jsx
// ❌ Without portal: Modal trapped in parent
function Parent() {
  return (
    <div style={{ position: 'relative', zIndex: 1, overflow: 'hidden' }}>
      {/* Modal clipped by parent overflow */}
      <div className="modal">Modal content</div>
    </div>
  );
}

// ✅ With portal: Modal escapes parent constraints
function Parent() {
  return (
    <div style={{ position: 'relative', zIndex: 1, overflow: 'hidden' }}>
      {createPortal(
        <div className="modal">Modal content</div>,
        document.body  // Renders outside parent
      )}
    </div>
  );
}
```

---

## Examples

### Example 1 — Basic modal with portal

```jsx
import { useState } from 'react';
import { createPortal } from 'react-dom';

function App() {
  const [isOpen, setIsOpen] = useState(false);
  
  return (
    <div>
      <h1>My App</h1>
      <button onClick={() => setIsOpen(true)}>Open Modal</button>
      
      {isOpen && (
        <Modal onClose={() => setIsOpen(false)}>
          <h2>Modal Title</h2>
          <p>This is modal content.</p>
        </Modal>
      )}
    </div>
  );
}

function Modal({ children, onClose }) {
  return createPortal(
    <div className="modal-overlay" onClick={onClose}>
      <div className="modal-content" onClick={e => e.stopPropagation()}>
        <button className="close-button" onClick={onClose}>×</button>
        {children}
      </div>
    </div>,
    document.body
  );
}
```

```css
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal-content {
  background: white;
  padding: 2rem;
  border-radius: 8px;
  max-width: 500px;
  width: 90%;
  position: relative;
}

.close-button {
  position: absolute;
  top: 1rem;
  right: 1rem;
  background: none;
  border: none;
  font-size: 1.5rem;
  cursor: pointer;
}
```

---

### Example 2 — Modal with focus trap and accessibility

```jsx
import { useEffect, useRef } from 'react';
import { createPortal } from 'react-dom';

function Modal({ children, onClose, title }) {
  const modalRef = useRef(null);
  const previousActiveElement = useRef(null);
  
  useEffect(() => {
    // Save currently focused element
    previousActiveElement.current = document.activeElement;
    
    // Focus modal
    modalRef.current?.focus();
    
    // Trap focus inside modal
    const handleTabKey = (e) => {
      if (e.key !== 'Tab') return;
      
      const focusableElements = modalRef.current?.querySelectorAll(
        'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
      );
      
      if (!focusableElements?.length) return;
      
      const firstElement = focusableElements[0];
      const lastElement = focusableElements[focusableElements.length - 1];
      
      if (e.shiftKey && document.activeElement === firstElement) {
        lastElement.focus();
        e.preventDefault();
      } else if (!e.shiftKey && document.activeElement === lastElement) {
        firstElement.focus();
        e.preventDefault();
      }
    };
    
    // Close on Escape
    const handleEscape = (e) => {
      if (e.key === 'Escape') {
        onClose();
      }
    };
    
    document.addEventListener('keydown', handleTabKey);
    document.addEventListener('keydown', handleEscape);
    
    // Cleanup: restore focus
    return () => {
      document.removeEventListener('keydown', handleTabKey);
      document.removeEventListener('keydown', handleEscape);
      previousActiveElement.current?.focus();
    };
  }, [onClose]);
  
  return createPortal(
    <div 
      className="modal-overlay" 
      onClick={onClose}
      role="dialog"
      aria-modal="true"
      aria-labelledby="modal-title"
    >
      <div 
        ref={modalRef}
        className="modal-content" 
        onClick={e => e.stopPropagation()}
        tabIndex={-1}
      >
        <h2 id="modal-title">{title}</h2>
        {children}
        <button onClick={onClose}>Close</button>
      </div>
    </div>,
    document.body
  );
}
```

---

### Example 3 — Tooltip with portal

```jsx
import { useState } from 'react';
import { createPortal } from 'react-dom';

function Tooltip({ children, content }) {
  const [isVisible, setIsVisible] = useState(false);
  const [position, setPosition] = useState({ x: 0, y: 0 });
  const targetRef = useRef(null);
  
  const handleMouseEnter = () => {
    if (targetRef.current) {
      const rect = targetRef.current.getBoundingClientRect();
      setPosition({
        x: rect.left + rect.width / 2,
        y: rect.top - 8  // 8px above element
      });
      setIsVisible(true);
    }
  };
  
  const handleMouseLeave = () => {
    setIsVisible(false);
  };
  
  return (
    <>
      <span
        ref={targetRef}
        onMouseEnter={handleMouseEnter}
        onMouseLeave={handleMouseLeave}
        style={{ position: 'relative', cursor: 'help' }}
      >
        {children}
      </span>
      
      {isVisible && createPortal(
        <div
          className="tooltip"
          style={{
            position: 'fixed',
            top: position.y,
            left: position.x,
            transform: 'translate(-50%, -100%)',
            background: '#333',
            color: 'white',
            padding: '0.5rem',
            borderRadius: '4px',
            fontSize: '0.875rem',
            pointerEvents: 'none',
            zIndex: 9999
          }}
        >
          {content}
        </div>,
        document.body
      )}
    </>
  );
}

// Usage
<p>
  Hover over <Tooltip content="This is helpful info">this text</Tooltip> to see tooltip.
</p>
```

---

### Example 4 — Toast notifications with portal

```jsx
import { createPortal } from 'react-dom';

function ToastContainer({ toasts, removeToast }) {
  return createPortal(
    <div className="toast-container">
      {toasts.map(toast => (
        <Toast 
          key={toast.id} 
          toast={toast} 
          onClose={() => removeToast(toast.id)} 
        />
      ))}
    </div>,
    document.body
  );
}

function Toast({ toast, onClose }) {
  useEffect(() => {
    const timer = setTimeout(onClose, toast.duration || 3000);
    return () => clearTimeout(timer);
  }, [toast.duration, onClose]);
  
  return (
    <div className={`toast toast--${toast.type}`}>
      <p>{toast.message}</p>
      <button onClick={onClose}>×</button>
    </div>
  );
}

// Usage with context
const ToastContext = createContext();

function ToastProvider({ children }) {
  const [toasts, setToasts] = useState([]);
  
  const addToast = (message, type = 'info', duration = 3000) => {
    const id = Date.now();
    setToasts(prev => [...prev, { id, message, type, duration }]);
  };
  
  const removeToast = (id) => {
    setToasts(prev => prev.filter(toast => toast.id !== id));
  };
  
  return (
    <ToastContext.Provider value={{ addToast }}>
      {children}
      <ToastContainer toasts={toasts} removeToast={removeToast} />
    </ToastContext.Provider>
  );
}

function useToast() {
  return useContext(ToastContext);
}

// Usage
function MyComponent() {
  const { addToast } = useToast();
  
  const handleSave = () => {
    // ... save logic
    addToast('Saved successfully!', 'success');
  };
  
  return <button onClick={handleSave}>Save</button>;
}
```

---

### Example 5 — Dropdown with portal (prevents overflow clipping)

```jsx
function Dropdown({ trigger, children }) {
  const [isOpen, setIsOpen] = useState(false);
  const [position, setPosition] = useState({ top: 0, left: 0 });
  const triggerRef = useRef(null);
  const dropdownRef = useRef(null);
  
  useEffect(() => {
    if (isOpen && triggerRef.current) {
      const rect = triggerRef.current.getBoundingClientRect();
      setPosition({
        top: rect.bottom + window.scrollY,
        left: rect.left + window.scrollX
      });
    }
  }, [isOpen]);
  
  // Close on outside click
  useEffect(() => {
    if (!isOpen) return;
    
    const handleClickOutside = (e) => {
      if (
        dropdownRef.current && 
        !dropdownRef.current.contains(e.target) &&
        !triggerRef.current?.contains(e.target)
      ) {
        setIsOpen(false);
      }
    };
    
    document.addEventListener('mousedown', handleClickOutside);
    return () => document.removeEventListener('mousedown', handleClickOutside);
  }, [isOpen]);
  
  return (
    <>
      <div ref={triggerRef} onClick={() => setIsOpen(!isOpen)}>
        {trigger}
      </div>
      
      {isOpen && createPortal(
        <div
          ref={dropdownRef}
          className="dropdown"
          style={{
            position: 'absolute',
            top: position.top,
            left: position.left,
            zIndex: 1000
          }}
        >
          {children}
        </div>,
        document.body
      )}
    </>
  );
}

// Usage
<Dropdown trigger={<button>Open Menu</button>}>
  <ul>
    <li>Option 1</li>
    <li>Option 2</li>
    <li>Option 3</li>
  </ul>
</Dropdown>
```

---

## Common Pitfalls

### 1. **Forgetting to stop propagation on modal content**

```jsx
// ❌ Clicking modal content closes modal
<div className="modal-overlay" onClick={onClose}>
  <div className="modal-content">
    {children}
  </div>
</div>

// ✅ Stop propagation on content
<div className="modal-overlay" onClick={onClose}>
  <div className="modal-content" onClick={e => e.stopPropagation()}>
    {children}
  </div>
</div>
```

---

### 2. **Not managing focus properly**

```jsx
// ❌ Focus remains on trigger element
function Modal({ children, onClose }) {
  return createPortal(
    <div className="modal">
      {children}
    </div>,
    document.body
  );
}

// ✅ Move focus into modal, restore on close
function Modal({ children, onClose }) {
  const modalRef = useRef(null);
  const previousFocus = useRef(null);
  
  useEffect(() => {
    previousFocus.current = document.activeElement;
    modalRef.current?.focus();
    
    return () => previousFocus.current?.focus();
  }, []);
  
  return createPortal(
    <div ref={modalRef} tabIndex={-1} className="modal">
      {children}
    </div>,
    document.body
  );
}
```

---

### 3. **Memory leaks: Not cleaning up portal root**

```jsx
// ❌ Portal root not removed
function Modal({ children }) {
  const el = document.createElement('div');
  document.body.appendChild(el);
  
  return createPortal(children, el);  // el never removed!
}

// ✅ Clean up on unmount
function Modal({ children }) {
  const el = useMemo(() => document.createElement('div'), []);
  
  useEffect(() => {
    document.body.appendChild(el);
    return () => document.body.removeChild(el);
  }, [el]);
  
  return createPortal(children, el);
}
```

---

### 4. **Not handling Escape key**

```jsx
// ❌ No keyboard support
function Modal({ children, onClose }) {
  return createPortal(
    <div className="modal-overlay" onClick={onClose}>
      <div className="modal-content">{children}</div>
    </div>,
    document.body
  );
}

// ✅ Close on Escape
function Modal({ children, onClose }) {
  useEffect(() => {
    const handleEscape = (e) => {
      if (e.key === 'Escape') onClose();
    };
    
    document.addEventListener('keydown', handleEscape);
    return () => document.removeEventListener('keydown', handleEscape);
  }, [onClose]);
  
  return createPortal(
    <div className="modal-overlay" onClick={onClose}>
      <div className="modal-content">{children}</div>
    </div>,
    document.body
  );
}
```

---

### 5. **Missing ARIA attributes**

```jsx
// ❌ Not accessible
<div className="modal">
  <h2>Title</h2>
  {children}
</div>

// ✅ Proper ARIA attributes
<div 
  className="modal"
  role="dialog"
  aria-modal="true"
  aria-labelledby="modal-title"
  aria-describedby="modal-description"
>
  <h2 id="modal-title">Title</h2>
  <div id="modal-description">{children}</div>
</div>
```

---

## FAQ

**Q: When should I use portals?**  
A: Modals, tooltips, dropdowns, notifications—anything that needs to escape parent DOM constraints.

**Q: Do portals affect event bubbling?**  
A: Events bubble through React tree, not DOM tree, so parent handlers still work.

**Q: Can I use multiple portals?**  
A: Yes, you can have as many portals as needed.

**Q: Where should portals render to?**  
A: Usually `document.body` or a dedicated `<div id="portal-root">`.

**Q: Do I need to clean up portals?**  
A: React handles portal cleanup, but remove custom portal roots manually.

**Q: How do portals help with z-index?**  
A: Rendering outside parent avoids stacking context issues.

**Q: Can portals render on the server (SSR)?**  
A: No, portals require DOM. Use hydration-aware approaches for SSR.

---

## Quick Self-Check

1. What problem do portals solve?
   (Rendering components outside parent DOM while maintaining React tree, avoiding z-index/overflow issues)

2. Do events bubble through DOM or React tree in portals?
   (React tree—parent handlers still work)

3. Where do portals typically render to?
   (`document.body` or dedicated portal root element)

4. What accessibility concerns do modals have?
   (Focus management, keyboard navigation, ARIA attributes)

5. When should you NOT use portals?
   (When parent-child DOM relationship is fine and no positioning/z-index issues exist)

---

## Further Practice

1. **Build:** Modal with portal, focus trap, and Escape key support
2. **Create:** Tooltip component with portal (positioned relative to trigger)
3. **Implement:** Toast notification system with portal
4. **Build:** Dropdown menu with portal (prevents overflow clipping)
5. **Add:** Accessibility features (ARIA, focus management, keyboard navigation)
6. **Create:** Context menu (right-click) with portal
7. **Build:** Lightbox image viewer with portal

---

## Key Takeaways

- Portals render components outside parent DOM while keeping React tree intact
- Use `ReactDOM.createPortal(children, domNode)` to create portals
- Common use cases: modals, tooltips, dropdowns, notifications
- Events bubble through React component tree, not DOM tree
- Portals solve z-index stacking and overflow clipping issues
- Render portals to `document.body` or dedicated portal root
- Always manage focus properly (trap focus, restore on close)
- Handle keyboard navigation (Escape, Tab, Arrow keys)
- Add ARIA attributes for accessibility (role, aria-modal, aria-labelledby)
- Clean up custom portal roots on component unmount
