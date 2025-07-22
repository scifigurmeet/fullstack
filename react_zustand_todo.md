# React + Zustand Todo List - Two File Version

A simple todo list application built with React and Zustand for state management using only two files.

## File 1: store.js

```javascript
import { create } from 'zustand'

const useTodoStore = create((set, get) => ({
  todos: [],
  filter: 'all', // 'all' | 'active' | 'completed'
  
  addTodo: (text) => set((state) => ({
    todos: [...state.todos, {
      id: Date.now(),
      text: text.trim(),
      completed: false,
      createdAt: new Date().toISOString()
    }]
  })),
  
  toggleTodo: (id) => set((state) => ({
    todos: state.todos.map(todo =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    )
  })),
  
  deleteTodo: (id) => set((state) => ({
    todos: state.todos.filter(todo => todo.id !== id)
  })),
  
  setFilter: (filter) => set({ filter }),
  
  clearCompleted: () => set((state) => ({
    todos: state.todos.filter(todo => !todo.completed)
  })),
  
  // Computed values
  getFilteredTodos: () => {
    const { todos, filter } = get()
    switch (filter) {
      case 'active':
        return todos.filter(todo => !todo.completed)
      case 'completed':
        return todos.filter(todo => todo.completed)
      default:
        return todos
    }
  },
  
  getStats: () => {
    const { todos } = get()
    return {
      total: todos.length,
      active: todos.filter(todo => !todo.completed).length,
      completed: todos.filter(todo => todo.completed).length
    }
  }
}))

export default useTodoStore
```

## File 2: App.js

```jsx
import React, { useState } from 'react'
import useTodoStore from './store'

// TodoItem Component
const TodoItem = ({ todo, onToggle, onDelete }) => {
  return (
    <div className="flex items-center gap-3 p-3 border border-gray-200 rounded-md hover:bg-gray-50">
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={() => onToggle(todo.id)}
        className="w-4 h-4 text-blue-600 rounded focus:ring-2 focus:ring-blue-500"
      />
      
      <span 
        className={`flex-1 ${
          todo.completed 
            ? 'line-through text-gray-500' 
            : 'text-gray-900'
        }`}
      >
        {todo.text}
      </span>
      
      <button
        onClick={() => onDelete(todo.id)}
        className="px-2 py-1 text-red-600 hover:bg-red-100 rounded transition-colors"
        aria-label="Delete todo"
      >
        ×
      </button>
    </div>
  )
}

// Main App Component
function App() {
  const [inputValue, setInputValue] = useState('')
  
  const {
    filter,
    addTodo,
    toggleTodo,
    deleteTodo,
    setFilter,
    clearCompleted,
    getFilteredTodos,
    getStats
  } = useTodoStore()
  
  const filteredTodos = getFilteredTodos()
  const stats = getStats()
  
  const handleSubmit = (e) => {
    e.preventDefault()
    if (inputValue.trim()) {
      addTodo(inputValue)
      setInputValue('')
    }
  }
  
  const filterButtons = [
    { key: 'all', label: 'All', count: stats.total },
    { key: 'active', label: 'Active', count: stats.active },
    { key: 'completed', label: 'Completed', count: stats.completed }
  ]
  
  return (
    <div className="min-h-screen bg-gray-100 py-8">
      <div className="max-w-md mx-auto mt-8 p-6 bg-white rounded-lg shadow-lg">
        <h1 className="text-2xl font-bold text-center mb-6 text-gray-800">
          Todo List
        </h1>
        
        {/* Add Todo Form */}
        <form onSubmit={handleSubmit} className="mb-6">
          <div className="flex gap-2">
            <input
              type="text"
              value={inputValue}
              onChange={(e) => setInputValue(e.target.value)}
              placeholder="Add a new todo..."
              className="flex-1 px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
            />
            <button
              type="submit"
              className="px-4 py-2 bg-blue-500 text-white rounded-md hover:bg-blue-600 focus:outline-none focus:ring-2 focus:ring-blue-500"
            >
              Add
            </button>
          </div>
        </form>
        
        {/* Filter Buttons */}
        <div className="flex gap-2 mb-4">
          {filterButtons.map(({ key, label, count }) => (
            <button
              key={key}
              onClick={() => setFilter(key)}
              className={`px-3 py-1 text-sm rounded-full transition-colors ${
                filter === key
                  ? 'bg-blue-500 text-white'
                  : 'bg-gray-200 text-gray-700 hover:bg-gray-300'
              }`}
            >
              {label} ({count})
            </button>
          ))}
        </div>
        
        {/* Todo List */}
        <div className="space-y-2 mb-4">
          {filteredTodos.length === 0 ? (
            <p className="text-gray-500 text-center py-4">
              {filter === 'active' ? 'No active todos' : 
               filter === 'completed' ? 'No completed todos' : 'No todos yet'}
            </p>
          ) : (
            filteredTodos.map(todo => (
              <TodoItem
                key={todo.id}
                todo={todo}
                onToggle={toggleTodo}
                onDelete={deleteTodo}
              />
            ))
          )}
        </div>
        
        {/* Clear Completed Button */}
        {stats.completed > 0 && (
          <button
            onClick={clearCompleted}
            className="w-full py-2 text-sm text-red-600 border border-red-300 rounded-md hover:bg-red-50 focus:outline-none focus:ring-2 focus:ring-red-500"
          >
            Clear Completed ({stats.completed})
          </button>
        )}
        
        {/* Stats */}
        <div className="mt-4 text-center text-sm text-gray-500">
          {stats.active} active • {stats.completed} completed • {stats.total} total
        </div>
      </div>
    </div>
  )
}

export default App
```