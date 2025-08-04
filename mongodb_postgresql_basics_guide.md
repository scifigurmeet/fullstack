# MongoDB + PostgreSQL Basics 🗄️
## Simple Beginner's Guide for Faculty Development Program

---

## Table of Contents 📚
1. [Database Fundamentals](#database-fundamentals)
2. [MongoDB Basics](#mongodb-basics)
3. [PostgreSQL Basics](#postgresql-basics)
4. [MongoDB vs PostgreSQL](#mongodb-vs-postgresql)
5. [MongoDB with Node.js](#mongodb-with-nodejs)
6. [PostgreSQL with Node.js](#postgresql-with-nodejs)
7. [When to Use Which?](#when-to-use-which)
8. [Quick Reference](#quick-reference)

---

## Database Fundamentals 💾

### What is a Database?
A database is like a **digital filing system** that stores and organizes information.

### Types of Databases

```mermaid
graph TD
    A[Databases] --> B[SQL/Relational]
    A --> C[NoSQL/Document]
    B --> D[PostgreSQL - Tables & Rows]
    C --> E[MongoDB - Collections & Documents]
    
    style D fill:#336791
    style E fill:#4caf50
```

### Key Differences

| Aspect | SQL (PostgreSQL) | NoSQL (MongoDB) |
|--------|------------------|-----------------|
| **Structure** | Fixed tables/columns | Flexible documents |
| **Data Format** | Rows and columns | JSON-like documents |
| **Query Language** | SQL | MongoDB queries |
| **Schema** | Must define first | Dynamic/flexible |

---

## MongoDB Basics 🍃

### What is MongoDB?
MongoDB stores data as **JSON-like documents** in collections.

### Installation

**Option 1: MongoDB Atlas (Cloud - Recommended)**
1. Go to [MongoDB Atlas](https://www.mongodb.com/cloud/atlas)
2. Create free account and cluster
3. Get connection string: 
   ```
   mongodb+srv://username:password@cluster.mongodb.net/myapp
   ```

**Option 2: Local Installation**
```bash
# Windows: Download from mongodb.com
# macOS:
brew install mongodb-community
brew services start mongodb-community

# Linux:
sudo apt install mongodb
sudo systemctl start mongodb
```

### Basic MongoDB Operations

**Using MongoDB Shell:**
```javascript
// Connect to database
use myapp

// Create documents
db.users.insertOne({
  name: "John Doe",
  email: "john@example.com",
  age: 30,
  hobbies: ["reading", "coding"]
})

// Read documents
db.users.find()                        // Get all
db.users.findOne({name: "John Doe"})   // Get one
db.users.find({age: {$gte: 25}})       // Age >= 25

// Update documents
db.users.updateOne(
  {name: "John Doe"}, 
  {$set: {age: 31}}
)

// Delete documents
db.users.deleteOne({name: "John Doe"})
```

### MongoDB Document Structure
```javascript
{
  "_id": ObjectId("64f123..."),  // Auto-generated ID
  "name": "John Doe",
  "email": "john@example.com",
  "age": 30,
  "address": {                   // Nested object
    "city": "New York",
    "zipCode": "10001"
  },
  "hobbies": ["reading", "coding"], // Array
  "createdAt": ISODate("2024-08-04T10:30:00Z")
}
```

---

## PostgreSQL Basics 🐘

### What is PostgreSQL?
PostgreSQL stores data in **tables with rows and columns** using SQL.

### Installation

**Option 1: Local Installation**
```bash
# Windows: Download from postgresql.org
# macOS:
brew install postgresql
brew services start postgresql

# Linux:
sudo apt install postgresql
sudo systemctl start postgresql
```

**Option 2: Docker (Easy)**
```bash
docker run --name postgres-db \
  -e POSTGRES_USER=myuser \
  -e POSTGRES_PASSWORD=mypass \
  -e POSTGRES_DB=myapp \
  -p 5432:5432 -d postgres:15
```

### Basic PostgreSQL Operations

**Using psql:**
```sql
-- Connect to database
\c myapp

-- Create table
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    age INTEGER,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Insert data
INSERT INTO users (name, email, age) VALUES 
('John Doe', 'john@example.com', 30),
('Jane Smith', 'jane@example.com', 25);

-- Select data
SELECT * FROM users;
SELECT name, email FROM users WHERE age >= 25;

-- Update data
UPDATE users SET age = 31 WHERE name = 'John Doe';

-- Delete data
DELETE FROM users WHERE name = 'John Doe';
```

### PostgreSQL Table Structure
```sql
-- Users table
| id | name     | email           | age | created_at          |
|----|----------|-----------------|-----|---------------------|
| 1  | John Doe | john@email.com  | 30  | 2024-08-04 10:30:00 |
| 2  | Jane     | jane@email.com  | 25  | 2024-08-04 11:00:00 |
```

---

## MongoDB vs PostgreSQL 🆚

### When to Use What?

**Use MongoDB when:**
- 🚀 Rapid development/prototyping
- 📱 Content management systems
- 🛒 Product catalogs
- 📊 Real-time analytics
- 🌐 APIs with changing data structure

**Use PostgreSQL when:**
- 💰 Financial applications
- 📈 Data analytics/reporting
- 🏢 Enterprise applications
- 🔄 Complex relationships between data
- 📋 Data integrity is critical

### Data Structure Comparison

**MongoDB (Flexible):**
```javascript
// User with posts embedded
{
  "name": "John Doe",
  "email": "john@example.com",
  "posts": [
    {
      "title": "My First Post",
      "content": "Hello world!",
      "tags": ["intro", "hello"]
    }
  ]
}
```

**PostgreSQL (Structured):**
```sql
-- Separate tables with relationships
-- Users table
| id | name     | email           |
|----|----------|-----------------|
| 1  | John Doe | john@email.com  |

-- Posts table
| id | user_id | title         | content      |
|----|---------|---------------|--------------|
| 1  | 1       | My First Post | Hello world! |
```

---

## MongoDB with Node.js 🍃

### Setup
```bash
npm install mongodb mongoose
```

### Basic Example
```javascript
const mongoose = require('mongoose');

// Connect to MongoDB
mongoose.connect('mongodb://localhost:27017/myapp')
  .then(() => console.log('🍃 Connected to MongoDB'))
  .catch(err => console.error('❌ MongoDB Error:', err));

// Define schema
const userSchema = new mongoose.Schema({
  name: { type: String, required: true },
  email: { type: String, required: true, unique: true },
  age: { type: Number, min: 0 },
  posts: [{
    title: String,
    content: String,
    createdAt: { type: Date, default: Date.now }
  }]
}, { timestamps: true });

const User = mongoose.model('User', userSchema);

// CRUD Operations
async function mongodbExample() {
  try {
    // Create
    const user = new User({
      name: "Alice Johnson",
      email: "alice@example.com", 
      age: 28,
      posts: [{ title: "Hello", content: "My first post!" }]
    });
    await user.save();
    console.log("✅ User created:", user._id);
    
    // Read
    const users = await User.find({ age: { $gte: 25 } });
    console.log("📖 Users found:", users.length);
    
    // Update
    await User.findByIdAndUpdate(user._id, { age: 29 });
    console.log("🔄 User updated");
    
    // Delete
    await User.findByIdAndDelete(user._id);
    console.log("🗑️ User deleted");
    
  } catch (error) {
    console.error("❌ Error:", error.message);
  }
}

mongodbExample();
```

---

## PostgreSQL with Node.js 🐘

### Setup
```bash
npm install pg
```

### Complete Example
```javascript
const { Pool } = require('pg');

// Database connection
const pool = new Pool({
  user: 'myuser',
  host: 'localhost',
  database: 'myapp',
  password: 'mypass',
  port: 5432,
});

async function postgresExample() {
  const client = await pool.connect();
  
  try {
    console.log('🐘 Connected to PostgreSQL');
    
    // Create table
    await client.query(`
      CREATE TABLE IF NOT EXISTS users (
        id SERIAL PRIMARY KEY,
        name VARCHAR(100) NOT NULL,
        email VARCHAR(255) UNIQUE NOT NULL,
        age INTEGER,
        created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
      )
    `);
    
    // Create user
    const insertResult = await client.query(
      'INSERT INTO users (name, email, age) VALUES ($1, $2, $3) RETURNING *',
      ['John Doe', 'john@example.com', 30]
    );
    const newUser = insertResult.rows[0];
    console.log('✅ User created:', newUser.id);
    
    // Read users
    const usersResult = await client.query('SELECT * FROM users WHERE age >= $1', [25]);
    console.log('📖 Users found:', usersResult.rows.length);
    
    // Update user
    await client.query(
      'UPDATE users SET age = $1 WHERE id = $2',
      [31, newUser.id]
    );
    console.log('🔄 User updated');
    
    // Delete user
    await client.query('DELETE FROM users WHERE id = $1', [newUser.id]);
    console.log('🗑️ User deleted');
    
  } catch (error) {
    console.error('❌ Error:', error.message);
  } finally {
    client.release();
  }
}

// Helper class for common operations
class UserService {
  static async createUser(name, email, age) {
    const result = await pool.query(
      'INSERT INTO users (name, email, age) VALUES ($1, $2, $3) RETURNING *',
      [name, email, age]
    );
    return result.rows[0];
  }
  
  static async getUserById(id) {
    const result = await pool.query('SELECT * FROM users WHERE id = $1', [id]);
    return result.rows[0];
  }
  
  static async getAllUsers() {
    const result = await pool.query('SELECT * FROM users ORDER BY created_at DESC');
    return result.rows;
  }
  
  static async updateUser(id, name, email, age) {
    const result = await pool.query(
      'UPDATE users SET name = $1, email = $2, age = $3 WHERE id = $4 RETURNING *',
      [name, email, age, id]
    );
    return result.rows[0];
  }
  
  static async deleteUser(id) {
    const result = await pool.query('DELETE FROM users WHERE id = $1 RETURNING *', [id]);
    return result.rows[0];
  }
}

// Usage example
async function demonstrateService() {
  try {
    const user = await UserService.createUser('Jane Smith', 'jane@example.com', 28);
    console.log('Created:', user);
    
    const foundUser = await UserService.getUserById(user.id);
    console.log('Found:', foundUser);
    
    const updatedUser = await UserService.updateUser(user.id, 'Jane Updated', 'jane.new@example.com', 29);
    console.log('Updated:', updatedUser);
    
    const allUsers = await UserService.getAllUsers();
    console.log('All users:', allUsers.length);
    
  } catch (error) {
    console.error('Service error:', error.message);
  }
}

postgresExample().then(() => demonstrateService());
```

---

## When to Use Which? 🤔

### Decision Tree

```mermaid
flowchart TD
    A[New Project] --> B{Data Structure?}
    B -->|Flexible/Changing| C[MongoDB]
    B -->|Fixed/Structured| D{Complex Relationships?}
    D -->|Yes| E[PostgreSQL]
    D -->|No| F[Either works]
    
    G{Team Experience?} --> H[SQL Knowledge?]
    H -->|Yes| I[PostgreSQL] 
    H -->|No| J[MongoDB]
    
    style C fill:#4caf50
    style E fill:#336791
    style I fill:#336791
    style J fill:#4caf50
```

### Real Examples

**MongoDB Perfect For:**
```javascript
// Blog post with flexible content
{
  "title": "My Blog Post",
  "content": "Long article...",
  "author": {
    "name": "John Doe",
    "bio": "Writer"
  },
  "tags": ["tech", "programming"],
  "comments": [
    { "user": "Jane", "text": "Great post!" }
  ],
  "metadata": {
    "readTime": 5,
    "wordCount": 1200
  }
}
```

**PostgreSQL Perfect For:**
```sql
-- E-commerce with strict relationships
CREATE TABLE customers (id, name, email);
CREATE TABLE orders (id, customer_id, total, status);
CREATE TABLE order_items (id, order_id, product_id, quantity);
CREATE TABLE products (id, name, price, stock);

-- Ensures data integrity with foreign keys
ALTER TABLE orders ADD FOREIGN KEY (customer_id) REFERENCES customers(id);
```

---

## Quick Reference 🗂️

### MongoDB Operations
```javascript
// Connection
mongoose.connect('mongodb://localhost:27017/myapp');

// Schema & Model
const schema = new mongoose.Schema({field: Type});
const Model = mongoose.model('Name', schema);

// CRUD
Model.create(data)              // Create
Model.find(query)               // Read all
Model.findById(id)              // Read one  
Model.findByIdAndUpdate(id, data) // Update
Model.findByIdAndDelete(id)     // Delete

// Common queries
Model.find({age: {$gte: 25}})   // Age >= 25
Model.find({name: /john/i})     // Contains "john" (case insensitive)
```

### PostgreSQL Operations
```javascript
// Connection
const pool = new Pool({user, host, database, password, port});

// CRUD with parameterized queries (secure)
pool.query('INSERT INTO users (name) VALUES ($1)', [name])
pool.query('SELECT * FROM users WHERE age >= $1', [25])
pool.query('UPDATE users SET name = $1 WHERE id = $2', [name, id])
pool.query('DELETE FROM users WHERE id = $1', [id])

// Common SQL patterns
'SELECT * FROM users ORDER BY created_at DESC LIMIT 10'
'SELECT COUNT(*) FROM users WHERE active = true'
'SELECT AVG(age) FROM users'
```

### Key Differences Summary

| Operation | MongoDB | PostgreSQL |
|-----------|---------|------------|
| **Create** | `insertOne({})` | `INSERT INTO table VALUES` |
| **Read** | `find({})` | `SELECT * FROM table WHERE` |
| **Update** | `updateOne({}, {$set: {}})` | `UPDATE table SET field = value` |
| **Delete** | `deleteOne({})` | `DELETE FROM table WHERE` |
| **Filter** | `{age: {$gte: 25}}` | `WHERE age >= 25` |
| **Sort** | `.sort({field: 1})` | `ORDER BY field ASC` |

---

## Key Takeaways 🎯

### What You've Learned:
1. ✅ **Database fundamentals** - SQL vs NoSQL differences
2. ✅ **MongoDB basics** - Documents, collections, flexible schema
3. ✅ **PostgreSQL basics** - Tables, rows, structured data
4. ✅ **Node.js integration** - Complete working examples for both
5. ✅ **When to use which** - Decision criteria and real examples
6. ✅ **CRUD operations** - Create, Read, Update, Delete in both systems

### Important Concepts:
- **Schema**: MongoDB is flexible, PostgreSQL is fixed
- **Relationships**: MongoDB embeds data, PostgreSQL uses foreign keys  
- **Queries**: MongoDB uses objects, PostgreSQL uses SQL
- **Scaling**: MongoDB scales horizontally, PostgreSQL vertically
- **ACID**: PostgreSQL full ACID, MongoDB document-level

### Quick Decision Guide:
- **Choose MongoDB** for: Rapid development, flexible data, content management
- **Choose PostgreSQL** for: Financial apps, complex relationships, data integrity

---

**🎉 Congratulations! You now understand both MongoDB and PostgreSQL fundamentals and can use them with Node.js!**

*You've completed all three topics in the Faculty Development Program series!* 🚀�