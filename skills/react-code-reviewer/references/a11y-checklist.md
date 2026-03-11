# Accessibility (a11y) Checklist

## Table of Contents
1. [Semantic HTML](#semantic-html)
2. [Keyboard Navigation](#keyboard-navigation)
3. [ARIA Patterns](#aria-patterns)
4. [Forms](#forms)
5. [Dynamic Content](#dynamic-content)
6. [Common Violations](#common-violations)

---

## Semantic HTML

### Structure Checklist
- [ ] Single `<main>` element per page
- [ ] Proper heading hierarchy (h1 → h2 → h3, no skipping)
- [ ] `<nav>` for navigation sections
- [ ] `<article>` for self-contained content
- [ ] `<section>` with accessible name for page regions
- [ ] `<button>` for actions, `<a>` for navigation

### Anti-patterns
```tsx
// ❌ Div with click handler
<div onClick={handleClick}>Click me</div>

// ✅ Semantic button
<button onClick={handleClick}>Click me</button>

// ❌ Span as link
<span onClick={() => navigate('/home')}>Home</span>

// ✅ Proper anchor
<a href="/home">Home</a>
// or with router:
<Link to="/home">Home</Link>
```

---

## Keyboard Navigation

### Focus Management
```tsx
// ✅ Focus trap in modal
function Modal({ isOpen, onClose, children }) {
  const firstFocusable = useRef(null);
  
  useEffect(() => {
    if (isOpen) {
      firstFocusable.current?.focus();
    }
  }, [isOpen]);
  
  const handleKeyDown = (e) => {
    if (e.key === 'Escape') onClose();
    // Implement focus trap for Tab key
  };
  
  return isOpen ? (
    <div role="dialog" aria-modal="true" onKeyDown={handleKeyDown}>
      <button ref={firstFocusable} onClick={onClose}>Close</button>
      {children}
    </div>
  ) : null;
}
```

### Skip Links
```tsx
// ✅ Skip to main content
function SkipLink() {
  return (
    <a 
      href="#main-content" 
      className="sr-only focus:not-sr-only"
    >
      Skip to main content
    </a>
  );
}
```

### Focus Indicators
```tsx
// ❌ Removing focus outline
button:focus { outline: none; }

// ✅ Custom focus indicator
button:focus-visible {
  outline: 2px solid #4A90D9;
  outline-offset: 2px;
}
```

---

## ARIA Patterns

### Interactive Components
```tsx
// ✅ Accessible dropdown
function Dropdown({ label, options, value, onChange }) {
  const [isOpen, setIsOpen] = useState(false);
  const id = useId();
  
  return (
    <div>
      <button
        id={`${id}-button`}
        aria-haspopup="listbox"
        aria-expanded={isOpen}
        aria-labelledby={`${id}-label ${id}-button`}
        onClick={() => setIsOpen(!isOpen)}
      >
        {value || 'Select...'}
      </button>
      
      {isOpen && (
        <ul
          role="listbox"
          aria-labelledby={`${id}-label`}
          tabIndex={-1}
        >
          {options.map(option => (
            <li
              key={option.value}
              role="option"
              aria-selected={option.value === value}
              onClick={() => onChange(option.value)}
            >
              {option.label}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

### Loading States
```tsx
// ✅ Announce loading state
function DataTable({ isLoading, data }) {
  return (
    <div aria-busy={isLoading} aria-live="polite">
      {isLoading ? (
        <span role="status">Loading data...</span>
      ) : (
        <table>{/* ... */}</table>
      )}
    </div>
  );
}
```

---

## Forms

### Labels and Descriptions
```tsx
// ✅ Properly associated labels
function FormField({ id, label, error, description, children }) {
  const descriptionId = description ? `${id}-description` : undefined;
  const errorId = error ? `${id}-error` : undefined;
  
  return (
    <div>
      <label htmlFor={id}>{label}</label>
      {description && (
        <p id={descriptionId}>{description}</p>
      )}
      {React.cloneElement(children, {
        id,
        'aria-describedby': [descriptionId, errorId].filter(Boolean).join(' ') || undefined,
        'aria-invalid': !!error,
      })}
      {error && (
        <p id={errorId} role="alert">{error}</p>
      )}
    </div>
  );
}
```

### Required Fields
```tsx
// ✅ Accessible required field
<label htmlFor="email">
  Email <span aria-hidden="true">*</span>
  <span className="sr-only">(required)</span>
</label>
<input
  id="email"
  type="email"
  required
  aria-required="true"
/>
```

---

## Dynamic Content

### Live Regions
```tsx
// ✅ Toast notifications
function Toast({ message, type }) {
  return (
    <div
      role="alert"
      aria-live={type === 'error' ? 'assertive' : 'polite'}
    >
      {message}
    </div>
  );
}

// ✅ Search results count
function SearchResults({ count }) {
  return (
    <p aria-live="polite" aria-atomic="true">
      {count} results found
    </p>
  );
}
```

### Route Changes
```tsx
// ✅ Announce page changes
function RouteAnnouncer() {
  const location = useLocation();
  const [announcement, setAnnouncement] = useState('');
  
  useEffect(() => {
    const pageTitle = document.title;
    setAnnouncement(`Navigated to ${pageTitle}`);
  }, [location]);
  
  return (
    <div
      role="status"
      aria-live="polite"
      aria-atomic="true"
      className="sr-only"
    >
      {announcement}
    </div>
  );
}
```

---

## Common Violations

### Images
```tsx
// ❌ Missing alt text
<img src="chart.png" />

// ✅ Descriptive alt text
<img src="chart.png" alt="Sales increased 25% from Q1 to Q2" />

// ✅ Decorative image
<img src="decoration.png" alt="" role="presentation" />
```

### Icons
```tsx
// ❌ Icon button without label
<button><Icon name="settings" /></button>

// ✅ With accessible name
<button aria-label="Settings">
  <Icon name="settings" aria-hidden="true" />
</button>

// ✅ Or with visible text
<button>
  <Icon name="settings" aria-hidden="true" />
  <span>Settings</span>
</button>
```

### Color Contrast
```tsx
// ❌ Relying only on color
<span className={isError ? 'text-red' : 'text-green'}>
  {status}
</span>

// ✅ Color + icon/text
<span className={isError ? 'text-red' : 'text-green'}>
  {isError ? '✗ Error: ' : '✓ Success: '}
  {status}
</span>
```

### Screen Reader Only Text
```css
/* Utility class for visually hidden but accessible text */
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}
```
