# React Testing Best Practices: Complete Guide to Testing React Applications

## Introduction

Testing is a crucial part of building reliable React applications. Well-tested code gives you confidence when refactoring, helps prevent bugs, and serves as documentation. In this comprehensive guide, we'll explore testing strategies, tools, and best practices for React applications.

## Testing Philosophy

### The Testing Trophy

Modern React testing follows the "Testing Trophy" approach:

```
       /\
      /  \     E2E Tests (10%)
     /    \    
    /------\   Integration Tests (30%)
   /        \  
  /----------\ Unit Tests (50%)
 /____________\ Static Analysis (10%)
```

- **Static Analysis**: TypeScript, ESLint, Prettier
- **Unit Tests**: Individual functions and components
- **Integration Tests**: Multiple components working together
- **E2E Tests**: Full user workflows

## Testing Tools

### Essential Testing Stack

1. **Jest**: Test runner and assertion library
2. **React Testing Library**: Component testing utilities
3. **MSW (Mock Service Worker)**: API mocking
4. **Cypress/Playwright**: E2E testing

### Setup

```bash
npm install --save-dev @testing-library/react @testing-library/jest-dom @testing-library/user-event jest-environment-jsdom
```

```javascript
// jest.config.js
module.exports = {
  testEnvironment: 'jest-environment-jsdom',
  setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],
  moduleNameMapper: {
    '\\.(css|less|scss|sass)$': 'identity-obj-proxy',
  },
};

// jest.setup.js
import '@testing-library/jest-dom';
```

## Unit Testing Components

### Basic Component Test

```jsx
// Button.jsx
export function Button({ onClick, children, disabled }) {
  return (
    <button onClick={onClick} disabled={disabled}>
      {children}
    </button>
  );
}

// Button.test.jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { Button } from './Button';

describe('Button', () => {
  it('renders with text', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByRole('button')).toHaveTextContent('Click me');
  });

  it('calls onClick when clicked', async () => {
    const handleClick = jest.fn();
    const user = userEvent.setup();
    
    render(<Button onClick={handleClick}>Click me</Button>);
    
    await user.click(screen.getByRole('button'));
    
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it('is disabled when disabled prop is true', () => {
    render(<Button disabled>Click me</Button>);
    expect(screen.getByRole('button')).toBeDisabled();
  });
});
```

### Testing Components with State

```jsx
// Counter.jsx
import { useState } from 'react';

export function Counter({ initialCount = 0 }) {
  const [count, setCount] = useState(initialCount);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <button onClick={() => setCount(count - 1)}>Decrement</button>
      <button onClick={() => setCount(0)}>Reset</button>
    </div>
  );
}

// Counter.test.jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { Counter } from './Counter';

describe('Counter', () => {
  it('starts with initial count', () => {
    render(<Counter initialCount={5} />);
    expect(screen.getByText('Count: 5')).toBeInTheDocument();
  });

  it('increments count when increment button is clicked', async () => {
    const user = userEvent.setup();
    render(<Counter />);
    
    await user.click(screen.getByRole('button', { name: /increment/i }));
    
    expect(screen.getByText('Count: 1')).toBeInTheDocument();
  });

  it('decrements count when decrement button is clicked', async () => {
    const user = userEvent.setup();
    render(<Counter initialCount={5} />);
    
    await user.click(screen.getByRole('button', { name: /decrement/i }));
    
    expect(screen.getByText('Count: 4')).toBeInTheDocument();
  });

  it('resets count when reset button is clicked', async () => {
    const user = userEvent.setup();
    render(<Counter initialCount={10} />);
    
    await user.click(screen.getByRole('button', { name: /reset/i }));
    
    expect(screen.getByText('Count: 0')).toBeInTheDocument();
  });
});
```

### Testing Forms

```jsx
// LoginForm.jsx
import { useState } from 'react';

export function LoginForm({ onSubmit }) {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');

  const handleSubmit = async (e) => {
    e.preventDefault();
    setError('');

    if (!email || !password) {
      setError('Email and password are required');
      return;
    }

    try {
      await onSubmit({ email, password });
    } catch (err) {
      setError(err.message);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        placeholder="Email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        aria-label="Email"
      />
      <input
        type="password"
        placeholder="Password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        aria-label="Password"
      />
      <button type="submit">Login</button>
      {error && <div role="alert">{error}</div>}
    </form>
  );
}

// LoginForm.test.jsx
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { LoginForm } from './LoginForm';

describe('LoginForm', () => {
  it('submits form with email and password', async () => {
    const handleSubmit = jest.fn().mockResolvedValue(undefined);
    const user = userEvent.setup();
    
    render(<LoginForm onSubmit={handleSubmit} />);
    
    await user.type(screen.getByLabelText(/email/i), 'test@example.com');
    await user.type(screen.getByLabelText(/password/i), 'password123');
    await user.click(screen.getByRole('button', { name: /login/i }));
    
    await waitFor(() => {
      expect(handleSubmit).toHaveBeenCalledWith({
        email: 'test@example.com',
        password: 'password123'
      });
    });
  });

  it('shows error when fields are empty', async () => {
    const handleSubmit = jest.fn();
    const user = userEvent.setup();
    
    render(<LoginForm onSubmit={handleSubmit} />);
    
    await user.click(screen.getByRole('button', { name: /login/i }));
    
    expect(screen.getByRole('alert')).toHaveTextContent(
      'Email and password are required'
    );
    expect(handleSubmit).not.toHaveBeenCalled();
  });

  it('displays error from failed submission', async () => {
    const handleSubmit = jest.fn().mockRejectedValue(
      new Error('Invalid credentials')
    );
    const user = userEvent.setup();
    
    render(<LoginForm onSubmit={handleSubmit} />);
    
    await user.type(screen.getByLabelText(/email/i), 'test@example.com');
    await user.type(screen.getByLabelText(/password/i), 'wrong');
    await user.click(screen.getByRole('button', { name: /login/i }));
    
    await waitFor(() => {
      expect(screen.getByRole('alert')).toHaveTextContent('Invalid credentials');
    });
  });
});
```

## Testing Hooks

### Custom Hook Testing

```jsx
// useCounter.js
import { useState, useCallback } from 'react';

export function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);

  const increment = useCallback(() => setCount(c => c + 1), []);
  const decrement = useCallback(() => setCount(c => c - 1), []);
  const reset = useCallback(() => setCount(initialValue), [initialValue]);

  return { count, increment, decrement, reset };
}

// useCounter.test.js
import { renderHook, act } from '@testing-library/react';
import { useCounter } from './useCounter';

describe('useCounter', () => {
  it('initializes with default value', () => {
    const { result } = renderHook(() => useCounter());
    expect(result.current.count).toBe(0);
  });

  it('initializes with custom value', () => {
    const { result } = renderHook(() => useCounter(10));
    expect(result.current.count).toBe(10);
  });

  it('increments count', () => {
    const { result } = renderHook(() => useCounter());
    
    act(() => {
      result.current.increment();
    });
    
    expect(result.current.count).toBe(1);
  });

  it('decrements count', () => {
    const { result } = renderHook(() => useCounter(5));
    
    act(() => {
      result.current.decrement();
    });
    
    expect(result.current.count).toBe(4);
  });

  it('resets to initial value', () => {
    const { result } = renderHook(() => useCounter(10));
    
    act(() => {
      result.current.increment();
      result.current.increment();
    });
    
    expect(result.current.count).toBe(12);
    
    act(() => {
      result.current.reset();
    });
    
    expect(result.current.count).toBe(10);
  });
});
```

## Testing Async Operations

### Mocking API Calls with MSW

```javascript
// mocks/handlers.js
import { rest } from 'msw';

export const handlers = [
  rest.get('/api/users', (req, res, ctx) => {
    return res(
      ctx.json([
        { id: 1, name: 'John Doe' },
        { id: 2, name: 'Jane Smith' }
      ])
    );
  }),

  rest.post('/api/login', async (req, res, ctx) => {
    const { email, password } = await req.json();
    
    if (email === 'test@example.com' && password === 'password') {
      return res(
        ctx.json({
          token: 'fake-token',
          user: { id: 1, email }
        })
      );
    }
    
    return res(
      ctx.status(401),
      ctx.json({ message: 'Invalid credentials' })
    );
  })
];

// mocks/server.js
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

export const server = setupServer(...handlers);

// jest.setup.js
import { server } from './mocks/server';

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

### Testing Components with API Calls

```jsx
// UserList.jsx
import { useEffect, useState } from 'react';

export function UserList() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetch('/api/users')
      .then(res => res.json())
      .then(data => {
        setUsers(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err.message);
        setLoading(false);
      });
  }, []);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}

// UserList.test.jsx
import { render, screen, waitFor } from '@testing-library/react';
import { UserList } from './UserList';

describe('UserList', () => {
  it('displays loading state initially', () => {
    render(<UserList />);
    expect(screen.getByText(/loading/i)).toBeInTheDocument();
  });

  it('displays users after loading', async () => {
    render(<UserList />);
    
    await waitFor(() => {
      expect(screen.getByText('John Doe')).toBeInTheDocument();
      expect(screen.getByText('Jane Smith')).toBeInTheDocument();
    });
  });

  it('handles error state', async () => {
    server.use(
      rest.get('/api/users', (req, res, ctx) => {
        return res(ctx.status(500));
      })
    );

    render(<UserList />);
    
    await waitFor(() => {
      expect(screen.getByText(/error/i)).toBeInTheDocument();
    });
  });
});
```

## Testing Context

```jsx
// AuthContext.jsx
import { createContext, useContext, useState } from 'react';

const AuthContext = createContext();

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);

  const login = (userData) => setUser(userData);
  const logout = () => setUser(null);

  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
}

// AuthContext.test.jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { AuthProvider, useAuth } from './AuthContext';

function TestComponent() {
  const { user, login, logout } = useAuth();

  return (
    <div>
      {user ? (
        <>
          <p>User: {user.name}</p>
          <button onClick={logout}>Logout</button>
        </>
      ) : (
        <button onClick={() => login({ name: 'John' })}>Login</button>
      )}
    </div>
  );
}

describe('AuthContext', () => {
  it('provides authentication functionality', async () => {
    const user = userEvent.setup();
    
    render(
      <AuthProvider>
        <TestComponent />
      </AuthProvider>
    );

    // Initially logged out
    expect(screen.getByRole('button', { name: /login/i })).toBeInTheDocument();

    // Login
    await user.click(screen.getByRole('button', { name: /login/i }));
    expect(screen.getByText('User: John')).toBeInTheDocument();

    // Logout
    await user.click(screen.getByRole('button', { name: /logout/i }));
    expect(screen.getByRole('button', { name: /login/i })).toBeInTheDocument();
  });
});
```

## Best Practices

### 1. Test User Behavior, Not Implementation

```jsx
// ❌ Bad: Testing implementation details
expect(component.state.count).toBe(1);

// ✅ Good: Testing user-visible behavior
expect(screen.getByText('Count: 1')).toBeInTheDocument();
```

### 2. Use Semantic Queries

```jsx
// ✅ Best: Accessible to everyone
screen.getByRole('button', { name: /submit/i });
screen.getByLabelText(/email/i);

// ⚠️ Okay: If role doesn't work
screen.getByText(/submit/i);

// ❌ Avoid: Implementation details
screen.getByTestId('submit-button');
screen.getByClassName('btn-primary');
```

### 3. Avoid Testing External Libraries

```jsx
// ❌ Don't test react-router
expect(mockNavigate).toHaveBeenCalledWith('/dashboard');

// ✅ Test your component behavior
expect(screen.getByRole('link')).toHaveAttribute('href', '/dashboard');
```

### 4. Keep Tests Simple and Focused

```jsx
// ✅ One assertion per test (or closely related)
it('displays user name', () => {
  render(<User name="John" />);
  expect(screen.getByText('John')).toBeInTheDocument();
});

it('displays user email', () => {
  render(<User email="john@example.com" />);
  expect(screen.getByText('john@example.com')).toBeInTheDocument();
});
```

## Conclusion

Effective React testing balances coverage with maintainability. Focus on testing user behavior, use integration tests for most scenarios, and reserve unit tests for complex logic. Remember: tests should give you confidence, not slow you down!

Happy testing!
