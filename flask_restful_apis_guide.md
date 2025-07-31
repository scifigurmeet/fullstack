# Flask-RESTful APIs - Complete Beginner's Guide 🌐

## 🎯 Learning Objectives
By the end of this guide, you will:
- Understand REST APIs and HTTP methods
- Build simple CRUD APIs using Flask-RESTful
- Handle JSON data and error responses
- Test APIs using curl and Python

---

## 📖 What is a REST API?

**REST API** = A way for applications to talk to each other using HTTP methods.

### HTTP Methods
| Method | Purpose | Example |
|--------|---------|---------|
| `GET` | Get data | Get all books |
| `POST` | Create data | Add new book |
| `PUT` | Update data | Update book |
| `DELETE` | Remove data | Delete book |

### REST URL Pattern
```
GET    /api/books      # Get all books
POST   /api/books      # Create new book
GET    /api/books/1    # Get book with ID 1
PUT    /api/books/1    # Update book 1
DELETE /api/books/1    # Delete book 1
```

---

## 🚀 Installation & Setup

```bash
pip install flask flask-restful
```

### Basic API Structure
```python
from flask import Flask
from flask_restful import Api, Resource

app = Flask(__name__)
api = Api(app)

class HelloAPI(Resource):
    def get(self):
        return {'message': 'Hello World! 🌍'}

api.add_resource(HelloAPI, '/api/hello')

if __name__ == '__main__':
    app.run(debug=True)
```

**Test:** `curl http://localhost:5000/api/hello`

---

## 🛠️ Building a Simple Book API

### Step 1: Data Model
```python
# Simple in-memory storage
books = [
    {"id": 1, "title": "Python Basics", "author": "John Doe"},
    {"id": 2, "title": "Flask Guide", "author": "Jane Smith"}
]
next_id = 3

def find_book(book_id):
    return next((book for book in books if book['id'] == book_id), None)
```

### Step 2: Book List Resource
```python
from flask import request
from flask_restful import Resource

class BookList(Resource):
    def get(self):
        """Get all books"""
        return {'books': books}, 200
    
    def post(self):
        """Create new book"""
        global next_id
        data = request.get_json()
        
        if not data or not data.get('title'):
            return {'error': 'Title is required'}, 400
        
        new_book = {
            'id': next_id,
            'title': data['title'],
            'author': data.get('author', 'Unknown')
        }
        books.append(new_book)
        next_id += 1
        
        return {'book': new_book}, 201
```

### Step 3: Single Book Resource
```python
class Book(Resource):
    def get(self, book_id):
        """Get single book"""
        book = find_book(book_id)
        if not book:
            return {'error': 'Book not found'}, 404
        return {'book': book}, 200
    
    def put(self, book_id):
        """Update book"""
        book = find_book(book_id)
        if not book:
            return {'error': 'Book not found'}, 404
        
        data = request.get_json()
        if data.get('title'):
            book['title'] = data['title']
        if data.get('author'):
            book['author'] = data['author']
        
        return {'book': book}, 200
    
    def delete(self, book_id):
        """Delete book"""
        book = find_book(book_id)
        if not book:
            return {'error': 'Book not found'}, 404
        
        books.remove(book)
        return {'message': 'Book deleted'}, 200
```

### Step 4: Complete App
```python
from flask import Flask, request
from flask_restful import Api, Resource

app = Flask(__name__)
api = Api(app)

# Data storage
books = [
    {"id": 1, "title": "Python Basics", "author": "John Doe"},
    {"id": 2, "title": "Flask Guide", "author": "Jane Smith"}
]
next_id = 3

def find_book(book_id):
    return next((book for book in books if book['id'] == book_id), None)

class BookList(Resource):
    def get(self):
        return {'books': books}, 200
    
    def post(self):
        global next_id
        data = request.get_json()
        
        if not data or not data.get('title'):
            return {'error': 'Title is required'}, 400
        
        new_book = {
            'id': next_id,
            'title': data['title'],
            'author': data.get('author', 'Unknown')
        }
        books.append(new_book)
        next_id += 1
        return {'book': new_book}, 201

class Book(Resource):
    def get(self, book_id):
        book = find_book(book_id)
        if not book:
            return {'error': 'Book not found'}, 404
        return {'book': book}, 200
    
    def put(self, book_id):
        book = find_book(book_id)
        if not book:
            return {'error': 'Book not found'}, 404
        
        data = request.get_json()
        if data.get('title'):
            book['title'] = data['title']
        if data.get('author'):
            book['author'] = data['author']
        return {'book': book}, 200
    
    def delete(self, book_id):
        book = find_book(book_id)
        if not book:
            return {'error': 'Book not found'}, 404
        books.remove(book)
        return {'message': 'Book deleted'}, 200

# Register routes
api.add_resource(BookList, '/api/books')
api.add_resource(Book, '/api/books/<int:book_id>')

if __name__ == '__main__':
    app.run(debug=True)
```

---

## 🧪 Testing Your API

### Using curl

```bash
# Get all books
curl http://localhost:5000/api/books

# Get single book
curl http://localhost:5000/api/books/1

# Create new book
curl -X POST http://localhost:5000/api/books \
  -H "Content-Type: application/json" \
  -d '{"title": "New Book", "author": "New Author"}'

# Update book
curl -X PUT http://localhost:5000/api/books/1 \
  -H "Content-Type: application/json" \
  -d '{"title": "Updated Title"}'

# Delete book
curl -X DELETE http://localhost:5000/api/books/1
```

### Using Python
```python
import requests

BASE_URL = 'http://localhost:5000/api/books'

# Get all books
response = requests.get(BASE_URL)
print(response.json())

# Create book
new_book = {"title": "Python Guide", "author": "Test Author"}
response = requests.post(BASE_URL, json=new_book)
print(response.json())

# Update book
update_data = {"title": "Updated Python Guide"}
response = requests.put(f'{BASE_URL}/3', json=update_data)
print(response.json())
```

---

## 🔧 Adding Validation

```python
def validate_book_data(data):
    errors = []
    if not data:
        errors.append("No data provided")
    elif not data.get('title'):
        errors.append("Title is required")
    elif len(data['title']) < 2:
        errors.append("Title too short")
    return errors

class BookList(Resource):
    def post(self):
        data = request.get_json()
        errors = validate_book_data(data)
        
        if errors:
            return {'errors': errors}, 400
        
        # Create book logic...
```

---

## 📊 HTTP Status Codes

```python
# Success
return data, 200  # OK
return data, 201  # Created

# Client Errors  
return error, 400  # Bad Request
return error, 404  # Not Found

# Server Error
return error, 500  # Internal Error
```

---

## ✅ Best Practices

### 1. **Consistent Response Format**
```python
# Good structure
{
    "success": true,
    "data": {...},
    "message": "Operation successful"
}

# Error structure
{
    "success": false,
    "error": "Error message",
    "details": [...]
}
```

### 2. **Input Validation**
```python
# Always validate input
if not request.is_json:
    return {'error': 'JSON required'}, 400

data = request.get_json()
if not data:
    return {'error': 'No data provided'}, 400
```

### 3. **Error Handling**
```python
try:
    # API logic here
    return {'success': True}, 200
except Exception as e:
    return {'error': str(e)}, 500
```

---

## 🚨 Common Mistakes

❌ **Wrong HTTP methods**
```python
# Wrong
@app.route('/create_user', methods=['GET'])

# Correct  
@app.route('/users', methods=['POST'])
```

❌ **Not validating input**
```python
# Wrong - no validation
title = data['title']  # Crashes if no title

# Correct
title = data.get('title')
if not title:
    return {'error': 'Title required'}, 400
```

❌ **Wrong status codes**
```python
# Wrong
return {'error': 'Not found'}, 200

# Correct
return {'error': 'Not found'}, 404
```

---

## 🎯 Quick Reference

### Resource Methods
```python
class MyResource(Resource):
    def get(self):        # GET request
        pass
    def post(self):       # POST request  
        pass
    def put(self):        # PUT request
        pass
    def delete(self):     # DELETE request
        pass
```

### Adding Routes
```python
api.add_resource(BookList, '/api/books')
api.add_resource(Book, '/api/books/<int:book_id>')
```

### Getting Request Data
```python
data = request.get_json()        # JSON data
param = request.args.get('name') # Query parameter
```

---

## 🔥 Challenge Exercise

**Build a User API** with:
- GET `/api/users` - List users
- POST `/api/users` - Create user (name, email required)
- GET `/api/users/<id>` - Get user by ID
- PUT `/api/users/<id>` - Update user
- DELETE `/api/users/<id>` - Delete user

**Bonus:** Add email validation and prevent duplicate emails.

---

## 📚 Summary

✅ **REST APIs** use HTTP methods for CRUD operations  
✅ **Flask-RESTful** makes building APIs easy with Resource classes  
✅ **Always validate** input data and handle errors  
✅ **Use proper** HTTP status codes  
✅ **Test thoroughly** with curl or Python requests  

### Next: JWT Authentication 🔐
In the next guide, we'll learn how to secure our APIs with JWT tokens!

---

*📖 This guide is part of the Faculty Development Program series on Flask.*