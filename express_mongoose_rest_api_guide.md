# REST APIs with Express + Mongoose 🌐
## Complete Beginner's Guide for Faculty Development Program

---

## Table of Contents 📚
1. [What are REST APIs?](#what-are-rest-apis)
2. [MongoDB Setup](#mongodb-setup)
3. [Express.js Basics](#expressjs-basics)
4. [Mongoose Fundamentals](#mongoose-fundamentals)
5. [Project Setup](#project-setup)
6. [Building Complete API](#building-complete-api)
7. [Testing Your API](#testing-your-api)
8. [Best Practices](#best-practices)
9. [Troubleshooting](#troubleshooting)

---

## What are REST APIs? 🤔

### Simple Explanation
A REST API is like a **waiter in a restaurant**:
- You (client) order food (make request)
- Waiter takes order to kitchen (server processes)
- Kitchen prepares food (database operations)
- Waiter brings food back (returns response)

### HTTP Methods
| Method | Purpose | Example URL | What it does |
|--------|---------|-------------|--------------|
| **GET** | Get data | `GET /api/users` | Retrieve all users |
| **GET** | Get one | `GET /api/users/123` | Get user by ID |
| **POST** | Create | `POST /api/users` | Create new user |
| **PUT** | Update | `PUT /api/users/123` | Update user |
| **DELETE** | Delete | `DELETE /api/users/123` | Delete user |

### Status Codes
- **200** ✅ Success (GET, PUT)
- **201** ✅ Created (POST)
- **404** ❌ Not Found
- **400** ❌ Bad Request
- **500** ❌ Server Error

---

## MongoDB Setup 🍃

### What is MongoDB?
MongoDB is a **NoSQL database** that stores data as documents (like JSON objects).

**SQL vs MongoDB:**
```javascript
// SQL Table
| id | name     | email           |
|----|----------|-----------------|
| 1  | John Doe | john@email.com  |

// MongoDB Document
{
  "_id": "64f123...",
  "name": "John Doe",
  "email": "john@email.com"
}
```

### Installation Options

**Option 1: MongoDB Atlas (Cloud - Recommended) ☁️**

1. **Create Account**: Go to [MongoDB Atlas](https://www.mongodb.com/cloud/atlas)
2. **Create Free Cluster**: Choose "Free Shared" option
3. **Create Database User**: 
   - Username: `myuser`
   - Password: `mypassword123`
4. **Whitelist IP**: Click "Allow Access from Anywhere"
5. **Get Connection String**: 
   ```
   mongodb+srv://myuser:mypassword123@cluster0.xxxxx.mongodb.net/myapp?retryWrites=true&w=majority
   ```

**Option 2: Local Installation 💻**

**Windows:**
1. Download from [MongoDB Download Center](https://www.mongodb.com/try/download/community)
2. Install with default settings
3. MongoDB starts automatically

**macOS:**
```bash
brew install mongodb-community
brew services start mongodb-community
```

**Linux:**
```bash
sudo apt update
sudo apt install -y mongodb
sudo systemctl start mongodb
```

### Verify Installation
```bash
# For local MongoDB
mongosh
# Should connect to: mongodb://127.0.0.1:27017
```

---

## Express.js Basics ⚡

### What is Express?
Express is a **web framework** that makes building APIs easy.

**Without Express (Hard):**
```javascript
const http = require('http');
// 50+ lines of complex code...
```

**With Express (Easy):**
```javascript
const express = require('express');
const app = express();

app.get('/users', (req, res) => {
  res.json({ message: 'Hello Users!' });
});

app.listen(3000);
```

### Key Concepts

**1. Middleware:** Functions that run between request and response
```javascript
app.use(express.json()); // Parse JSON data
app.use((req, res, next) => {
  console.log('Request received');
  next(); // Continue to next middleware
});
```

**2. Routes:** Define API endpoints
```javascript
app.get('/users', handler);     // GET request
app.post('/users', handler);    // POST request
app.put('/users/:id', handler); // PUT with parameter
```

**3. Request & Response:**
```javascript
app.get('/users', (req, res) => {
  console.log(req.method);   // GET
  console.log(req.params);   // URL parameters
  console.log(req.body);     // Request body
  
  res.json({ data: 'response' }); // Send JSON
  res.status(201).send('Created'); // Set status
});
```

---

## Mongoose Fundamentals 🍃

### What is Mongoose?
Mongoose is a **bridge** between JavaScript and MongoDB that provides:
- 📋 **Schema definitions** (data structure)
- ✅ **Validation** (data rules)
- 🔧 **Helper methods** (easy database operations)

### Core Concepts

**1. Schema - Define Structure:**
```javascript
const userSchema = new mongoose.Schema({
  name: { 
    type: String, 
    required: true 
  },
  email: { 
    type: String, 
    required: true, 
    unique: true 
  },
  age: { 
    type: Number, 
    min: 0, 
    max: 120 
  }
});
```

**2. Model - Create Operations:**
```javascript
const User = mongoose.model('User', userSchema);
```

**3. Document - Actual Data:**
```javascript
const user = new User({
  name: 'John',
  email: 'john@email.com',
  age: 30
});
```

---

## Project Setup 🛠️

### Step 1: Create Project
```bash
mkdir my-blog-api
cd my-blog-api
npm init -y
```

### Step 2: Install Packages
```bash
npm install express mongoose dotenv cors
npm install --save-dev nodemon
```

### Step 3: Project Structure
```
my-blog-api/
├── server.js
├── models/
│   └── User.js
├── .env
└── package.json
```

### Step 4: Configure Scripts (package.json)
```json
{
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  }
}
```

### Step 5: Environment Variables (.env)
```env
PORT=3000
MONGODB_URI=mongodb+srv://myuser:mypassword123@cluster0.xxxxx.mongodb.net/blogapi
# For local: MONGODB_URI=mongodb://localhost:27017/blogapi
```

---

## Building Complete API 🚀

### Step 1: User Model (models/User.js)
```javascript
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
  name: {
    type: String,
    required: [true, 'Name is required'],
    trim: true,
    minlength: [2, 'Name too short'],
    maxlength: [50, 'Name too long']
  },
  email: {
    type: String,
    required: [true, 'Email is required'],
    unique: true,
    lowercase: true,
    match: [/^\w+@\w+\.\w+$/, 'Invalid email format']
  },
  age: {
    type: Number,
    min: [0, 'Age cannot be negative'],
    max: [120, 'Age too high']
  },
  bio: {
    type: String,
    maxlength: [500, 'Bio too long']
  },
  isActive: {
    type: Boolean,
    default: true
  }
}, {
  timestamps: true // Adds createdAt and updatedAt
});

// Custom methods
userSchema.methods.getPublicProfile = function() {
  return {
    id: this._id,
    name: this.name,
    bio: this.bio,
    createdAt: this.createdAt
  };
};

module.exports = mongoose.model('User', userSchema);
```

### Step 2: Complete Server (server.js)
```javascript
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
require('dotenv').config();

const User = require('./models/User');

const app = express();

// Middleware
app.use(cors());
app.use(express.json());

// Connect to MongoDB
mongoose.connect(process.env.MONGODB_URI)
  .then(() => console.log('🍃 Connected to MongoDB'))
  .catch(err => console.error('❌ MongoDB Error:', err));

// Routes

// Welcome route
app.get('/', (req, res) => {
  res.json({
    message: '🚀 Welcome to Blog API',
    endpoints: {
      users: '/api/users',
      singleUser: '/api/users/:id'
    }
  });
});

// GET all users with filtering
app.get('/api/users', async (req, res) => {
  try {
    // Build query
    let query = {};
    
    // Filter by active status
    if (req.query.active) {
      query.isActive = req.query.active === 'true';
    }
    
    // Search by name
    if (req.query.name) {
      query.name = { $regex: req.query.name, $options: 'i' };
    }
    
    // Pagination
    const page = parseInt(req.query.page) || 1;
    const limit = parseInt(req.query.limit) || 5;
    const skip = (page - 1) * limit;
    
    const users = await User.find(query)
      .limit(limit)
      .skip(skip)
      .sort({ createdAt: -1 });
      
    const total = await User.countDocuments(query);
    
    res.json({
      success: true,
      count: users.length,
      total,
      page,
      pages: Math.ceil(total / limit),
      data: users
    });
  } catch (error) {
    res.status(500).json({
      success: false,
      message: error.message
    });
  }
});

// GET single user
app.get('/api/users/:id', async (req, res) => {
  try {
    const user = await User.findById(req.params.id);
    
    if (!user) {
      return res.status(404).json({
        success: false,
        message: 'User not found'
      });
    }
    
    res.json({
      success: true,
      data: user
    });
  } catch (error) {
    if (error.name === 'CastError') {
      return res.status(404).json({
        success: false,
        message: 'User not found'
      });
    }
    
    res.status(500).json({
      success: false,
      message: error.message
    });
  }
});

// POST create user
app.post('/api/users', async (req, res) => {
  try {
    const user = await User.create(req.body);
    
    res.status(201).json({
      success: true,
      message: 'User created successfully',
      data: user
    });
  } catch (error) {
    if (error.name === 'ValidationError') {
      const errors = Object.values(error.errors).map(e => e.message);
      return res.status(400).json({
        success: false,
        message: 'Validation Error',
        errors
      });
    }
    
    if (error.code === 11000) {
      return res.status(400).json({
        success: false,
        message: 'Email already exists'
      });
    }
    
    res.status(500).json({
      success: false,
      message: error.message
    });
  }
});

// PUT update user
app.put('/api/users/:id', async (req, res) => {
  try {
    const user = await User.findByIdAndUpdate(
      req.params.id,
      req.body,
      { new: true, runValidators: true }
    );
    
    if (!user) {
      return res.status(404).json({
        success: false,
        message: 'User not found'
      });
    }
    
    res.json({
      success: true,
      message: 'User updated successfully',
      data: user
    });
  } catch (error) {
    if (error.name === 'ValidationError') {
      const errors = Object.values(error.errors).map(e => e.message);
      return res.status(400).json({
        success: false,
        message: 'Validation Error',
        errors
      });
    }
    
    if (error.name === 'CastError') {
      return res.status(404).json({
        success: false,
        message: 'User not found'
      });
    }
    
    res.status(500).json({
      success: false,
      message: error.message
    });
  }
});

// DELETE user
app.delete('/api/users/:id', async (req, res) => {
  try {
    const user = await User.findByIdAndDelete(req.params.id);
    
    if (!user) {
      return res.status(404).json({
        success: false,
        message: 'User not found'
      });
    }
    
    res.json({
      success: true,
      message: 'User deleted successfully'
    });
  } catch (error) {
    if (error.name === 'CastError') {
      return res.status(404).json({
        success: false,
        message: 'User not found'
      });
    }
    
    res.status(500).json({
      success: false,
      message: error.message
    });
  }
});

// Error handling
app.use((err, req, res, next) => {
  res.status(500).json({
    success: false,
    message: 'Server Error'
  });
});

// 404 handler
app.use('*', (req, res) => {
  res.status(404).json({
    success: false,
    message: 'Route not found'
  });
});

// Start server
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`🚀 Server running on http://localhost:${PORT}`);
});
```

### Step 3: Run Your API
```bash
npm run dev
```

You should see:
```
🍃 Connected to MongoDB
🚀 Server running on http://localhost:3000
```

---

## Testing Your API 🧪

### Using curl Commands

**1. Create users:**
```bash
curl -X POST http://localhost:3000/api/users \
  -H "Content-Type: application/json" \
  -d '{"name": "John Doe", "email": "john@example.com", "age": 30, "bio": "Software developer"}'

curl -X POST http://localhost:3000/api/users \
  -H "Content-Type: application/json" \
  -d '{"name": "Jane Smith", "email": "jane@example.com", "age": 25, "bio": "Designer"}'
```

**2. Get all users:**
```bash
curl http://localhost:3000/api/users
```

**3. Get users with filtering:**
```bash
# Get active users only
curl "http://localhost:3000/api/users?active=true"

# Search by name
curl "http://localhost:3000/api/users?name=john"

# Pagination
curl "http://localhost:3000/api/users?page=1&limit=2"
```

**4. Get single user:**
```bash
curl http://localhost:3000/api/users/USER_ID_HERE
```

**5. Update user:**
```bash
curl -X PUT http://localhost:3000/api/users/USER_ID_HERE \
  -H "Content-Type: application/json" \
  -d '{"name": "John Updated", "age": 31}'
```

**6. Delete user:**
```bash
curl -X DELETE http://localhost:3000/api/users/USER_ID_HERE
```

### Using Browser/Postman
- **GET requests**: Visit URLs directly in browser
- **POST/PUT/DELETE**: Use Postman or Thunder Client (VS Code extension)

### Sample Responses

**Success Response:**
```json
{
  "success": true,
  "count": 2,
  "total": 2,
  "page": 1,
  "pages": 1,
  "data": [
    {
      "_id": "64f123abc456def789",
      "name": "John Doe",
      "email": "john@example.com",
      "age": 30,
      "bio": "Software developer",
      "isActive": true,
      "createdAt": "2024-08-04T10:30:00.000Z",
      "updatedAt": "2024-08-04T10:30:00.000Z"
    }
  ]
}
```

**Error Response:**
```json
{
  "success": false,
  "message": "Validation Error",
  "errors": ["Name is required", "Email is required"]
}
```

---

## Best Practices 📝

### 1. Always Use Try-Catch
```javascript
app.get('/api/users', async (req, res) => {
  try {
    const users = await User.find();
    res.json({ success: true, data: users });
  } catch (error) {
    res.status(500).json({ success: false, message: error.message });
  }
});
```

### 2. Validate Input Data
```javascript
// In your schema
name: {
  type: String,
  required: [true, 'Name is required'],
  minlength: [2, 'Name too short']
}
```

### 3. Use Consistent Response Format
```javascript
// Success
{ success: true, data: {...} }

// Error
{ success: false, message: "Error description" }
```

### 4. Handle Different Error Types
```javascript
if (error.name === 'ValidationError') {
  // Handle validation errors
} else if (error.code === 11000) {
  // Handle duplicate key errors
} else if (error.name === 'CastError') {
  // Handle invalid ObjectId
}
```

### 5. Use Environment Variables
```javascript
// .env file
PORT=3000
MONGODB_URI=your_connection_string

// server.js
const PORT = process.env.PORT || 3000;
```

---

## Troubleshooting 🔧

### Common Issues

**1. "Cannot connect to MongoDB"**
- Check your connection string in `.env`
- Verify MongoDB is running (local) or cluster is active (Atlas)
- Check network access (Atlas)

**2. "ValidationError"**
- Check required fields are provided
- Verify data types match schema
- Check field length limits

**3. "E11000 duplicate key error"**
- Email already exists in database
- Check unique fields in schema

**4. "CastError"**
- Invalid ObjectId format
- User ID doesn't exist

### Debug Tips
```javascript
// Add logging
console.log('Request body:', req.body);
console.log('Query params:', req.query);

// Check database connection
mongoose.connection.on('connected', () => {
  console.log('✅ MongoDB connected');
});

mongoose.connection.on('error', (err) => {
  console.log('❌ MongoDB error:', err);
});
```

---

## Key Takeaways 🎯

### What You've Learned:
1. ✅ **REST API concepts** and HTTP methods
2. ✅ **MongoDB setup** (both cloud and local)
3. ✅ **Express.js** for building web servers
4. ✅ **Mongoose** for database operations
5. ✅ **Complete CRUD operations**
6. ✅ **Error handling** and validation
7. ✅ **API testing** with real examples

### Important Concepts:
- **Schema**: Data structure definition
- **Model**: Database operations interface
- **Middleware**: Functions between request/response
- **Validation**: Data integrity rules
- **CRUD**: Create, Read, Update, Delete operations

### Quick Reference:
```javascript
// Mongoose Operations
User.find()              // Get all
User.findById(id)        // Get one
User.create(data)        // Create
User.findByIdAndUpdate() // Update
User.findByIdAndDelete() // Delete

// Express Routes
app.get()    // GET request
app.post()   // POST request
app.put()    // PUT request
app.delete() // DELETE request
```

---

## Next Steps 🚀

1. **Practice**: Build your own API with different models (Posts, Comments)
2. **Add Authentication**: JWT tokens for secure access
3. **File Uploads**: Handle image uploads
4. **Relationships**: Connect Users with Posts
5. **Advanced Queries**: Aggregation and complex searches

**Congratulations! 🎉 You can now build complete REST APIs with Express and Mongoose!**

*Ready for the final topic: MongoDB + PostgreSQL basics!* 📚