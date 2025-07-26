# Django Deployment with Docker, Gunicorn, Nginx - Simple Guide

## What We're Building

```
Internet → Nginx (web server) → Gunicorn (app server) → Django (your app)
```

**Why each piece?**
- **Django**: Your web application
- **Gunicorn**: Runs Django in production (replaces `python manage.py runserver`)
- **Nginx**: Handles static files and forwards requests to Gunicorn
- **Docker**: Packages everything together for easy deployment

## Step 1: Create Simple Django Project

```bash
# Create project
mkdir simple_deploy
cd simple_deploy
django-admin startproject myapp .
```

**Project structure:**
```
simple_deploy/
├── myapp/
├── manage.py
├── requirements.txt
├── Dockerfile
├── docker-compose.yml
└── nginx.conf
```

## Step 2: Create Requirements File

```txt
# requirements.txt
Django==4.2.7
gunicorn==21.2.0
```

**What**: Python packages needed
**Why**: Docker needs to know what to install

## Step 3: Update Django Settings

```python
# myapp/settings.py - Add these lines at the bottom
import os

# Allow Docker containers to connect
ALLOWED_HOSTS = ['localhost', '127.0.0.1', 'web']

# Static files for production
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
```

**What**: Configuration changes
**Why**: 
- `ALLOWED_HOSTS`: Security - only these domains can access your app
- `STATIC_ROOT`: Where to put CSS/JS files for production

## Step 4: Create Simple View

```python
# myapp/urls.py
from django.contrib import admin
from django.urls import path
from django.http import HttpResponse

def home(request):
    return HttpResponse("<h1>🚀 Django + Docker + Nginx Works!</h1>")

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', home),
]
```

**What**: A simple homepage
**Why**: To test if everything works

## Step 5: Create Dockerfile

```dockerfile
# Dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

RUN python manage.py collectstatic --noinput

EXPOSE 8000

CMD ["gunicorn", "--bind", "0.0.0.0:8000", "myapp.wsgi:application"]
```

**What each line does:**
- `FROM python:3.11-slim`: Start with Python installed
- `WORKDIR /app`: Set working directory inside container
- `COPY requirements.txt .`: Copy requirements file
- `RUN pip install -r requirements.txt`: Install Python packages
- `COPY . .`: Copy all project files
- `RUN python manage.py collectstatic`: Collect CSS/JS files
- `EXPOSE 8000`: Tell Docker this app uses port 8000
- `CMD ["gunicorn"...]`: Start Gunicorn server instead of Django dev server

## Step 6: Create Nginx Config

```nginx
# nginx.conf
events {}

http {
    upstream app {
        server web:8000;
    }

    server {
        listen 80;

        location /static/ {
            alias /app/staticfiles/;
        }

        location / {
            proxy_pass http://app;
            proxy_set_header Host $host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
}
```

**What each part does:**
- `upstream app { server web:8000; }`: Define where Django app runs
- `listen 80`: Nginx listens on port 80 (web traffic)
- `location /static/`: Serve CSS/JS files directly (faster)
- `location /`: Forward everything else to Django app
- `proxy_set_header`: Pass client info to Django

## Step 7: Create Docker Compose

```yaml
# docker-compose.yml
version: '3.8'

services:
  web:
    build: .
    volumes:
      - static_volume:/app/staticfiles

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - static_volume:/app/staticfiles
    depends_on:
      - web

volumes:
  static_volume:
```

**What each part does:**
- `web:` - Your Django app container
- `build: .` - Build from Dockerfile in current directory
- `volumes:` - Share static files between containers
- `nginx:` - Web server container
- `ports: "80:80"` - Make port 80 accessible from outside
- `depends_on:` - Start web container first

## Step 8: Deploy

```bash
# Build and start containers
docker-compose up --build

# Visit http://localhost
# You should see: "🚀 Django + Docker + Nginx Works!"
```

**What happens:**
1. Docker builds your Django app
2. Starts Django with Gunicorn on port 8000
3. Starts Nginx on port 80
4. Nginx forwards requests to Django

## Step 9: Common Commands

```bash
# Stop containers
docker-compose down

# Rebuild after changes
docker-compose up --build

# View logs
docker-compose logs web    # Django logs
docker-compose logs nginx  # Nginx logs

# Run Django commands
docker-compose exec web python manage.py migrate
docker-compose exec web python manage.py createsuperuser
```

## Step 10: Production Additions

For real production, add these files:

**.env file** (keep secrets safe):
```env
SECRET_KEY=your-very-secret-key-here
DEBUG=False
```

**Update settings.py**:
```python
import os
from decouple import config

SECRET_KEY = config('SECRET_KEY')
DEBUG = config('DEBUG', default=False, cast=bool)
```

**Add to requirements.txt**:
```txt
python-decouple==3.8
```

## Troubleshooting

**Container won't start?**
```bash
docker-compose logs web
```

**Can't connect to app?**
- Check `ALLOWED_HOSTS` in settings.py
- Make sure ports aren't blocked

**Static files not loading?**
```bash
docker-compose exec web python manage.py collectstatic
```

## Summary

**What you built:**
1. **Django app** running with **Gunicorn** (production server)
2. **Nginx** serving static files and forwarding requests
3. **Docker** containers that work together
4. **Easy deployment** with one command

**Why this setup:**
- **Nginx**: Fast static file serving, handles many connections
- **Gunicorn**: Proper Python WSGI server for production
- **Docker**: Same environment everywhere (dev/staging/production)
- **Scalable**: Can add more web containers easily

**Next steps:**
- Add PostgreSQL database
- Add SSL certificates (HTTPS)
- Add monitoring and logging
- Deploy to cloud (AWS, DigitalOcean, etc.)