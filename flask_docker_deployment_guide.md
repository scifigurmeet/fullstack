# Flask Deployment with Docker + Gunicorn - Complete Beginner's Guide 🐳

## 🎯 Learning Objectives
By the end of this guide, you will:
- Understand what Docker and Gunicorn are
- Create a production-ready Flask app
- Build and run Docker containers
- Deploy your app for real users

---

## 📖 What is Docker & Gunicorn?

### Docker
**Docker** = Packages your app with everything it needs to run anywhere.

**Without Docker:**
```
Your Computer: ✅ Works
Friend's Computer: ❌ "It doesn't work on my machine!"
Server: ❌ Missing dependencies
```

**With Docker:**
```
Your Computer: ✅ Works
Friend's Computer: ✅ Works  
Server: ✅ Works
Anywhere: ✅ Works!
```

### Gunicorn
**Gunicorn** = A production web server for Flask apps.

**Development (Flask built-in server):**
```python
app.run(debug=True)  # Only for testing! ⚠️
```

**Production (Gunicorn):**
```bash
gunicorn app:app  # For real users! ✅
```

---

## 🚀 Step-by-Step Deployment

### Step 1: Create Simple Flask App

**File: `app.py`**
```python
from flask import Flask, jsonify

app = Flask(__name__)

@app.route('/')
def home():
    return jsonify({
        'message': 'Hello from Docker! 🐳',
        'status': 'running',
        'version': '1.0.0'
    })

@app.route('/health')
def health():
    return jsonify({
        'status': 'healthy',
        'service': 'flask-app'
    })

@app.route('/api/users')
def users():
    return jsonify({
        'users': [
            {'id': 1, 'name': 'John Doe'},
            {'id': 2, 'name': 'Jane Smith'}
        ]
    })

if __name__ == '__main__':
    # This only runs in development
    app.run(debug=True, host='0.0.0.0')
```

### Step 2: Create Requirements File

**File: `requirements.txt`**
```
Flask==2.3.3
gunicorn==21.2.0
```

### Step 3: Create Dockerfile

**File: `Dockerfile`**
```dockerfile
# Use Python base image
FROM python:3.9-slim

# Set working directory inside container
WORKDIR /app

# Copy requirements first (for better caching)
COPY requirements.txt .

# Install Python packages
RUN pip install --no-cache-dir -r requirements.txt

# Copy app code
COPY app.py .

# Expose port 8000
EXPOSE 8000

# Run with Gunicorn (production server)
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "app:app"]
```

---

## 🏗️ Project Structure
```
my-flask-app/
├── app.py
├── requirements.txt
├── Dockerfile
└── README.md
```

---

## 🐳 Docker Commands

### Step 1: Install Docker
Download from: https://www.docker.com/get-started

**Test Docker is working:**
```bash
docker --version
# Should show: Docker version X.X.X
```

### Step 2: Build Docker Image
```bash
# Build image with tag "my-flask-app"
docker build -t my-flask-app .
```

**What happens:**
- Docker reads the Dockerfile
- Downloads Python 3.9
- Installs Flask and Gunicorn
- Copies your app code
- Creates a ready-to-run container

### Step 3: Run Container
```bash
# Run container on port 8000
docker run -p 8000:8000 my-flask-app
```

**What happens:**
- Container starts
- Gunicorn serves your Flask app
- App is available at http://localhost:8000

### Step 4: Test Your App
Visit these URLs:
- http://localhost:8000 - Home page
- http://localhost:8000/health - Health check
- http://localhost:8000/api/users - API endpoint

---

## 🧪 Testing Commands

### Test with curl
```bash
# Home page
curl http://localhost:8000

# Health check
curl http://localhost:8000/health

# API endpoint
curl http://localhost:8000/api/users
```

### Check if container is running
```bash
# List running containers
docker ps

# Stop container
docker stop <container_id>

# Remove container
docker rm <container_id>
```

---

## 🔧 Useful Docker Commands

### Development Workflow
```bash
# Build image
docker build -t my-flask-app .

# Run container (remove after stop)
docker run --rm -p 8000:8000 my-flask-app

# Run in background
docker run -d -p 8000:8000 my-flask-app

# View logs
docker logs <container_id>

# Stop all containers
docker stop $(docker ps -q)
```

### Image Management
```bash
# List all images
docker images

# Remove image
docker rmi my-flask-app

# Remove unused images
docker image prune
```

---

## 🌐 Environment Variables

### Adding Environment Config

**Updated `app.py`:**
```python
import os
from flask import Flask, jsonify

app = Flask(__name__)

# Get environment variables
ENV = os.environ.get('FLASK_ENV', 'development')
DEBUG = os.environ.get('DEBUG', 'False').lower() == 'true'

@app.route('/')
def home():
    return jsonify({
        'message': 'Hello from Docker! 🐳',
        'environment': ENV,
        'debug': DEBUG,
        'version': '1.0.0'
    })

@app.route('/config')
def config():
    return jsonify({
        'environment': ENV,
        'debug': DEBUG,
        'database_url': os.environ.get('DATABASE_URL', 'Not set')
    })

if __name__ == '__main__':
    app.run(debug=DEBUG, host='0.0.0.0')
```

### Running with Environment Variables
```bash
# Set environment variables
docker run -p 8000:8000 \
  -e FLASK_ENV=production \
  -e DEBUG=false \
  -e DATABASE_URL=postgresql://user:pass@db:5432/mydb \
  my-flask-app
```

---

## 📦 Docker Compose (Optional)

**File: `docker-compose.yml`**
```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "8000:8000"
    environment:
      - FLASK_ENV=production
      - DEBUG=false
    restart: unless-stopped
```

### Using Docker Compose
```bash
# Start services
docker-compose up

# Start in background
docker-compose up -d

# Stop services
docker-compose down

# View logs
docker-compose logs
```

---

## 🚀 Production Deployment

### 1. Improved Dockerfile for Production

**File: `Dockerfile.prod`**
```dockerfile
FROM python:3.9-slim

# Create non-root user
RUN adduser --disabled-password --gecos '' appuser

WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy app
COPY app.py .

# Change ownership to app user
RUN chown -R appuser:appuser /app
USER appuser

EXPOSE 8000

# Production command with more workers
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "--workers", "4", "app:app"]
```

### 2. Production Build
```bash
# Build production image
docker build -f Dockerfile.prod -t my-flask-app:prod .

# Run production container
docker run -p 8000:8000 my-flask-app:prod
```

### 3. Health Check in Dockerfile
```dockerfile
# Add health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8000/health || exit 1
```

---

## 🌍 Deploy to Cloud

### 1. Deploy to Heroku
```bash
# Install Heroku CLI
npm install -g heroku

# Login to Heroku
heroku login

# Create app
heroku create my-flask-app

# Deploy with container
heroku container:push web
heroku container:release web

# Open app
heroku open
```

### 2. Deploy to DigitalOcean/AWS
```bash
# Build and tag image
docker build -t my-flask-app .
docker tag my-flask-app your-registry/my-flask-app

# Push to registry
docker push your-registry/my-flask-app

# Pull and run on server
docker pull your-registry/my-flask-app
docker run -d -p 80:8000 your-registry/my-flask-app
```

---

## 🛡️ Production Best Practices

### 1. Security
```dockerfile
# Don't run as root
USER appuser

# Use specific Python version
FROM python:3.9.18-slim

# Remove unnecessary packages
RUN apt-get purge -y --auto-remove
```

### 2. Performance
```bash
# Multiple workers
gunicorn --workers 4 --bind 0.0.0.0:8000 app:app

# Worker class for async
gunicorn --worker-class gevent --workers 4 app:app
```

### 3. Monitoring
```python
import logging

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

@app.route('/')
def home():
    logger.info("Home page accessed")
    return jsonify({'message': 'Hello!'})
```

---

## ✅ Quick Reference

### Essential Commands
```bash
# Build image
docker build -t my-app .

# Run container
docker run -p 8000:8000 my-app

# List containers
docker ps

# Stop container
docker stop <container_id>

# View logs
docker logs <container_id>
```

### File Structure
```
my-app/
├── app.py          # Flask application
├── requirements.txt # Python dependencies  
├── Dockerfile      # Docker instructions
└── docker-compose.yml # Optional: multi-service setup
```

### Production Checklist
- ✅ Use Gunicorn (not Flask dev server)
- ✅ Set environment variables
- ✅ Add health check endpoint
- ✅ Use non-root user in container
- ✅ Configure proper logging
- ✅ Add restart policy

---

## 🚨 Common Issues

### ❌ "Port already in use"
```bash
# Find what's using port 8000
lsof -i :8000

# Kill process
kill -9 <process_id>

# Or use different port
docker run -p 8001:8000 my-app
```

### ❌ "Container exits immediately"
```bash
# Check logs for errors
docker logs <container_id>

# Run interactively to debug
docker run -it my-app /bin/bash
```

### ❌ "Can't connect to app"
```bash
# Make sure container is running
docker ps

# Check if port is mapped correctly
docker run -p 8000:8000 my-app  # Host:Container
```

---

## 🔥 Simple Challenge

**Deploy a Todo API:**
1. Create Flask app with GET/POST /api/todos endpoints
2. Create Dockerfile
3. Build and run with Docker
4. Test all endpoints work
5. Add environment variable for app name

**Bonus:** Use docker-compose to add a Redis container for data storage.

---

## 📚 Summary

✅ **Docker** packages your app to run anywhere  
✅ **Gunicorn** serves Flask apps in production  
✅ **Dockerfile** defines how to build your container  
✅ **Environment variables** configure your app  
✅ **Health checks** monitor app status  
✅ **Multi-stage builds** optimize image size  

### 🎉 Congratulations!
You've completed the Flask Faculty Development Program series:
1. ✅ Flask Blueprints
2. ✅ Flask-RESTful APIs  
3. ✅ JWT Authentication
4. ✅ JWT-Extended Features
5. ✅ Celery Background Tasks
6. ✅ Docker Deployment

You now have the skills to build and deploy production Flask applications! 🚀

---

*📖 This completes the Faculty Development Program series on Flask.*