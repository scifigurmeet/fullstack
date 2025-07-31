# Flask + Celery Study Guide

## Overview

This guide covers building a simple Flask web application with Celery for background task processing. The example creates an addition service that processes calculations asynchronously.

## Key Concepts

### Flask
- **Web Framework**: Python micro-framework for building web applications
- **Routes**: URL endpoints that handle HTTP requests
- **JSON API**: Returns data in JSON format for API consumption

### Celery
- **Task Queue**: Distributed task queue system for handling background jobs
- **Broker**: Message broker (Redis) that stores and delivers tasks
- **Backend**: Storage system for task results
- **Worker**: Process that executes tasks from the queue

### Redis
- **Message Broker**: Stores tasks waiting to be processed
- **Result Backend**: Stores task results and status
- **In-Memory Database**: Fast key-value storage system

## Architecture Flow

```
Client Request → Flask App → Queue Task → Redis → Celery Worker → Process Task → Store Result → Client Retrieves Result
```

1. **Client sends POST request** to `/add` endpoint with numbers
2. **Flask app receives request** and validates data
3. **Task queued** using `add_task.delay(a, b)`
4. **Redis stores task** in message queue
5. **Celery worker picks up task** from queue
6. **Worker executes task** with time delay simulation
7. **Result stored** in Redis backend
8. **Client polls** `/result/<task_id>` to get status/result

## Code Breakdown

### Flask Application Setup
```python
app = Flask(__name__)
```
- Creates Flask application instance
- Handles HTTP requests and responses

### Celery Configuration
```python
celery = Celery('app', 
                broker='redis://localhost:6379',
                backend='redis://localhost:6379')
```
- **app**: Application name identifier
- **broker**: Where tasks are queued (Redis)
- **backend**: Where results are stored (Redis)

### Windows-Specific Settings
```python
celery.conf.update(
    task_serializer='json',      # Serialize tasks as JSON
    accept_content=['json'],     # Only accept JSON content
    result_serializer='json',    # Serialize results as JSON
    timezone='UTC',              # Use UTC timezone
    enable_utc=True,            # Enable UTC
    worker_pool='solo'          # Use solo pool for Windows
)
```
- **solo pool**: Avoids Windows process forking issues
- **JSON serialization**: Ensures data compatibility

### Task Definition
```python
@celery.task
def add_task(a, b):
    time.sleep(3)  # Simulate work
    return a + b
```
- **@celery.task**: Decorator makes function a Celery task
- **time.sleep(3)**: Simulates long-running operation
- **return**: Task result stored in backend

### API Endpoints

#### POST /add
```python
@app.route('/add', methods=['POST'])
def add():
    data = request.json
    task = add_task.delay(a, b)
    return jsonify({'task_id': task.id})
```
- **request.json**: Parses JSON request body
- **delay()**: Queues task for background execution
- **task.id**: Unique identifier for tracking

#### GET /result/<task_id>
```python
@app.route('/result/<task_id>')
def result(task_id):
    task = add_task.AsyncResult(task_id)
    if task.state == 'PENDING':
        return jsonify({'status': 'PENDING', 'result': None})
    elif task.state == 'SUCCESS':
        return jsonify({'status': 'SUCCESS', 'result': task.result})
    else:
        return jsonify({'status': task.state, 'result': str(task.result)})
```
- **AsyncResult**: Gets task status and result
- **task.state**: Current task status (PENDING, SUCCESS, FAILURE)
- **task.result**: Task return value or exception

## Task States

| State | Description |
|-------|-------------|
| **PENDING** | Task waiting to be processed |
| **STARTED** | Task has begun execution |
| **SUCCESS** | Task completed successfully |
| **FAILURE** | Task failed with an error |
| **RETRY** | Task is being retried |
| **REVOKED** | Task was cancelled |

## Setup Commands

### 1. Install Dependencies
```bash
pip install flask celery redis
```

### 2. Start Redis Server
```bash
redis-server
```

### 3. Start Celery Worker (Windows)
```bash
celery -A app.celery worker --pool=solo --loglevel=info
```

### 4. Run Flask Application
```bash
python app.py
```

## Testing the Application

### Start a Task
```bash
curl -X POST http://localhost:5000/add \
  -H "Content-Type: application/json" \
  -d '{"a": 5, "b": 3}'
```

**Response:**
```json
{
  "task_id": "6a299afa-830c-4ad0-a033-6a240ba1db96"
}
```

### Check Task Status (Immediately)
```bash
curl http://localhost:5000/result/6a299afa-830c-4ad0-a033-6a240ba1db96
```

**Response:**
```json
{
  "status": "PENDING",
  "result": null
}
```

### Check Task Result (After 3+ seconds)
```bash
curl http://localhost:5000/result/6a299afa-830c-4ad0-a033-6a240ba1db96
```

**Response:**
```json
{
  "status": "SUCCESS",
  "result": 8
}
```

## Common Issues & Solutions

### Windows Process Forking Error
**Error:** `ValueError: not enough values to unpack (expected 3, got 0)`

**Solution:** Use solo worker pool:
```bash
celery -A app.celery worker --pool=solo
```

### JSON Serialization Error
**Error:** `Object of type ValueError is not JSON serializable`

**Solution:** Convert exceptions to strings:
```python
return jsonify({'status': task.state, 'result': str(task.result)})
```

### Connection Refused
**Error:** `ConnectionError: Error connecting to Redis`

**Solution:** Ensure Redis server is running:
```bash
redis-server
```

## Key Benefits

### Asynchronous Processing
- **Non-blocking**: Web requests don't wait for long tasks
- **Scalable**: Multiple workers can process tasks in parallel
- **Reliable**: Tasks persist in queue even if worker restarts

### Separation of Concerns
- **Web Layer**: Handles HTTP requests/responses
- **Task Layer**: Processes background jobs
- **Storage Layer**: Manages data persistence

### User Experience
- **Immediate Response**: Users get task ID instantly
- **Progress Tracking**: Can check task status anytime
- **Error Handling**: Failed tasks return error messages

## Production Considerations

### Monitoring
- Use `flower` for Celery monitoring: `pip install flower`
- Monitor Redis memory usage and connections
- Log task execution times and failure rates

### Scaling
- Run multiple Celery workers for parallel processing
- Use different queues for different task types
- Configure Redis for persistence and clustering

### Security
- Secure Redis with authentication
- Validate all input data thoroughly
- Use HTTPS for production APIs

## Extended Learning

### Advanced Celery Features
- **Periodic Tasks**: Schedule recurring jobs
- **Task Routing**: Send tasks to specific workers
- **Callbacks**: Chain tasks together
- **Error Handling**: Retry failed tasks automatically

### Alternative Brokers
- **RabbitMQ**: More features but complex setup
- **Amazon SQS**: Cloud-based message queue
- **Database**: Use Django ORM as broker

### Performance Optimization
- **Connection Pooling**: Reuse Redis connections
- **Batch Processing**: Process multiple items per task
- **Result Compression**: Reduce memory usage for large results