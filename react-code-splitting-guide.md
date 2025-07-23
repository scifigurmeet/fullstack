# React Code Splitting & Lazy Loading Study Guide
*Beginner-Friendly Training Material*

## Table of Contents
1. [Project Setup](#setup)
2. [What is Code Splitting?](#what-is)
3. [React.lazy Basics](#react-lazy)
4. [Suspense Component](#suspense)
5. [Hands-On Examples](#examples)
6. [Quick Tips](#tips)

## Project Setup {#setup}

### Create New React App
```bash
npx create-react-app lazy-demo
cd lazy-demo
npm start
```

Your folder structure:
```
lazy-demo/
├── src/
│   ├── App.js
│   ├── index.js
│   └── ...
```

## What is Code Splitting? {#what-is}

**Problem:** Your React app loads everything at once = slow initial load

**Solution:** Load components only when needed = faster initial load

**Simple Example:**
- Without splitting: Load Home + About + Dashboard pages immediately
- With splitting: Load Home immediately, load About/Dashboard when user visits them

## React.lazy Basics {#react-lazy}

### Basic Syntax
```javascript
const MyComponent = React.lazy(() => import('./MyComponent'));
```

**What this does:**
- Creates a "lazy" component
- Only loads when component is actually used
- Must be wrapped in `<Suspense>`

## Suspense Component {#suspense}

Shows loading message while lazy component loads:

```javascript
<Suspense fallback={<div>Loading...</div>}>
  <MyComponent />
</Suspense>
```

## Hands-On Examples {#examples}

### Example 1: Basic Lazy Component

**Step 1:** Create `src/HeavyComponent.js`
```javascript
import React from 'react';

const HeavyComponent = () => {
  return (
    <div style={{ padding: '20px', backgroundColor: '#f0f8ff', border: '2px solid green' }}>
      <h2>🎉 I'm Heavy!</h2>
      <p>I was loaded lazily!</p>
    </div>
  );
};

export default HeavyComponent;
```

**Step 2:** Update `src/App.js`
```javascript
import React, { Suspense, useState } from 'react';

// 🔥 Lazy load the component
const HeavyComponent = React.lazy(() => import('./HeavyComponent'));

function App() {
  const [show, setShow] = useState(false);

  return (
    <div style={{ padding: '20px' }}>
      <h1>Lazy Loading Demo</h1>
      
      <button onClick={() => setShow(!show)}>
        {show ? 'Hide' : 'Show'} Heavy Component
      </button>

      {show && (
        <Suspense fallback={<div>Loading Heavy Component...</div>}>
          <HeavyComponent />
        </Suspense>
      )}
    </div>
  );
}

export default App;
```

**Test it:**
1. Open browser DevTools (F12)
2. Go to Network tab
3. Click the button
4. Watch for new JS file loading!

### Example 2: Route-Based Splitting

**Step 1:** Install React Router
```bash
npm install react-router-dom
```

**Step 2:** Create page files

**File: `src/Home.js`**
```javascript
import React from 'react';

const Home = () => (
  <div style={{ padding: '20px' }}>
    <h1>🏠 Home Page</h1>
    <p>This loads immediately</p>
  </div>
);

export default Home;
```

**File: `src/About.js`**
```javascript
import React from 'react';

const About = () => (
  <div style={{ padding: '20px', backgroundColor: '#fff3cd' }}>
    <h1>ℹ️ About Page</h1>
    <p>This loaded lazily when you clicked About!</p>
  </div>
);

export default About;
```

**Step 3:** Create `src/RouterApp.js`
```javascript
import React, { Suspense } from 'react';
import { BrowserRouter as Router, Routes, Route, Link } from 'react-router-dom';

// 🔥 Lazy load pages
const Home = React.lazy(() => import('./Home'));
const About = React.lazy(() => import('./About'));

const PageLoader = () => (
  <div style={{ padding: '20px', textAlign: 'center' }}>
    <div>Loading page...</div>
  </div>
);

function RouterApp() {
  return (
    <Router>
      <nav style={{ padding: '20px', backgroundColor: '#f8f9fa' }}>
        <Link to="/" style={{ marginRight: '20px' }}>Home</Link>
        <Link to="/about">About</Link>
      </nav>
      
      <Suspense fallback={<PageLoader />}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/about" element={<About />} />
        </Routes>
      </Suspense>
    </Router>
  );
}

export default RouterApp;
```

**Step 4:** Update `src/index.js`
```javascript
import React from 'react';
import ReactDOM from 'react-dom/client';
import RouterApp from './RouterApp'; // Change this line

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<RouterApp />);
```

**Test it:**
1. Open DevTools → Network tab
2. Navigate between Home and About
3. See separate JS chunks loading!

### Example 3: Conditional Loading

**Step 1:** Create `src/ChatWidget.js`
```javascript
import React from 'react';

const ChatWidget = () => (
  <div style={{
    position: 'fixed',
    bottom: '20px',
    right: '20px',
    width: '300px',
    height: '200px',
    backgroundColor: 'white',
    border: '1px solid #ccc',
    borderRadius: '10px',
    padding: '20px'
  }}>
    <h3>💬 Chat</h3>
    <p>Chat widget loaded lazily!</p>
  </div>
);

export default ChatWidget;
```

**Step 2:** Create `src/ConditionalApp.js`
```javascript
import React, { Suspense, useState } from 'react';

// 🔥 Lazy load chat widget
const ChatWidget = React.lazy(() => import('./ChatWidget'));

function ConditionalApp() {
  const [showChat, setShowChat] = useState(false);

  return (
    <div style={{ padding: '20px' }}>
      <h1>Conditional Loading Demo</h1>
      
      <button onClick={() => setShowChat(!showChat)}>
        {showChat ? 'Close' : 'Open'} Chat
      </button>

      {showChat && (
        <Suspense fallback={<div>Loading chat...</div>}>
          <ChatWidget />
        </Suspense>
      )}
    </div>
  );
}

export default ConditionalApp;
```

**Test it:** Change import in `index.js` to use `ConditionalApp`

## Quick Tips {#tips}

### ✅ Do's
```javascript
// ✅ Always use default export
export default MyComponent;

// ✅ Always wrap with Suspense
<Suspense fallback={<div>Loading...</div>}>
  <LazyComponent />
</Suspense>

// ✅ Use descriptive loading messages
fallback={<div>Loading dashboard...</div>}
```

### ❌ Don'ts
```javascript
// ❌ Named exports won't work
export { MyComponent };

// ❌ No Suspense wrapper
<LazyComponent /> // Will crash!

// ❌ Generic loading
fallback={<div>Loading...</div>}
```

### Check Bundle Size
```bash
npm run build
```
Look at the `build/static/js/` folder - you'll see multiple chunk files!

### Common Issues
1. **"Element type is invalid"** → Check if component has default export
2. **"A React component suspended"** → Missing Suspense wrapper
3. **Component not loading** → Check file path in import()

### When to Use Lazy Loading
- ✅ Different pages/routes
- ✅ Heavy components (charts, editors)
- ✅ Admin panels or rare features
- ✅ Third-party libraries
- ❌ Small, frequently used components