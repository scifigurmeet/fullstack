# 🚀 Advanced React Hooks Study Guide - Part 1

## ⏰ Time Allocation: 40 minutes

---

## 📚 Table of Contents
1. [useReducer Hook](#-usereducer-hook)
2. [useMemo Hook](#-usememo-hook)
3. [useCallback Hook](#-usecallback-hook)
4. [Performance Optimization](#-performance-optimization)
5. [Best Practices](#-best-practices)

---

## 🔄 useReducer Hook

### 🎯 What & When?
- Alternative to `useState` for **complex state logic**
- Multiple state updates or state depends on previous state
- Pattern: `const [state, dispatch] = useReducer(reducer, initialState)`

### 💡 Simple Example
```javascript
import React, { useReducer } from 'react';

// 1. Reducer function
function counterReducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    case 'reset':
      return { count: 0 };
    default:
      return state;
  }
}

// 2. Component
function Counter() {
  const [state, dispatch] = useReducer(counterReducer, { count: 0 });

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'reset' })}>Reset</button>
    </div>
  );
}
```

### 🛒 Real-World Example
```javascript
// Shopping cart reducer
function cartReducer(state, action) {
  switch (action.type) {
    case 'ADD_ITEM':
      const existingItem = state.items.find(item => item.id === action.payload.id);
      if (existingItem) {
        return {
          ...state,
          items: state.items.map(item =>
            item.id === action.payload.id
              ? { ...item, quantity: item.quantity + 1 }
              : item
          )
        };
      }
      return {
        ...state,
        items: [...state.items, { ...action.payload, quantity: 1 }]
      };
    
    case 'REMOVE_ITEM':
      return {
        ...state,
        items: state.items.filter(item => item.id !== action.payload)
      };
    
    default:
      return state;
  }
}

function ShoppingCart() {
  const [cart, dispatch] = useReducer(cartReducer, { items: [] });

  const addItem = (item) => dispatch({ type: 'ADD_ITEM', payload: item });
  const removeItem = (id) => dispatch({ type: 'REMOVE_ITEM', payload: id });

  return (
    <div>
      <h3>Cart Items: {cart.items.length}</h3>
      <button onClick={() => addItem({ id: 1, name: 'Laptop', price: 999 })}>
        Add Laptop
      </button>
      {cart.items.map(item => (
        <div key={item.id}>
          {item.name} x{item.quantity}
          <button onClick={() => removeItem(item.id)}>Remove</button>
        </div>
      ))}
    </div>
  );
}
```

---

## 🧠 useMemo Hook

### 🎯 What & When?
- **Cache expensive calculations** between re-renders
- Prevent unnecessary computations when dependencies don't change
- Pattern: `const cachedValue = useMemo(() => calculation, [dependencies])`

### 💡 Simple Example
```javascript
import React, { useState, useMemo } from 'react';

function ExpensiveCalculation() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('');

  // ❌ Without useMemo - runs on EVERY render (even when name changes)
  const expensiveValue = heavyCalculation(count);

  // ✅ With useMemo - only runs when count changes
  const memoizedValue = useMemo(() => {
    console.log('🧮 Calculating...');
    return heavyCalculation(count);
  }, [count]); // Only recalculate when count changes

  return (
    <div>
      <p>Result: {memoizedValue}</p>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <input 
        value={name} 
        onChange={(e) => setName(e.target.value)} 
        placeholder="Type here (watch console)"
      />
    </div>
  );
}

function heavyCalculation(num) {
  // Simulate expensive calculation
  let result = 0;
  for (let i = 0; i < 1000000; i++) {
    result += num * i;
  }
  return result;
}
```

### 🎨 Object Memoization Example
```javascript
import React, { useState, useMemo } from 'react';

const ChildComponent = React.memo(({ config }) => {
  console.log('🔄 Child rendered');
  return <div>Theme: {config.theme}</div>;
});

function Parent() {
  const [count, setCount] = useState(0);
  const [theme, setTheme] = useState('light');

  // ❌ New object every render - child re-renders unnecessarily
  const configBad = {
    theme: theme,
    apiUrl: 'https://api.example.com'
  };

  // ✅ Same object reference when theme doesn't change
  const configGood = useMemo(() => ({
    theme: theme,
    apiUrl: 'https://api.example.com'
  }), [theme]);

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
        Toggle Theme
      </button>
      <ChildComponent config={configGood} />
    </div>
  );
}
```

---

## ⚡ useCallback Hook

### 🎯 What & When?
- **Cache function definitions** between re-renders
- Prevent child component re-renders when function props don't change
- Pattern: `const cachedFn = useCallback(fn, [dependencies])`

### 💡 Simple Example
```javascript
import React, { useState, useCallback } from 'react';

const Button = React.memo(({ onClick, children }) => {
  console.log(`🔄 Rendering button: ${children}`);
  return <button onClick={onClick}>{children}</button>;
});

function App() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('');

  // ❌ New function every render - button re-renders unnecessarily
  const handleClickBad = () => {
    setCount(count + 1);
  };

  // ✅ Same function reference - button doesn't re-render
  const handleClickGood = useCallback(() => {
    setCount(prev => prev + 1); // Use prev to avoid count dependency
  }, []); // Empty deps - function never changes

  return (
    <div>
      <p>Count: {count}</p>
      <input 
        value={name} 
        onChange={(e) => setName(e.target.value)}
        placeholder="Type here (watch console)"
      />
      <Button onClick={handleClickGood}>Increment</Button>
    </div>
  );
}
```

### 🎯 Dependencies Example
```javascript
function SearchComponent({ searchTerm }) {
  const [results, setResults] = useState([]);

  // Function that depends on searchTerm
  const performSearch = useCallback(async () => {
    if (searchTerm) {
      const data = await fetch(`/api/search?q=${searchTerm}`);
      setResults(await data.json());
    }
  }, [searchTerm]); // Re-create when searchTerm changes

  // Use in useEffect
  useEffect(() => {
    performSearch();
  }, [performSearch]); // Safe because performSearch is stable

  return <div>Results: {results.length}</div>;
}
```

---

## 🚀 Performance Optimization

### Combining All Three Hooks
```javascript
import React, { useState, useMemo, useCallback, useReducer } from 'react';

const todosReducer = (state, action) => {
  switch (action.type) {
    case 'ADD_TODO':
      return [...state, { id: Date.now(), text: action.text, done: false }];
    case 'TOGGLE_TODO':
      return state.map(todo =>
        todo.id === action.id ? { ...todo, done: !todo.done } : todo
      );
    default:
      return state;
  }
};

function TodoApp() {
  const [todos, dispatch] = useReducer(todosReducer, []);
  const [filter, setFilter] = useState('all');
  const [newTodo, setNewTodo] = useState('');

  // ✅ useMemo for expensive filtering
  const filteredTodos = useMemo(() => {
    switch (filter) {
      case 'active': return todos.filter(todo => !todo.done);
      case 'completed': return todos.filter(todo => todo.done);
      default: return todos;
    }
  }, [todos, filter]);

  // ✅ useCallback for stable event handlers
  const addTodo = useCallback(() => {
    if (newTodo.trim()) {
      dispatch({ type: 'ADD_TODO', text: newTodo });
      setNewTodo('');
    }
  }, [newTodo]);

  const toggleTodo = useCallback((id) => {
    dispatch({ type: 'TOGGLE_TODO', id });
  }, []);

  return (
    <div>
      <input 
        value={newTodo}
        onChange={(e) => setNewTodo(e.target.value)}
        placeholder="Add todo"
      />
      <button onClick={addTodo}>Add</button>
      
      <select value={filter} onChange={(e) => setFilter(e.target.value)}>
        <option value="all">All</option>
        <option value="active">Active</option>
        <option value="completed">Completed</option>
      </select>

      {filteredTodos.map(todo => (
        <TodoItem 
          key={todo.id} 
          todo={todo} 
          onToggle={toggleTodo} 
        />
      ))}
    </div>
  );
}

const TodoItem = React.memo(({ todo, onToggle }) => (
  <div>
    <input 
      type="checkbox" 
      checked={todo.done}
      onChange={() => onToggle(todo.id)}
    />
    {todo.text}
  </div>
));
```

---

## ✅ Best Practices

### ❌ Common Mistakes
```javascript
// ❌ Don't memoize primitive calculations
const double = useMemo(() => count * 2, [count]); // Unnecessary

// ❌ Missing dependencies
const callback = useCallback(() => {
  console.log(someValue); // someValue should be in deps
}, []); // Missing someValue dependency

// ❌ Over-optimization
const simpleString = useMemo(() => 'Hello World', []); // Unnecessary
```

### ✅ Good Patterns
```javascript
// ✅ Memoize expensive operations
const expensiveResult = useMemo(() => 
  largeArray.filter(...).sort(...).map(...), 
  [largeArray]
);

// ✅ Stable event handlers
const handleClick = useCallback(() => {
  onClick(id);
}, [onClick, id]);

// ✅ Proper reducer structure
const reducer = (state, action) => {
  switch (action.type) {
    case 'UPDATE':
      return { ...state, ...action.payload }; // New object
    default:
      return state;
  }
};
```

---

## 🎯 Quick Reference

| Hook | Use Case | Pattern |
|------|----------|---------|
| **useReducer** | Complex state logic | `useReducer(reducer, initialState)` |
| **useMemo** | Cache expensive calculations | `useMemo(() => calc, [deps])` |
| **useCallback** | Stable function references | `useCallback(fn, [deps])` |

### 🔍 When to Use Each

- **useReducer**: Multiple related state updates, complex state logic
- **useMemo**: Expensive calculations, preventing object recreation
- **useCallback**: Event handlers, functions passed to optimized components

### 🚨 Remember
- **Measure before optimizing** - use React DevTools Profiler
- **Dependencies matter** - include ALL used variables
- **Don't over-optimize** - simple calculations don't need memoization

---

**Next**: Part 2 covers Custom Hooks - creating reusable stateful logic! 🎣