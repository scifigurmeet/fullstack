# 🐻 Zustand State Management Study Guide - Part 3

## ⏰ Time Allocation: 40 minutes

---

## 📚 Table of Contents
1. [What is Zustand?](#-what-is-zustand)
2. [Basic Setup & Usage](#-basic-setup--usage)
3. [Core Patterns](#-core-patterns)
4. [Advanced Features](#-advanced-features)
5. [Real-World Examples](#-real-world-examples)
6. [Best Practices](#-best-practices)

---

## 🎯 What is Zustand?

### 📖 Simple Definition
Zustand is a **small, fast, and scalable** state management library for React. It's much simpler than Redux but more powerful than useState for global state.

**"Zustand" = "State" in German** 🇩🇪

### 🤔 Why Choose Zustand?

#### vs useState (Context)
```javascript
// ❌ useState + Context (Complex for global state)
const ThemeContext = createContext();
const ThemeProvider = ({ children }) => {
  const [theme, setTheme] = useState('light');
  return (
    <ThemeContext.Provider value={{theme, setTheme}}>
      {children}
    </ThemeContext.Provider>
  );
};

// ✅ Zustand (Simple global state)
const useTheme = create((set) => ({
  theme: 'light',
  setTheme: (theme) => set({ theme })
}));
```

#### vs Redux
| Feature | Redux | Zustand |
|---------|-------|---------|
| **Boilerplate** | 📦 Lots | 🪶 Minimal |
| **Bundle Size** | 🐘 ~100kb | 🐭 ~2kb |
| **Learning Curve** | 📚 Steep | 🛝 Gentle |
| **DevTools** | ✅ Built-in | ✅ Optional |
| **TypeScript** | 🟡 Complex | ✅ Simple |

### 🏗️ Core Architecture

```mermaid
graph TD
    A[Zustand Store] --> B[Component 1]
    A --> C[Component 2]
    A --> D[Component 3]
    
    B --> E[get() - Read State]
    B --> F[set() - Update State]
    
    style A fill:#fff3e0
    style B fill:#e1f5fe
    style C fill:#e1f5fe
    style D fill:#e1f5fe
    style E fill:#e8f5e8
    style F fill:#ffe8e8
```

**Key Points:**
- **One store** can be used by multiple components
- **Direct subscription** - no providers needed
- **Automatic re-renders** when state changes
- **Simple API** - just `get`, `set`, and store functions

---

## 🚀 Basic Setup & Usage

### 📦 Installation
```bash
npm install zustand
# or
yarn add zustand
```

### 🏁 Your First Store

#### Step 1: Create a Store
```javascript
import { create } from 'zustand';

// Simple counter store
const useCounterStore = create((set, get) => ({
  // State
  count: 0,
  
  // Actions
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
  reset: () => set({ count: 0 }),
  
  // Computed values (getters)
  getDoubleCount: () => get().count * 2,
}));
```

**Explanation:**
- `create()` - Creates the store
- `set()` - Updates state (triggers re-renders)
- `get()` - Reads current state
- Return object contains state + actions

#### Step 2: Use in Components
```javascript
function Counter() {
  // Subscribe to specific state
  const count = useCounterStore((state) => state.count);
  const { increment, decrement, reset } = useCounterStore();
  
  return (
    <div>
      <h2>Count: {count}</h2>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
}

function DisplayCounter() {
  // Only subscribe to count - won't re-render if other state changes
  const count = useCounterStore((state) => state.count);
  
  return <p>Current count is: {count}</p>;
}

function DoubleDisplay() {
  const getDoubleCount = useCounterStore((state) => state.getDoubleCount);
  
  return <p>Double count: {getDoubleCount()}</p>;
}
```

**Magic:** Each component automatically re-renders when subscribed state changes! ✨

---

## 🎨 Core Patterns

### 1. 🎛️ State Updates

#### Simple Updates
```javascript
const useStore = create((set) => ({
  name: '',
  age: 0,
  
  // Direct property update
  setName: (name) => set({ name }),
  setAge: (age) => set({ age }),
  
  // Multiple properties
  updateUser: (name, age) => set({ name, age }),
  
  // Function-based update (access previous state)
  incrementAge: () => set((state) => ({ age: state.age + 1 })),
}));
```

#### Object Updates
```javascript
const useUserStore = create((set) => ({
  user: {
    id: null,
    name: '',
    preferences: {
      theme: 'light',
      language: 'en'
    }
  },
  
  // ✅ Shallow merge (Zustand merges top-level)
  updateUser: (userData) => set((state) => ({
    user: { ...state.user, ...userData }
  })),
  
  // ✅ Deep nested updates
  updatePreference: (key, value) => set((state) => ({
    user: {
      ...state.user,
      preferences: {
        ...state.user.preferences,
        [key]: value
      }
    }
  })),
}));

// Usage
function UserSettings() {
  const { user, updateUser, updatePreference } = useUserStore();
  
  return (
    <div>
      <input 
        value={user.name}
        onChange={(e) => updateUser({ name: e.target.value })}
        placeholder="Name"
      />
      
      <select 
        value={user.preferences.theme}
        onChange={(e) => updatePreference('theme', e.target.value)}
      >
        <option value="light">Light</option>
        <option value="dark">Dark</option>
      </select>
    </div>
  );
}
```

### 2. 📋 Array Operations

```javascript
const useTodosStore = create((set, get) => ({
  todos: [],
  
  // Add todo
  addTodo: (text) => set((state) => ({
    todos: [...state.todos, {
      id: Date.now(),
      text,
      completed: false
    }]
  })),
  
  // Remove todo
  removeTodo: (id) => set((state) => ({
    todos: state.todos.filter(todo => todo.id !== id)
  })),
  
  // Toggle todo
  toggleTodo: (id) => set((state) => ({
    todos: state.todos.map(todo =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    )
  })),
  
  // Clear completed
  clearCompleted: () => set((state) => ({
    todos: state.todos.filter(todo => !todo.completed)
  })),
  
  // Computed values
  get completedCount() {
    return get().todos.filter(todo => todo.completed).length;
  },
  
  get pendingCount() {
    return get().todos.filter(todo => !todo.completed).length;
  }
}));

// Usage
function TodoApp() {
  const { todos, addTodo, toggleTodo, removeTodo, completedCount } = useTodosStore();
  const [newTodo, setNewTodo] = useState('');
  
  const handleSubmit = (e) => {
    e.preventDefault();
    if (newTodo.trim()) {
      addTodo(newTodo);
      setNewTodo('');
    }
  };
  
  return (
    <div>
      <form onSubmit={handleSubmit}>
        <input 
          value={newTodo}
          onChange={(e) => setNewTodo(e.target.value)}
          placeholder="Add todo..."
        />
        <button type="submit">Add</button>
      </form>
      
      <p>Completed: {completedCount} / {todos.length}</p>
      
      {todos.map(todo => (
        <div key={todo.id}>
          <input 
            type="checkbox"
            checked={todo.completed}
            onChange={() => toggleTodo(todo.id)}
          />
          <span style={{ 
            textDecoration: todo.completed ? 'line-through' : 'none' 
          }}>
            {todo.text}
          </span>
          <button onClick={() => removeTodo(todo.id)}>Delete</button>
        </div>
      ))}
    </div>
  );
}
```

---

## 🔥 Advanced Features

### 1. 🎯 Selective Subscriptions (Performance)

```javascript
const useStore = create((set) => ({
  user: { name: 'John', email: 'john@email.com' },
  posts: ['Post 1', 'Post 2'],
  theme: 'light',
  
  updateUser: (user) => set({ user }),
  addPost: (post) => set((state) => ({ posts: [...state.posts, post] })),
  setTheme: (theme) => set({ theme }),
}));

function OptimizedComponent() {
  // ✅ Only re-renders when user.name changes
  const userName = useStore((state) => state.user.name);
  
  // ✅ Only re-renders when posts array changes
  const posts = useStore((state) => state.posts);
  
  // ❌ Re-renders on ANY state change
  const allState = useStore();
  
  return (
    <div>
      <h2>{userName}</h2>
      <p>Posts: {posts.length}</p>
    </div>
  );
}
```

**Pro Tip:** Always subscribe to the smallest piece of state you need! 🎯

### 2. 🔄 Middleware

#### Persistence (localStorage)
```javascript
import { persist } from 'zustand/middleware';

const useSettingsStore = create(
  persist(
    (set) => ({
      theme: 'light',
      language: 'en',
      notifications: true,
      
      setTheme: (theme) => set({ theme }),
      setLanguage: (language) => set({ language }),
      toggleNotifications: () => set((state) => ({ 
        notifications: !state.notifications 
      })),
    }),
    {
      name: 'app-settings', // localStorage key
      // Optional: only persist specific keys
      partialize: (state) => ({ 
        theme: state.theme, 
        language: state.language 
      }),
    }
  )
);

// Usage - automatically persists to localStorage! 💾
function SettingsPanel() {
  const { theme, language, setTheme, setLanguage } = useSettingsStore();
  
  return (
    <div>
      <select value={theme} onChange={(e) => setTheme(e.target.value)}>
        <option value="light">Light</option>
        <option value="dark">Dark</option>
      </select>
      
      <select value={language} onChange={(e) => setLanguage(e.target.value)}>
        <option value="en">English</option>
        <option value="es">Spanish</option>
      </select>
      
      <p>Settings are automatically saved! 🎉</p>
    </div>
  );
}
```

#### DevTools Integration
```javascript
import { devtools } from 'zustand/middleware';

const useStore = create(
  devtools(
    (set) => ({
      count: 0,
      increment: () => set((state) => ({ count: state.count + 1 }), 'increment'),
      decrement: () => set((state) => ({ count: state.count - 1 }), 'decrement'),
    }),
    { name: 'counter-store' } // Shows in Redux DevTools
  )
);
```

### 3. 🏪 Multiple Stores

```javascript
// User store
export const useUserStore = create((set) => ({
  user: null,
  isAuthenticated: false,
  
  login: (userData) => set({ user: userData, isAuthenticated: true }),
  logout: () => set({ user: null, isAuthenticated: false }),
}));

// Cart store
export const useCartStore = create((set, get) => ({
  items: [],
  
  addItem: (product) => set((state) => ({
    items: [...state.items, { ...product, quantity: 1 }]
  })),
  
  getTotalPrice: () => {
    const items = get().items;
    return items.reduce((total, item) => total + (item.price * item.quantity), 0);
  }
}));

// Component using multiple stores
function Checkout() {
  const user = useUserStore((state) => state.user);
  const { items, getTotalPrice } = useCartStore();
  
  return (
    <div>
      <h2>Checkout for {user?.name}</h2>
      <p>Items: {items.length}</p>
      <p>Total: ${getTotalPrice()}</p>
    </div>
  );
}
```

---

## 🌟 Real-World Examples

### 🛒 Shopping Cart Store

```javascript
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

const useCartStore = create(
  persist(
    (set, get) => ({
      // State
      items: [],
      isOpen: false,
      
      // Cart actions
      addItem: (product) => {
        const items = get().items;
        const existingItem = items.find(item => item.id === product.id);
        
        if (existingItem) {
          // Update quantity
          set({
            items: items.map(item =>
              item.id === product.id
                ? { ...item, quantity: item.quantity + 1 }
                : item
            )
          });
        } else {
          // Add new item
          set({
            items: [...items, { ...product, quantity: 1 }]
          });
        }
      },
      
      removeItem: (productId) => set((state) => ({
        items: state.items.filter(item => item.id !== productId)
      })),
      
      updateQuantity: (productId, quantity) => {
        if (quantity <= 0) {
          get().removeItem(productId);
          return;
        }
        
        set((state) => ({
          items: state.items.map(item =>
            item.id === productId ? { ...item, quantity } : item
          )
        }));
      },
      
      clearCart: () => set({ items: [] }),
      
      // UI actions
      openCart: () => set({ isOpen: true }),
      closeCart: () => set({ isOpen: false }),
      toggleCart: () => set((state) => ({ isOpen: !state.isOpen })),
      
      // Computed values
      get itemCount() {
        return get().items.reduce((total, item) => total + item.quantity, 0);
      },
      
      get totalPrice() {
        return get().items.reduce(
          (total, item) => total + (item.price * item.quantity), 
          0
        );
      },
    }),
    { name: 'shopping-cart' }
  )
);

// Usage Components
function ProductCard({ product }) {
  const addItem = useCartStore((state) => state.addItem);
  
  return (
    <div className="product-card">
      <h3>{product.name}</h3>
      <p>${product.price}</p>
      <button onClick={() => addItem(product)}>
        Add to Cart 🛒
      </button>
    </div>
  );
}

function CartIcon() {
  const { itemCount, toggleCart } = useCartStore();
  
  return (
    <button onClick={toggleCart} className="cart-icon">
      🛒 {itemCount > 0 && <span className="badge">{itemCount}</span>}
    </button>
  );
}

function CartSidebar() {
  const { items, isOpen, closeCart, updateQuantity, totalPrice } = useCartStore();
  
  if (!isOpen) return null;
  
  return (
    <div className="cart-sidebar">
      <div className="cart-header">
        <h2>Shopping Cart</h2>
        <button onClick={closeCart}>×</button>
      </div>
      
      {items.length === 0 ? (
        <p>Your cart is empty</p>
      ) : (
        <>
          {items.map(item => (
            <div key={item.id} className="cart-item">
              <h4>{item.name}</h4>
              <p>${item.price} each</p>
              <div>
                <button 
                  onClick={() => updateQuantity(item.id, item.quantity - 1)}
                >
                  -
                </button>
                <span>{item.quantity}</span>
                <button 
                  onClick={() => updateQuantity(item.id, item.quantity + 1)}
                >
                  +
                </button>
              </div>
            </div>
          ))}
          
          <div className="cart-footer">
            <h3>Total: ${totalPrice.toFixed(2)}</h3>
            <button className="checkout-btn">Checkout</button>
          </div>
        </>
      )}
    </div>
  );
}
```

### 🔐 Authentication Store

```javascript
const useAuthStore = create(
  persist(
    (set, get) => ({
      // State
      user: null,
      token: null,
      isLoading: false,
      error: null,
      
      // Computed
      get isAuthenticated() {
        return !!get().token;
      },
      
      // Actions
      login: async (email, password) => {
        set({ isLoading: true, error: null });
        
        try {
          // Simulate API call
          const response = await fetch('/api/login', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ email, password })
          });
          
          const data = await response.json();
          
          if (response.ok) {
            set({
              user: data.user,
              token: data.token,
              isLoading: false,
              error: null
            });
          } else {
            set({
              error: data.message,
              isLoading: false
            });
          }
        } catch (error) {
          set({
            error: 'Login failed. Please try again.',
            isLoading: false
          });
        }
      },
      
      logout: () => set({
        user: null,
        token: null,
        error: null
      }),
      
      clearError: () => set({ error: null }),
    }),
    {
      name: 'auth-storage',
      partialize: (state) => ({ 
        user: state.user, 
        token: state.token 
      })
    }
  )
);

// Usage
function LoginForm() {
  const { login, isLoading, error, clearError } = useAuthStore();
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  
  const handleSubmit = (e) => {
    e.preventDefault();
    login(email, password);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input 
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Email"
        required
      />
      
      <input 
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        placeholder="Password"
        required
      />
      
      {error && (
        <div className="error">
          {error}
          <button type="button" onClick={clearError}>×</button>
        </div>
      )}
      
      <button type="submit" disabled={isLoading}>
        {isLoading ? 'Logging in...' : 'Login'}
      </button>
    </form>
  );
}

function UserProfile() {
  const { user, logout, isAuthenticated } = useAuthStore();
  
  if (!isAuthenticated) {
    return <LoginForm />;
  }
  
  return (
    <div>
      <h2>Welcome, {user.name}! 👋</h2>
      <p>Email: {user.email}</p>
      <button onClick={logout}>Logout</button>
    </div>
  );
}
```

---

## ✅ Best Practices

### 🎯 Store Organization

#### ✅ Good Patterns
```javascript
// ✅ Group related state and actions
const useUserStore = create((set, get) => ({
  // State section
  user: null,
  preferences: {},
  isLoading: false,
  error: null,
  
  // Actions section
  setUser: (user) => set({ user }),
  updatePreferences: (prefs) => set((state) => ({
    preferences: { ...state.preferences, ...prefs }
  })),
  
  // Async actions section
  fetchUser: async (id) => {
    set({ isLoading: true });
    try {
      const user = await api.getUser(id);
      set({ user, isLoading: false, error: null });
    } catch (error) {
      set({ error: error.message, isLoading: false });
    }
  },
  
  // Computed values section
  get isAuthenticated() {
    return !!get().user;
  }
}));
```

#### ❌ Anti-patterns
```javascript
// ❌ Too many unrelated things in one store
const useBadStore = create((set) => ({
  user: null,
  todos: [],
  cartItems: [],
  weatherData: {},
  theme: 'light',
  // This store is doing too much!
}));

// ❌ Putting local component state in global store
const useUIStore = create((set) => ({
  modalOpen: false, // Should be useState in component
  selectedTab: 0,   // Should be useState in component
  formData: {},     // Should be useState in component
}));
```

### 🚀 Performance Tips

```javascript
// ✅ Subscribe to specific slices
function MyComponent() {
  // Good - only re-renders when name changes
  const name = useStore((state) => state.user.name);
  
  // Bad - re-renders on any store change
  const { user } = useStore();
  const name = user.name;
}

// ✅ Use shallow comparison for objects
import { shallow } from 'zustand/shallow';

function MyComponent() {
  // Only re-renders when user object properties change
  const user = useStore((state) => state.user, shallow);
}

// ✅ Memoize selectors
const selectTodosByStatus = (status) => (state) => 
  state.todos.filter(todo => todo.status === status);

function TodoList({ status }) {
  const todos = useStore(selectTodosByStatus(status));
  // ...
}
```

### 🧪 Testing

```javascript
// Easy to test stores independently
import { useCounterStore } from './store';

describe('Counter Store', () => {
  beforeEach(() => {
    useCounterStore.setState({ count: 0 }); // Reset state
  });
  
  test('increments count', () => {
    const { increment } = useCounterStore.getState();
    
    increment();
    
    expect(useCounterStore.getState().count).toBe(1);
  });
  
  test('resets count', () => {
    useCounterStore.setState({ count: 5 });
    const { reset } = useCounterStore.getState();
    
    reset();
    
    expect(useCounterStore.getState().count).toBe(0);
  });
});
```

---

## 🎯 Quick Reference

### 📋 Essential API

| Method | Purpose | Example |
|--------|---------|---------|
| `create()` | Create store | `create((set, get) => ({}))` |
| `set()` | Update state | `set({ count: 1 })` |
| `get()` | Read current state | `get().count` |
| `useStore()` | Subscribe in component | `useStore(state => state.count)` |
| `useStore.getState()` | Get state outside component | `useStore.getState().count` |
| `useStore.setState()` | Update state outside component | `useStore.setState({ count: 1 })` |

### 🏗️ Store Template

```javascript
const useStore = create((set, get) => ({
  // 📊 State
  data: initialData,
  loading: false,
  error: null,
  
  // 🔧 Simple actions
  setData: (data) => set({ data }),
  setLoading: (loading) => set({ loading }),
  
  // 🔄 Complex actions
  updateData: (updates) => set((state) => ({
    data: { ...state.data, ...updates }
  })),
  
  // 🌐 Async actions
  fetchData: async () => {
    set({ loading: true, error: null });
    try {
      const data = await api.getData();
      set({ data, loading: false });
    } catch (error) {
      set({ error: error.message, loading: false });
    }
  },
  
  // 🧮 Computed values
  get processedData() {
    return get().data.filter(item => item.active);
  }
}));
```

---

## 🎉 Summary

**🧠 Key Concepts:**
- **Simple API**: `create()`, `set()`, `get()` - that's it!
- **No boilerplate**: Direct state updates, no reducers/actions
- **Automatic subscriptions**: Components re-render when subscribed state changes
- **Flexible**: Use with or without middleware, multiple stores, etc.

**💡 Why Zustand Rocks:**
- **Small bundle size** (~2KB vs Redux ~100KB)
- **Simple learning curve** (no complex patterns)
- **Great performance** (selective subscriptions)
- **TypeScript friendly** (excellent type inference)
- **No providers needed** (works anywhere)

**🎯 When to Use:**
- ✅ Global state that multiple components need
- ✅ Complex state logic that's outgrown useState
- ✅ Want simple API without Redux complexity
- ✅ Need persistent state (with middleware)

**🚫 When NOT to Use:**
- ❌ Simple local component state (use useState)
- ❌ Already happy with Redux setup
- ❌ Need time-travel debugging (Redux DevTools)

**🏆 Best Practices:**
- Keep stores focused and cohesive
- Use selective subscriptions for performance
- Separate async logic into dedicated actions
- Test store logic independently
- Use TypeScript for better DX

**You're now ready to build scalable React apps with Zustand!** 🚀🐻