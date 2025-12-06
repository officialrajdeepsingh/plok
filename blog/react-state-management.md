# React State Management: From Context API to Redux

## Introduction

State management is one of the most crucial aspects of building React applications. As your application grows, managing state becomes increasingly complex. In this guide, we'll explore various state management solutions in React, from built-in options to popular third-party libraries.

## Understanding State in React

State represents the data that changes over time in your application. It can be:

- **Local State**: Data managed within a single component
- **Global State**: Data shared across multiple components
- **Server State**: Data fetched from external sources
- **URL State**: Data stored in the URL (query parameters, path)

## Built-In React State Management

### 1. Component State with useState

Perfect for local component state:

```jsx
import { useState } from 'react';

function TodoList() {
  const [todos, setTodos] = useState([]);
  const [input, setInput] = useState('');

  const addTodo = () => {
    setTodos([...todos, { id: Date.now(), text: input }]);
    setInput('');
  };

  return (
    <div>
      <input 
        value={input}
        onChange={(e) => setInput(e.target.value)}
      />
      <button onClick={addTodo}>Add Todo</button>
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </div>
  );
}
```

### 2. Context API for Global State

Share state across components without prop drilling:

```jsx
import { createContext, useContext, useState } from 'react';

// Create context
const AuthContext = createContext();

// Provider component
export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [isAuthenticated, setIsAuthenticated] = useState(false);

  const login = async (email, password) => {
    const userData = await authenticateUser(email, password);
    setUser(userData);
    setIsAuthenticated(true);
  };

  const logout = () => {
    setUser(null);
    setIsAuthenticated(false);
  };

  return (
    <AuthContext.Provider value={{ user, isAuthenticated, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

// Custom hook for using auth context
export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
}

// Usage in components
function LoginButton() {
  const { login, isAuthenticated } = useAuth();

  if (isAuthenticated) {
    return <div>You're logged in!</div>;
  }

  return (
    <button onClick={() => login('user@example.com', 'password')}>
      Login
    </button>
  );
}
```

### 3. useReducer for Complex State Logic

When state logic becomes complex, useReducer provides a more structured approach:

```jsx
import { useReducer } from 'react';

const initialState = {
  cart: [],
  total: 0,
  itemCount: 0
};

function cartReducer(state, action) {
  switch (action.type) {
    case 'ADD_ITEM':
      const newCart = [...state.cart, action.payload];
      return {
        ...state,
        cart: newCart,
        itemCount: newCart.length,
        total: state.total + action.payload.price
      };
    
    case 'REMOVE_ITEM':
      const filteredCart = state.cart.filter(item => item.id !== action.payload);
      const removedItem = state.cart.find(item => item.id === action.payload);
      return {
        ...state,
        cart: filteredCart,
        itemCount: filteredCart.length,
        total: state.total - removedItem.price
      };
    
    case 'CLEAR_CART':
      return initialState;
    
    default:
      return state;
  }
}

function ShoppingCart() {
  const [state, dispatch] = useReducer(cartReducer, initialState);

  const addItem = (item) => {
    dispatch({ type: 'ADD_ITEM', payload: item });
  };

  const removeItem = (id) => {
    dispatch({ type: 'REMOVE_ITEM', payload: id });
  };

  return (
    <div>
      <h2>Cart ({state.itemCount} items)</h2>
      <p>Total: ${state.total}</p>
      {state.cart.map(item => (
        <div key={item.id}>
          {item.name} - ${item.price}
          <button onClick={() => removeItem(item.id)}>Remove</button>
        </div>
      ))}
    </div>
  );
}
```

## Redux: Enterprise-Grade State Management

Redux is the most popular state management library for React:

```jsx
// store.js
import { configureStore, createSlice } from '@reduxjs/toolkit';

// Create a slice
const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0 },
  reducers: {
    increment: (state) => {
      state.value += 1;
    },
    decrement: (state) => {
      state.value -= 1;
    },
    incrementByAmount: (state, action) => {
      state.value += action.payload;
    }
  }
});

export const { increment, decrement, incrementByAmount } = counterSlice.actions;

export const store = configureStore({
  reducer: {
    counter: counterSlice.reducer
  }
});

// App.js
import { Provider } from 'react-redux';
import { store } from './store';

function App() {
  return (
    <Provider store={store}>
      <Counter />
    </Provider>
  );
}

// Counter.js
import { useSelector, useDispatch } from 'react-redux';
import { increment, decrement } from './store';

function Counter() {
  const count = useSelector((state) => state.counter.value);
  const dispatch = useDispatch();

  return (
    <div>
      <h1>{count}</h1>
      <button onClick={() => dispatch(increment())}>+</button>
      <button onClick={() => dispatch(decrement())}>-</button>
    </div>
  );
}
```

## Zustand: Lightweight Alternative

Zustand provides a simpler API with less boilerplate:

```jsx
import create from 'zustand';

const useStore = create((set) => ({
  bears: 0,
  increasePopulation: () => set((state) => ({ bears: state.bears + 1 })),
  removeAllBears: () => set({ bears: 0 })
}));

function BearCounter() {
  const bears = useStore((state) => state.bears);
  return <h1>{bears} bears around here...</h1>;
}

function Controls() {
  const increasePopulation = useStore((state) => state.increasePopulation);
  return <button onClick={increasePopulation}>Add bear</button>;
}
```

## When to Use What?

### Use useState when:
- State is local to a single component
- Simple state updates
- No need to share state

### Use Context API when:
- Need to share state across many components
- Moderate complexity
- Don't need time-travel debugging

### Use useReducer when:
- Complex state logic
- Multiple sub-values
- State transitions depend on previous state

### Use Redux when:
- Large-scale applications
- Need time-travel debugging
- Complex state interactions
- Middleware needed

### Use Zustand when:
- Want Redux-like features with simpler API
- Medium-sized applications
- Don't need Redux DevTools

## Best Practices

1. **Keep state as local as possible**: Don't make everything global
2. **Normalize your state**: Avoid deeply nested structures
3. **Separate concerns**: UI state vs. server state
4. **Use TypeScript**: Type-safe state management
5. **Consider server state libraries**: React Query or SWR for API data
6. **Avoid prop drilling**: Use Context API or state management libraries
7. **Memoize selectors**: Use reselect with Redux

## Conclusion

Choose your state management solution based on your application's needs. Start simple with useState and Context API, and only introduce more complex solutions like Redux when necessary. Remember, the best state management is the simplest one that meets your requirements.

Happy state managing!
