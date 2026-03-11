# Performance Patterns Reference

## Table of Contents
1. [Re-render Prevention](#re-render-prevention)
2. [Memoization Strategies](#memoization-strategies)
3. [Bundle Optimization](#bundle-optimization)
4. [Runtime Performance](#runtime-performance)

---

## Re-render Prevention

### Component Splitting Pattern
```tsx
// ❌ Entire component re-renders when count changes
function App() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <ExpensiveTree />
      <button onClick={() => setCount(c => c + 1)}>{count}</button>
    </div>
  );
}

// ✅ Isolate state to minimize re-render scope
function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}

function App() {
  return (
    <div>
      <ExpensiveTree />
      <Counter />
    </div>
  );
}
```

### Children as Props Pattern
```tsx
// ✅ Children don't re-render when parent state changes
function ScrollTracker({ children }) {
  const [scroll, setScroll] = useState(0);
  
  useEffect(() => {
    const handler = () => setScroll(window.scrollY);
    window.addEventListener('scroll', handler);
    return () => window.removeEventListener('scroll', handler);
  }, []);
  
  return <div data-scroll={scroll}>{children}</div>;
}

// Usage: <ScrollTracker><ExpensiveChild /></ScrollTracker>
```

### React.memo Checklist
Use `React.memo` when:
- Component renders often with same props
- Component is expensive to render
- Parent re-renders frequently but props are stable

Avoid `React.memo` when:
- Props change on most renders
- Component is cheap to render
- Premature optimization without profiling

```tsx
// ✅ Proper memo with custom comparison
const ListItem = memo(function ListItem({ item, onSelect }) {
  return <li onClick={() => onSelect(item.id)}>{item.name}</li>;
}, (prev, next) => prev.item.id === next.item.id);
```

---

## Memoization Strategies

### useMemo Guidelines
```tsx
// ✅ Expensive computation
const sortedItems = useMemo(
  () => items.sort((a, b) => a.price - b.price),
  [items]
);

// ✅ Referential equality for child props
const chartData = useMemo(
  () => ({ labels, values, colors }),
  [labels, values, colors]
);

// ❌ Over-memoization (cheap operation)
const fullName = useMemo(
  () => `${firstName} ${lastName}`,
  [firstName, lastName]
);
```

### useCallback Guidelines
```tsx
// ✅ Callback passed to memoized child
const handleClick = useCallback((id) => {
  dispatch({ type: 'SELECT', id });
}, [dispatch]);

// ✅ Callback used in useEffect dependency
const fetchData = useCallback(async () => {
  const data = await api.get(`/items/${id}`);
  setData(data);
}, [id]);

useEffect(() => {
  fetchData();
}, [fetchData]);
```

---

## Bundle Optimization

### Code Splitting Patterns
```tsx
// ✅ Route-level splitting
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Settings = lazy(() => import('./pages/Settings'));

// ✅ Feature-level splitting
const ChartWidget = lazy(() => import('./components/ChartWidget'));

// ✅ Conditional loading
const AdminPanel = lazy(() => 
  user.isAdmin ? import('./AdminPanel') : import('./UserPanel')
);
```

### Dynamic Import Triggers
```tsx
// ✅ Load on interaction
function Editor() {
  const [Editor, setEditor] = useState(null);
  
  const loadEditor = async () => {
    const { RichTextEditor } = await import('./RichTextEditor');
    setEditor(() => RichTextEditor);
  };
  
  return Editor ? <Editor /> : <button onClick={loadEditor}>Edit</button>;
}
```

---

## Runtime Performance

### List Virtualization
```tsx
// ✅ Virtual list for large datasets
import { FixedSizeList } from 'react-window';

function VirtualList({ items }) {
  const Row = ({ index, style }) => (
    <div style={style}>{items[index].name}</div>
  );
  
  return (
    <FixedSizeList
      height={400}
      itemCount={items.length}
      itemSize={35}
    >
      {Row}
    </FixedSizeList>
  );
}
```

### Debouncing and Throttling
```tsx
// ✅ Debounced search input
function SearchInput({ onSearch }) {
  const [query, setQuery] = useState('');
  
  const debouncedSearch = useMemo(
    () => debounce((q) => onSearch(q), 300),
    [onSearch]
  );
  
  useEffect(() => {
    debouncedSearch(query);
    return () => debouncedSearch.cancel();
  }, [query, debouncedSearch]);
  
  return <input value={query} onChange={e => setQuery(e.target.value)} />;
}
```

### Image Optimization
```tsx
// ✅ Lazy loading with placeholder
function OptimizedImage({ src, alt }) {
  return (
    <img
      src={src}
      alt={alt}
      loading="lazy"
      decoding="async"
      style={{ contentVisibility: 'auto' }}
    />
  );
}
```
