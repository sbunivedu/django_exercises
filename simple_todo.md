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

The following steps add the "delete" feature to this **To-Do app**.

## **1. Update the Delete View** (`views.py`)
```python
from django.shortcuts import redirect, get_object_or_404
from django.contrib import messages
from .models import Task

def task_delete_view(request, task_id):
    """ Deletes a task after user confirmation """
    task = get_object_or_404(Task, id=task_id)

    if request.method == "POST":  # Ensure deletion happens only via POST
        task.delete()
        messages.success(request, "Task deleted successfully!")
        return redirect('task_list')  # Redirect to task list after deletion

    return redirect('task_list')  # Redirect if accessed via GET
```
- Uses **`messages`** to show a success message after deletion.
- Redirects the user back to the task list.

---

## **2. Update URLs** (`urls.py`)
```python
from django.urls import path
from .views import task_delete_view

urlpatterns = [
    path('delete/<int:task_id>/', task_delete_view, name='task_delete_view'),
]
```

---

## **3. Modify Task List Template to Include a "Delete" Button** (`task_list.html`)
```html
<h1>Task List</h1>
{% if messages %}
    <ul>
        {% for message in messages %}
            <li style="color: green;">{{ message }}</li>
        {% endfor %}
    </ul>
{% endif %}

<ul>
    {% for task in tasks %}
        <li>
            {{ task.title }} - {{ task.description }}
            <form action="{% url 'task_delete_view' task.id %}" method="post" style="display:inline;">
                {% csrf_token %}
                <button type="submit" onclick="return confirm('Are you sure you want to delete this task?');">
                    Delete
                </button>
            </form>
        </li>
    {% endfor %}
</ul>
```
### **How This Works**
✔ **CSRF Protection** – Includes `{% csrf_token %}` to prevent CSRF attacks.
✔ **Confirmation Prompt** – Uses `onclick="return confirm(...)"` to ask before deletion.
✔ **Redirects after Deletion** – The user is taken back to the task list after a task is removed.

### **Adding an Undo Feature for Task Deletion**

To allow users to undo task deletion, we'll **store deleted tasks temporarily** and **provide a "Restore" option** after deletion.

---

## **1. Modify the Delete View to Support Undo** (`views.py`)
Instead of permanently deleting the task, we’ll **store it in the session** and allow restoration within a time limit.

```python
from django.shortcuts import redirect, get_object_or_404, render
from django.contrib import messages
from .models import Task

def task_delete_view(request, task_id):
    """ Soft deletes a task and allows undo within a session """
    task = get_object_or_404(Task, id=task_id)

    if request.method == "POST":
        # Store task details in session for undo
        request.session['deleted_task'] = {
            'id': task.id,
            'title': task.title,
            'description': task.description
        }
        task.delete()
        messages.success(request, "Task deleted! You can undo this action.")

    return redirect('task_list')  # Redirect to task list

def task_restore_view(request):
    """ Restores the last deleted task if available """
    deleted_task = request.session.get('deleted_task')

    if deleted_task:
        Task.objects.create(
            id=deleted_task['id'],
            title=deleted_task['title'],
            description=deleted_task['description']
        )
        messages.success(request, "Task restored successfully!")
        del request.session['deleted_task']  # Remove from session after restoring

    return redirect('task_list')
```
- **Stores task details in `request.session`** before deletion.
- **Restores the task if the user clicks "Undo"** before the session expires.

---

## **2. Update URLs** (`urls.py`)
```python
from django.urls import path
from .views import task_delete_view, task_restore_view

urlpatterns = [
    path('delete/<int:task_id>/', task_delete_view, name='task_delete_view'),
    path('restore/', task_restore_view, name='task_restore_view'),
]
```

---

## **3. Modify Task List Template to Show Undo Option** (`task_list.html`)
```html
<h1>Task List</h1>

{% if messages %}
    <ul>
        {% for message in messages %}
            <li style="color: green;">{{ message }}</li>
        {% endfor %}
    </ul>
{% endif %}

<ul>
    {% for task in tasks %}
        <li>
            {{ task.title }} - {{ task.description }}
            <form action="{% url 'task_delete_view' task.id %}" method="post" style="display:inline;">
                {% csrf_token %}
                <button type="submit" onclick="return confirm('Are you sure you want to delete this task?');">
                    Delete
                </button>
            </form>
        </li>
    {% endfor %}
</ul>

{% if request.session.deleted_task %}
    <form action="{% url 'task_restore_view' %}" method="post">
        {% csrf_token %}
        <button type="submit">Undo Delete</button>
    </form>
{% endif %}
```
---

## **Final Features**
✔ **Soft Delete with Undo** – Task is **temporarily stored** instead of being permanently removed.
✔ **Undo Button** – Appears only if a task was recently deleted.
✔ **Session-Based Storage** – Undo is possible **until the session expires** or another task is deleted.

### **Adding a Timeout-Based Undo Option for Task Deletion**

To make **undo available only for 30 seconds**, we'll store the **deletion time in the session** and hide the undo option when time runs out.

---

## **1. Modify the Delete View to Include a Timestamp** (`views.py`)
We'll store both the deleted task **and the timestamp** in the session.

```python
from django.shortcuts import redirect, get_object_or_404
from django.contrib import messages
from django.utils.timezone import now
from .models import Task

def task_delete_view(request, task_id):
    """ Soft deletes a task and allows undo for 30 seconds """
    task = get_object_or_404(Task, id=task_id)

    if request.method == "POST":
        request.session['deleted_task'] = {
            'id': task.id,
            'title': task.title,
            'description': task.description,
            'deleted_at': now().timestamp()  # Store deletion timestamp
        }
        task.delete()
        messages.success(request, "Task deleted! You have 30 seconds to undo.")

    return redirect('task_list')

def task_restore_view(request):
    """ Restores the last deleted task if within the allowed time """
    deleted_task = request.session.get('deleted_task')

    if deleted_task:
        time_elapsed = now().timestamp() - deleted_task['deleted_at']

        if time_elapsed <= 30:  # Allow undo only if within 30 seconds
            Task.objects.create(
                id=deleted_task['id'],
                title=deleted_task['title'],
                description=deleted_task['description']
            )
            messages.success(request, "Task restored successfully!")
            del request.session['deleted_task']  # Remove from session after restoring
        else:
            messages.error(request, "Undo period has expired!")

    return redirect('task_list')
```
- **Stores deletion time in `deleted_at`.**
- **Checks if 30 seconds have passed before allowing restore.**
- **Shows an error message if the undo period has expired.**

---

## **2. Update Task List Template to Hide Undo When Time Runs Out** (`task_list.html`)
```html
<h1>Task List</h1>

{% if messages %}
    <ul>
        {% for message in messages %}
            <li style="color: green;">{{ message }}</li>
        {% endfor %}
    </ul>
{% endif %}

<ul>
    {% for task in tasks %}
        <li>
            {{ task.title }} - {{ task.description }}
            <form action="{% url 'task_delete_view' task.id %}" method="post" style="display:inline;">
                {% csrf_token %}
                <button type="submit" onclick="return confirm('Are you sure you want to delete this task?');">
                    Delete
                </button>
            </form>
        </li>
    {% endfor %}
</ul>

{% if request.session.deleted_task %}
    <div id="undo-section">
        <form action="{% url 'task_restore_view' %}" method="post">
            {% csrf_token %}
            <button type="submit">Undo Delete</button>
        </form>
        <p id="timer">Undo available for <span id="time-left">30</span> seconds</p>
    </div>

    <script>
        let timeLeft = 30;
        function countdown() {
            if (timeLeft > 0) {
                timeLeft--;
                document.getElementById("time-left").innerText = timeLeft;
                setTimeout(countdown, 1000);
            } else {
                document.getElementById("undo-section").style.display = "none"; // Hide undo option
            }
        }
        countdown();
    </script>
{% endif %}
```
---

## **Final Features**
✔ **Undo Available for 30 Seconds** – Uses a timestamp to check time limits.
✔ **Automatic Hide** – The undo button disappears when time runs out.
✔ **User-Friendly Timer** – A **JavaScript countdown** shows remaining time.

### **Storing Deleted Tasks in the Database for Persistent Undo**

Instead of using **session storage**, we'll **add a "soft delete" feature** by keeping deleted tasks in a separate table. This allows **undo even after page refresh** and prevents accidental loss.

---

## **1. Create a Model to Store Deleted Tasks** (`models.py`)

```python
from django.db import models
from django.utils.timezone import now

class Task(models.Model):
    title = models.CharField(max_length=255)
    description = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)

class DeletedTask(models.Model):
    """ Stores recently deleted tasks for undo functionality """
    title = models.CharField(max_length=255)
    description = models.TextField()
    deleted_at = models.DateTimeField(default=now)  # Track deletion time

    def is_undo_available(self):
        """ Check if the task can still be restored (within 30 seconds) """
        return (now() - self.deleted_at).seconds <= 30
```
- **`DeletedTask` stores soft-deleted tasks.**
- **`is_undo_available()` checks if 30 seconds have passed.**

---

## **2. Update Views to Support Soft Delete and Undo** (`views.py`)

```python
from django.shortcuts import redirect, get_object_or_404, render
from django.contrib import messages
from django.utils.timezone import now
from .models import Task, DeletedTask

def task_delete_view(request, task_id):
    """ Soft deletes a task and moves it to DeletedTask table """
    task = get_object_or_404(Task, id=task_id)

    if request.method == "POST":
        # Save task details in DeletedTask
        DeletedTask.objects.create(title=task.title, description=task.description)
        task.delete()
        messages.success(request, "Task deleted! You have 30 seconds to undo.")

    return redirect('task_list')

def task_restore_view(request):
    """ Restores the most recent deleted task within 30 seconds """
    deleted_task = DeletedTask.objects.order_by('-deleted_at').first()

    if deleted_task and deleted_task.is_undo_available():
        Task.objects.create(title=deleted_task.title, description=deleted_task.description)
        deleted_task.delete()  # Remove from DeletedTask table
        messages.success(request, "Task restored successfully!")
    else:
        messages.error(request, "Undo period has expired!")

    return redirect('task_list')
```
- **Deleted tasks are stored in `DeletedTask`.**
- **Tasks can be restored if within 30 seconds.**
- **Deletes from `DeletedTask` after undo to keep data clean.**

---

## **3. Update URLs** (`urls.py`)

```python
from django.urls import path
from .views import task_delete_view, task_restore_view

urlpatterns = [
    path('delete/<int:task_id>/', task_delete_view, name='task_delete_view'),
    path('restore/', task_restore_view, name='task_restore_view'),
]
```

---

## **4. Modify Task List Template to Show Undo Button** (`task_list.html`)

```html
<h1>Task List</h1>

{% if messages %}
    <ul>
        {% for message in messages %}
            <li style="color: green;">{{ message }}</li>
        {% endfor %}
    </ul>
{% endif %}

<ul>
    {% for task in tasks %}
        <li>
            {{ task.title }} - {{ task.description }}
            <form action="{% url 'task_delete_view' task.id %}" method="post" style="display:inline;">
                {% csrf_token %}
                <button type="submit" onclick="return confirm('Are you sure you want to delete this task?');">
                    Delete
                </button>
            </form>
        </li>
    {% endfor %}
</ul>

{% with deleted_task=deleted_tasks|first %}
    {% if deleted_task and deleted_task.is_undo_available %}
        <div id="undo-section">
            <form action="{% url 'task_restore_view' %}" method="post">
                {% csrf_token %}
                <button type="submit">Undo Delete</button>
            </form>
            <p id="timer">Undo available for <span id="time-left">30</span> seconds</p>
        </div>

        <script>
            let timeLeft = 30;
            function countdown() {
                if (timeLeft > 0) {
                    timeLeft--;
                    document.getElementById("time-left").innerText = timeLeft;
                    setTimeout(countdown, 1000);
                } else {
                    document.getElementById("undo-section").style.display = "none"; // Hide undo option
                }
            }
            countdown();
        </script>
    {% endif %}
{% endwith %}
```
- **Gets the most recent deleted task from `DeletedTask`.**
- **Shows undo button if within 30 seconds.**
- **JavaScript countdown hides button after time expires.**

---

## **Final Features**
✔ **Soft Delete with Undo Stored in Database** – Undo **even after page refresh.**
✔ **30-Second Time Limit** – Ensures old deleted tasks don’t clutter the database.
✔ **Automatic Cleanup** – Tasks are removed from `DeletedTask` after restoration.

### **Automatically Removing Expired Deleted Tasks**

To prevent **DeletedTask** from filling up with old records, we'll **automatically clean up expired tasks** using:
✔ A **scheduled cleanup job** (for periodic deletion)
✔ A **check in the undo function** (removing expired tasks when accessed)

---

## **1. Create a Cleanup Function for Expired Tasks** (`views.py`)

```python
from django.utils.timezone import now
from .models import DeletedTask

def clean_expired_deleted_tasks():
    """ Deletes tasks older than 30 seconds from the DeletedTask table """
    expiration_time = now().timestamp() - 30  # Get timestamp 30 seconds ago
    DeletedTask.objects.filter(deleted_at__lt=now()).delete()  # Remove expired tasks
```
- This function removes **tasks older than 30 seconds**.

---

## **2. Call Cleanup in Restore View** (`views.py`)

```python
from django.shortcuts import redirect
from django.contrib import messages
from .models import Task, DeletedTask

def task_restore_view(request):
    """ Restores a task if within 30 seconds, else cleans expired tasks """
    deleted_task = DeletedTask.objects.order_by('-deleted_at').first()

    if deleted_task:
        if deleted_task.is_undo_available():
            Task.objects.create(title=deleted_task.title, description=deleted_task.description)
            deleted_task.delete()  # Remove restored task
            messages.success(request, "Task restored successfully!")
        else:
            messages.error(request, "Undo period expired!")

    # Cleanup old tasks after checking for restoration
    clean_expired_deleted_tasks()

    return redirect('task_list')
```
✔ **Deletes expired tasks** when a user tries to restore.
✔ **Keeps database clean automatically.**

---

## **3. Set Up a Periodic Cleanup Task** (Optional)

If you want to remove expired tasks **even if no one clicks restore**, use Django's **management commands** or **Celery periodic tasks**.

### **Method 1: Django Management Command**
Create a **custom command** to run cleanup periodically.

1️⃣ **Create the Command**
Run:
```bash
mkdir -p myapp/management/commands
touch myapp/management/commands/clean_deleted_tasks.py
```

2️⃣ **Write the Cleanup Command** (`clean_deleted_tasks.py`)
```python
from django.core.management.base import BaseCommand
from myapp.views import clean_expired_deleted_tasks

class Command(BaseCommand):
    help = 'Cleans expired deleted tasks'

    def handle(self, *args, **kwargs):
        clean_expired_deleted_tasks()
        self.stdout.write("Expired deleted tasks removed.")
```

3️⃣ **Schedule the Command in a Cron Job**
Edit `crontab`:
```bash
crontab -e
```
Add:
```bash
* * * * * /path/to/your/venv/bin/python /path/to/your/manage.py clean_deleted_tasks
```
This runs **every minute** to clean expired tasks.

---

### **Method 2: Using Celery (If Your App Uses It)**
If you’re using **Celery**, define a scheduled task.

1️⃣ **Add Task in `tasks.py`**
```python
from celery import shared_task
from .views import clean_expired_deleted_tasks

@shared_task
def periodic_clean_deleted_tasks():
    clean_expired_deleted_tasks()
```

2️⃣ **Schedule It in `celery.py`**
```python
from celery.schedules import crontab

app.conf.beat_schedule = {
    'clean_deleted_tasks_every_minute': {
        'task': 'myapp.tasks.periodic_clean_deleted_tasks',
        'schedule': crontab(minute='*'),  # Runs every minute
    },
}
```
✅ **Celery will clean expired tasks every minute**.

---

## **Final Features**
✔ **Expired Deleted Tasks Are Removed Automatically**
✔ **Cleanup Runs Periodically or on Restore Attempt**
✔ **Keeps the Database Clean Without Manual Work**

### **Extending Undo to Multiple Deleted Tasks**

Currently, the system only restores **the most recent deleted task**. Now, we'll modify it so that users can **choose from multiple recently deleted tasks** within **30 seconds**.

---

## **1. Update the Restore View to List Multiple Deleted Tasks** (`views.py`)

```python
from django.shortcuts import render, redirect, get_object_or_404
from django.contrib import messages
from django.utils.timezone import now
from .models import Task, DeletedTask

def task_delete_view(request, task_id):
    """ Soft deletes a task and moves it to DeletedTask table """
    task = get_object_or_404(Task, id=task_id)

    if request.method == "POST":
        DeletedTask.objects.create(title=task.title, description=task.description, deleted_at=now())
        task.delete()
        messages.success(request, "Task deleted! You have 30 seconds to undo.")

    return redirect('task_list')

def task_restore_view(request, deleted_task_id):
    """ Restores a specific deleted task if within 30 seconds """
    deleted_task = get_object_or_404(DeletedTask, id=deleted_task_id)

    if deleted_task.is_undo_available():
        Task.objects.create(title=deleted_task.title, description=deleted_task.description)
        deleted_task.delete()  # Remove from DeletedTask table
        messages.success(request, "Task restored successfully!")
    else:
        messages.error(request, "Undo period expired!")

    # Cleanup old deleted tasks
    clean_expired_deleted_tasks()

    return redirect('task_list')
```
✔ **Users can select which task to restore instead of just the last one.**
✔ **Expired tasks get cleaned automatically.**

---

## **2. Update URLs to Support Multiple Undo Choices** (`urls.py`)

```python
from django.urls import path
from .views import task_delete_view, task_restore_view

urlpatterns = [
    path('delete/<int:task_id>/', task_delete_view, name='task_delete_view'),
    path('restore/<int:deleted_task_id>/', task_restore_view, name='task_restore_view'),
]
```
✔ **Users can restore a specific task using `/restore/<task_id>/`.**

---

## **3. Modify Task List Template to Show Multiple Undo Options** (`task_list.html`)

```html
<h1>Task List</h1>

{% if messages %}
    <ul>
        {% for message in messages %}
            <li style="color: green;">{{ message }}</li>
        {% endfor %}
    </ul>
{% endif %}

<ul>
    {% for task in tasks %}
        <li>
            {{ task.title }} - {{ task.description }}
            <form action="{% url 'task_delete_view' task.id %}" method="post" style="display:inline;">
                {% csrf_token %}
                <button type="submit" onclick="return confirm('Are you sure you want to delete this task?');">
                    Delete
                </button>
            </form>
        </li>
    {% endfor %}
</ul>

{% if deleted_tasks %}
    <h2>Recently Deleted Tasks (Undo Available for 30 Seconds)</h2>
    <ul>
        {% for deleted_task in deleted_tasks %}
            {% if deleted_task.is_undo_available %}
                <li>
                    {{ deleted_task.title }} - {{ deleted_task.description }}
                    <form action="{% url 'task_restore_view' deleted_task.id %}" method="post">
                        {% csrf_token %}
                        <button type="submit">Undo</button>
                    </form>
                    <p>Undo expires in <span class="countdown" data-time="{{ deleted_task.deleted_at|date:'U' }}"></span> seconds</p>
                </li>
            {% endif %}
        {% endfor %}
    </ul>

    <script>
        function updateCountdowns() {
            let countdowns = document.querySelectorAll(".countdown");
            let now = Math.floor(Date.now() / 1000);

            countdowns.forEach(function(el) {
                let deleteTime = parseInt(el.dataset.time);
                let timeLeft = 30 - (now - deleteTime);

                if (timeLeft > 0) {
                    el.textContent = timeLeft;
                } else {
                    el.closest("li").remove();  // Hide expired tasks
                }
            });

            setTimeout(updateCountdowns, 1000);
        }

        updateCountdowns();
    </script>
{% endif %}
```
✔ **Displays all recently deleted tasks with individual undo buttons.**
✔ **JavaScript updates countdown timers dynamically.**

---

## **4. Update `views.py` to Pass Deleted Tasks to Template**

Modify the **task list view** to pass deleted tasks to the template.

```python
def task_list_view(request):
    """ Displays tasks and recently deleted tasks """
    tasks = Task.objects.all()
    deleted_tasks = DeletedTask.objects.filter(deleted_at__gte=now()).order_by('-deleted_at')

    return render(request, 'task_list.html', {'tasks': tasks, 'deleted_tasks': deleted_tasks})
```
✔ **Fetches deleted tasks sorted by deletion time.**
✔ **Only tasks deleted within 30 seconds are shown.**

---

## **Final Features**
✔ **Users Can Undo Multiple Deleted Tasks**
✔ **Each Deleted Task Has Its Own Countdown Timer**
✔ **Expired Tasks Are Automatically Removed**

### **Adding an Admin Panel Option to Configure the Undo Time Limit**

We'll allow **admin users** to set a custom undo time limit **without modifying the code**. This will be stored in the database and dynamically retrieved when checking if a task can be restored.

---

## **1. Create a Model for Configuration Settings** (`models.py`)

```python
from django.db import models

class AppConfig(models.Model):
    """ Stores configuration settings for the app """
    undo_time_limit = models.IntegerField(default=30, help_text="Time (in seconds) to allow task undo.")

    def __str__(self):
        return f"Undo Time Limit: {self.undo_time_limit} seconds"

    @classmethod
    def get_undo_time_limit(cls):
        """ Fetches the undo time limit, defaults to 30 seconds if not set """
        config = cls.objects.first()
        return config.undo_time_limit if config else 30
```
✔ **Stores the undo time limit in the database.**
✔ **Allows updating the time limit from the Django admin panel.**
✔ **Provides a method (`get_undo_time_limit()`) to fetch the value dynamically.**

---

## **2. Register AppConfig in the Admin Panel** (`admin.py`)

```python
from django.contrib import admin
from .models import AppConfig

@admin.register(AppConfig)
class AppConfigAdmin(admin.ModelAdmin):
    list_display = ('undo_time_limit',)
```
✔ **Admins can modify the undo time limit via the Django admin interface.**
✔ **Only one config entry is needed—no need to create multiple records.**

---

## **3. Update the Deleted Task Model to Use the Configurable Undo Time** (`models.py`)

Modify the `DeletedTask` model to use the **dynamic undo time** instead of a fixed 30 seconds.

```python
from django.utils.timezone import now

class DeletedTask(models.Model):
    title = models.CharField(max_length=255)
    description = models.TextField()
    deleted_at = models.DateTimeField(auto_now_add=True)

    def is_undo_available(self):
        """ Checks if the undo time has not expired based on the configured limit """
        undo_time_limit = AppConfig.get_undo_time_limit()
        return (now() - self.deleted_at).total_seconds() < undo_time_limit
```
✔ **Uses the admin-configurable undo time instead of a hardcoded 30 seconds.**

---

## **4. Modify Views to Use the Configurable Time** (`views.py`)

### **Update Restore View**
```python
def task_restore_view(request, deleted_task_id):
    """ Restores a task if within the configured undo time limit """
    deleted_task = get_object_or_404(DeletedTask, id=deleted_task_id)

    if deleted_task.is_undo_available():
        Task.objects.create(title=deleted_task.title, description=deleted_task.description)
        deleted_task.delete()
        messages.success(request, "Task restored successfully!")
    else:
        messages.error(request, "Undo period expired!")

    clean_expired_deleted_tasks()
    return redirect('task_list')
```
✔ **Automatically respects the configured undo time limit.**

---

### **5. Modify JavaScript Countdown to Reflect Configurable Time** (`task_list.html`)

Update the template so JavaScript dynamically adjusts countdowns.

```html
<script>
    let undoTimeLimit = {{ deleted_tasks.0.get_undo_time_limit|default:30 }};  // Get from Django

    function updateCountdowns() {
        let countdowns = document.querySelectorAll(".countdown");
        let now = Math.floor(Date.now() / 1000);

        countdowns.forEach(function(el) {
            let deleteTime = parseInt(el.dataset.time);
            let timeLeft = undoTimeLimit - (now - deleteTime);

            if (timeLeft > 0) {
                el.textContent = timeLeft;
            } else {
                el.closest("li").remove();  // Hide expired tasks
            }
        });

        setTimeout(updateCountdowns, 1000);
    }

    updateCountdowns();
</script>
```
✔ **The undo time in the UI dynamically updates based on the admin setting.**

---

## **6. Allow Admins to Modify Undo Time in Django Admin**

1. **Run Migrations**
   ```bash
   python manage.py makemigrations
   python manage.py migrate
   ```

2. **Create a Default Configuration Entry**
   ```bash
   python manage.py shell
   >>> from myapp.models import AppConfig
   >>> AppConfig.objects.create(undo_time_limit=30)
   ```

3. **Login to Django Admin**
   - Navigate to `/admin/`
   - Find **AppConfig**
   - Modify the **undo time limit**

---

## **Final Features**
✔ **Admins Can Adjust the Undo Time Without Code Changes**
✔ **Tasks Expire Based on the Configured Time**
✔ **UI Automatically Updates Undo Timers**

### **Adding an Optional Warning Popup Before Undo Time Expires**

We'll modify the **JavaScript countdown** to display a **warning popup** when only **5 seconds** remain before the undo time expires.

---

## **1. Update the JavaScript Countdown to Show a Warning**

Modify **`task_list.html`** to include a **popup alert** when the remaining time reaches 5 seconds.

```html
<script>
    let undoTimeLimit = {{ deleted_tasks.0.get_undo_time_limit|default:30 }};  // Get from Django

    function updateCountdowns() {
        let countdowns = document.querySelectorAll(".countdown");
        let now = Math.floor(Date.now() / 1000);

        countdowns.forEach(function(el) {
            let deleteTime = parseInt(el.dataset.time);
            let timeLeft = undoTimeLimit - (now - deleteTime);

            if (timeLeft > 0) {
                el.textContent = timeLeft;

                // Show warning at 5 seconds
                if (timeLeft === 5 && !el.dataset.warned) {
                    alert("⚠️ Warning! This task will be permanently deleted in 5 seconds!");
                    el.dataset.warned = "true";  // Mark as warned to avoid multiple alerts
                }
            } else {
                el.closest("li").remove();  // Hide expired tasks
            }
        });

        setTimeout(updateCountdowns, 1000);
    }

    updateCountdowns();
</script>
```
✔ **Displays a warning 5 seconds before a task is permanently deleted.**
✔ **Prevents multiple alerts for the same task.**
✔ **Hides expired tasks automatically.**

---

## **2. (Optional) Style the Warning Message with a Visible Alert Box**

Instead of using `alert()`, we can show a **visible warning box** in the UI.

### **Modify the Template to Include a Warning Box**

```html
<div id="undo-warning" style="display: none; background: yellow; padding: 10px; margin: 10px 0;">
    ⚠️ <strong>Warning:</strong> A task is about to be permanently deleted!
</div>

<script>
    let undoTimeLimit = {{ deleted_tasks.0.get_undo_time_limit|default:30 }};
    let warningBox = document.getElementById("undo-warning");

    function updateCountdowns() {
        let countdowns = document.querySelectorAll(".countdown");
        let now = Math.floor(Date.now() / 1000);
        let showWarning = false;

        countdowns.forEach(function(el) {
            let deleteTime = parseInt(el.dataset.time);
            let timeLeft = undoTimeLimit - (now - deleteTime);

            if (timeLeft > 0) {
                el.textContent = timeLeft;

                // Show warning at 5 seconds
                if (timeLeft <= 5) {
                    showWarning = true;
                }
            } else {
                el.closest("li").remove();  // Hide expired tasks
            }
        });

        warningBox.style.display = showWarning ? "block" : "none";  // Show/hide warning box

        setTimeout(updateCountdowns, 1000);
    }

    updateCountdowns();
</script>
```
✔ **Displays a visible warning banner instead of an alert box.**
✔ **Hides the warning when no tasks are close to expiration.**

---

## **Final Features**
✅ **Warns Users Before Tasks Are Permanently Deleted**
✅ **Option for Popup Alert or Visible UI Warning**
✅ **Automatically Hides Expired Tasks**

### **Adding an Admin-Only Grace Period for Task Recovery**

We’ll extend the system to include a **grace period** after a task expires. During this time, the task will remain recoverable but **only accessible to admins**.

---

### **1. Update the `DeletedTask` Model** (`models.py`)

Add a **grace period** by introducing a method to check whether a task is recoverable by admins even after user expiration.

```python
from django.db import models
from django.utils.timezone import now

class AppConfig(models.Model):
    """Stores app-wide configuration settings"""
    undo_time_limit = models.IntegerField(default=30, help_text="Time (in seconds) to allow task undo.")
    admin_grace_period = models.IntegerField(default=300, help_text="Admin grace period in seconds (e.g., 300 = 5 minutes).")

    def __str__(self):
        return f"Undo Time: {self.undo_time_limit}s, Admin Grace: {self.admin_grace_period}s"

    @classmethod
    def get_undo_time_limit(cls):
        """Returns the user undo time (default: 30 seconds)"""
        config = cls.objects.first()
        return config.undo_time_limit if config else 30

    @classmethod
    def get_admin_grace_period(cls):
        """Returns the admin grace period (default: 300 seconds = 5 minutes)"""
        config = cls.objects.first()
        return config.admin_grace_period if config else 300


class DeletedTask(models.Model):
    """Stores soft-deleted tasks with recovery windows"""
    title = models.CharField(max_length=255)
    description = models.TextField()
    deleted_at = models.DateTimeField(auto_now_add=True)

    def is_user_undo_available(self):
        """Checks if the task is still recoverable by a user"""
        return (now() - self.deleted_at).total_seconds() < AppConfig.get_undo_time_limit()

    def is_admin_undo_available(self):
        """Checks if the task is still recoverable by an admin (extended window)"""
        return (now() - self.deleted_at).total_seconds() < AppConfig.get_admin_grace_period()
```

✔ **Allows configuration of both the user undo time and admin grace period.**
✔ **Admins can restore tasks even after the user undo window expires.**

---

### **2. Update the Restore View** (`views.py`)

We’ll check whether the user is an **admin** and allow recovery during the grace period.

```python
from django.shortcuts import redirect, get_object_or_404, render
from django.contrib import messages
from django.utils.timezone import now
from .models import Task, DeletedTask

def task_restore_view(request, deleted_task_id):
    """Restores a task if within the user or admin grace period"""
    deleted_task = get_object_or_404(DeletedTask, id=deleted_task_id)

    if deleted_task.is_user_undo_available() or (request.user.is_staff and deleted_task.is_admin_undo_available()):
        Task.objects.create(title=deleted_task.title, description=deleted_task.description)
        deleted_task.delete()
        messages.success(request, "Task successfully restored!")
    else:
        messages.error(request, "The undo period has expired!")

    clean_expired_deleted_tasks()  # Automatically clear old entries
    return redirect('task_list')

def clean_expired_deleted_tasks():
    """Removes tasks that have passed both the user and admin grace periods"""
    DeletedTask.objects.all().delete()  # Automatically removes expired tasks
```

✔ **Users can only undo within their defined window.**
✔ **Admins can undo within the extended grace period.**
✔ **Expired tasks are automatically cleaned up.**

---

### **3. Update the Admin Panel Configuration** (`admin.py`)

Allow admins to configure **both time limits** through Django’s admin interface.

```python
from django.contrib import admin
from .models import AppConfig

@admin.register(AppConfig)
class AppConfigAdmin(admin.ModelAdmin):
    list_display = ('undo_time_limit', 'admin_grace_period')
```

---

### **4. Update the Template to Show Admin-Only Undo** (`task_list.html`)

We will display an **"Admin Recovery"** section that only appears for **staff users**.

```html
{% if messages %}
    <ul>
        {% for message in messages %}
            <li style="color: green;">{{ message }}</li>
        {% endfor %}
    </ul>
{% endif %}

<h1>Task List</h1>

<ul>
    {% for task in tasks %}
        <li>
            {{ task.title }} - {{ task.description }}
            <form action="{% url 'task_delete_view' task.id %}" method="post" style="display:inline;">
                {% csrf_token %}
                <button type="submit" onclick="return confirm('Are you sure you want to delete this task?');">
                    Delete
                </button>
            </form>
        </li>
    {% endfor %}
</ul>

{% if deleted_tasks %}
    <h2>Recently Deleted Tasks (Undo Available for {{ deleted_tasks.0.get_undo_time_limit }} seconds)</h2>
    <ul>
        {% for deleted_task in deleted_tasks %}
            {% if deleted_task.is_user_undo_available %}
                <li>
                    {{ deleted_task.title }}
                    <form action="{% url 'task_restore_view' deleted_task.id %}" method="post" style="display:inline;">
                        {% csrf_token %}
                        <button type="submit">Undo</button>
                    </form>
                    <span class="countdown" data-time="{{ deleted_task.deleted_at|date:'U' }}"></span> seconds left
                </li>
            {% endif %}
        {% endfor %}
    </ul>
{% endif %}

{% if user.is_staff %}
    <h2>Admin Recovery (Tasks within {{ deleted_tasks.0.get_admin_grace_period }} seconds)</h2>
    <ul>
        {% for deleted_task in deleted_tasks %}
            {% if deleted_task.is_admin_undo_available and not deleted_task.is_user_undo_available %}
                <li>
                    {{ deleted_task.title }}
                    <form action="{% url 'task_restore_view' deleted_task.id %}" method="post" style="display:inline;">
                        {% csrf_token %}
                        <button type="submit">Restore (Admin)</button>
                    </form>
                </li>
            {% endif %}
        {% endfor %}
    </ul>
{% endif %}

<script>
    function updateCountdowns() {
        let countdowns = document.querySelectorAll(".countdown");
        let now = Math.floor(Date.now() / 1000);
        let undoTimeLimit = {{ deleted_tasks.0.get_undo_time_limit|default:30 }};

        countdowns.forEach(function(el) {
            let deleteTime = parseInt(el.dataset.time);
            let timeLeft = undoTimeLimit - (now - deleteTime);
            if (timeLeft > 0) {
                el.textContent = timeLeft;
            } else {
                el.closest("li").remove();  // Hide expired tasks
            }
        });
        setTimeout(updateCountdowns, 1000);
    }

    updateCountdowns();
</script>
```

✔ **Users see their own undo option.**
✔ **Admins have a separate "Admin Recovery" section.**
✔ **Timers automatically update and hide expired tasks.**

---

### **5. Final Touch: Run Migrations and Create Config**
1. **Run the migrations:**
   ```bash
   python manage.py makemigrations
   python manage.py migrate
   ```

2. **Create the initial configuration:**
   ```bash
   python manage.py shell
   >>> from myapp.models import AppConfig
   >>> AppConfig.objects.create(undo_time_limit=30, admin_grace_period=300)
   ```

---

## **🎯 Final Features**
✅ **User Undo:** Users can undo tasks within a configurable time (default: 30s).
✅ **Admin Grace Period:** Admins can recover tasks within a longer window (default: 5 min).
✅ **Automatic Cleanup:** Tasks are removed once both time windows expire.

### **Adding Audit Tracking for Task Deletion and Restoration**

To enhance accountability, we’ll track **who deleted** and **who restored** each task. This will help in auditing user actions in the system.

---

## **1. Update the `DeletedTask` Model** (`models.py`)

We'll add fields to store the **user who deleted** and the **user who restored** the task.

```python
from django.db import models
from django.contrib.auth.models import User
from django.utils.timezone import now

class DeletedTask(models.Model):
    """Stores soft-deleted tasks with audit tracking"""
    title = models.CharField(max_length=255)
    description = models.TextField()
    deleted_at = models.DateTimeField(auto_now_add=True)
    deleted_by = models.ForeignKey(User, on_delete=models.SET_NULL, null=True, related_name='deleted_tasks')
    restored_at = models.DateTimeField(null=True, blank=True)
    restored_by = models.ForeignKey(User, on_delete=models.SET_NULL, null=True, blank=True, related_name='restored_tasks')

    def is_user_undo_available(self):
        """Checks if the task is still recoverable by a user"""
        return (now() - self.deleted_at).total_seconds() < AppConfig.get_undo_time_limit()

    def is_admin_undo_available(self):
        """Checks if the task is still recoverable by an admin (extended window)"""
        return (now() - self.deleted_at).total_seconds() < AppConfig.get_admin_grace_period()

    def __str__(self):
        return f"{self.title} (Deleted by: {self.deleted_by}, Restored by: {self.restored_by})"
```

✔ **Tracks who deleted and restored the task.**
✔ **Stores timestamps for deletion and restoration.**

---

## **2. Update the Delete View to Log the User** (`views.py`)

Modify the **task deletion** logic to store the deleting user.

```python
from django.shortcuts import redirect, get_object_or_404
from django.contrib import messages
from django.utils.timezone import now
from .models import Task, DeletedTask

def task_delete_view(request, task_id):
    """Soft-deletes a task and logs the user who deleted it"""
    task = get_object_or_404(Task, id=task_id)

    # Move to DeletedTask model
    DeletedTask.objects.create(
        title=task.title,
        description=task.description,
        deleted_by=request.user,  # Track who deleted it
        deleted_at=now()
    )

    task.delete()
    messages.success(request, "Task deleted. Undo available for a limited time.")
    return redirect('task_list')
```

✔ **Stores the user who deleted the task.**
✔ **Retains task details for recovery.**

---

## **3. Update the Restore View to Log the User** (`views.py`)

Modify the **task restore** logic to track who restored it.

```python
def task_restore_view(request, deleted_task_id):
    """Restores a deleted task and logs the user who restored it"""
    deleted_task = get_object_or_404(DeletedTask, id=deleted_task_id)

    if deleted_task.is_user_undo_available() or (request.user.is_staff and deleted_task.is_admin_undo_available()):
        Task.objects.create(title=deleted_task.title, description=deleted_task.description)

        # Log who restored it
        deleted_task.restored_at = now()
        deleted_task.restored_by = request.user
        deleted_task.save()

        messages.success(request, "Task successfully restored!")
    else:
        messages.error(request, "The undo period has expired!")

    clean_expired_deleted_tasks()
    return redirect('task_list')
```

✔ **Tracks the user who restored the task.**
✔ **Logs the restoration time.**

---

## **4. Display Audit Logs in the Admin Panel** (`admin.py`)

To make it easy to view deletions and restorations in Django Admin:

```python
from django.contrib import admin
from .models import DeletedTask

@admin.register(DeletedTask)
class DeletedTaskAdmin(admin.ModelAdmin):
    list_display = ('title', 'deleted_by', 'deleted_at', 'restored_by', 'restored_at')
    list_filter = ('deleted_by', 'restored_by')
    search_fields = ('title', 'deleted_by__username', 'restored_by__username')
```

✔ **Admins can search for tasks deleted or restored by specific users.**
✔ **Filter logs to track user actions.**

---

## **5. Update the Frontend to Show Who Deleted or Restored a Task**

Modify `task_list.html` to **show the user who deleted or restored a task**.

```html
{% if deleted_tasks %}
    <h2>Recently Deleted Tasks</h2>
    <ul>
        {% for deleted_task in deleted_tasks %}
            <li>
                <strong>{{ deleted_task.title }}</strong>
                <br>
                Deleted by: {{ deleted_task.deleted_by }} at {{ deleted_task.deleted_at|date:"H:i:s" }}

                {% if deleted_task.is_user_undo_available %}
                    <form action="{% url 'task_restore_view' deleted_task.id %}" method="post" style="display:inline;">
                        {% csrf_token %}
                        <button type="submit">Undo</button>
                    </form>
                    <span class="countdown" data-time="{{ deleted_task.deleted_at|date:'U' }}"></span> seconds left
                {% endif %}
            </li>
        {% endfor %}
    </ul>
{% endif %}

{% if user.is_staff %}
    <h2>Admin Recovery</h2>
    <ul>
        {% for deleted_task in deleted_tasks %}
            {% if deleted_task.is_admin_undo_available and not deleted_task.is_user_undo_available %}
                <li>
                    <strong>{{ deleted_task.title }}</strong>
                    <br>
                    Deleted by: {{ deleted_task.deleted_by }} at {{ deleted_task.deleted_at|date:"H:i:s" }}
                    {% if deleted_task.restored_by %}
                        <br>Restored by: {{ deleted_task.restored_by }} at {{ deleted_task.restored_at|date:"H:i:s" }}
                    {% endif %}

                    <form action="{% url 'task_restore_view' deleted_task.id %}" method="post" style="display:inline;">
                        {% csrf_token %}
                        <button type="submit">Restore (Admin)</button>
                    </form>
                </li>
            {% endif %}
        {% endfor %}
    </ul>
{% endif %}
```

✔ **Displays who deleted the task.**
✔ **Shows restoration logs.**

---

## **6. Migrate and Test the Changes**

1. **Run Migrations:**
   ```bash
   python manage.py makemigrations
   python manage.py migrate
   ```

2. **Test Deletion & Restoration**
   - **Delete a task as a regular user** → Should log the deleting user.
   - **Restore within undo time** → Should log the restoring user.
   - **Admin restores after user undo expires** → Should still log the admin.

---

## **🎯 Final Features**
✅ **Tracks who deleted a task.**
✅ **Tracks who restored a task.**
✅ **Admins can audit deletions and recoveries.**

### **Sending Email Notifications for Task Deletion and Restoration**

To improve accountability, we’ll send an email notification **when a task is deleted or restored**. The email will include:
✔ **Who deleted/restored the task**
✔ **Timestamp of the action**
✔ **Task details**
✔ **Undo instructions (if applicable)**

---

## **1. Configure Django Email Settings (`settings.py`)**

Ensure Django is configured to send emails. Update your `settings.py`:

```python
EMAIL_BACKEND = "django.core.mail.backends.smtp.EmailBackend"
EMAIL_HOST = "smtp.gmail.com"  # Change this based on your email provider
EMAIL_PORT = 587
EMAIL_USE_TLS = True
EMAIL_HOST_USER = "your_email@gmail.com"
EMAIL_HOST_PASSWORD = "your_email_password"  # Use environment variables for security
DEFAULT_FROM_EMAIL = "Task Manager <your_email@gmail.com>"
```

✅ **Uses SMTP for sending emails**
✅ **Supports TLS encryption for security**

---

## **2. Create an Email Utility Function (`utils.py`)**

To simplify email sending, create a helper function:

```python
from django.core.mail import send_mail
from django.conf import settings

def send_task_email(subject, message, recipient_list):
    """Sends an email notification for task actions."""
    send_mail(
        subject,
        message,
        settings.DEFAULT_FROM_EMAIL,
        recipient_list,
        fail_silently=False,
    )
```

✅ **Reusable function to send task-related emails**

---

## **3. Send Email on Task Deletion (`views.py`)**

Modify the **task delete view** to send an email:

```python
from django.shortcuts import redirect, get_object_or_404
from django.contrib import messages
from django.utils.timezone import now
from django.contrib.auth.models import User
from .models import Task, DeletedTask
from .utils import send_task_email

def task_delete_view(request, task_id):
    """Soft-deletes a task and notifies the user"""
    task = get_object_or_404(Task, id=task_id)
    deleted_task = DeletedTask.objects.create(
        title=task.title,
        description=task.description,
        deleted_by=request.user,
        deleted_at=now()
    )

    task.delete()
    messages.success(request, "Task deleted. Undo available for a limited time.")

    # Send email notification
    subject = "Task Deleted Notification"
    message = (
        f"Dear {request.user.username},\n\n"
        f"The task '{deleted_task.title}' was deleted by you on {deleted_task.deleted_at.strftime('%Y-%m-%d %H:%M:%S')}.\n\n"
        "If this was a mistake, you can undo this action within the allowed time.\n\n"
        "Best,\nTask Manager Team"
    )

    send_task_email(subject, message, [request.user.email])

    return redirect('task_list')
```

✅ **Sends email to the user when they delete a task**
✅ **Includes undo instructions**

---

## **4. Send Email on Task Restoration (`views.py`)**

Modify the **task restore view** to send an email:

```python
def task_restore_view(request, deleted_task_id):
    """Restores a deleted task and notifies the user"""
    deleted_task = get_object_or_404(DeletedTask, id=deleted_task_id)

    if deleted_task.is_user_undo_available() or (request.user.is_staff and deleted_task.is_admin_undo_available()):
        restored_task = Task.objects.create(
            title=deleted_task.title,
            description=deleted_task.description
        )

        deleted_task.restored_at = now()
        deleted_task.restored_by = request.user
        deleted_task.save()

        messages.success(request, "Task successfully restored!")

        # Send email notification
        subject = "Task Restored Notification"
        message = (
            f"Dear {request.user.username},\n\n"
            f"The task '{restored_task.title}' has been successfully restored by {request.user.username} on {deleted_task.restored_at.strftime('%Y-%m-%d %H:%M:%S')}.\n\n"
            "Best,\nTask Manager Team"
        )

        send_task_email(subject, message, [request.user.email])

    else:
        messages.error(request, "The undo period has expired!")

    return redirect('task_list')
```

✅ **Sends an email to the user when they restore a task**
✅ **Confirms that the task has been successfully recovered**

---

## **5. Test Email Functionality**

1. **Run the Django development server**
   ```bash
   python manage.py runserver
   ```

2. **Perform a test:**
   - Delete a task → Check if the email is received.
   - Restore a task → Verify the restoration email.

3. **If debugging is needed**, configure Django to print emails in the console:
   Add this to `settings.py`:
   ```python
   EMAIL_BACKEND = "django.core.mail.backends.console.EmailBackend"
   ```

---

## **🎯 Final Features**
✅ **Email sent when a task is deleted**
✅ **Email sent when a task is restored**
✅ **Includes timestamps and undo information**

### **Sending Admin Notifications for Task Deletion and Restoration**

Now, let's configure Django to send **email notifications to admins** when a task is deleted or restored. The email will include:
- **Task details**
- **Who deleted/restored the task**
- **Timestamp of the action**

---

## **1. Modify Email Sending Function for Admin Notifications (`utils.py`)**

We’ll add logic to send notifications to **admins** in addition to the user performing the action.

```python
from django.core.mail import send_mail
from django.conf import settings
from django.contrib.auth.models import User

def send_task_email(subject, message, recipient_list):
    """Sends an email notification for task actions."""
    send_mail(
        subject,
        message,
        settings.DEFAULT_FROM_EMAIL,
        recipient_list,
        fail_silently=False,
    )

def send_admin_task_notification(subject, message):
    """Sends an email to all admins about task deletion or restoration."""
    admins = User.objects.filter(is_staff=True)
    admin_emails = [admin.email for admin in admins]

    if admin_emails:
        send_task_email(subject, message, admin_emails)
```

✔ **`send_admin_task_notification`** sends emails to all admins.
✔ **`send_task_email`** is used for both user and admin notifications.

---

## **2. Send Email to Admins on Task Deletion (`views.py`)**

Modify the **task delete view** to also notify **admins** when a task is deleted.

```python
def task_delete_view(request, task_id):
    """Soft-deletes a task and notifies the user and admins"""
    task = get_object_or_404(Task, id=task_id)
    deleted_task = DeletedTask.objects.create(
        title=task.title,
        description=task.description,
        deleted_by=request.user,
        deleted_at=now()
    )

    task.delete()
    messages.success(request, "Task deleted. Undo available for a limited time.")

    # Send email notification to the user
    subject = "Task Deleted Notification"
    message = (
        f"Dear {request.user.username},\n\n"
        f"The task '{deleted_task.title}' was deleted by you on {deleted_task.deleted_at.strftime('%Y-%m-%d %H:%M:%S')}.\n\n"
        "If this was a mistake, you can undo this action within the allowed time.\n\n"
        "Best,\nTask Manager Team"
    )

    send_task_email(subject, message, [request.user.email])

    # Send email notification to admins
    subject_admin = "Admin Notification: Task Deleted"
    message_admin = (
        f"Task '{deleted_task.title}' was deleted by {request.user.username} on {deleted_task.deleted_at.strftime('%Y-%m-%d %H:%M:%S')}.\n\n"
        "Please review any pending actions if necessary.\n\n"
        "Best,\nTask Manager Team"
    )
    send_admin_task_notification(subject_admin, message_admin)

    return redirect('task_list')
```

✔ **Sends an email to admins** with task deletion details.
✔ **Notifies both the user and admins about the deletion**.

---

## **3. Send Email to Admins on Task Restoration (`views.py`)**

Similarly, modify the **task restore view** to send an email to **admins** when a task is restored.

```python
def task_restore_view(request, deleted_task_id):
    """Restores a deleted task and notifies the user and admins"""
    deleted_task = get_object_or_404(DeletedTask, id=deleted_task_id)

    if deleted_task.is_user_undo_available() or (request.user.is_staff and deleted_task.is_admin_undo_available()):
        restored_task = Task.objects.create(
            title=deleted_task.title,
            description=deleted_task.description
        )

        deleted_task.restored_at = now()
        deleted_task.restored_by = request.user
        deleted_task.save()

        messages.success(request, "Task successfully restored!")

        # Send email notification to the user
        subject = "Task Restored Notification"
        message = (
            f"Dear {request.user.username},\n\n"
            f"The task '{restored_task.title}' has been successfully restored by {request.user.username} on {deleted_task.restored_at.strftime('%Y-%m-%d %H:%M:%S')}.\n\n"
            "Best,\nTask Manager Team"
        )

        send_task_email(subject, message, [request.user.email])

        # Send email notification to admins
        subject_admin = "Admin Notification: Task Restored"
        message_admin = (
            f"Task '{restored_task.title}' was restored by {request.user.username} on {deleted_task.restored_at.strftime('%Y-%m-%d %H:%M:%S')}.\n\n"
            "Please review the restored task if needed.\n\n"
            "Best,\nTask Manager Team"
        )
        send_admin_task_notification(subject_admin, message_admin)

    else:
        messages.error(request, "The undo period has expired!")

    return redirect('task_list')
```

✔ **Sends an email to admins** when a task is restored.
✔ **Notifies both the user and admins about the restoration**.

---

## **4. Test Email Notifications to Admins**

1. **Run the Django development server**:
   ```bash
   python manage.py runserver
   ```

2. **Perform a test:**
   - **Delete a task** → Verify that both the user and **all admins** receive the deletion email.
   - **Restore a task** → Verify that both the user and **all admins** receive the restoration email.

3. **Debugging email issues**:
   - Use `EMAIL_BACKEND = "django.core.mail.backends.console.EmailBackend"` to print emails to the console during testing.

---

## **🎯 Final Features**
✅ **Email sent to the user and all admins when a task is deleted**.
✅ **Email sent to the user and all admins when a task is restored**.
✅ **Admins are notified about actions performed by other users**.

### **Setting Up Daily/Weekly Email Summaries for Admins about Task Activity**

To ensure admins are regularly updated on the activity in the task manager, we’ll set up automated email summaries that provide details of tasks that were **created**, **deleted**, and **restored** over a daily or weekly period.

---

## **1. Create a Management Command to Send Summaries**

We’ll create a custom Django **management command** to gather task activity over a period of time and send a summary email to all admins.

### **Step 1: Create the Management Command**

1. In your Django app directory, create a folder named `management/commands` (if it doesn't already exist).

2. Inside the `commands` folder, create a Python file, e.g., `send_task_activity_summary.py`.

```bash
your_project/
    your_app/
        management/
            commands/
                send_task_activity_summary.py
```

3. Implement the command to fetch task activity and send the summary:

```python
# your_app/management/commands/send_task_activity_summary.py
from django.core.management.base import BaseCommand
from django.utils.timezone import now
from django.core.mail import send_mail
from django.conf import settings
from your_app.models import Task, DeletedTask
from datetime import timedelta

class Command(BaseCommand):
    help = "Send daily/weekly summary of task activity to admins."

    def handle(self, *args, **options):
        # Calculate time window (last 24 hours or week)
        time_window = timedelta(days=1)  # For daily summaries
        # time_window = timedelta(weeks=1)  # For weekly summaries
        start_time = now() - time_window

        # Fetch task activity in the last 24 hours
        tasks_created = Task.objects.filter(created_at__gte=start_time)
        tasks_deleted = DeletedTask.objects.filter(deleted_at__gte=start_time)
        tasks_restored = DeletedTask.objects.filter(restored_at__gte=start_time)

        # Create summary content
        summary = f"Task Activity Summary ({start_time.strftime('%Y-%m-%d %H:%M:%S')} - {now().strftime('%Y-%m-%d %H:%M:%S')})\n\n"

        summary += "Tasks Created:\n"
        for task in tasks_created:
            summary += f"- {task.title} (ID: {task.id})\n"

        summary += "\nTasks Deleted:\n"
        for task in tasks_deleted:
            summary += f"- {task.title} (ID: {task.id})\n"

        summary += "\nTasks Restored:\n"
        for task in tasks_restored:
            summary += f"- {task.title} (ID: {task.id})\n"

        # Send email to all admins
        admins = User.objects.filter(is_staff=True)
        admin_emails = [admin.email for admin in admins]
        if admin_emails:
            send_mail(
                subject="Task Activity Summary",
                message=summary,
                from_email=settings.DEFAULT_FROM_EMAIL,
                recipient_list=admin_emails,
                fail_silently=False,
            )

        self.stdout.write(self.style.SUCCESS('Successfully sent task activity summary to admins'))
```

### **Explanation:**
- **Daily/Weekly Window:** You can adjust the `time_window` variable to either **24 hours** (daily) or **1 week** (weekly) based on your needs.
- **Fetching Task Activity:** The command fetches all tasks created, deleted, and restored during the time window.
- **Summary Content:** It creates a simple summary containing the task titles and IDs.
- **Sending Emails:** It then sends the summary email to all admins using the `send_mail` function.

---

## **2. Set Up a Cron Job (Linux/Mac) or Task Scheduler (Windows)**

To run this command daily or weekly, we need to set up a scheduled task. Depending on your operating system, this can be done using **cron jobs** (Linux/Mac) or **Task Scheduler** (Windows).

### **Linux/Mac: Use Cron Jobs**

1. Open your crontab file:

```bash
crontab -e
```

2. Add an entry for a daily or weekly cron job:

- **Daily (at midnight):**
```bash
0 0 * * * /path/to/your/virtualenv/bin/python /path/to/your/project/manage.py send_task_activity_summary
```

- **Weekly (every Sunday at midnight):**
```bash
0 0 * * 0 /path/to/your/virtualenv/bin/python /path/to/your/project/manage.py send_task_activity_summary
```

### **Windows: Use Task Scheduler**

1. Open **Task Scheduler** and create a **New Task**.
2. Set it to run the `manage.py send_task_activity_summary` command daily or weekly.
3. Set the **Action** to execute the Python script as follows:
   - **Program/Script:** `python`
   - **Add arguments:** `manage.py send_task_activity_summary`
   - **Start in:** (your project directory)

---

## **3. Test the Summary Command**

1. Run the management command manually to test:

```bash
python manage.py send_task_activity_summary
```

2. Verify that the admins received the email with the correct summary content.

---

## **4. Verify Email Delivery**

Ensure emails are being sent correctly by checking the admin inbox and the **console output**. If you're debugging, you can use:

```python
EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'
```

This will print emails to the console during testing.

---

## **🎯 Final Features**
✅ **Daily or weekly email summaries** of task activities.
✅ **Admins are kept informed** about task creation, deletion, and restoration.
✅ **Automation via cron jobs or Task Scheduler** to ensure timely updates.
