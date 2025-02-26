### **Django Forms Exercise: Form Validation & Bootstrap Styling**

#### **Objective**
Students will create a Django form that validates user input and applies Bootstrap styling for a visually appealing form. The form will collect a **task title** and **description**, ensuring:
✅ The **title** is required and has a maximum length of 100 characters.
✅ The **description** is optional but has a **minimum length of 10 characters** if provided.

---

### **Step 1: Create a Django Form with Validation**
Create a file `forms.py` in your Django app.

```python
from django import forms

class TaskForm(forms.Form):
    title = forms.CharField(
        max_length=100,
        required=True,
        widget=forms.TextInput(attrs={'class': 'form-control', 'placeholder': 'Enter task title'})
    )
    description = forms.CharField(
        required=False,
        min_length=10,
        widget=forms.Textarea(attrs={'class': 'form-control', 'placeholder': 'Task description (optional)'}),
    )
```

---

### **Step 2: Create the View**
Modify `views.py` to handle form validation.

```python
from django.shortcuts import render
from .forms import TaskForm

def task_form_view(request):
    form = TaskForm(request.POST or None)

    if request.method == 'POST' and form.is_valid():
        title = form.cleaned_data['title']
        description = form.cleaned_data['description']
        return render(request, 'task_success.html', {'title': title, 'description': description})

    return render(request, 'task_form.html', {'form': form})
```

---

### **Step 3: Create the Template with Bootstrap**
Create `templates/task_form.html`:

```html
{% load static %}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Task Form</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body class="container mt-5">
    <h2 class="mb-4">Create a Task</h2>
    <form method="POST" class="card p-4 shadow">
        {% csrf_token %}

        <div class="mb-3">
            <label for="{{ form.title.id_for_label }}" class="form-label">Task Title</label>
            {{ form.title }}
            {% if form.title.errors %}
                <div class="text-danger">{{ form.title.errors.0 }}</div>
            {% endif %}
        </div>

        <div class="mb-3">
            <label for="{{ form.description.id_for_label }}" class="form-label">Task Description</label>
            {{ form.description }}
            {% if form.description.errors %}
                <div class="text-danger">{{ form.description.errors.0 }}</div>
            {% endif %}
        </div>

        <button type="submit" class="btn btn-primary">Submit Task</button>
    </form>
</body>
</html>
```

---

### **Step 4: Success Page**
Create `templates/task_success.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Task Created</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body class="container mt-5">
    <div class="alert alert-success">
        <h4>Task Created Successfully!</h4>
        <p><strong>Title:</strong> {{ title }}</p>
        <p><strong>Description:</strong> {{ description }}</p>
        <a href="{% url 'task_form' %}" class="btn btn-primary">Create Another Task</a>
    </div>
</body>
</html>
```

---

### **Step 5: Configure URL**
Modify `urls.py`:

```python
from django.urls import path
from .views import task_form_view

urlpatterns = [
    path('task/', task_form_view, name='task_form'),
]
```

---

### **Test Your Form**
1. Run the server:
   ```bash
   python manage.py runserver
   ```
2. Go to `http://127.0.0.1:8000/task/`
3. Try:
   - Submitting an empty form → Should show an error.
   - Entering a short description (<10 characters) → Should show an error.
   - Submitting a valid task → Should redirect to a success page.

---

### **What Students Learn**
✅ Using Django Forms for input validation.
✅ Customizing Bootstrap-styled form elements.
✅ Displaying form errors and success messages.
