# Flask-JWT-Extended - Complete Beginner's Guide 🔧

## 🎯 Learning Objectives
By the end of this guide, you will:
- Understand Flask-JWT-Extended benefits over basic JWT
- Implement access and refresh tokens
- Create token blacklisting for logout
- Handle token callbacks and protection

---

## 📖 What is Flask-JWT-Extended?

**Flask-JWT-Extended** = A powerful extension that adds advanced JWT features to Flask.

### Basic JWT vs JWT-Extended

**Basic JWT:**
```python
# Manual everything
token = jwt.encode(payload, secret)
payload = jwt.decode(token, secret)
```

**JWT-Extended:**
```python
# Built-in functions
access_token = create_access_token(identity=user_id)
current_user = get_jwt_identity()
```

### 🌟 Key Benefits
1. **Access & Refresh Tokens** - Better security
2. **Token Blacklisting** - Proper logout
3. **Built-in Decorators** - Easy route protection
4. **Automatic Error Handling** - Less code

---

## 🚀 Installation & Setup

```bash
pip install flask flask-jwt-extended bcrypt
```

---

## 🛠️ Simple JWT-Extended App

```python
from flask import Flask, request, jsonify
from flask_jwt_extended import (
    JWTManager, jwt_required, create_access_token, create_refresh_token,
    get_jwt_identity, get_jwt
)
import bcrypt
from datetime import timedelta

app = Flask(__name__)

# Configuration
app.config['JWT_SECRET_KEY'] = 'your-secret-key-change-this'
app.config['JWT_ACCESS_TOKEN_EXPIRES'] = timedelta(hours=1)

jwt = JWTManager(app)

# Simple user storage
users = [{
    'id': 1,
    'username': 'admin',
    'password': bcrypt.hashpw('password123'.encode('utf-8'), bcrypt.gensalt())
}]

# Token blacklist
blacklisted_tokens = set()

def find_user(username):
    return next((user for user in users if user['username'] == username), None)

def check_password(password, hashed):
    return bcrypt.checkpw(password.encode('utf-8'), hashed)

# JWT Callbacks
@jwt.token_in_blocklist_loader
def check_if_token_revoked(jwt_header, jwt_payload):
    return jwt_payload['jti'] in blacklisted_tokens

# Routes
@app.route('/api/login', methods=['POST'])
def login():
    data = request.get_json()
    
    if not data or not data.get('username') or not data.get('password'):
        return jsonify({'error': 'Username and password required'}), 400
    
    user = find_user(data['username'])
    if not user or not check_password(data['password'], user['password']):
        return jsonify({'error': 'Invalid credentials'}), 401
    
    # Create tokens
    access_token = create_access_token(identity=user['id'])
    refresh_token = create_refresh_token(identity=user['id'])
    
    return jsonify({
        'access_token': access_token,
        'refresh_token': refresh_token
    }), 200

@app.route('/api/refresh', methods=['POST'])
@jwt_required(refresh=True)
def refresh():
    current_user_id = get_jwt_identity()
    new_token = create_access_token(identity=current_user_id)
    return jsonify({'access_token': new_token}), 200

@app.route('/api/logout', methods=['POST'])
@jwt_required()
def logout():
    jti = get_jwt()['jti']  # JWT ID
    blacklisted_tokens.add(jti)
    return jsonify({'message': 'Logged out successfully'}), 200

@app.route('/api/protected', methods=['GET'])
@jwt_required()
def protected():
    current_user_id = get_jwt_identity()
    return jsonify({
        'message': f'Hello user {current_user_id}!',
        'user_id': current_user_id
    }), 200

if __name__ == '__main__':
    print("🚀 JWT-Extended API running...")
    print("🔐 Login: POST /api/login")
    print("🔄 Refresh: POST /api/refresh") 
    print("🚪 Logout: POST /api/logout")
    print("🛡️ Protected: GET /api/protected")
    app.run(debug=True)
```

---

## 🧪 Testing the API

### 1. Login
```bash
curl -X POST http://localhost:5000/api/login \
  -H "Content-Type: application/json" \
  -d '{"username": "admin", "password": "password123"}'
```

**Response:**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

### 2. Access Protected Route
```bash
ACCESS_TOKEN="your-access-token-here"

curl -X GET http://localhost:5000/api/protected \
  -H "Authorization: Bearer $ACCESS_TOKEN"
```

### 3. Refresh Token
```bash
REFRESH_TOKEN="your-refresh-token-here"

curl -X POST http://localhost:5000/api/refresh \
  -H "Authorization: Bearer $REFRESH_TOKEN"
```

### 4. Logout
```bash
curl -X POST http://localhost:5000/api/logout \
  -H "Authorization: Bearer $ACCESS_TOKEN"
```

### 5. Test After Logout (Should Fail)
```bash
curl -X GET http://localhost:5000/api/protected \
  -H "Authorization: Bearer $ACCESS_TOKEN"
# Response: {"msg": "Token has been revoked"}
```

---

## 🔧 Key Features Explained

### 1. Access & Refresh Tokens
```python
# Access token - short-lived (1 hour)
access_token = create_access_token(identity=user_id)

# Refresh token - long-lived (30 days default)
refresh_token = create_refresh_token(identity=user_id)
```

### 2. Token Protection
```python
@jwt_required()              # Requires access token
@jwt_required(refresh=True)  # Requires refresh token
```

### 3. Token Blacklisting
```python
@jwt.token_in_blocklist_loader
def check_if_token_revoked(jwt_header, jwt_payload):
    return jwt_payload['jti'] in blacklisted_tokens
```

### 4. Getting User Info
```python
@jwt_required()
def some_route():
    user_id = get_jwt_identity()  # Get user ID from token
    jwt_data = get_jwt()          # Get full JWT data
```

---

## 🛡️ Adding User Registration

```python
@app.route('/api/register', methods=['POST'])
def register():
    data = request.get_json()
    
    if not data or not data.get('username') or not data.get('password'):
        return jsonify({'error': 'Username and password required'}), 400
    
    # Check if user exists
    if find_user(data['username']):
        return jsonify({'error': 'Username already exists'}), 400
    
    # Create new user
    new_user = {
        'id': len(users) + 1,
        'username': data['username'],
        'password': bcrypt.hashpw(data['password'].encode('utf-8'), bcrypt.gensalt())
    }
    users.append(new_user)
    
    # Create tokens
    access_token = create_access_token(identity=new_user['id'])
    refresh_token = create_refresh_token(identity=new_user['id'])
    
    return jsonify({
        'message': 'User created successfully',
        'access_token': access_token,
        'refresh_token': refresh_token
    }), 201
```

---

## 📊 Configuration Options

```python
# Token expiration times
app.config['JWT_ACCESS_TOKEN_EXPIRES'] = timedelta(minutes=15)  # Short
app.config['JWT_REFRESH_TOKEN_EXPIRES'] = timedelta(days=30)    # Long

# Secret key (use environment variable in production)
app.config['JWT_SECRET_KEY'] = os.environ.get('JWT_SECRET_KEY', 'fallback-key')

# Token location (headers, cookies, query string)
app.config['JWT_TOKEN_LOCATION'] = ['headers']
```

---

## 🚨 Error Handling

```python
# Handle expired tokens
@jwt.expired_token_loader
def expired_token_callback(jwt_header, jwt_payload):
    return jsonify({'error': 'Token has expired'}), 401

# Handle invalid tokens  
@jwt.invalid_token_loader
def invalid_token_callback(error):
    return jsonify({'error': 'Invalid token'}), 401

# Handle missing tokens
@jwt.unauthorized_loader
def missing_token_callback(error):
    return jsonify({'error': 'Authorization token required'}), 401
```

---

## ✅ Best Practices

### 1. **Short Access Tokens**
```python
# Good - 15 minutes
app.config['JWT_ACCESS_TOKEN_EXPIRES'] = timedelta(minutes=15)

# Bad - too long
app.config['JWT_ACCESS_TOKEN_EXPIRES'] = timedelta(days=1)
```

### 2. **Always Use Refresh Tokens**
```python
# Always provide both tokens
access_token = create_access_token(identity=user_id)
refresh_token = create_refresh_token(identity=user_id)
```

### 3. **Implement Logout**
```python
# Always implement token blacklisting for logout
@jwt.token_in_blocklist_loader
def check_if_token_revoked(jwt_header, jwt_payload):
    return jwt_payload['jti'] in blacklisted_tokens
```

---

## 🎯 Quick Reference

### Essential Functions
```python
# Create tokens
create_access_token(identity=user_id)
create_refresh_token(identity=user_id)

# Protect routes
@jwt_required()
@jwt_required(refresh=True)

# Get user info
get_jwt_identity()  # User ID
get_jwt()          # Full JWT data
```

### Token Flow
```
1. Login → Get access + refresh tokens
2. Use access token for API calls
3. When access expires → Use refresh to get new access
4. Logout → Blacklist tokens
```

---

## 🔥 Challenge Exercise

**Build a Simple Todo API:**
1. User registration and login
2. Create/read todos (protected routes)
3. Token refresh functionality
4. Logout with token blacklisting

**Requirements:**
- Use JWT-Extended for authentication
- Implement proper error handling
- Test all endpoints with curl

---

## 📚 Summary

✅ **JWT-Extended** simplifies JWT implementation  
✅ **Access & Refresh tokens** provide better security  
✅ **Token blacklisting** enables proper logout  
✅ **Built-in decorators** make route protection easy  
✅ **Error handling** is automatic and customizable

### Next: Celery Basics ⚡
In the next guide, we'll learn background task processing!

---

*📖 This guide is part of the Faculty Development Program series on Flask.*