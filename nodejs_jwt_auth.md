# Node.js JWT Authentication & Authorization 🔐
## Faculty Development Program - Part 3 of 3

---

## Table of Contents 📚
1. [What is JWT?](#what-is-jwt)
2. [Setup & Installation](#setup--installation)
3. [User Registration & Login](#user-registration--login)
4. [JWT Middleware](#jwt-middleware)
5. [Role-Based Authorization](#role-based-authorization)
6. [Complete Working Example](#complete-working-example)
7. [Testing the API](#testing-the-api)
8. [Exercises](#exercises)

---

## What is JWT? 🎫

**JWT (JSON Web Token)** = A secure way to transmit user information between client and server.

### Authentication vs Authorization
- **Authentication**: "Who are you?" (Login)
- **Authorization**: "What can you do?" (Permissions)

### JWT Structure
```
Header.Payload.Signature
```

Example JWT contains:
```javascript
{
  "userId": 123,
  "username": "john",
  "role": "admin",
  "exp": 1234567890  // Expiration time
}
```

---

## Setup & Installation 🚀

```bash
npm init -y
npm install express jsonwebtoken bcryptjs
```

### Basic Server Setup

```javascript
// server.js
const express = require('express');
const jwt = require('jsonwebtoken');
const bcryptjs = require('bcryptjs');

const app = express();
app.use(express.json());

const JWT_SECRET = 'your-super-secret-key';
const users = []; // In-memory storage (use database in production)

const PORT = 3000;
app.listen(PORT, () => {
    console.log(`Server running on http://localhost:${PORT}`);
});
```

---

## User Registration & Login 👤

### Registration Endpoint

```javascript
app.post('/register', async (req, res) => {
    try {
        const { username, email, password } = req.body;
        
        // Validation
        if (!username || !email || !password) {
            return res.status(400).json({ error: 'All fields required' });
        }
        
        // Check if user exists
        if (users.find(u => u.email === email)) {
            return res.status(400).json({ error: 'User already exists' });
        }
        
        // Hash password
        const hashedPassword = await bcryptjs.hash(password, 10);
        
        // Create user
        const newUser = {
            id: users.length + 1,
            username,
            email,
            password: hashedPassword,
            role: 'user'
        };
        users.push(newUser);
        
        // Create JWT
        const token = jwt.sign(
            { userId: newUser.id, username, email, role: 'user' },
            JWT_SECRET,
            { expiresIn: '24h' }
        );
        
        res.status(201).json({ 
            message: 'User registered successfully', 
            token 
        });
        
    } catch (error) {
        res.status(500).json({ error: 'Registration failed' });
    }
});
```

### Login Endpoint

```javascript
app.post('/login', async (req, res) => {
    try {
        const { email, password } = req.body;
        
        // Find user
        const user = users.find(u => u.email === email);
        if (!user) {
            return res.status(401).json({ error: 'Invalid credentials' });
        }
        
        // Check password
        const isValidPassword = await bcryptjs.compare(password, user.password);
        if (!isValidPassword) {
            return res.status(401).json({ error: 'Invalid credentials' });
        }
        
        // Create JWT
        const token = jwt.sign(
            { userId: user.id, username: user.username, email: user.email, role: user.role },
            JWT_SECRET,
            { expiresIn: '24h' }
        );
        
        res.json({ 
            message: 'Login successful', 
            token,
            user: { id: user.id, username: user.username, role: user.role }
        });
        
    } catch (error) {
        res.status(500).json({ error: 'Login failed' });
    }
});
```

---

## JWT Middleware 🛡️

### Authentication Middleware

```javascript
// Verify JWT token
function authenticateToken(req, res, next) {
    const authHeader = req.headers['authorization'];
    const token = authHeader && authHeader.split(' ')[1]; // Bearer TOKEN
    
    if (!token) {
        return res.status(401).json({ error: 'Access token required' });
    }
    
    jwt.verify(token, JWT_SECRET, (err, user) => {
        if (err) {
            return res.status(403).json({ error: 'Invalid or expired token' });
        }
        req.user = user;
        next();
    });
}

// Protected route example
app.get('/profile', authenticateToken, (req, res) => {
    res.json({
        message: 'Profile accessed successfully',
        user: {
            id: req.user.userId,
            username: req.user.username,
            email: req.user.email,
            role: req.user.role
        }
    });
});
```

---

## Role-Based Authorization 👑

### Authorization Middleware

```javascript
// Check user role
function requireRole(role) {
    return (req, res, next) => {
        if (!req.user) {
            return res.status(401).json({ error: 'Authentication required' });
        }
        
        if (req.user.role !== role) {
            return res.status(403).json({ error: `${role} access required` });
        }
        
        next();
    };
}

// Admin-only route
app.get('/admin/users', authenticateToken, requireRole('admin'), (req, res) => {
    res.json({
        message: 'Admin access granted',
        totalUsers: users.length,
        users: users.map(u => ({
            id: u.id,
            username: u.username,
            email: u.email,
            role: u.role
        }))
    });
});

// Create admin user for testing
app.post('/create-admin', async (req, res) => {
    const hashedPassword = await bcryptjs.hash('admin123', 10);
    
    const adminUser = {
        id: users.length + 1,
        username: 'admin',
        email: 'admin@example.com',
        password: hashedPassword,
        role: 'admin'
    };
    
    users.push(adminUser);
    
    res.json({
        message: 'Admin created',
        login: { email: 'admin@example.com', password: 'admin123' }
    });
});
```

---

## Complete Working Example 🎯

```javascript
// complete-jwt-server.js
const express = require('express');
const jwt = require('jsonwebtoken');
const bcryptjs = require('bcryptjs');

const app = express();
app.use(express.json());

const JWT_SECRET = 'your-super-secret-key';
const users = [];

// Middleware
function authenticateToken(req, res, next) {
    const authHeader = req.headers['authorization'];
    const token = authHeader && authHeader.split(' ')[1];
    
    if (!token) {
        return res.status(401).json({ error: 'Access token required' });
    }
    
    jwt.verify(token, JWT_SECRET, (err, user) => {
        if (err) return res.status(403).json({ error: 'Invalid token' });
        req.user = user;
        next();
    });
}

function requireRole(role) {
    return (req, res, next) => {
        if (req.user.role !== role) {
            return res.status(403).json({ error: `${role} access required` });
        }
        next();
    };
}

// Routes
app.post('/register', async (req, res) => {
    try {
        const { username, email, password } = req.body;
        
        if (!username || !email || !password) {
            return res.status(400).json({ error: 'All fields required' });
        }
        
        if (users.find(u => u.email === email)) {
            return res.status(400).json({ error: 'User exists' });
        }
        
        const hashedPassword = await bcryptjs.hash(password, 10);
        const newUser = {
            id: users.length + 1,
            username,
            email,
            password: hashedPassword,
            role: 'user'
        };
        users.push(newUser);
        
        const token = jwt.sign(
            { userId: newUser.id, username, email, role: 'user' },
            JWT_SECRET,
            { expiresIn: '24h' }
        );
        
        res.status(201).json({ message: 'Registered successfully', token });
    } catch (error) {
        res.status(500).json({ error: 'Registration failed' });
    }
});

app.post('/login', async (req, res) => {
    try {
        const { email, password } = req.body;
        
        const user = users.find(u => u.email === email);
        if (!user || !(await bcryptjs.compare(password, user.password))) {
            return res.status(401).json({ error: 'Invalid credentials' });
        }
        
        const token = jwt.sign(
            { userId: user.id, username: user.username, email, role: user.role },
            JWT_SECRET,
            { expiresIn: '24h' }
        );
        
        res.json({ message: 'Login successful', token });
    } catch (error) {
        res.status(500).json({ error: 'Login failed' });
    }
});

app.get('/profile', authenticateToken, (req, res) => {
    res.json({ message: 'Profile data', user: req.user });
});

app.get('/admin', authenticateToken, requireRole('admin'), (req, res) => {
    res.json({ 
        message: 'Admin panel', 
        totalUsers: users.length,
        users: users.map(u => ({ id: u.id, username: u.username, role: u.role }))
    });
});

// Helper route to create admin
app.post('/setup-admin', async (req, res) => {
    const adminExists = users.find(u => u.role === 'admin');
    if (adminExists) {
        return res.json({ message: 'Admin already exists' });
    }
    
    const hashedPassword = await bcryptjs.hash('admin123', 10);
    users.push({
        id: users.length + 1,
        username: 'admin',
        email: 'admin@test.com',
        password: hashedPassword,
        role: 'admin'
    });
    
    res.json({ 
        message: 'Admin created', 
        credentials: { email: 'admin@test.com', password: 'admin123' }
    });
});

// Info route
app.get('/', (req, res) => {
    res.json({
        message: 'JWT Auth API',
        endpoints: {
            'POST /register': 'Register user',
            'POST /login': 'Login user',
            'GET /profile': 'Get profile (auth required)',
            'GET /admin': 'Admin panel (admin role)',
            'POST /setup-admin': 'Create admin user'
        }
    });
});

const PORT = 3000;
app.listen(PORT, () => {
    console.log(`🚀 Server running on http://localhost:${PORT}`);
    console.log('📋 Visit http://localhost:3000 for API info');
});
```

---

## Testing the API 🧪

### 1. Start the Server
```bash
node complete-jwt-server.js
```

### 2. Create Admin User
```bash
curl -X POST http://localhost:3000/setup-admin
```

### 3. Register a Regular User
```bash
curl -X POST http://localhost:3000/register \
  -H "Content-Type: application/json" \
  -d '{"username":"john","email":"john@test.com","password":"password123"}'
```

### 4. Login and Get Token
```bash
curl -X POST http://localhost:3000/login \
  -H "Content-Type: application/json" \
  -d '{"email":"john@test.com","password":"password123"}'
```

### 5. Access Protected Route
```bash
# Replace YOUR_TOKEN with the actual token from login
curl -X GET http://localhost:3000/profile \
  -H "Authorization: Bearer YOUR_TOKEN"
```

### 6. Test Admin Access
```bash
# Login as admin first
curl -X POST http://localhost:3000/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@test.com","password":"admin123"}'

# Then access admin route with admin token
curl -X GET http://localhost:3000/admin \
  -H "Authorization: Bearer ADMIN_TOKEN"
```

---

## Exercises 📝

### Exercise 1: Add Logout
```javascript
// Your task: Implement logout functionality
// POST /logout - Invalidate current token
// Hint: Maintain a blacklist of invalid tokens
```

### Exercise 2: Password Change
```javascript
// Your task: Add password change endpoint
// POST /change-password - Change user password
// Requirements: Old password + new password
// Must be authenticated
```

### Exercise 3: User Management
```javascript
// Your task: Add user management for admins
// DELETE /admin/users/:id - Delete user (admin only)
// PUT /admin/users/:id/role - Change user role (admin only)
```

---

## Summary 📋

### What You Learned:
1. **JWT Basics**: Structure and purpose
2. **Authentication**: Register/Login with password hashing
3. **Authorization**: Role-based access control
4. **Security**: Token verification and protection
5. **Middleware**: Protecting routes automatically

### Key Security Points:
- ✅ Always hash passwords
- ✅ Use strong JWT secrets
- ✅ Set token expiration
- ✅ Validate all inputs
- ✅ Never expose passwords in responses

### Production Checklist:
- Use environment variables for secrets
- Add input validation
- Implement rate limiting
- Use HTTPS
- Add refresh tokens
- Use a real database

---

**🎉 Congratulations!**

You've completed the **Node.js Mastery Series**:
1. ✅ Async Patterns
2. ✅ Error Handling  
3. ✅ JWT Authentication

*You're now ready to build secure Node.js applications!* 🚀