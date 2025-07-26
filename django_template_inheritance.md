# Django Template Inheritance - Complete Beginner Guide

This guide will walk you through creating a basic Django project with template inheritance from scratch.

## Prerequisites

- Python installed on your system
- Basic command line knowledge

## Step 1: Install Django

Open your terminal/command prompt and run:

```bash
pip install django
```

## Step 2: Create a New Django Project

```bash
django-admin startproject myproject
cd myproject
```

## Step 3: Create a Django App

```bash
python manage.py startapp myapp
```

## Step 4: Register the App

Edit `myproject/settings.py` and add your app to INSTALLED_APPS:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'myapp',  # Add this line
]
```

## Step 5: Create Templates Directory

Create the following directory structure:

```
myproject/
├── myapp/
│   └── templates/
│       └── myapp/
│           ├── base.html
│           ├── home.html
│           └── about.html
└── myproject/
```

Create the directories:

```bash
mkdir -p myapp/templates/myapp
```

## Step 6: Create the Base Template

Create `myapp/templates/myapp/base.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}My Website{% endblock %}</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 20px;
            background-color: #f5f5f5;
        }
        .header {
            background-color: #333;
            color: white;
            padding: 20px;
            text-align: center;
            margin-bottom: 20px;
        }
        .nav {
            background-color: #666;
            padding: 10px;
            margin-bottom: 20px;
        }
        .nav a {
            color: white;
            text-decoration: none;
            margin-right: 20px;
        }
        .content {
            background-color: white;
            padding: 20px;
            border-radius: 5px;
        }
        .footer {
            background-color: #333;
            color: white;
            text-align: center;
            padding: 10px;
            margin-top: 20px;
        }
    </style>
</head>
<body>
    <div class="header">
        <h1>My Website</h1>
    </div>
    
    <div class="nav">
        <a href="{% url 'home' %}">Home</a>
        <a href="{% url 'about' %}">About</a>
    </div>
    
    <div class="content">
        {% block content %}
        <p>Welcome to my website!</p>
        {% endblock %}
    </div>
    
    <div class="footer">
        <p>&copy; 2025 My Website. All rights reserved.</p>
    </div>
</body>
</html>
```

## Step 7: Create Child Templates

Create `myapp/templates/myapp/home.html`:

```html
{% extends 'myapp/base.html' %}

{% block title %}Home - My Website{% endblock %}

{% block content %}
    <h2>Welcome to the Home Page</h2>
    <p>This is the home page content. It inherits the layout from the base template.</p>
    <p>Notice how the header, navigation, and footer are automatically included!</p>
{% endblock %}
```

Create `myapp/templates/myapp/about.html`:

```html
{% extends 'myapp/base.html' %}

{% block title %}About - My Website{% endblock %}

{% block content %}
    <h2>About Us</h2>
    <p>This is the about page content. It also inherits from the same base template.</p>
    <p>Template inheritance allows us to:</p>
    <ul>
        <li>Reuse common HTML structure</li>
        <li>Maintain consistent design</li>
        <li>Reduce code duplication</li>
        <li>Make updates easier</li>
    </ul>
{% endblock %}
```

## Step 8: Create Views

Edit `myapp/views.py`:

```python
from django.shortcuts import render

def home(request):
    return render(request, 'myapp/home.html')

def about(request):
    return render(request, 'myapp/about.html')
```

## Step 9: Create URL Configuration

Create `myapp/urls.py`:

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.home, name='home'),
    path('about/', views.about, name='about'),
]
```

## Step 10: Include App URLs in Project

Edit `myproject/urls.py`:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('myapp.urls')),
]
```

## Step 11: Run the Development Server

```bash
python manage.py runserver
```

## Step 12: View Your Website

Open your web browser and go to:
- Home page: http://127.0.0.1:8000/
- About page: http://127.0.0.1:8000/about/

## How Template Inheritance Works

### Key Concepts:

1. **Base Template (`base.html`)**: Contains the common HTML structure with `{% block %}` tags that define areas where child templates can insert content.

2. **Child Templates (`home.html`, `about.html`)**: Use `{% extends %}` to inherit from the base template and `{% block %}` to override specific sections.

3. **Block Tags**: Define sections that can be overridden:
   - `{% block title %}` - Page title
   - `{% block content %}` - Main content area

### Template Inheritance Flow:

1. Child template declares `{% extends 'myapp/base.html' %}`
2. Django loads the base template
3. Django replaces each `{% block %}` in the base template with the corresponding block from the child template
4. If a block isn't defined in the child template, the default content from the base template is used

## Project Structure Summary

```
myproject/
├── manage.py
├── myproject/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
└── myapp/
    ├── __init__.py
    ├── admin.py
    ├── apps.py
    ├── models.py
    ├── tests.py
    ├── views.py
    ├── urls.py
    └── templates/
        └── myapp/
            ├── base.html
            ├── home.html
            └── about.html
```

You now have a working Django application with template inheritance! Both pages share the same header, navigation, and footer, but have different content in the main area.