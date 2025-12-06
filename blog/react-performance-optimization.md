# React Performance Optimization: Best Practices and Techniques

## Introduction

React is fast by default, but as your application grows, you might encounter performance bottlenecks. Understanding how to optimize React applications is crucial for delivering great user experiences. In this guide, we'll explore proven techniques to make your React apps blazingly fast.

## Understanding React's Rendering

Before optimizing, understand how React renders:

1. **Reconciliation**: React compares the new virtual DOM with the previous one
2. **Rendering**: React updates only changed DOM elements
3. **Component Re-rendering**: When state or props change, components re-render

### What Causes Re-renders?

- State updates
- Props changes
- Parent component re-renders
- Context value changes

## Optimization Techniques

### 1. React.memo - Prevent Unnecessary Re-renders

Memoize functional components to prevent re-renders when props haven't changed:

```jsx
import { memo } from 'react';

// Without memo - re-renders every time parent renders
function ExpensiveComponent({ data }) {
  console.log('Rendering expensive component');
  return <div>{/* Complex UI */}</div>;
}

// With memo - only re-renders when data changes
const OptimizedComponent = memo(function ExpensiveComponent({ data }) {
  console.log('Rendering expensive component');
  return <div>{/* Complex UI */}</div>;
});

// Custom comparison function
const SmartComponent = memo(
  function Component({ user }) {
    return <div>{user.name}</div>;
  },
  (prevProps, nextProps) => {
    // Return true if props are equal (skip re-render)
    return prevProps.user.id === nextProps.user.id;
  }
);
```

### 2. useMemo - Memoize Expensive Calculations

Cache computed values to avoid recalculating on every render:

```jsx
import { useMemo, useState } from 'react';

function DataTable({ data, filter }) {
  // Without useMemo - recalculates on every render
  const filteredData = data.filter(item => 
    item.name.includes(filter)
  );

  // With useMemo - only recalculates when dependencies change
  const optimizedData = useMemo(() => {
    console.log('Filtering data...');
    return data.filter(item => item.name.includes(filter));
  }, [data, filter]);

  return (
    <table>
      {optimizedData.map(item => (
        <tr key={item.id}>
          <td>{item.name}</td>
        </tr>
      ))}
    </table>
  );
}

// Example with expensive calculation
function Statistics({ numbers }) {
  const stats = useMemo(() => {
    console.log('Calculating statistics...');
    return {
      sum: numbers.reduce((a, b) => a + b, 0),
      average: numbers.reduce((a, b) => a + b, 0) / numbers.length,
      max: Math.max(...numbers),
      min: Math.min(...numbers)
    };
  }, [numbers]);

  return (
    <div>
      <p>Sum: {stats.sum}</p>
      <p>Average: {stats.average}</p>
      <p>Max: {stats.max}</p>
      <p>Min: {stats.min}</p>
    </div>
  );
}
```

### 3. useCallback - Memoize Functions

Prevent function recreation on every render:

```jsx
import { useCallback, useState, memo } from 'react';

// Child component that receives callback
const Button = memo(({ onClick, children }) => {
  console.log('Button rendered');
  return <button onClick={onClick}>{children}</button>;
});

function Parent() {
  const [count, setCount] = useState(0);
  const [other, setOther] = useState(0);

  // Without useCallback - new function on every render
  const handleClick = () => {
    setCount(c => c + 1);
  };

  // With useCallback - same function reference
  const optimizedClick = useCallback(() => {
    setCount(c => c + 1);
  }, []); // Empty deps - function never changes

  return (
    <div>
      <p>Count: {count}</p>
      <p>Other: {other}</p>
      {/* This button re-renders even when 'other' changes */}
      <Button onClick={handleClick}>Bad</Button>
      {/* This button only re-renders when dependencies change */}
      <Button onClick={optimizedClick}>Good</Button>
      <button onClick={() => setOther(o => o + 1)}>
        Update Other
      </button>
    </div>
  );
}
```

### 4. Code Splitting with React.lazy

Split your code into smaller chunks and load them on demand:

```jsx
import { lazy, Suspense } from 'react';

// Instead of regular import
// import HeavyComponent from './HeavyComponent';

// Use lazy loading
const HeavyComponent = lazy(() => import('./HeavyComponent'));
const Dashboard = lazy(() => import('./Dashboard'));
const Profile = lazy(() => import('./Profile'));

function App() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <HeavyComponent />
      </Suspense>
    </div>
  );
}

// Route-based code splitting
import { BrowserRouter, Routes, Route } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<div>Loading page...</div>}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/dashboard" element={<Dashboard />} />
          <Route path="/profile" element={<Profile />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}
```

### 5. Virtualization for Long Lists

Render only visible items in long lists:

```jsx
import { FixedSizeList } from 'react-window';

// Without virtualization - renders all 10,000 items
function SlowList({ items }) {
  return (
    <div>
      {items.map(item => (
        <div key={item.id}>{item.name}</div>
      ))}
    </div>
  );
}

// With virtualization - only renders visible items
function FastList({ items }) {
  const Row = ({ index, style }) => (
    <div style={style}>
      {items[index].name}
    </div>
  );

  return (
    <FixedSizeList
      height={600}
      itemCount={items.length}
      itemSize={50}
      width="100%"
    >
      {Row}
    </FixedSizeList>
  );
}

// Variable size list
import { VariableSizeList } from 'react-window';

function DynamicList({ items }) {
  const getItemSize = (index) => {
    // Calculate dynamic height based on content
    return items[index].content.length > 100 ? 100 : 50;
  };

  const Row = ({ index, style }) => (
    <div style={style}>
      <h3>{items[index].title}</h3>
      <p>{items[index].content}</p>
    </div>
  );

  return (
    <VariableSizeList
      height={600}
      itemCount={items.length}
      itemSize={getItemSize}
      width="100%"
    >
      {Row}
    </VariableSizeList>
  );
}
```

### 6. Debouncing and Throttling

Limit the frequency of expensive operations:

```jsx
import { useState, useCallback } from 'react';
import { debounce } from 'lodash';

function SearchBox() {
  const [results, setResults] = useState([]);

  // Debounce search - waits for user to stop typing
  const debouncedSearch = useCallback(
    debounce(async (query) => {
      const data = await fetch(`/api/search?q=${query}`);
      const results = await data.json();
      setResults(results);
    }, 300),
    []
  );

  return (
    <input
      onChange={(e) => debouncedSearch(e.target.value)}
      placeholder="Search..."
    />
  );
}

// Throttle scroll events
import { throttle } from 'lodash';

function InfiniteScroll() {
  const handleScroll = useCallback(
    throttle(() => {
      // Load more items
      loadMoreItems();
    }, 200),
    []
  );

  useEffect(() => {
    window.addEventListener('scroll', handleScroll);
    return () => window.removeEventListener('scroll', handleScroll);
  }, [handleScroll]);

  return <div>{/* Content */}</div>;
}
```

### 7. Web Workers for Heavy Computations

Offload CPU-intensive tasks to background threads:

```jsx
// worker.js
self.addEventListener('message', (e) => {
  const { data } = e;
  
  // Heavy computation
  const result = performExpensiveCalculation(data);
  
  self.postMessage(result);
});

// Component.jsx
import { useEffect, useState } from 'react';

function HeavyCalculation({ data }) {
  const [result, setResult] = useState(null);

  useEffect(() => {
    const worker = new Worker(new URL('./worker.js', import.meta.url));
    
    worker.postMessage(data);
    
    worker.onmessage = (e) => {
      setResult(e.data);
    };

    return () => worker.terminate();
  }, [data]);

  return <div>Result: {result}</div>;
}
```

### 8. Optimize Context Usage

Prevent unnecessary re-renders from context:

```jsx
import { createContext, useContext, useState, memo } from 'react';

// Split context by update frequency
const UserContext = createContext();
const ThemeContext = createContext();

function App() {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');

  return (
    <UserContext.Provider value={{ user, setUser }}>
      <ThemeContext.Provider value={{ theme, setTheme }}>
        <Main />
      </ThemeContext.Provider>
    </UserContext.Provider>
  );
}

// Memoize context consumers
const ThemedComponent = memo(() => {
  const { theme } = useContext(ThemeContext);
  return <div className={theme}>Themed content</div>;
});

// Use context selectors
function useUserName() {
  const { user } = useContext(UserContext);
  return user?.name;
}
```

### 9. Optimize Images

```jsx
// Next.js Image component - automatic optimization
import Image from 'next/image';

function OptimizedImage() {
  return (
    <Image
      src="/large-image.jpg"
      alt="Description"
      width={800}
      height={600}
      loading="lazy"
      placeholder="blur"
      blurDataURL="data:image/jpeg;base64,..."
    />
  );
}

// Regular React - lazy loading
function LazyImage({ src, alt }) {
  return (
    <img
      src={src}
      alt={alt}
      loading="lazy"
      decoding="async"
    />
  );
}
```

### 10. Avoid Inline Functions and Objects

```jsx
// Bad - creates new function on every render
function BadComponent() {
  return <button onClick={() => console.log('clicked')}>Click</button>;
}

// Good - stable reference
function GoodComponent() {
  const handleClick = useCallback(() => {
    console.log('clicked');
  }, []);

  return <button onClick={handleClick}>Click</button>;
}

// Bad - creates new object on every render
function BadProps() {
  return <Component style={{ margin: 10 }} />;
}

// Good - stable reference
const STYLE = { margin: 10 };
function GoodProps() {
  return <Component style={STYLE} />;
}
```

## Performance Monitoring

### React DevTools Profiler

Use the Profiler to identify performance bottlenecks:

```jsx
import { Profiler } from 'react';

function onRenderCallback(
  id, // component identifier
  phase, // "mount" or "update"
  actualDuration, // time spent rendering
  baseDuration, // estimated time without memoization
  startTime, // when render started
  commitTime, // when changes committed
  interactions // Set of interactions
) {
  console.log(`${id} took ${actualDuration}ms to render`);
}

function App() {
  return (
    <Profiler id="App" onRender={onRenderCallback}>
      <Main />
    </Profiler>
  );
}
```

## Best Practices Summary

1. **Measure First**: Use profiling tools before optimizing
2. **Don't Premature Optimize**: Optimize when you have a problem
3. **Memoize Appropriately**: Don't overuse memo/useMemo/useCallback
4. **Split Code**: Use lazy loading for large components
5. **Virtualize Lists**: Use react-window for long lists
6. **Optimize Images**: Use appropriate formats and lazy loading
7. **Debounce/Throttle**: Limit expensive operations
8. **Keep State Local**: Don't lift state unnecessarily
9. **Use Production Build**: Development mode is slower
10. **Monitor Performance**: Continuously track metrics

## Conclusion

Performance optimization is an iterative process. Start with measurements, identify bottlenecks, apply appropriate techniques, and measure again. Remember, premature optimization is the root of all evil - optimize only when necessary!

Happy optimizing!
