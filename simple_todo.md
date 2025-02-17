### **Exercise: Managing Tasks with Django Forms**

#### **Objective:**
In this exercise, you will create a simple **To-Do app** where users can add and edit tasks using Django Forms. The goal is to understand how Django Forms facilitate data passage from the client to the server and how the same form can be used for both creating and updating tasks.

---

### **Instructions:**

1. **Set Up Your Django App**
   - Create a new Django app named `todo`.
   - Define a `Task` model with fields for `title`, `description`, and `completed` (boolean).
   - Apply migrations to set up the database.

2. **Create a Form for Task Management**
   - Define a Django form that maps to the `Task` model.
   - The form should include fields for entering a task title, description, and marking it as completed.

3. **Implement Views for Adding and Editing Tasks**
   - Create a view that displays the form and allows users to submit new tasks.
   - Ensure that, on successful submission, the task is saved in the database and the user is redirected to a list view.
   - Implement another view that allows users to edit existing tasks using the same form.
   - The form should be pre-populated with the existing task’s data when editing.

4. **Set Up Templates**
   - Create templates for displaying the task form and task list.
   - Include appropriate buttons for submitting new tasks and saving changes when editing.

5. **Define URL Routes**
   - Set up URL patterns for both the "add task" and "edit task" views.

6. **Test the Functionality**
   - Run the server and test adding new tasks.
   - Select a task and try editing its details.
   - Observe how the same form is used for both adding and editing.

---

### **Guiding Questions:**
- How does Django handle the form submission process when creating a new task?
- How does Django retrieve and pre-populate form data when editing an existing task?
- What happens when an invalid form is submitted?
- How does the form ensure data validation before saving it to the database?

---

### **Step-by-Step Solution**

#### **1. Set Up the Django App**
First, create a new Django app called `todo`:

```bash
python manage.py startapp todo
```

Register the app in `settings.py`:

```python
INSTALLED_APPS = [
    # other apps...
    'todo',
]
```

#### **2. Define the Task Model**
In `todo/models.py`, define the `Task` model:

```python
from django.db import models

class Task(models.Model):
    title = models.CharField(max_length=255)
    description = models.TextField(blank=True)
    completed = models.BooleanField(default=False)

    def __str__(self):
        return self.title
```

Run migrations:

```bash
python manage.py makemigrations
python manage.py migrate
```

#### **3. Create the Task Form**
In `todo/forms.py`, create a form for handling task data:

```python
from django import forms
from .models import Task

class TaskForm(forms.ModelForm):
    class Meta:
        model = Task
        fields = ['title', 'description', 'completed']
```

#### **4. Implement Views for Adding and Editing Tasks**
In `todo/views.py`, create views for handling task creation and editing.

```python
from django.shortcuts import render, redirect, get_object_or_404
from .models import Task
from .forms import TaskForm

def task_list(request):
    tasks = Task.objects.all()
    return render(request, 'todo/task_list.html', {'tasks': tasks})

def task_create(request):
    if request.method == "POST":
        form = TaskForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect('task_list')
    else:
        form = TaskForm()
    return render(request, 'todo/task_form.html', {'form': form})

def task_edit(request, task_id):
    task = get_object_or_404(Task, id=task_id)
    if request.method == "POST":
        form = TaskForm(request.POST, instance=task)
        if form.is_valid():
            form.save()
            return redirect('task_list')
    else:
        form = TaskForm(instance=task)
    return render(request, 'todo/task_form.html', {'form': form})
```

#### **5. Set Up URL Routing**
In `todo/urls.py`, define routes for task listing, creation, and editing:

```python
from django.urls import path
from .views import task_list, task_create, task_edit

urlpatterns = [
    path('', task_list, name='task_list'),
    path('add/', task_create, name='task_create'),
    path('edit/<int:task_id>/', task_edit, name='task_edit'),
]
```

In `myproject/urls.py`, include the `todo` app’s URLs:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('tasks/', include('todo.urls')),
]
```

#### **6. Create Templates**
Create the `todo/templates/todo/task_list.html` file for listing tasks:

```html
<h1>Task List</h1>
<a href="{% url 'task_create' %}">Add New Task</a>
<ul>
    {% for task in tasks %}
        <li>
            {{ task.title }} - {% if task.completed %}Completed{% else %}Pending{% endif %}
            <a href="{% url 'task_edit' task.id %}">Edit</a>
        </li>
    {% empty %}
        <p>No tasks available.</p>
    {% endfor %}
</ul>
```

Create the `todo/templates/todo/task_form.html` file for adding and editing tasks:

```html
<h1>{% if form.instance.pk %}Edit Task{% else %}Add New Task{% endif %}</h1>
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Save</button>
</form>
<a href="{% url 'task_list' %}">Back to List</a>
```

#### **7. Run the Server and Test**
Start the development server:

```bash
python manage.py runserver
```

- Visit `http://127.0.0.1:8000/tasks/` to see the task list.
- Click "Add New Task" to create a task.
- Click "Edit" next to a task to modify its details.

---

### **Expected Behavior:**
- When visiting `tasks/`, users see a list of tasks.
- Clicking “Add New Task” shows a form to enter a title, description, and completion status.
- Submitting the form saves the task and redirects back to the list.
- Clicking “Edit” on an existing task pre-populates the form, allowing users to update the task details.

---

### **Review Questions:**
1. How does the form handle both new task creation and editing an existing task?
2. What happens if you submit a blank form?
3. How can you improve the user experience, such as adding validation messages or better styling?

This exercise helps students understand how Django Forms handle data input and updates while keeping the implementation simple and reusable.

