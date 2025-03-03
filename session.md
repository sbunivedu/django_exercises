### **Django Exercise: Understanding Sessions & Cookies**

#### **Objective**
This exercise helps students explore how Django manages user sessions using cookies and session storage. They will:
✅ **Set and retrieve session data** in Django views.
✅ **Inspect cookies and session data** using browser developer tools.
✅ **Understand how Django stores session information on the server.**

---

## **Step 1: Enable Sessions in Django**
Django comes with session management enabled by default. Ensure `django.contrib.sessions.middleware.SessionMiddleware` is present in `settings.py`:

```python
INSTALLED_APPS = [
    'django.contrib.sessions',  # Ensures session handling
    ...
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',  # Enables session management
    ...
]

# Define session storage (default is database-backed sessions)
SESSION_ENGINE = 'django.contrib.sessions.backends.db'
# Alternative: File-based sessions
# SESSION_ENGINE = 'django.contrib.sessions.backends.file'
```

Run migrations to set up session storage:

```bash
python manage.py migrate
```

---

## **Step 2: Create Views to Demonstrate Session Handling**
In `views.py`:

```python
from django.shortcuts import render, redirect
from django.http import HttpResponse

def set_session(request):
    request.session['username'] = 'Student'
    request.session['visits'] = request.session.get('visits', 0) + 1
    return HttpResponse("Session set! Go to <a href='/get-session/'>Get Session</a>")

def get_session(request):
    username = request.session.get('username', 'Guest')
    visits = request.session.get('visits', 0)

    if visits > 5:
        del request.session['visits']  # Reset visit count after 5 visits

    response = f"""
    <p>Username: {username}</p>
    <p>Visit Count: {visits}</p>
    <p>Session Key: {request.session.session_key}</p>
    <p>Check cookies in your browser! <a href='/delete-session/'>Clear Session</a></p>
    """
    return HttpResponse(response)

def delete_session(request):
    request.session.flush()  # Clears all session data
    return HttpResponse("Session deleted! <a href='/set-session/'>Set Session Again</a>")
```

---

## **Step 3: Define URLs**
In `urls.py`:

```python
from django.urls import path
from .views import set_session, get_session, delete_session

urlpatterns = [
    path('set-session/', set_session, name='set_session'),
    path('get-session/', get_session, name='get_session'),
    path('delete-session/', delete_session, name='delete_session'),
]
```

---

## **Step 4: Explore Session and Cookie Behavior**
1. **Start the Django server:**
   ```bash
   python manage.py runserver
   ```
2. **Visit** `http://127.0.0.1:8000/set-session/`
   - This sets the session values.
   - Check the response and click the "Get Session" link.
3. **Visit** `http://127.0.0.1:8000/get-session/`
   - Displays stored session data.
   - Shows a session key.
   - Refresh multiple times to observe how `visits` changes.
   - After 5 visits, `visits` resets to 0.
4. **Visit** `http://127.0.0.1:8000/delete-session/`
   - Deletes the session and resets data.

---

## **Step 5: Inspect Cookies & Sessions in Browser**
### **Check Cookies**
1. Open **Developer Tools** in your browser (`F12` or `Ctrl+Shift+I` in Chrome).
2. Go to **Application → Storage → Cookies → http://127.0.0.1:8000/**.
3. Look for the `sessionid` cookie.
   - Django stores the session key in the client-side cookie.
   - This key links to session data stored on the server.

### **Check Session Data in the Database**
Run the following SQL query to inspect stored sessions (if using database-backed sessions):

```bash
sqlite3 db.sqlite3 "SELECT * FROM django_session;"
```

Or, if using Django's shell:

```bash
python manage.py shell
>>> from django.contrib.sessions.models import Session
>>> Session.objects.all().values()
```

---

## **Key Takeaways**
✅ **Django stores session data on the server**, not in cookies.
✅ **Cookies only store a session key**, which links to session data on the backend.
✅ **Session data is persistent until deleted or expired** (default is 2 weeks).
✅ **Students can modify session data in views and observe changes in cookies and storage.**
