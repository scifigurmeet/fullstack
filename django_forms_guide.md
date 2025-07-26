# Django Forms - Step by Step Guide

Let's learn Django forms by starting simple and gradually adding features.

## Initial Setup

```bash
django-admin startproject simpleforms
cd simpleforms
python manage.py startapp myapp
```

Add to `simpleforms/settings.py`:
```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'myapp',  # Add this
]
```

Create templates directory:
```bash
mkdir -p myapp/templates/myapp
```

---

## Step 1: Basic HTML Form (No Django Forms Yet)

Let's start with a simple HTML form to understand the basics.

### Create `myapp/templates/myapp/simple.html`:
```html
<!DOCTYPE html>
<html>
<head>
    <title>Simple Form</title>
    <style>
        body { font-family: Arial; padding: 20px; }
        input, button { padding: 8px; margin: 5px; }
        .result { background: #e8f5e9; padding: 10px; margin: 10px 0; }
    </style>
</head>
<body>
    <h2>Step 1: Basic HTML Form</h2>
    
    <form method="GET">
        <input type="text" name="username" placeholder="Enter your name">
        <button type="submit">Submit</button>
    </form>
    
    {% if request.GET.username %}
        <div class="result">
            Hello {{ request.GET.username }}!
        </div>
    {% endif %}
    
    <p><strong>What's happening:</strong></p>
    <ul>
        <li>Form uses GET method - data appears in URL</li>
        <li>No Django forms yet, just plain HTML</li>
        <li>Data accessed with request.GET.username</li>
    </ul>
</body>
</html>
```

### Create `myapp/views.py`:
```python
from django.shortcuts import render

def simple_form(request):
    return render(request, 'myapp/simple.html')
```

### Create `myapp/urls.py`:
```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.simple_form, name='simple'),
]
```

### Update `simpleforms/urls.py`:
```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('myapp.urls')),
]
```

**Run and test:**
```bash
python manage.py runserver
```
Visit http://127.0.0.1:8000/ and try the form!

---

## Step 2: Using Django Forms

Now let's use Django's form system instead of plain HTML.

### Create `myapp/forms.py`:
```python
from django import forms

class SimpleForm(forms.Form):
    username = forms.CharField(max_length=50, label='Your Name')
```

### Update `myapp/views.py`:
```python
from django.shortcuts import render
from .forms import SimpleForm

def simple_form(request):
    form = SimpleForm()
    message = None
    
    if request.method == 'GET' and 'username' in request.GET:
        form = SimpleForm(request.GET)
        if form.is_valid():
            message = f"Hello {form.cleaned_data['username']}!"
    
    return render(request, 'myapp/simple.html', {
        'form': form,
        'message': message
    })
```

### Update `myapp/templates/myapp/simple.html`:
```html
<!DOCTYPE html>
<html>
<head>
    <title>Django Form</title>
    <style>
        body { font-family: Arial; padding: 20px; }
        input, button { padding: 8px; margin: 5px; }
        .result { background: #e8f5e9; padding: 10px; margin: 10px 0; }
    </style>
</head>
<body>
    <h2>Step 2: Django Forms</h2>
    
    <form method="GET">
        {{ form.username.label_tag }}
        {{ form.username }}
        <button type="submit">Submit</button>
    </form>
    
    {% if message %}
        <div class="result">{{ message }}</div>
    {% endif %}
    
    <p><strong>What's new:</strong></p>
    <ul>
        <li>Using Django Form class instead of plain HTML</li>
        <li>form.is_valid() checks if data is correct</li>
        <li>form.cleaned_data gives us clean, validated data</li>
    </ul>
</body>
</html>
```

**Test it:** The form now uses Django's form system!

---

## Step 3: POST Method and CSRF

Let's switch to POST method and learn about CSRF protection.

### Update `myapp/views.py`:
```python
from django.shortcuts import render
from .forms import SimpleForm

def simple_form(request):
    form = SimpleForm()
    message = None
    
    if request.method == 'POST':
        form = SimpleForm(request.POST)
        if form.is_valid():
            message = f"Hello {form.cleaned_data['username']}!"
    
    return render(request, 'myapp/simple.html', {
        'form': form,
        'message': message
    })
```

### Update `myapp/templates/myapp/simple.html`:
```html
<!DOCTYPE html>
<html>
<head>
    <title>POST Form with CSRF</title>
    <style>
        body { font-family: Arial; padding: 20px; }
        input, button { padding: 8px; margin: 5px; }
        .result { background: #e8f5e9; padding: 10px; margin: 10px 0; }
    </style>
</head>
<body>
    <h2>Step 3: POST Method + CSRF Protection</h2>
    
    <form method="POST">
        {% csrf_token %}
        {{ form.username.label_tag }}
        {{ form.username }}
        <button type="submit">Submit</button>
    </form>
    
    {% if message %}
        <div class="result">{{ message }}</div>
    {% endif %}
    
    <p><strong>What's new:</strong></p>
    <ul>
        <li>Changed to POST method - data not visible in URL</li>
        <li>Added {% verbatim %}{% csrf_token %}{% endverbatim %} - protects against attacks</li>
        <li>POST is better for forms that change data</li>
    </ul>
</body>
</html>
```

**Test it:** Notice data no longer appears in URL, and CSRF token protects the form.

---

## Step 4: Form Validation

Let's add validation to make our form smarter.

### Update `myapp/forms.py`:
```python
from django import forms

class SimpleForm(forms.Form):
    username = forms.CharField(
        max_length=50, 
        label='Your Name',
        min_length=2,
        help_text='Enter at least 2 characters'
    )
    
    def clean_username(self):
        username = self.cleaned_data.get('username')
        if username and username.lower() == 'admin':
            raise forms.ValidationError("Username 'admin' is not allowed!")
        return username
```

### Update template to show errors:
```html
<!DOCTYPE html>
<html>
<head>
    <title>Form with Validation</title>
    <style>
        body { font-family: Arial; padding: 20px; }
        input, button { padding: 8px; margin: 5px; }
        .result { background: #e8f5e9; padding: 10px; margin: 10px 0; }
        .error { color: red; font-size: 12px; }
        .helptext { color: gray; font-size: 12px; }
    </style>
</head>
<body>
    <h2>Step 4: Form Validation</h2>
    
    <form method="POST">
        {% csrf_token %}
        
        <div>
            {{ form.username.label_tag }}
            {{ form.username }}
            {% if form.username.help_text %}
                <div class="helptext">{{ form.username.help_text }}</div>
            {% endif %}
            {% if form.username.errors %}
                <div class="error">{{ form.username.errors }}</div>
            {% endif %}
        </div>
        
        <button type="submit">Submit</button>
    </form>
    
    {% if message %}
        <div class="result">{{ message }}</div>
    {% endif %}
    
    <p><strong>What's new:</strong></p>
    <ul>
        <li>Added min_length validation</li>
        <li>Custom validation: 'admin' username not allowed</li>
        <li>Help text guides users</li>
        <li>Error messages show validation problems</li>
    </ul>
    
    <p><strong>Try:</strong> Enter 'admin', or just one character to see validation!</p>
</body>
</html>
```

**Test it:** Try entering "admin" or single character to see validation in action!

---

## Step 5: POST-Redirect-GET Pattern

Let's prevent duplicate submissions by redirecting after successful POST.

### Update `myapp/views.py`:
```python
from django.shortcuts import render, redirect
from django.contrib import messages
from .forms import SimpleForm

def simple_form(request):
    if request.method == 'POST':
        form = SimpleForm(request.POST)
        if form.is_valid():
            username = form.cleaned_data['username']
            messages.success(request, f"Hello {username}! Form submitted successfully.")
            return redirect('simple')  # Redirect after POST
    else:
        form = SimpleForm()
    
    return render(request, 'myapp/simple.html', {'form': form})
```

### Update template to show messages:
```html
<!DOCTYPE html>
<html>
<head>
    <title>POST-Redirect-GET</title>
    <style>
        body { font-family: Arial; padding: 20px; }
        input, button { padding: 8px; margin: 5px; }
        .messages { background: #e8f5e9; padding: 10px; margin: 10px 0; }
        .error { color: red; font-size: 12px; }
        .helptext { color: gray; font-size: 12px; }
    </style>
</head>
<body>
    <h2>Step 5: POST-Redirect-GET Pattern</h2>
    
    {% if messages %}
        {% for message in messages %}
            <div class="messages">{{ message }}</div>
        {% endfor %}
    {% endif %}
    
    <form method="POST">
        {% csrf_token %}
        
        <div>
            {{ form.username.label_tag }}
            {{ form.username }}
            {% if form.username.help_text %}
                <div class="helptext">{{ form.username.help_text }}</div>
            {% endif %}
            {% if form.username.errors %}
                <div class="error">{{ form.username.errors }}</div>
            {% endif %}
        </div>
        
        <button type="submit">Submit</button>
    </form>
    
    <p><strong>What's new:</strong></p>
    <ul>
        <li>After successful POST, redirects to GET request</li>
        <li>Uses Django messages to show success</li>
        <li>Prevents duplicate submissions on browser refresh</li>
        <li>Form is cleared after successful submission</li>
    </ul>
    
    <p><strong>Try:</strong> Submit form, then refresh page - no duplicate submission!</p>
</body>
</html>
```

---

## Step 6: Working with Models

Let's save form data to database using models.

### Create `myapp/models.py`:
```python
from django.db import models

class Contact(models.Model):
    name = models.CharField(max_length=50)
    created_at = models.DateTimeField(auto_now_add=True)
    
    def __str__(self):
        return self.name
```

### Run migrations:
```bash
python manage.py makemigrations
python manage.py migrate
```

### Update `myapp/forms.py`:
```python
from django import forms
from .models import Contact

class SimpleForm(forms.ModelForm):
    class Meta:
        model = Contact
        fields = ['name']
        labels = {'name': 'Your Name'}
        help_texts = {'name': 'Enter at least 2 characters'}
    
    def clean_name(self):
        name = self.cleaned_data.get('name')
        if name and len(name) < 2:
            raise forms.ValidationError("Name must be at least 2 characters!")
        if name and name.lower() == 'admin':
            raise forms.ValidationError("Username 'admin' is not allowed!")
        return name
```

### Update `myapp/views.py`:
```python
from django.shortcuts import render, redirect
from django.contrib import messages
from .forms import SimpleForm
from .models import Contact

def simple_form(request):
    if request.method == 'POST':
        form = SimpleForm(request.POST)
        if form.is_valid():
            contact = form.save()  # Save to database
            messages.success(request, f"Hello {contact.name}! Your data has been saved.")
            return redirect('simple')
    else:
        form = SimpleForm()
    
    # Show recent contacts
    recent_contacts = Contact.objects.all().order_by('-created_at')[:5]
    
    return render(request, 'myapp/simple.html', {
        'form': form,
        'recent_contacts': recent_contacts
    })
```

### Final template:
```html
<!DOCTYPE html>
<html>
<head>
    <title>Forms with Database</title>
    <style>
        body { font-family: Arial; padding: 20px; }
        input, button { padding: 8px; margin: 5px; }
        .messages { background: #e8f5e9; padding: 10px; margin: 10px 0; }
        .error { color: red; font-size: 12px; }
        .helptext { color: gray; font-size: 12px; }
        .contacts { background: #f5f5f5; padding: 15px; margin: 15px 0; }
    </style>
</head>
<body>
    <h2>Step 6: Forms with Database Storage</h2>
    
    {% if messages %}
        {% for message in messages %}
            <div class="messages">{{ message }}</div>
        {% endfor %}
    {% endif %}
    
    <form method="POST">
        {% csrf_token %}
        
        <div>
            {{ form.name.label_tag }}
            {{ form.name }}
            {% if form.name.help_text %}
                <div class="helptext">{{ form.name.help_text }}</div>
            {% endif %}
            {% if form.name.errors %}
                <div class="error">{{ form.name.errors }}</div>
            {% endif %}
        </div>
        
        <button type="submit">Submit & Save</button>
    </form>
    
    {% if recent_contacts %}
        <div class="contacts">
            <h3>Recent Contacts:</h3>
            {% for contact in recent_contacts %}
                <p>{{ contact.name }} - {{ contact.created_at|date:"M d, Y H:i" }}</p>
            {% endfor %}
        </div>
    {% endif %}
    
    <p><strong>What's new:</strong></p>
    <ul>
        <li>Using ModelForm instead of regular Form</li>
        <li>Data is saved to database with form.save()</li>
        <li>Shows recent contacts from database</li>
        <li>Automatic validation from model fields</li>
    </ul>
</body>
</html>
```

---

## Summary

You've learned Django forms step by step:

1. **Basic HTML Form** - Understanding GET requests
2. **Django Forms** - Using Form classes and validation
3. **POST + CSRF** - Secure form submission
4. **Validation** - Custom validation rules
5. **POST-Redirect-GET** - Preventing duplicate submissions
6. **Model Forms** - Saving data to database

Each step built on the previous one, making it easy to understand how Django forms work!

**Next Steps:** Try adding more fields, different field types (email, textarea, choices), or create multiple forms.