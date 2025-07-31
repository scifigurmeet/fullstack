# JWT with Flask - Complete Beginner's Guide 🔐

## 🎯 Learning Objectives
By the end of this guide, you will:
- Understand what JWT tokens are and why they're used
- Implement JWT authentication in Flask APIs
- Create login/register endpoints with JWT
- Protect API routes with JWT verification
- Handle token expiration and refresh

---

## 📖 What is JWT?

**JWT (JSON Web Token)** = A secure way to transmit information between parties as a JSON object.

### 🤔 Why Use JWT?

**Without JWT (Traditional Sessions):**
```mermaid
graph LR
    A[User Login] --> B[Server Creates Session]
    B --> C[Session Stored on Server]
    C --> D[Session ID in Cookie]
    D --> E[User Makes Request]
    E --> F[Server Checks Session Store]
```

**With JWT (Stateless):**
```mermaid
graph LR
    A[User Login] --> B[Server Creates JWT]
    B --> C[JWT Sent to Client]
    C --> D[Client Stores JWT]
    D --> E[JWT in Request Header]
    E --> F[Server Verifies JWT]
```

### 🌟 JWT Benefits

1. **Stateless** - No server-side session storage
2. **Scalable** - Works across multiple servers
3. **Self-contained** - All info in the token
4. **Cross-platform** - Works with any technology

### JWT Structure
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c

Header.Payload.Signature
```

---

## 🚀 Installation & Setup

```bash
pip install flask pyjwt bcrypt
```

### Basic JWT Functions
```python
import jwt
import bcrypt
from datetime import datetime, timedelta
from functools import wraps
from flask import Flask, request, jsonify

app = Flask(__name__)
app.config['SECRET_KEY'] = 'your-secret-key-change-this'

def generate_token(user_id):
    """Generate JWT token"""
    payload = {
        'user_id': user_id,
        'exp': datetime.utcnow() + timedelta(hours=24)  # Expires in 24 hours
    }
    return jwt.encode(payload, app.config['SECRET_KEY'], algorithm='HS256')

def verify_token(token):
    """Verify JWT token"""
    try:
        payload = jwt.decode(token, app.config['SECRET_KEY'], algorithms=['HS256'])
        return payload['user_id']
    except jwt.ExpiredSignatureError:
        return None  # Token expired
    except jwt.InvalidTokenError:
        return None  # Invalid token

def hash_password(password):
    """Hash password with bcrypt"""
    return bcrypt.hashpw(password.encode('utf-8'), bcrypt.gensalt())

def check_password(password, hashed):
    """Check if password matches hash"""
    return bcrypt.checkpw(password.encode('utf-8'), hashed)
```

---

## 🛠️ Building JWT Authentication

### Step 1: User Data Storage
```python
# Simple in-memory storage (use database in real apps)
users = [
    {
        'id': 1,
        'username': 'admin',
        'email': 'admin@example.com',
        'password': hash_password('password123')
    }
]
next_user_id = 2

def find_user_by_username(username):
    return next((user for user in users if user['username'] == username), None)

def find_user_by_id(user_id):
    return next((user for user in users if user['id'] == user_id), None)
```

### Step 2: Registration Endpoint
```python
@app.route('/api/register', methods=['POST'])
def register():
    data = request.get_json()
    
    # Validation
    if not data or not data.get('username') or not data.get('password'):
        return jsonify({'error': 'Username and password required'}), 400
    
    # Check if user exists
    if find_user_by_username(data['username']):
        return jsonify({'error': 'Username already exists'}), 400
    
    # Create new user
    global next_user_id
    new_user = {
        'id': next_user_id,
        'username': data['username'],
        'email': data.get('email', ''),
        'password': hash_password(data['password'])
    }
    users.append(new_user)
    next_user_id += 1
    
    # Generate token
    token = generate_token(new_user['id'])
    
    return jsonify({
        'message': 'User created successfully',
        'token': token,
        'user': {
            'id': new_user['id'],
            'username': new_user['username'],
            'email': new_user['email']
        }
    }), 201
```

### Step 3: Login Endpoint
```python
@app.route('/api/login', methods=['POST'])
def login():
    data = request.get_json()
    
    # Validation
    if not data or not data.get('username') or not data.get('password'):
        return jsonify({'error': 'Username and password required'}), 400
    
    # Find user
    user = find_user_by_username(data['username'])
    if not user:
        return jsonify({'error': 'Invalid credentials'}), 401
    
    # Check password
    if not check_password(data['password'], user['password']):
        return jsonify({'error': 'Invalid credentials'}), 401
    
    # Generate token
    token = generate_token(user['id'])
    
    return jsonify({
        'message': 'Login successful',
        'token': token,
        'user': {
            'id': user['id'],
            'username': user['username'],
            'email': user['email']
        }
    }), 200
```

### Step 4: JWT Protection Decorator
```python
def token_required(f):
    """Decorator to protect routes with JWT"""
    @wraps(f)
    def decorated(*args, **kwargs):
        token = None
        
        # Get token from header
        if 'Authorization' in request.headers:
            auth_header = request.headers['Authorization']
            try:
                token = auth_header.split(" ")[1]  # "Bearer <token>"
            except IndexError:
                return jsonify({'error': 'Invalid token format'}), 401
        
        if not token:
            return jsonify({'error': 'Token is missing'}), 401
        
        # Verify token
        user_id = verify_token(token)
        if user_id is None:
            return jsonify({'error': 'Token is invalid or expired'}), 401
        
        # Get current user
        current_user = find_user_by_id(user_id)
        if not current_user:
            return jsonify({'error': 'User not found'}), 401
        
        # Pass current user to the route
        return f(current_user, *args, **kwargs)
    
    return decorated
```

### Step 5: Protected Routes
```python
@app.route('/api/profile', methods=['GET'])
@token_required
def get_profile(current_user):
    """Get user profile - requires JWT token"""
    return jsonify({
        'user': {
            'id': current_user['id'],
            'username': current_user['username'],
            'email': current_user['email']
        }
    }), 200

@app.route('/api/protected', methods=['GET'])
@token_required
def protected_route(current_user):
    """Example protected route"""
    return jsonify({
        'message': f'Hello {current_user["username"]}! This is a protected route.',
        'user_id': current_user['id']
    }), 200
```

---

## 📋 Complete Example App

```python
import jwt
import bcrypt
from datetime import datetime, timedelta
from functools import wraps
from flask import Flask, request, jsonify

app = Flask(__name__)
app.config['SECRET_KEY'] = 'your-secret-key-change-this'

# Helper functions
def generate_token(user_id):
    payload = {
        'user_id': user_id,
        'exp': datetime.utcnow() + timedelta(hours=24)
    }
    return jwt.encode(payload, app.config['SECRET_KEY'], algorithm='HS256')

def verify_token(token):
    try:
        payload = jwt.decode(token, app.config['SECRET_KEY'], algorithms=['HS256'])
        return payload['user_id']
    except:
        return None

def hash_password(password):
    return bcrypt.hashpw(password.encode('utf-8'), bcrypt.gensalt())

def check_password(password, hashed):
    return bcrypt.checkpw(password.encode('utf-8'), hashed)

# Data storage
users = [{
    'id': 1,
    'username': 'admin',
    'email': 'admin@example.com',
    'password': hash_password('password123')
}]
next_user_id = 2

def find_user_by_username(username):
    return next((user for user in users if user['username'] == username), None)

def find_user_by_id(user_id):
    return next((user for user in users if user['id'] == user_id), None)

# Decorator
def token_required(f):
    @wraps(f)
    def decorated(*args, **kwargs):
        token = None
        if 'Authorization' in request.headers:
            auth_header = request.headers['Authorization']
            try:
                token = auth_header.split(" ")[1]
            except IndexError:
                return jsonify({'error': 'Invalid token format'}), 401
        
        if not token:
            return jsonify({'error': 'Token is missing'}), 401
        
        user_id = verify_token(token)
        if user_id is None:
            return jsonify({'error': 'Token is invalid or expired'}), 401
        
        current_user = find_user_by_id(user_id)
        if not current_user:
            return jsonify({'error': 'User not found'}), 401
        
        return f(current_user, *args, **kwargs)
    return decorated

# Routes
@app.route('/api/register', methods=['POST'])
def register():
    data = request.get_json()
    
    if not data or not data.get('username') or not data.get('password'):
        return jsonify({'error': 'Username and password required'}), 400
    
    if find_user_by_username(data['username']):
        return jsonify({'error': 'Username already exists'}), 400
    
    global next_user_id
    new_user = {
        'id': next_user_id,
        'username': data['username'],
        'email': data.get('email', ''),
        'password': hash_password(data['password'])
    }
    users.append(new_user)
    next_user_id += 1
    
    token = generate_token(new_user['id'])
    
    return jsonify({
        'message': 'User created successfully',
        'token': token,
        'user': {'id': new_user['id'], 'username': new_user['username']}
    }), 201

@app.route('/api/login', methods=['POST'])
def login():
    data = request.get_json()
    
    if not data or not data.get('username') or not data.get('password'):
        return jsonify({'error': 'Username and password required'}), 400
    
    user = find_user_by_username(data['username'])
    if not user or not check_password(data['password'], user['password']):
        return jsonify({'error': 'Invalid credentials'}), 401
    
    token = generate_token(user['id'])
    
    return jsonify({
        'message': 'Login successful',
        'token': token,
        'user': {'id': user['id'], 'username': user['username']}
    }), 200

@app.route('/api/profile', methods=['GET'])
@token_required
def get_profile(current_user):
    return jsonify({
        'user': {
            'id': current_user['id'],
            'username': current_user['username'],
            'email': current_user['email']
        }
    }), 200

@app.route('/api/protected', methods=['GET'])
@token_required
def protected_route(current_user):
    return jsonify({
        'message': f'Hello {current_user["username"]}!',
        'timestamp': datetime.utcnow().isoformat()
    }), 200

if __name__ == '__main__':
    print("🚀 JWT API Server running...")
    print("📝 Register: POST /api/register")
    print("🔐 Login: POST /api/login")
    print("👤 Profile: GET /api/profile (protected)")
    app.run(debug=True)
```

---

## 🧪 Testing JWT Authentication

### Step 1: Register User
```bash
curl -X POST http://localhost:5000/api/register \
  -H "Content-Type: application/json" \
  -d '{
    "username": "testuser",
    "password": "mypassword",
    "email": "test@example.com"
  }'
```

**Response:**
```json
{
  "message": "User created successfully",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {"id": 2, "username": "testuser"}
}
```

### Step 2: Login
```bash
curl -X POST http://localhost:5000/api/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": "testuser",
    "password": "mypassword"
  }'
```

### Step 3: Access Protected Route
```bash
# Save token from login response
TOKEN="your-jwt-token-here"

curl -X GET http://localhost:5000/api/profile \
  -H "Authorization: Bearer $TOKEN"
```

### Step 4: Test Without Token (Should Fail)
```bash
curl -X GET http://localhost:5000/api/profile
# Returns: {"error": "Token is missing"}
```

---

## 🔧 Advanced Features

### 1. Token Refresh
```python
@app.route('/api/refresh', methods=['POST'])
@token_required
def refresh_token(current_user):
    """Generate new token for authenticated user"""
    new_token = generate_token(current_user['id'])
    return jsonify({
        'message': 'Token refreshed',
        'token': new_token
    }), 200
```

### 2. Logout (Token Blacklist)
```python
# Simple blacklist (use Redis in production)
blacklisted_tokens = set()

def is_token_blacklisted(token):
    return token in blacklisted_tokens

@app.route('/api/logout', methods=['POST'])
@token_required
def logout(current_user):
    """Blacklist current token"""
    token = request.headers['Authorization'].split(" ")[1]
    blacklisted_tokens.add(token)
    return jsonify({'message': 'Logged out successfully'}), 200

# Update token verification
def verify_token(token):
    if is_token_blacklisted(token):
        return None
    try:
        payload = jwt.decode(token, app.config['SECRET_KEY'], algorithms=['HS256'])
        return payload['user_id']
    except:
        return None
```

### 3. Custom Token Payload
```python
def generate_token(user_id, username, role='user'):
    """Generate token with custom data"""
    payload = {
        'user_id': user_id,
        'username': username,
        'role': role,
        'iat': datetime.utcnow(),  # Issued at
        'exp': datetime.utcnow() + timedelta(hours=24)
    }
    return jwt.encode(payload, app.config['SECRET_KEY'], algorithm='HS256')
```

---

## 🛡️ Security Best Practices

### 1. **Strong Secret Key**
```python
import secrets

# Generate strong secret key
app.config['SECRET_KEY'] = secrets.token_hex(32)

# Or use environment variable
import os
app.config['SECRET_KEY'] = os.environ.get('SECRET_KEY', 'fallback-key')
```

### 2. **Token Expiration**
```python
# Short-lived tokens
payload = {
    'user_id': user_id,
    'exp': datetime.utcnow() + timedelta(minutes=15)  # 15 minutes
}
```

### 3. **HTTPS Only**
```python
# Set secure headers
@app.after_request
def after_request(response):
    response.headers['X-Content-Type-Options'] = 'nosniff'
    response.headers['X-Frame-Options'] = 'DENY'
    response.headers['X-XSS-Protection'] = '1; mode=block'
    return response
```

---

## ✅ Best Practices

### 1. **Token Storage (Client-side)**
```javascript
// ✅ Good - Store in memory/state
const [token, setToken] = useState(null);

// ❌ Avoid - localStorage (XSS risk)
localStorage.setItem('token', token);

// ⚠️ Acceptable - httpOnly cookies (if possible)
// Set-Cookie: token=...; HttpOnly; Secure; SameSite=Strict
```

### 2. **Error Handling**
```python
# ✅ Generic error messages
return jsonify({'error': 'Invalid credentials'}), 401

# ❌ Specific errors (security risk)
return jsonify({'error': 'Username not found'}), 401
```

### 3. **Token Validation**
```python
def token_required(f):
    @wraps(f)
    def decorated(*args, **kwargs):
        # Always validate token format, signature, and expiration
        # Handle all possible JWT exceptions
        pass
    return decorated
```

---

## 🚨 Common Mistakes

❌ **Storing sensitive data in JWT**
```python
# Wrong - password in token
payload = {'user_id': 1, 'password': 'secret'}

# Correct - only non-sensitive data
payload = {'user_id': 1, 'role': 'user'}
```

❌ **Not handling token expiration**
```python
# Wrong - no expiration handling
try:
    payload = jwt.decode(token, secret, algorithms=['HS256'])
except:
    return 'Error'

# Correct - handle specific exceptions
try:
    payload = jwt.decode(token, secret, algorithms=['HS256'])
except jwt.ExpiredSignatureError:
    return 'Token expired'
except jwt.InvalidTokenError:
    return 'Invalid token'
```

❌ **Weak secret keys**
```python
# Wrong
SECRET_KEY = 'secret123'

# Correct  
SECRET_KEY = secrets.token_hex(32)
```

---

## 🎯 Quick Reference

### Generate Token
```python
token = jwt.encode(payload, secret, algorithm='HS256')
```

### Verify Token
```python
payload = jwt.decode(token, secret, algorithms=['HS256'])
```

### Authorization Header
```
Authorization: Bearer <token>
```

### Token Structure
```python
payload = {
    'user_id': 123,
    'exp': datetime.utcnow() + timedelta(hours=24)
}
```

---

## 🔥 Challenge Exercise

**Enhance the JWT API:**
1. Add user roles (admin, user)
2. Create role-based protection decorator
3. Add password change endpoint (protected)
4. Implement token refresh mechanism
5. Add basic rate limiting for login attempts

**Bonus:** Create a simple frontend that stores and uses JWT tokens.

---

## 📚 Summary

✅ **JWT tokens** provide stateless authentication  
✅ **Flask + PyJWT** makes implementation easy  
✅ **Always hash passwords** with bcrypt  
✅ **Protect routes** with decorators  
✅ **Handle token expiration** and errors properly  
✅ **Use strong secret keys** and HTTPS in production  

### Next: JWT-Extended 🔧
In the next guide, we'll explore Flask-JWT-Extended for advanced JWT features!

---

*📖 This guide is part of the Faculty Development Program series on Flask.*