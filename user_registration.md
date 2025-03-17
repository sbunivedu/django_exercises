### **Adding User Registration Using Django Admin Module**

**Objective:**
Students will learn how to use Django’s built-in **admin module** to manage user registration and authentication.

### **Step 1: Set Up Django Project**
If not already set up, start a new Django project and app:
```sh
django-admin startproject user_admin_project
cd user_admin_project
django-admin startapp accounts
```

### **Step 2: Enable the Admin Module**
Ensure `django.contrib.admin` is included in `INSTALLED_APPS` in `settings.py`:
```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'accounts',  # Our new app
]
```

### **Step 3: Create a Superuser**
Run the following command and follow the prompts to create an admin user:
```sh
python manage.py createsuperuser
```

### **Step 4: Register Users in Django Admin**
Modify `accounts/admin.py` to manage users via the Django admin panel:
```python
from django.contrib import admin
from django.contrib.auth.models import User

admin.site.register(User)
```
Now, users can be added, edited, and deleted via the Django admin panel.

### **Step 5: Create a User Registration Form**
In `accounts/forms.py`, create a user registration form:
```python
from django import forms
from django.contrib.auth.models import User

class UserRegistrationForm(forms.ModelForm):
    password = forms.CharField(widget=forms.PasswordInput)

    class Meta:
        model = User
        fields = ['username', 'email', 'password']
```

### **Step 6: Create a View for User Registration**
Modify `accounts/views.py`:
```python
from django.shortcuts import render, redirect
from django.contrib.auth import login
from .forms import UserRegistrationForm

def register(request):
    if request.method == 'POST':
        form = UserRegistrationForm(request.POST)
        if form.is_valid():
            user = form.save(commit=False)
            user.set_password(form.cleaned_data['password'])  # Encrypt password
            user.save()
            login(request, user)
            return redirect('admin:index')  # Redirect to admin panel
    else:
        form = UserRegistrationForm()
    return render(request, 'accounts/register.html', {'form': form})
```

### **Step 7: Create a Registration Template**
In `templates/accounts/register.html`:
```html
{% extends "admin/base_site.html" %}
{% block content %}
<h2>User Registration</h2>
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Register</button>
</form>
{% endblock %}
```

### **Step 8: Configure URLs**
Modify `accounts/urls.py`:
```python
from django.urls import path
from .views import register

urlpatterns = [
    path('register/', register, name='register'),
]
```
Include this in `user_admin_project/urls.py`:
```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('accounts/', include('accounts.urls')),
]
```

### **Step 9: Run and Test**
Start the server:
```sh
python manage.py runserver
```
- **Visit** `/admin/` to access the admin panel.
- **Go to** `/accounts/register/` to register a user.
- **Login to the admin panel** with the registered user.

### **Reflection Questions for Students**
1. How does Django’s admin module simplify user management?
2. What is the role of `set_password()` in user registration?
3. How can you extend the registration form to include additional fields like first name and last name?
