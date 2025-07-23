# React Error Boundaries Study Guide
*Beginner-Friendly Training Material*

## Table of Contents
1. [What are Error Boundaries?](#what-are)
2. [Why Use Error Boundaries?](#why-use)
3. [Creating Error Boundaries](#creating)
4. [Hands-On Examples](#examples)
5. [Error Boundaries + Lazy Loading](#with-lazy)
6. [Best Practices](#best-practices)

## What are Error Boundaries? {#what-are}

Error Boundaries are React components that **catch JavaScript errors** in their child components and show a fallback UI instead of crashing the entire app.

**Think of it like:**
- A safety net that catches falling components
- A try-catch block for React components
- Insurance for your app - when something breaks, show a nice error message

### What Errors Do They Catch?
✅ **Catches:**
- Errors in component rendering
- Errors in lifecycle methods
- Errors in constructors

❌ **Does NOT catch:**
- Event handler errors
- Async errors (setTimeout, promises)
- Server-side rendering errors
- Errors in the error boundary itself

## Why Use Error Boundaries? {#why-use}

**Without Error Boundary:**
```
Component crashes → Entire app shows blank white screen → Bad user experience
```

**With Error Boundary:**
```
Component crashes → Error boundary shows "Something went wrong" → Rest of app works fine
```

## Creating Error Boundaries {#creating}

### Basic Error Boundary Class

**File: `src/ErrorBoundary.js`**
```javascript
import React from 'react';

class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  // This method catches the error
  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  // This method logs the error details
  componentDidCatch(error, errorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      // Show fallback UI when there's an error
      return (
        <div style={{
          padding: '20px',
          border: '2px solid #ff6b6b',
          borderRadius: '8px',
          backgroundColor: '#ffe0e0',
          textAlign: 'center'
        }}>
          <h2>🚨 Oops! Something went wrong</h2>
          <p>Don't worry, the rest of the app is still working.</p>
          <button 
            onClick={() => this.setState({ hasError: false, error: null })}
            style={{
              padding: '10px 20px',
              backgroundColor: '#ff6b6b',
              color: 'white',
              border: 'none',
              borderRadius: '5px',
              cursor: 'pointer'
            }}
          >
            Try Again
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}

export default ErrorBoundary;
```

## Hands-On Examples {#examples}

### Example 1: Basic Error Boundary

**Step 1:** Create a component that can crash

**File: `src/BuggyComponent.js`**
```javascript
import React, { useState } from 'react';

const BuggyComponent = () => {
  const [count, setCount] = useState(0);

  if (count === 3) {
    // This will cause an error!
    throw new Error('I crashed because count reached 3!');
  }

  return (
    <div style={{ 
      padding: '20px', 
      border: '2px solid #4CAF50', 
      borderRadius: '8px',
      margin: '10px 0'
    }}>
      <h3>🐛 Buggy Counter</h3>
      <p>Count: {count}</p>
      <p>⚠️ I will crash when count reaches 3!</p>
      <button 
        onClick={() => setCount(count + 1)}
        style={{
          padding: '8px 16px',
          backgroundColor: '#4CAF50',
          color: 'white',
          border: 'none',
          borderRadius: '4px',
          cursor: 'pointer'
        }}
      >
        Increment (+1)
      </button>
    </div>
  );
};

export default BuggyComponent;
```

**Step 2:** Create app with Error Boundary

**File: `src/ErrorBoundaryApp.js`**
```javascript
import React from 'react';
import ErrorBoundary from './ErrorBoundary';
import BuggyComponent from './BuggyComponent';

function ErrorBoundaryApp() {
  return (
    <div style={{ padding: '20px' }}>
      <h1>Error Boundary Demo</h1>
      
      <div style={{ marginBottom: '30px' }}>
        <h2>Component WITHOUT Error Boundary</h2>
        <p>⚠️ This will crash the entire app!</p>
        <BuggyComponent />
      </div>

      <div>
        <h2>Component WITH Error Boundary</h2>
        <p>✅ This will show a nice error message!</p>
        <ErrorBoundary>
          <BuggyComponent />
        </ErrorBoundary>
      </div>

      <div style={{
        marginTop: '30px',
        padding: '15px',
        backgroundColor: '#e7f3ff',
        borderRadius: '5px'
      }}>
        <h3>🎯 Test Instructions:</h3>
        <ol>
          <li>Click increment on FIRST counter until it reaches 3</li>
          <li>See how entire app crashes (white screen)</li>
          <li>Refresh page</li>
          <li>Click increment on SECOND counter until it reaches 3</li>
          <li>See how only that component shows error, rest works!</li>
        </ol>
      </div>
    </div>
  );
}

export default ErrorBoundaryApp;
```

**Step 3:** Test it

Update `src/index.js`:
```javascript
import React from 'react';
import ReactDOM from 'react-dom/client';
import ErrorBoundaryApp from './ErrorBoundaryApp';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<ErrorBoundaryApp />);
```

### Example 2: Multiple Error Boundaries

**Step 1:** Create different buggy components

**File: `src/NetworkComponent.js`**
```javascript
import React, { useState } from 'react';

const NetworkComponent = () => {
  const [shouldFail, setShouldFail] = useState(false);

  if (shouldFail) {
    throw new Error('Network request failed!');
  }

  return (
    <div style={{ 
      padding: '15px', 
      border: '2px solid #2196F3',
      borderRadius: '5px'
    }}>
      <h4>🌐 Network Component</h4>
      <p>Simulates network operations</p>
      <button 
        onClick={() => setShouldFail(true)}
        style={{
          padding: '5px 10px',
          backgroundColor: '#ff4444',
          color: 'white',
          border: 'none',
          borderRadius: '3px'
        }}
      >
        Simulate Network Error
      </button>
    </div>
  );
};

export default NetworkComponent;
```

**File: `src/DataComponent.js`**
```javascript
import React, { useState } from 'react';

const DataComponent = () => {
  const [shouldFail, setShouldFail] = useState(false);

  if (shouldFail) {
    throw new Error('Data processing failed!');
  }

  return (
    <div style={{ 
      padding: '15px', 
      border: '2px solid #FF9800',
      borderRadius: '5px'
    }}>
      <h4>📊 Data Component</h4>
      <p>Handles data processing</p>
      <button 
        onClick={() => setShouldFail(true)}
        style={{
          padding: '5px 10px',
          backgroundColor: '#ff4444',
          color: 'white',
          border: 'none',
          borderRadius: '3px'
        }}
      >
        Simulate Data Error
      </button>
    </div>
  );
};

export default DataComponent;
```

**Step 2:** Create app with multiple boundaries

**File: `src/MultiErrorApp.js`**
```javascript
import React from 'react';
import ErrorBoundary from './ErrorBoundary';
import NetworkComponent from './NetworkComponent';
import DataComponent from './DataComponent';

function MultiErrorApp() {
  return (
    <div style={{ padding: '20px' }}>
      <h1>Multiple Error Boundaries</h1>
      
      <div style={{ 
        display: 'grid', 
        gridTemplateColumns: '1fr 1fr', 
        gap: '20px',
        marginTop: '20px'
      }}>
        <ErrorBoundary>
          <NetworkComponent />
        </ErrorBoundary>
        
        <ErrorBoundary>
          <DataComponent />
        </ErrorBoundary>
      </div>
      
      <div style={{
        marginTop: '20px',
        padding: '15px',
        backgroundColor: '#f0f8ff',
        borderRadius: '5px'
      }}>
        <h3>🎯 Try this:</h3>
        <p>Click either error button - only that component will show error, the other keeps working!</p>
      </div>
    </div>
  );
}

export default MultiErrorApp;
```

## Error Boundaries + Lazy Loading {#with-lazy}

### Enhanced Error Boundary for Lazy Components

**File: `src/LazyErrorBoundary.js`**
```javascript
import React from 'react';

class LazyErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null, errorInfo: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    this.setState({ errorInfo });
    console.error('Lazy loading error:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return (
        <div style={{
          padding: '20px',
          border: '2px solid #ff6b6b',
          borderRadius: '8px',
          backgroundColor: '#ffe0e0',
          textAlign: 'center'
        }}>
          <h3>📦 Failed to Load Component</h3>
          <p>The component couldn't be loaded. This might be due to:</p>
          <ul style={{ textAlign: 'left', display: 'inline-block' }}>
            <li>Network issues</li>
            <li>Component code errors</li>
            <li>Missing files</li>
          </ul>
          
          <div style={{ marginTop: '15px' }}>
            <button 
              onClick={() => window.location.reload()}
              style={{
                padding: '10px 20px',
                backgroundColor: '#4CAF50',
                color: 'white',
                border: 'none',
                borderRadius: '5px',
                cursor: 'pointer',
                marginRight: '10px'
              }}
            >
              Reload Page
            </button>
            
            <button 
              onClick={() => this.setState({ hasError: false, error: null })}
              style={{
                padding: '10px 20px',
                backgroundColor: '#2196F3',
                color: 'white',
                border: 'none',
                borderRadius: '5px',
                cursor: 'pointer'
              }}
            >
              Try Again
            </button>
          </div>
        </div>
      );
    }

    return this.props.children;
  }
}

export default LazyErrorBoundary;
```

### Complete Lazy + Error Boundary Example

**Step 1:** Create a lazy component that might fail

**File: `src/LazyBuggyComponent.js`**
```javascript
import React, { useState } from 'react';

const LazyBuggyComponent = () => {
  const [shouldCrash, setShouldCrash] = useState(false);

  if (shouldCrash) {
    throw new Error('Lazy component crashed!');
  }

  return (
    <div style={{ 
      padding: '20px', 
      border: '2px solid #9C27B0',
      borderRadius: '8px',
      backgroundColor: '#f3e5f5'
    }}>
      <h3>🚀 Lazy Loaded Component</h3>
      <p>I was loaded on-demand and I'm protected by error boundaries!</p>
      <button 
        onClick={() => setShouldCrash(true)}
        style={{
          padding: '8px 16px',
          backgroundColor: '#ff4444',
          color: 'white',
          border: 'none',
          borderRadius: '4px',
          cursor: 'pointer'
        }}
      >
        Crash This Component
      </button>
    </div>
  );
};

export default LazyBuggyComponent;
```

**Step 2:** Create combined app

**File: `src/LazyWithErrorApp.js`**
```javascript
import React, { Suspense, useState } from 'react';
import LazyErrorBoundary from './LazyErrorBoundary';

// 🔥 Lazy load component
const LazyBuggyComponent = React.lazy(() => import('./LazyBuggyComponent'));

const LoadingSpinner = () => (
  <div style={{ 
    padding: '20px', 
    textAlign: 'center',
    border: '2px dashed #ccc',
    borderRadius: '8px'
  }}>
    <div style={{
      width: '30px',
      height: '30px',
      border: '3px solid #f3f3f3',
      borderTop: '3px solid #3498db',
      borderRadius: '50%',
      animation: 'spin 1s linear infinite',
      margin: '0 auto 10px'
    }}></div>
    Loading component...
    
    <style jsx>{`
      @keyframes spin {
        0% { transform: rotate(0deg); }
        100% { transform: rotate(360deg); }
      }
    `}</style>
  </div>
);

function LazyWithErrorApp() {
  const [showLazy, setShowLazy] = useState(false);

  return (
    <div style={{ padding: '20px' }}>
      <h1>Lazy Loading + Error Boundaries</h1>
      
      <button 
        onClick={() => setShowLazy(!showLazy)}
        style={{
          padding: '12px 24px',
          backgroundColor: showLazy ? '#ff4444' : '#4CAF50',
          color: 'white',
          border: 'none',
          borderRadius: '6px',
          cursor: 'pointer',
          fontSize: '16px'
        }}
      >
        {showLazy ? 'Hide' : 'Load'} Lazy Component
      </button>

      {showLazy && (
        <div style={{ marginTop: '20px' }}>
          <LazyErrorBoundary>
            <Suspense fallback={<LoadingSpinner />}>
              <LazyBuggyComponent />
            </Suspense>
          </LazyErrorBoundary>
        </div>
      )}
      
      <div style={{
        marginTop: '30px',
        padding: '15px',
        backgroundColor: '#e8f5e8',
        borderRadius: '5px'
      }}>
        <h3>🎯 What This Demonstrates:</h3>
        <ul>
          <li>✅ Lazy loading with loading state</li>
          <li>✅ Error boundary catches component errors</li>
          <li>✅ Graceful error handling</li>
          <li>✅ Recovery options for users</li>
        </ul>
      </div>
    </div>
  );
}

export default LazyWithErrorApp;
```

## Best Practices {#best-practices}

### 1. Where to Place Error Boundaries

```javascript
// ✅ Good: Around route components
<Router>
  <ErrorBoundary>
    <Routes>
      <Route path="/" element={<Home />} />
    </Routes>
  </ErrorBoundary>
</Router>

// ✅ Good: Around feature sections
<div>
  <Header />
  <ErrorBoundary>
    <MainContent />
  </ErrorBoundary>
  <Footer />
</div>

// ✅ Good: Around lazy components
<ErrorBoundary>
  <Suspense fallback={<Loading />}>
    <LazyComponent />
  </Suspense>
</ErrorBoundary>
```

### 2. Simple Error Boundary Hook (React 16.8+)

**File: `src/SimpleErrorBoundary.js`**
```javascript
import React from 'react';

const SimpleErrorBoundary = ({ children, fallback }) => {
  return (
    <ErrorBoundaryClass fallback={fallback}>
      {children}
    </ErrorBoundaryClass>
  );
};

class ErrorBoundaryClass extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError() {
    return { hasError: true };
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback || <div>Something went wrong.</div>;
    }
    return this.props.children;
  }
}

export default SimpleErrorBoundary;
```

**Usage:**
```javascript
<SimpleErrorBoundary fallback={<div>Oops! Error occurred.</div>}>
  <MyComponent />
</SimpleErrorBoundary>
```

### 3. Quick Testing

**Create `src/TestErrorApp.js`:**
```javascript
import React from 'react';
import ErrorBoundary from './ErrorBoundary';

const CrashButton = () => {
  const crash = () => {
    throw new Error('Test error!');
  };

  return (
    <button onClick={crash} style={{ padding: '10px', backgroundColor: 'red', color: 'white' }}>
      Click to Crash
    </button>
  );
};

function TestErrorApp() {
  return (
    <div style={{ padding: '20px' }}>
      <h1>Quick Error Test</h1>
      
      <ErrorBoundary>
        <CrashButton />
      </ErrorBoundary>
    </div>
  );
}

export default TestErrorApp;
```

### Key Points to Remember

✅ **Do:**
- Use class components for error boundaries
- Place boundaries strategically in your component tree
- Provide helpful error messages to users
- Include recovery options (retry buttons)
- Log errors for debugging

❌ **Don't:**
- Wrap every single component
- Catch errors in event handlers with error boundaries
- Forget to handle async errors separately
- Show technical error messages to users

### Common Error Boundary Locations
1. **Route level** - Catch page-level errors
2. **Feature level** - Isolate feature failures
3. **Lazy component level** - Handle loading failures
4. **Third-party component level** - Isolate external library issues