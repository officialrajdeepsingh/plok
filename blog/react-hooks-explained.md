# React Hooks Explained: Complete Guide to Modern React

## Introduction

React Hooks, introduced in React 16.8, have transformed the way we write React components. They allow you to use state and other React features without writing class components. In this comprehensive guide, we'll explore the most commonly used hooks and their practical applications.

## What Are React Hooks?

Hooks are functions that let you "hook into" React state and lifecycle features from functional components. They make it easier to reuse stateful logic between components and organize related code together.

### Rules of Hooks

Before diving in, remember these two essential rules:

1. **Only call hooks at the top level** - Don't call hooks inside loops, conditions, or nested functions
2. **Only call hooks from React functions** - Call them from functional components or custom hooks

## Essential React Hooks

### 1. useState - Managing Component State

The `useState` hook lets you add state to functional components:

```jsx
import { useState } from 'react';

function LoginForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [isLoading, setIsLoading] = useState(false);

  const handleSubmit = async (e) => {
    e.preventDefault();
    setIsLoading(true);
    // Handle login logic
    setIsLoading(false);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input 
        value={email} 
        onChange={(e) => setEmail(e.target.value)} 
        placeholder="Email"
      />
      <input 
        type="password"
        value={password} 
        onChange={(e) => setPassword(e.target.value)}
        placeholder="Password"
      />
      <button disabled={isLoading}>
        {isLoading ? 'Loading...' : 'Login'}
      </button>
    </form>
  );
}
```

### 2. useEffect - Side Effects and Lifecycle

The `useEffect` hook performs side effects in functional components:

```jsx
import { useState, useEffect } from 'react';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    let cancelled = false;

    async function fetchUser() {
      setLoading(true);
      const response = await fetch(`/api/users/${userId}`);
      const data = await response.json();
      
      if (!cancelled) {
        setUser(data);
        setLoading(false);
      }
    }

    fetchUser();

    // Cleanup function
    return () => {
      cancelled = true;
    };
  }, [userId]); // Re-run when userId changes

  if (loading) return <div>Loading...</div>;
  return <div>Welcome, {user.name}!</div>;
}
```

### 3. useContext - Accessing Context

The `useContext` hook makes it easy to consume context values:

```jsx
import { createContext, useContext } from 'react';

const ThemeContext = createContext('light');

function ThemedButton() {
  const theme = useContext(ThemeContext);
  
  return (
    <button className={`btn-${theme}`}>
      I'm a {theme} themed button
    </button>
  );
}
```

### 4. useRef - Accessing DOM Elements

The `useRef` hook creates a mutable reference that persists across renders:

```jsx
import { useRef, useEffect } from 'react';

function AutoFocusInput() {
  const inputRef = useRef(null);

  useEffect(() => {
    inputRef.current.focus();
  }, []);

  return <input ref={inputRef} placeholder="I'm auto-focused!" />;
}
```

### 5. useMemo and useCallback - Performance Optimization

These hooks help optimize performance by memoizing values and functions:

```jsx
import { useMemo, useCallback, useState } from 'react';

function ExpensiveComponent({ data, onUpdate }) {
  const [filter, setFilter] = useState('');

  // Memoize expensive computation
  const filteredData = useMemo(() => {
    return data.filter(item => 
      item.name.toLowerCase().includes(filter.toLowerCase())
    );
  }, [data, filter]);

  // Memoize callback function
  const handleUpdate = useCallback((id) => {
    onUpdate(id);
  }, [onUpdate]);

  return (
    <div>
      <input 
        value={filter}
        onChange={(e) => setFilter(e.target.value)}
        placeholder="Filter..."
      />
      {filteredData.map(item => (
        <Item key={item.id} data={item} onUpdate={handleUpdate} />
      ))}
    </div>
  );
}
```

## Custom Hooks

Create your own hooks to reuse stateful logic:

```jsx
function useLocalStorage(key, initialValue) {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      return initialValue;
    }
  });

  const setValue = (value) => {
    try {
      setStoredValue(value);
      window.localStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
      console.error(error);
    }
  };

  return [storedValue, setValue];
}

// Usage
function App() {
  const [name, setName] = useLocalStorage('name', 'Guest');
  
  return (
    <input 
      value={name}
      onChange={(e) => setName(e.target.value)}
    />
  );
}
```

## Best Practices

1. **Use ESLint plugin**: Install `eslint-plugin-react-hooks` to enforce rules
2. **Keep hooks at the top level**: Never conditionally call hooks
3. **Name custom hooks with "use" prefix**: Makes them recognizable as hooks
4. **Clean up effects**: Return cleanup functions when needed
5. **Optimize with useMemo and useCallback**: But don't overuse them

## Conclusion

React Hooks have made functional components more powerful and cleaner. They eliminate the need for class components in most cases and make code more reusable and maintainable. Start incorporating hooks in your React applications today!
