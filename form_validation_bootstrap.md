### **Advanced Django Forms Exercise: Multi-Field Validation & Bootstrap Styling**

#### **Objective**
Students will create a Django form that validates user input with advanced **multi-field validation rules**, ensuring:
✅ **Title and description must not be identical.**
✅ **Deadline must be in the future.**
✅ **Priority (High, Medium, Low) must be specified if the deadline is within 3 days.**

---

## **Step 1: Define the Django Form with Advanced Validation**
Create `forms.py` in your Django app:

```python
from django import forms
from django.core.exceptions import ValidationError
from django.utils.timezone import now
from datetime import timedelta

PRIORITY_CHOICES = [
    ('', 'Select Priority'),
    ('High', 'High'),
    ('Medium', 'Medium'),
    ('Low', 'Low'),
]

class TaskForm(forms.Form):
    title = forms.CharField(
        max_length=100,
        required=True,
        widget=forms.TextInput(attrs={'class': 'form-control', 'placeholder': 'Enter task title'}),
    )
    description = forms.CharField(
        required=True,
        min_length=10,
        widget=forms.Textarea(attrs={'class': 'form-control', 'placeholder': 'Task description'}),
    )
    deadline = forms.DateTimeField(
        required=True,
        widget=forms.DateTimeInput(attrs={'type': 'datetime-local', 'class': 'form-control'}),
    )
    priority = forms.ChoiceField(
        choices=PRIORITY_CHOICES,
        required=False,
        widget=forms.Select(attrs={'class': 'form-select'}),
    )

    def clean(self):
        cleaned_data = super().clean()
        title = cleaned_data.get('title')
        description = cleaned_data.get('description')
        deadline = cleaned_data.get('deadline')
        priority = cleaned_data.get('priority')

        # Rule 1: Title and Description must not be identical
        if title and description and title.strip().lower() == description.strip().lower():
            raise ValidationError("Title and description cannot be the same.")

        # Rule 2: Deadline must be in the future
        if deadline and deadline <= now():
            raise ValidationError("Deadline must be in the future.")

        # Rule 3: Priority must be set if the deadline is within 3 days
        if deadline and deadline <= now() + timedelta(days=3) and not priority:
            raise ValidationError("Priority must be specified if the deadline is within 3 days.")

        return cleaned_data
```

---

## **Step 2: Create the View with Validation Handling**
Modify `views.py`:

```python
from django.shortcuts import render
from .forms import TaskForm

def task_form_view(request):
    form = TaskForm(request.POST or None)

    if request.method == 'POST' and form.is_valid():
        return render(request, 'task_success.html', {'form': form.cleaned_data})

    return render(request, 'task_form.html', {'form': form})
```

---

## **Step 3: Create the Bootstrap-Styled Form Template**
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

        <div class="mb-3">
            <label for="{{ form.deadline.id_for_label }}" class="form-label">Deadline</label>
            {{ form.deadline }}
            {% if form.deadline.errors %}
                <div class="text-danger">{{ form.deadline.errors.0 }}</div>
            {% endif %}
        </div>

        <div class="mb-3">
            <label for="{{ form.priority.id_for_label }}" class="form-label">Priority</label>
            {{ form.priority }}
            {% if form.priority.errors %}
                <div class="text-danger">{{ form.priority.errors.0 }}</div>
            {% endif %}
        </div>

        {% if form.non_field_errors %}
            <div class="alert alert-danger">
                {{ form.non_field_errors.0 }}
            </div>
        {% endif %}

        <button type="submit" class="btn btn-primary">Submit Task</button>
    </form>
</body>
</html>
```

---

## **Step 4: Success Page**
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
        <p><strong>Title:</strong> {{ form.title }}</p>
        <p><strong>Description:</strong> {{ form.description }}</p>
        <p><strong>Deadline:</strong> {{ form.deadline }}</p>
        <p><strong>Priority:</strong> {{ form.priority|default:"Not set" }}</p>
        <a href="{% url 'task_form' %}" class="btn btn-primary">Create Another Task</a>
    </div>
</body>
</html>
```

---

## **Step 5: Configure URL**
Modify `urls.py`:

```python
from django.urls import path
from .views import task_form_view

urlpatterns = [
    path('task/', task_form_view, name='task_form'),
]
```

---

## **Test Your Form**
1. Run the server:
   ```bash
   python manage.py runserver
   ```
2. Go to `http://127.0.0.1:8000/task/`
3. Try:
   - Submitting the **same title and description** → Should show an error.
   - Setting the **deadline in the past** → Should show an error.
   - Setting the **deadline within 3 days but leaving priority empty** → Should show an error.
   - Submitting **valid data** → Should show a success page.

---

### **What Students Learn**
✅ Implementing **multi-field validation** in Django forms.
✅ Handling **form-level errors** (beyond individual field errors).
✅ Using **Bootstrap styling** to enhance form appearance.
✅ Displaying validation messages effectively.
