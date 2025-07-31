# Celery with Flask - Complete Beginner's Guide ⚡

## 🎯 Learning Objectives
By the end of this guide, you will:
- Understand what Celery is and why it's needed
- Set up Celery with Flask (step by step)
- Create simple background tasks
- Check task status

---

## 📖 What is Celery?

**Celery** = Runs slow tasks in the background so users don't wait.

### The Problem
```python
@app.route('/send-email')
def send_email():
    time.sleep(5)  # Email takes 5 seconds
    return "Email sent!"  # User waits 5 seconds! 😴
```

### The Solution
```python
@app.route('/send-email')  
def send_email():
    send_email_task.delay()  # Runs in background
    return "Email sending!"  # User gets instant response! ⚡
```

### What You Need
1. **Flask App** - Your web application
2. **Redis** - Stores tasks (like a todo list)
3. **Celery Worker** - Runs the background tasks

---

## 🚀 Step-by-Step Setup

### Step 1: Install Everything
```bash
pip install flask celery redis
```

### Step 2: Install Redis (Task Storage)
**What is Redis?** Think of it as a simple database that stores your tasks.

```bash
# Windows (easiest way)
docker run -d -p 6379:6379 redis:alpine

# Mac
brew install redis
redis-server

# Ubuntu
sudo apt install redis-server
redis-server
```

**Test Redis is working:**
```bash
redis-cli ping
# Should return: PONG
```

---

## 🛠️ Simple Example

### Complete App (app.py)
```python
from flask import Flask, jsonify
from celery import Celery
import time

# Create Flask app
app = Flask(__name__)

# Tell Celery where Redis is
app.config['CELERY_BROKER_URL'] = 'redis://localhost:6379'
app.config['CELERY_RESULT_BACKEND'] = 'redis://localhost:6379'

# Create Celery
celery = Celery(app.name, broker=app.config['CELERY_BROKER_URL'])
celery.conf.update(app.config)

# Background task
@celery.task
def send_email_task(email):
    """This runs in background"""
    print(f"Sending email to {email}...")
    time.sleep(3)  # Pretend sending email takes 3 seconds
    print(f"Email sent to {email}!")
    return f"Email sent to {email}"

# Web routes
@app.route('/send-email/<email>')
def send_email(email):
    """User calls this - gets instant response"""
    task = send_email_task.delay(email)  # Start background task
    return jsonify({
        'message': 'Email is being sent!',
        'task_id': task.id
    })

@app.route('/check-task/<task_id>')
def check_task(task_id):
    """Check if task is done"""
    task = celery.AsyncResult(task_id)
    
    if task.state == 'PENDING':
        return jsonify({'status': 'Task is waiting...'})
    elif task.state == 'SUCCESS':
        return jsonify({
            'status': 'Task completed!',
            'result': task.result
        })
    else:
        return jsonify({'status': 'Task failed'})

@app.route('/')
def home():
    return jsonify({
        'message': 'Celery + Flask Demo',
        'try': [
            'GET /send-email/test@example.com',
            'GET /check-task/<task_id>'
        ]
    })

if __name__ == '__main__':
    print("🚀 Flask app starting...")
    print("📧 Try: http://localhost:5000/send-email/test@example.com")
    app.run(debug=True)
```

---

## 🏃‍♂️ Running Everything

### Terminal 1: Start Redis
```bash
redis-server
# Leave this running
```

### Terminal 2: Start Celery Worker
```bash
celery -A app.celery worker --loglevel=info
# Leave this running - it will show task progress
```

### Terminal 3: Start Flask App
```bash
python app.py
# Your web app is now running
```

---

## 🧪 Testing

### 1. Send Email (Background Task)
Visit: `http://localhost:5000/send-email/john@example.com`

**Response:**
```json
{
  "message": "Email is being sent!",
  "task_id": "abc-123-def"
}
```

### 2. Check Task Status
Visit: `http://localhost:5000/check-task/abc-123-def`

**Response (while running):**
```json
{
  "status": "Task is waiting..."
}
```

**Response (after completion):**
```json
{
  "status": "Task completed!",
  "result": "Email sent to john@example.com"
}
```

**Watch the Celery Worker terminal** - you'll see the task running!

---

## 🔧 More Examples

### Heavy Computation Task
```python
@celery.task
def calculate_numbers():
    """Heavy calculation in background"""
    print("Starting calculation...")
    total = 0
    for i in range(1000000):
        total += i
    print("Calculation done!")
    return f"Result: {total}"

@app.route('/calculate')
def start_calculation():
    task = calculate_numbers.delay()
    return jsonify({
        'message': 'Calculation started!',
        'task_id': task.id
    })
```

### File Processing Task
```python
@celery.task
def process_file(filename):
    """Process file in background"""
    print(f"Processing {filename}...")
    time.sleep(5)  # Simulate file processing
    print(f"File {filename} processed!")
    return f"File {filename} ready for download"

@app.route('/process/<filename>')
def process_file_route(filename):
    task = process_file.delay(filename)
    return jsonify({
        'message': f'Processing {filename}...',
        'task_id': task.id
    })
```

---

## 🛡️ Error Handling

### Simple Error Handling
```python
@celery.task
def risky_task():
    """Task that might fail"""
    try:
        # Something that might fail
        result = 10 / 0
        return result
    except Exception as e:
        return f"Error: {str(e)}"

@app.route('/risky')
def start_risky_task():
    task = risky_task.delay()
    return jsonify({'task_id': task.id})
```

---

## 📊 Understanding What Happens

### The Flow
```
1. User visits /send-email/john@example.com
2. Flask creates a task and puts it in Redis
3. Flask returns task_id immediately (user happy!)
4. Celery worker picks up task from Redis
5. Celery worker runs send_email_task()
6. User can check progress with task_id
```

### Why This is Great
- ✅ **User gets instant response** (no waiting)
- ✅ **Tasks run in background** (doesn't block the app)
- ✅ **Reliable** (tasks are saved in Redis)
- ✅ **Scalable** (add more workers if needed)

---

## 🚨 Common Issues

### ❌ "Connection refused" Error
**Problem:** Redis is not running
**Solution:** Start Redis first
```bash
redis-server
```

### ❌ Tasks Never Execute  
**Problem:** Celery worker not running
**Solution:** Start worker
```bash
celery -A app.celery worker --loglevel=info
```

### ❌ "Task is waiting..." Forever
**Problem:** Worker crashed or not connected
**Solution:** Restart worker, check Redis connection

---

## ✅ Simple Rules

### 1. **Always Keep These Running:**
```bash
Terminal 1: redis-server
Terminal 2: celery -A app.celery worker --loglevel=info  
Terminal 3: python app.py
```

### 2. **Task Names Should Be Clear:**
```python
# ✅ Good
@celery.task
def send_welcome_email(user_email):
    pass

# ❌ Bad  
@celery.task
def task1(data):
    pass
```

### 3. **Always Return Something:**
```python
@celery.task
def my_task():
    # Do work
    return "Task completed!"  # Always return result
```

---

## 🎯 Quick Reference

### Create Task
```python
@celery.task
def my_task(param):
    return "result"
```

### Start Task
```python
task = my_task.delay("value")
task_id = task.id
```

### Check Task
```python
task = celery.AsyncResult(task_id)
if task.state == 'SUCCESS':
    result = task.result
```

### Required Commands
```bash
redis-server                                 # Start Redis
celery -A app.celery worker --loglevel=info # Start worker  
python app.py                               # Start Flask
```

---

## 🔥 Simple Challenge

**Create a "Report Generator":**
1. Route: `/generate-report` - starts background task
2. Task takes 10 seconds to "generate report"
3. Route: `/report-status/<task_id>` - checks if done
4. Return "Report ready!" when complete

**Hint:** Use `time.sleep(10)` to simulate report generation.

---

## 📚 Summary

✅ **Celery** runs slow tasks in background  
✅ **Redis** stores the tasks  
✅ **Workers** do the actual work  
✅ **Users get instant responses** (no waiting!)  
✅ **Three terminals** needed: Redis, Worker, Flask  

### Next: Docker Deployment 🐳
In the final guide, we'll learn to deploy everything with Docker!

---

*📖 This guide is part of the Faculty Development Program series on Flask.*