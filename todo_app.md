# **To-Do App with CRUD, User Login, and Undo Feature**

## **Project Overview**
In this project, students will build a Django-based **To-Do App** that allows users to **create, read, update, and delete (CRUD) tasks**. The app will include:
- [x] **User authentication** (login, logout, registration).
- [x] **A decent UI** using Bootstrap for styling.
- [x] **Undo delete functionality** using Django sessions.

This project reinforces **Django fundamentals**, including models, views, forms, templates, authentication, and session management.

## **📌 Step 1: Set Up Your Django Project**
### **1.1. Create a Django Project and App**
Run the following commands:
```bash
django-admin startproject todo_project
cd todo_project
django-admin startapp todo
```
Add `todo` to `INSTALLED_APPS` in `settings.py`:
```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'todo.apps.TodoConfig',
]
```

### **1.2. Set Up Static & Media Files**
Add the following to `settings.py`:
```python
STATIC_URL = '/static/'
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'
```

## **📌 Step 2: Create the To-Do Model**
In `models.py` inside the `todo` app:
```python
from django.db import models
from django.contrib.auth.models import User

class Task(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE)  # User-specific tasks
    title = models.CharField(max_length=255)
    description = models.TextField(blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    completed = models.BooleanField(default=False)

    def __str__(self):
        return self.title
```
Run migrations:
```bash
python manage.py makemigrations
python manage.py migrate
```

## **📌 Step 3: Implement User Authentication**
Django provides built-in authentication views.
Add this to `urls.py` in `todo_project`:
```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('accounts/', include('django.contrib.auth.urls')),  # Includes login/logout
    path('', include('todo.urls')),  # Main app
]
```

Create a `templates/registration/login.html` file:
```html
{% extends 'base.html' %}
{% block content %}
<h2>Login</h2>
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Login</button>
</form>
{% endblock %}
```
Resources:
- Django Authentication: [https://docs.djangoproject.com/en/stable/topics/auth/default/](https://docs.djangoproject.com/en/stable/topics/auth/default/)

## **📌 Step 4: Implement CRUD Operations**
### **4.1. Create Task Forms**
In `forms.py`:
```python
from django import forms
from .models import Task

class TaskForm(forms.ModelForm):
    class Meta:
        model = Task
        fields = ['title', 'description', 'completed']
```

### **4.2. Create Views**
In `views.py`:
```python
from django.shortcuts import render, redirect, get_object_or_404
from django.contrib.auth.decorators import login_required
from .models import Task
from .forms import TaskForm

@login_required
def task_list(request):
    tasks = Task.objects.filter(user=request.user)
    return render(request, 'todo/task_list.html', {'tasks': tasks})

@login_required
def task_create(request):
    if request.method == 'POST':
        form = TaskForm(request.POST)
        if form.is_valid():
            task = form.save(commit=False)
            task.user = request.user
            task.save()
            return redirect('task_list')
    else:
        form = TaskForm()
    return render(request, 'todo/task_form.html', {'form': form})

@login_required
def task_edit(request, task_id):
    task = get_object_or_404(Task, id=task_id, user=request.user)
    if request.method == 'POST':
        form = TaskForm(request.POST, instance=task)
        if form.is_valid():
            form.save()
            return redirect('task_list')
    else:
        form = TaskForm(instance=task)
    return render(request, 'todo/task_form.html', {'form': form})

@login_required
def task_delete(request, task_id):
    task = get_object_or_404(Task, id=task_id, user=request.user)
    request.session['undo_task'] = {'id': task.id, 'title': task.title, 'description': task.description}
    task.delete()
    return redirect('task_list')

@login_required
def task_undo(request):
    undo_task = request.session.pop('undo_task', None)
    if undo_task:
        Task.objects.create(user=request.user, title=undo_task['title'], description=undo_task['description'])
    return redirect('task_list')
```

### **4.3. Define URLs**
In `urls.py`:
```python
from django.urls import path
from .views import task_list, task_create, task_edit, task_delete, task_undo

urlpatterns = [
    path('', task_list, name='task_list'),
    path('new/', task_create, name='task_create'),
    path('<int:task_id>/edit/', task_edit, name='task_edit'),
    path('<int:task_id>/delete/', task_delete, name='task_delete'),
    path('undo/', task_undo, name='task_undo'),
]
```

## **📌 Step 5: Build the UI with Bootstrap**
Create `templates/todo/task_list.html`:
```html
{% extends 'base.html' %}
{% block content %}
<h2>To-Do List</h2>
<a href="{% url 'task_create' %}" class="btn btn-success">New Task</a>
<ul class="list-group mt-3">
    {% for task in tasks %}
        <li class="list-group-item">
            <strong>{{ task.title }}</strong> - {{ task.description }}
            <a href="{% url 'task_edit' task.id %}" class="btn btn-primary btn-sm">Edit</a>
            <a href="{% url 'task_delete' task.id %}" class="btn btn-danger btn-sm">Delete</a>
        </li>
    {% empty %}
        <li class="list-group-item">No tasks yet.</li>
    {% endfor %}
</ul>
<a href="{% url 'task_undo' %}" class="btn btn-warning mt-3">Undo Last Delete</a>
{% endblock %}
```

Resources:
- Bootstrap: [https://getbootstrap.com/docs/5.3/getting-started/introduction/](https://getbootstrap.com/docs/5.3/getting-started/introduction/)


## **📌 Step 6: Test the Project**
Run the server:
```bash
python manage.py runserver
```
Test the following:
- [x] Register a new user
- [x] Add, edit, and delete tasks
- [x] Check **undo delete** feature

---

## **📌 Step 7: Deploy the Project (Optional)**
Students can deploy the app using **Render** or **Heroku**.
Guide: [https://docs.djangoproject.com/en/stable/howto/deployment/](https://docs.djangoproject.com/en/stable/howto/deployment/)
