# Flask Background Tasks - Quick Guide

## What This Does
Creates background tasks in Flask using Python threading - no Celery or Redis needed!

## How It Works
1. User starts a task → Gets immediate response with task ID
2. Task runs in background thread
3. User checks status using task ID
4. Gets result when task completes

## Core Code
```python
from flask import Flask, jsonify
import threading, time, uuid

app = Flask(__name__)
tasks = {}  # Store task results

def background_work(task_id, x, y):
    time.sleep(2)  # Simulate work
    tasks[task_id] = {'status': 'completed', 'result': x + y}

@app.route('/start/<int:x>/<int:y>')
def start_task(x, y):
    task_id = str(uuid.uuid4())
    tasks[task_id] = {'status': 'running'}
    
    thread = threading.Thread(target=background_work, args=(task_id, x, y))
    thread.start()
    
    return jsonify({'task_id': task_id, 'status': 'started'})

@app.route('/status/<task_id>')
def check_status(task_id):
    if task_id not in tasks:
        return jsonify({'error': 'Task not found'}), 404
    
    task = tasks[task_id]
    if task['status'] == 'completed':
        return jsonify({'status': 'done', 'result': task['result']})
    else:
        return jsonify({'status': 'running'})

if __name__ == '__main__':
    app.run(debug=True)
```

## Usage
```bash
# Run server
python app.py

# Start task
curl http://localhost:5000/start/10/5
# Returns: {"task_id": "abc-123", "status": "started"}

# Check status
curl http://localhost:5000/status/abc-123
# Returns: {"status": "done", "result": 15}
```

## Key Concepts
- **Threading**: Runs tasks in background without blocking
- **Task Storage**: Uses dictionary to track task status
- **UUID**: Creates unique task identifiers
- **Non-blocking**: User gets immediate response

## When to Use
✅ Simple apps, learning, prototypes  
❌ Production systems, high traffic

## Limitations
- Tasks lost on server restart
- Limited to single server
- No persistence or retry logic

That's it! 🚀