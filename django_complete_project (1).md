# 🐍 Super Simple Django - Just One List
## Your First Django App with ONE View and ONE Loop

---

## 🛠️ Project Setup

### Step 1: Create Project
```bash
# Create folder
mkdir simple_django
cd simple_django

# Create virtual environment
python -m venv venv

# Activate it
# Windows:
venv\Scripts\activate
# Mac/Linux:
source venv/bin/activate

# Install Django
pip install django
```

### Step 2: Create Django Project
```bash
# Create project
django-admin startproject mysite .

# Create app
python manage.py startapp myapp
```

---

## ⚙️ Settings

### 📄 mysite/settings.py
```python
# Find INSTALLED_APPS and add your app:
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'myapp',  # Add this line only
]
```

---

## 👁️ One Simple View

### 📄 myapp/views.py
```python
from django.shortcuts import render

def home(request):
    # Simple Python list
    fruits = ['Apple', 'Banana', 'Orange', 'Mango', 'Grapes']
    
    # Send list to template
    return render(request, 'home.html', {'fruits': fruits})
```

---

## 📄 One Simple Template

### Step 1: Create Template Folder
```bash
# Create templates folder
mkdir myapp/templates
```

### 📄 myapp/templates/home.html
```html
<!DOCTYPE html>
<html>
<head>
    <title>My Simple Django App</title>
    <style>
        body { font-family: Arial; padding: 20px; }
        h1 { color: blue; }
        li { 
            background: lightblue; 
            margin: 5px; 
            padding: 10px; 
            list-style: none; 
            border-radius: 5px;
        }
    </style>
</head>
<body>
    <h1>🍎 My Fruit List</h1>
    
    <p>Here are my fruits:</p>
    
    <ul>
        {% for fruit in fruits %}
            <li>{{ forloop.counter }}. {{ fruit }}</li>
        {% endfor %}
    </ul>
    
    <p>Total fruits: {{ fruits|length }}</p>
</body>
</html>
```

---

## 🌐 URL Setup

### 📄 mysite/urls.py
```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('myapp.urls')),
]
```

### 📄 myapp/urls.py
```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.home, name='home'),
]
```

---

## 🚀 Run the App

```bash
# Start server
python manage.py runserver

# Visit: http://127.0.0.1:8000/
```

---

## 📁 Final File Structure

```
simple_django/
├── venv/
├── mysite/
│   ├── __init__.py
│   ├── settings.py      # Added 'myapp'
│   ├── urls.py          # Added include
│   ├── wsgi.py
│   └── asgi.py
├── myapp/
│   ├── templates/
│   │   └── home.html    # Your template
│   ├── __init__.py
│   ├── views.py         # Your view
│   ├── urls.py          # Your URLs
│   └── (other files)
├── manage.py
└── db.sqlite3
```

---

## ✅ What You Get

**One webpage showing:**
- A blue title "🍎 My Fruit List"
- 5 fruits in colored boxes with numbers
- Total count of fruits
- All from one Python list!

**What the loop does:**
- `{% for fruit in fruits %}` - Loops through each fruit
- `{{ forloop.counter }}` - Shows 1, 2, 3, 4, 5
- `{{ fruit }}` - Shows Apple, Banana, etc.
- `{{ fruits|length }}` - Shows total count (5)

**That's it! Just:**
- ✅ 1 Python list
- ✅ 1 view function  
- ✅ 1 template with 1 loop
- ✅ Basic styling

**Total time: 10 minutes**
**Total code: ~50 lines**