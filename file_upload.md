### **Django Exercise: File Upload & Serving Media**

#### **Objective**
Students will create a Django web app that allows users to upload files (e.g., images, PDFs) and display the uploaded files on a webpage. This will help them practice:
✅ Handling **file uploads** in Django using `FileField` and `ImageField`.
✅ Serving **uploaded media files** in development.
✅ Storing file paths in a **Django model**.

---

## **Step 1: Configure Django to Handle Media Files**
In `settings.py`, add the following configurations:

```python
import os

MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
```

Modify `urls.py` to serve media files in development:

```python
from django.conf import settings
from django.conf.urls.static import static
from django.urls import path
from .views import file_upload_view

urlpatterns = [
    path('upload/', file_upload_view, name='file_upload'),
]

if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

---

## **Step 2: Create a Django Model for File Upload**
In `models.py`:

```python
from django.db import models

class UploadedFile(models.Model):
    title = models.CharField(max_length=255)
    file = models.FileField(upload_to='uploads/')
    uploaded_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return self.title
```

Run migrations:

```bash
python manage.py makemigrations
python manage.py migrate
```

---

## **Step 3: Create the Upload Form**
In `forms.py`:

```python
from django import forms
from .models import UploadedFile

class UploadFileForm(forms.ModelForm):
    class Meta:
        model = UploadedFile
        fields = ['title', 'file']
        widgets = {
            'title': forms.TextInput(attrs={'class': 'form-control', 'placeholder': 'Enter file title'}),
            'file': forms.ClearableFileInput(attrs={'class': 'form-control'}),
        }
```

---

## **Step 4: Create the View for Handling File Upload**
In `views.py`:

```python
from django.shortcuts import render, redirect
from .forms import UploadFileForm
from .models import UploadedFile

def file_upload_view(request):
    if request.method == 'POST':
        form = UploadFileForm(request.POST, request.FILES)
        if form.is_valid():
            form.save()
            return redirect('file_upload')
    else:
        form = UploadFileForm()

    files = UploadedFile.objects.all()
    return render(request, 'file_upload.html', {'form': form, 'files': files})
```

---

## **Step 5: Create the File Upload Template**
Create `templates/file_upload.html`:

```html
{% load static %}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>File Upload</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body class="container mt-5">
    <h2 class="mb-4">Upload a File</h2>

    <form method="POST" enctype="multipart/form-data" class="card p-4 shadow">
        {% csrf_token %}
        <div class="mb-3">
            <label for="{{ form.title.id_for_label }}" class="form-label">File Title</label>
            {{ form.title }}
        </div>
        <div class="mb-3">
            <label for="{{ form.file.id_for_label }}" class="form-label">Select File</label>
            {{ form.file }}
        </div>
        <button type="submit" class="btn btn-primary">Upload</button>
    </form>

    <hr>

    <h3>Uploaded Files</h3>
    <ul class="list-group">
        {% for file in files %}
            <li class="list-group-item">
                <strong>{{ file.title }}</strong> -
                <a href="{{ file.file.url }}" target="_blank">View File</a>
            </li>
        {% empty %}
            <li class="list-group-item">No files uploaded yet.</li>
        {% endfor %}
    </ul>
</body>
</html>
```

---

## **Step 6: Test the File Upload Feature**
1. Run the server:
   ```bash
   python manage.py runserver
   ```
2. Visit `http://127.0.0.1:8000/upload/`
3. Try:
   - Uploading **different file types** (images, PDFs, etc.).
   - Checking if files are **saved in the media folder**.
   - Clicking the uploaded file **links** to ensure they are accessible.

---

### **What Students Learn**
✅ **File uploads** using `FileField` in Django.
✅ **Handling uploaded files** in a model.
✅ **Serving media files** in development.
✅ **Bootstrap styling** for file input.

